# Go GMP 调度模型

#go #golang #gmp #scheduler #抢占式调度 #goroutine #协程 #gmp 

> 所谓 `GMP`，其中 `G` 是指 `goroutine`；`M` 是 `Machine` 的缩写，指的是工作线程；`P` 则是指处理器 `Processor`，代表了一组资源，`M` 要想执行 `G` 的代码，必须持有一个 `P` 才行。

> 简单来说，`GMP` 就是 `Task`、`Worker`、`Resource` 的关系，`G` 和 `P` 都是 `Go` 语言实现的抽象程度更高的组件，对于工作线程而言，`Machine` 一词表明了它与具体的操作系统和平台密切相关，对具体平台的适配和特殊处理等大多在这一层实现。

## 从 GM 到 GMP

### GM 模型调度的几个明显问题

1. 用一个全局的 `mutex` 保护一个全局的 `runq`（就绪队列），造成对锁的竞争异常严重
2. `G` 每次执行都会被分发到随机的 `M` 上，造成在不同 `M` 之间频繁切换，破坏了程序的局部性
3. 每个 `M` 都会关联一个内存分配缓存 `mcache`，造成大量内存开销，进一步使数据的局部性变差。实际上只有执行 `Go` 代码的 `M` 才真正需要 `mcache`，那些阻塞在系统调用中的 `M` 根本不需要，而实际执行 `Go` 代码的 `M` 可能仅占总数的 `1%`
4. 在存在系统调用的情况下，工作线程经常被阻塞和解除阻塞，从而增加了很多开销

> 通过 `runtime.GOMAXPROCS()` 函数可以精准的控制 `P` 的个数

## GMP 的数据结构

### G

#### 源码

`runtime.g`

```go
type g struct {  
   // Stack parameters.   
   // stack describes the actual stack memory: [stack.lo, stack.hi).  
   // stackguard0 is the stack pointer compared in the Go stack growth prologue.   
   // It is stack.lo+StackGuard normally, but can be StackPreempt to trigger a preemption.  
   // stackguard1 is the stack pointer compared in the C stack growth prologue.   
   // It is stack.lo+StackGuard on g0 and gsignal stacks.   
   // It is ~0 on other goroutine stacks, to trigger a call to morestackc (and crash).  
   stack       stack   // offset known to runtime/cgo  
   stackguard0 uintptr // offset known to liblink  
   stackguard1 uintptr // offset known to liblink  
  
   _panic    *_panic // innermost panic - offset known to liblink  
   _defer    *_defer // innermost defer  
   m         *m      // current m; offset known to arm liblink  
   sched     gobuf  
   syscallsp uintptr // if status==Gsyscall, syscallsp = sched.sp to use during gc   
   syscallpc uintptr // if status==Gsyscall, syscallpc = sched.pc to use during gc   
   stktopsp  uintptr // expected sp at top of stack, to check in traceback  
   // param is a generic pointer parameter field used to pass   
   // values in particular contexts where other storage for the   
   // parameter would be difficult to find. It is currently used   
   // in three ways:   
   // 1. When a channel operation wakes up a blocked goroutine, it sets param to   
   //    point to the sudog of the completed blocking operation.   
   // 2. By gcAssistAlloc1 to signal back to its caller that the goroutine completed   
   //    the GC cycle. It is unsafe to do so in any other way, because the goroutine's   
   //    stack may have moved in the meantime.   
   // 3. By debugCallWrap to pass parameters to a new goroutine because allocating a   
   //    closure in the runtime is forbidden.   
   param        unsafe.Pointer  
   atomicstatus atomic.Uint32  
   stackLock    uint32 // sigprof/scang lock; TODO: fold in to atomicstatus  
   goid         uint64  
   schedlink    guintptr  
   waitsince    int64      // approx time when the g become blocked   
   waitreason   waitReason // if status==Gwaiting  
  
   preempt       bool // preemption signal, duplicates stackguard0 = stackpreempt   
   preemptStop   bool // transition to _Gpreempted on preemption; otherwise, just 
   deschedule   preemptShrink bool // shrink stack at synchronous safe point  
   // asyncSafePoint is set if g is stopped at an asynchronous   
   // safe point. This means there are frames on the stack  
   // without precise pointer information.  
   asyncSafePoint bool  
  
   paniconfault bool // panic (instead of crash) on unexpected fault address  
   gcscandone   bool // g has scanned stack; protected by _Gscan bit in status   
   throwsplit   bool // must not split stack  
   // activeStackChans indicates that there are unlocked channels   
   // pointing into this goroutine's stack. If true, stack  
   // copying needs to acquire channel locks to protect these  
   // areas of the stack.  
   activeStackChans bool  
   // parkingOnChan indicates that the goroutine is about to   
   // park on a chansend or chanrecv. Used to signal an unsafe point  
   // for stack shrinking.   
   parkingOnChan atomic.Bool  
  
   raceignore     int8     // ignore race detection events  
   sysblocktraced bool     // StartTrace has emitted EvGoInSyscall about this goroutine   
   tracking       bool     // whether we're tracking this G for sched latency statistics   
   trackingSeq    uint8    // used to decide whether to track this G  
   trackingStamp  int64    // timestamp of when the G last started being tracked  
   runnableTime   int64    // the amount of time spent runnable, cleared when running, only used when tracking  
   sysexitticks   int64    // cputicks when syscall has returned (for tracing)   
   traceseq       uint64   // trace event sequencer   
   tracelastp     puintptr // last P emitted an event for this goroutine  
   lockedm        muintptr  
   sig            uint32  
   writebuf       []byte  
   sigcode0       uintptr  
   sigcode1       uintptr  
   sigpc          uintptr  
   gopc           uintptr         // pc of go statement that created this goroutine  
   ancestors      *[]ancestorInfo // ancestor information goroutine(s) that created this goroutine (only used if debug.tracebackancestors)  
   startpc        uintptr         // pc of goroutine function  
   racectx        uintptr  
   waiting        *sudog         // sudog structures this g is waiting on (that have a valid elem ptr); in lock order   
   cgoCtxt        []uintptr      // cgo traceback context   
   labels         unsafe.Pointer // profiler labels  
   timer          *timer         // cached timer for time.Sleep  
   selectDone     atomic.Uint32  // are we participating in a select and did someone win the race?  
  
   // goroutineProfiled indicates the status of this goroutine's stack for the   
   // current in-progress goroutine profile   
   goroutineProfiled goroutineProfileStateHolder  
  
   // Per-G GC state  
  
   // gcAssistBytes is this G's GC assist credit in terms of   
   // bytes allocated. If this is positive, then the G has credit   
   // to allocate gcAssistBytes bytes without assisting. If this   
   // is negative, then the G must correct this by performing   
   // scan work. We track this in bytes to make it fast to update   
   // and check for debt in the malloc hot path. The assist ratio   
   // determines how this corresponds to scan work debt.   
   gcAssistBytes int64  
}
```

#### 部分字段用途

- `stack`：描述了 `goroutine` 的栈空间
- `stackguard0`：
- `stackguard1`：原理同 `stackguard0` 差不多，只不过是被 `g0` 和 `gsignal` 中的 `C` 代码使用
- `m`：关联到正在执行当前 `G` 的工作线程 `M`
- `sched`：被调度器用来保存 goroutine 的执行上下文
- `atomicstatus`：用来表示当前 `G` 的状态
- `goid`：当前 `goroutine` 的全局唯一 ID
- `schedlink`：被调度器用于实现内部链表、队列，对应的 `gunitptr` 类型从逻辑上等价于 `*g`，而底层类型却是个 `uintptr`，这样是为了避免写障碍
- `preempt`：为 `true` 时，调度器会在合适的时机触发一次抢占
- `lockedm`：关联到与当前 `G` 绑定的 `M`，可参考 `LockOSThread`
- `waiting`：主要用于实现 `channel` 中的等待队列
- `timer`：`runtime` 内部实现的计时器类型，主要用来支持 `time.Sleep`

#### atomicstatus 字段的取值及其含义

```go
const (  
   // G status  
   //   
   // Beyond indicating the general state of a G, the G status   
   // acts like a lock on the goroutine's stack (and hence its   
   // ability to execute user code).   
   //   
   // If you add to this list, add to the list   
   // of "okay during garbage collection" status   
   // in mgcmark.go too.   
   //   
   // TODO(austin): The _Gscan bit could be much lighter-weight.   
   // For example, we could choose not to run _Gscanrunnable   
   // goroutines found in the run queue, rather than CAS-looping   
   // until they become _Grunnable. And transitions like   
   // _Gscanwaiting -> _Gscanrunnable are actually okay because   
   // they don't affect stack ownership.  
   // _Gidle means this goroutine was just allocated and has not   
   // yet been initialized.   
   _Gidle = iota // 0  
  
   // _Grunnable means this goroutine is on a run queue. It is   
   // not currently executing user code. The stack is not owned.   
   _Grunnable // 1  
  
   // _Grunning means this goroutine may execute user code. The   
   // stack is owned by this goroutine. It is not on a run queue.   
   // It is assigned an M and a P (g.m and g.m.p are valid).   
   _Grunning // 2  
  
   // _Gsyscall means this goroutine is executing a system call.   
   // It is not executing user code. The stack is owned by this   
   // goroutine. It is not on a run queue. It is assigned an M.   
   _Gsyscall // 3  
  
   // _Gwaiting means this goroutine is blocked in the runtime.   
   // It is not executing user code. It is not on a run queue,   
   // but should be recorded somewhere (e.g., a channel wait   
   // queue) so it can be ready()d when necessary. The stack is   
   // not owned *except* that a channel operation may read or   
   // write parts of the stack under the appropriate channel   
   // lock. Otherwise, it is not safe to access the stack after a   
   // goroutine enters _Gwaiting (e.g., it may get moved).   
   _Gwaiting // 4  
  
   // _Gmoribund_unused is currently unused, but hardcoded in gdb   
   // scripts.   
   _Gmoribund_unused // 5  
  
   // _Gdead means this goroutine is currently unused. It may be   
   // just exited, on a free list, or just being initialized. It   
   // is not executing user code. It may or may not have a stack  
   // allocated. The G and its stack (if any) are owned by the M   
   // that is exiting the G or that obtained the G from the free   
   // list.   
   _Gdead // 6  
  
   // _Genqueue_unused is currently unused.   
   _Genqueue_unused // 7  
  
   // _Gcopystack means this goroutine's stack is being moved. It   
   // is not executing user code and is not on a run queue. The   
   // stack is owned by the goroutine that put it in _Gcopystack.  
   _Gcopystack // 8  
  
   // _Gpreempted means this goroutine stopped itself for a   
   // suspendG preemption. It is like _Gwaiting, but nothing is  
   // yet responsible for ready()ing it. Some suspendG must CAS   
   // the status to _Gwaiting to take responsibility for   
   // ready()ing this G.  
   _Gpreempted // 9  
  
   // _Gscan combined with one of the above states other than   
   // _Grunning indicates that GC is scanning the stack. The  
   // goroutine is not executing user code and the stack is owned   
   // by the goroutine that set the _Gscan bit.   
   //   
   // _Gscanrunning is different: it is used to briefly block  
   // state transitions while GC signals the G to scan its own   
   // stack. This is otherwise like _Grunning.  
   //   
   // atomicstatus&~Gscan gives the state the goroutine will   
   // return to when the scan completes.   
   _Gscan          = 0x1000  
   _Gscanrunnable  = _Gscan + _Grunnable  // 0x1001  
   _Gscanrunning   = _Gscan + _Grunning   // 0x1002  
   _Gscansyscall   = _Gscan + _Gsyscall   // 0x1003  
   _Gscanwaiting   = _Gscan + _Gwaiting   // 0x1004  
   _Gscanpreempted = _Gscan + _Gpreempted // 0x1009  
)
```

- 

G 几种暂停的方式：

- `gosched`：将当前的 `G` 暂停，保存堆栈状态，以 `_Grunable` 状态放入 Global 队列中，让当前 `M` 继续执行其他任务。无需对 `G` 执行唤醒操作，因为总会有 `M` 从 Global 队列取得并执行 G，抢占调度即使用该方式。
- `gopark`：与 `gosched` 的最大区别在于 `gopark` 没有将 `G` 放回执行队列，而是位于某个等待队列中（如 `channel` 的 `waitq`，此时 `G` 状态为 `_Gwaitting`）, 因此 `G` 必须被手动唤醒（通过 `goready`），否则会丢失任务。应用层阻塞通常使用这种方式。
- `notesleep`：既不让出 `M`，也不让 `G` 和 `P` 重新调度，直接让线程休眠直到被唤醒（`notewakeup`），该方式更快，通常用于 `gcMark`，`stopm` 这类自旋场景。
- `notesleepg`：阻塞 `G` 和 `M`，放开 `P`，`P` 可以和其它 `M` 绑定继续执行，比如可能阻塞的系统调用会主动调用 `entersyscallblock`，则会触发 `notesleepg`。
- `goexit`：立即终止 `G` 任务，不管其处于调用堆栈的哪个层次，在终止前，确保所有 `defer` 正确执行。


### M

#### 源码

`runtime.m`

```go
type m struct {  
   g0      *g     // goroutine with scheduling stack  
   morebuf gobuf  // gobuf arg to morestack  
   divmod  uint32 // div/mod denominator for arm - known to liblink  
   _       uint32 // align next field to 8 bytes  
  
   // Fields not known to debuggers.  
   procid        uint64            // for debuggers, but offset not hard-coded  
   gsignal       *g                // signal-handling g  
   goSigStack    gsignalStack      // Go-allocated signal handling stack  
   sigmask       sigset            // storage for saved signal mask  
   tls           [tlsSlots]uintptr // thread-local storage (for x86 extern register)  
   mstartfn      func()  
   curg          *g       // current running goroutine  
   caughtsig     guintptr // goroutine running during fatal signal   
   p             puintptr // attached p for executing go code (nil if not executing go code)   
   nextp         puintptr  
   oldp          puintptr // the p that was attached before executing a syscall  
   id            int64  
   mallocing     int32  
   throwing      throwType  
   preemptoff    string // if != "", keep curg running on this m  
   locks         int32  
   dying         int32  
   profilehz     int32  
   spinning      bool // m is out of work and is actively looking for work  
   blocked       bool // m is blocked on a note  
   newSigstack   bool // minit on C thread called sigaltstack  
   printlock     int8  
   incgo         bool          // m is executing a cgo call   
   isextra       bool          // m is an extra m  
   freeWait      atomic.Uint32 // Whether it is safe to free g0 and delete m (one of freeMRef, freeMStack, freeMWait)  
   fastrand      uint64  
   needextram    bool  
   traceback     uint8  
   ncgocall      uint64        // number of cgo calls in total  
   ncgo          int32         // number of cgo calls currently in progress  
   cgoCallersUse atomic.Uint32 // if non-zero, cgoCallers in use temporarily   
   cgoCallers    *cgoCallers   // cgo traceback if crashing in cgo call   park          note  
   alllink       *m // on allm  
   schedlink     muintptr  
   lockedg       guintptr  
   createstack   [32]uintptr // stack that created this thread.   
   lockedExt     uint32      // tracking for external LockOSThread  
   lockedInt     uint32      // tracking for internal lockOSThread  
   nextwaitm     muintptr    // next m waiting for lock  
   waitunlockf   func(*g, unsafe.Pointer) bool  
   waitlock      unsafe.Pointer  
   waittraceev   byte  
   waittraceskip int  
   startingtrace bool  
   syscalltick   uint32  
   freelink      *m // on sched.freem  
  
   // these are here because they are too large to be on the stack  
   // of low-level NOSPLIT functions.  
   libcall   libcall  
   libcallpc uintptr // for cpu profiler  
   libcallsp uintptr  
   libcallg  guintptr  
   syscall   libcall // stores syscall parameters on windows  
   
   vdsoSP uintptr // SP for traceback while in VDSO call (0 if not in call)   
   vdsoPC uintptr // PC for traceback while in VDSO call  
   
   // preemptGen counts the number of completed preemption   
   // signals. This is used to detect when a preemption is   
   // requested, but fails.   
   preemptGen atomic.Uint32  
  
   // Whether this is a pending preemption signal on this M.  
   signalPending atomic.Uint32  
  
   dlogPerM  
   
   mOS  
   
   // Up to 10 locks held by this m, maintained by the lock ranking code.   
   locksHeldLen int  
   locksHeld    [10]heldLockInfo  
}
```

#### 部分字段用途

- `g0`：并不是一个真正的 `goroutine`，他的栈是由操作系统分配的，初始大小比普通 `goroutine` 的栈要大，被用作调度器执行的栈
- `gsignal`：本质上是用来处理信号的栈，因为一些 `UNIX` 系统支持位信号处理器配置独立的栈
- `curg `：指向 `M ` 当前正在执行的 ` G `
- `p`：`GMP` 中的 `P`，即关联当当前 `M` 上的处理器
- `nextp`：用来将 `P` 传递给 `M`，调度器一般是在 `M` 阻塞时为 `m.nextp` 赋值，等到 `M` 开始运行后会尝试从 `nextp` 处获取 `P` 进行关联
- `oldp`：用来暂存执行系统调用之前关联的 `P`
- `id`：`M` 的唯一 `ID`
- `preemptoff`：不为空时表示要关闭对 `curg` 的抢占，字符串内容给出了相关的原因
- `locks`：记录了当前 `M` 持有的锁数量，不为 0 时能够阻止抢占发生
- `spinning`：表示当前 `M` 正处于自旋状态
- `park`：用来支持 `M` 的挂起与唤醒，可以很方便的实现每个 `M` 的单独挂起与唤醒
- `alllink`：把所有的 `M` 连起来，构成 `allm` 链表
- `schedlink`：被调度器用来实现链表，如空闲的 `M` 链表
- `lockedg`：关联到与当前 `M` 绑定的 `G`，可参考 `LockOSThread`
- `freelink`：用来把已经退出运行的 `M` 连起来，构成 `sched.freem` 链表，方便下次分配时复用

### P

#### 源码

`runtime.p`

```go
type p struct {  
   id          int32  
   status      uint32 // one of pidle/prunning/...  
   link        puintptr  
   schedtick   uint32     // incremented on every scheduler call  
   syscalltick uint32     // incremented on every system call  
   sysmontick  sysmontick // last tick observed by sysmon  
   m           muintptr   // back-link to associated m (nil if idle)   
   mcache      *mcache  
   pcache      pageCache  
   raceprocctx uintptr  
  
   deferpool    []*_defer // pool of available defer structs (see panic.go)  
   deferpoolbuf [32]*_defer  
  
   // Cache of goroutine ids, amortizes accesses to runtime·sched.goidgen.  
   goidcache    uint64  
   goidcacheend uint64  
  
   // Queue of runnable goroutines. Accessed without lock.  
   runqhead uint32  
   runqtail uint32  
   runq     [256]guintptr  
   // runnext, if non-nil, is a runnable G that was ready'd by  
   // the current G and should be run next instead of what's in   
   // runq if there's time remaining in the running G's time   
   // slice. It will inherit the time left in the current time  
   // slice. If a set of goroutines is locked in a  
   // communicate-and-wait pattern, this schedules that set as a   
   // unit and eliminates the (potentially large) scheduling   
   // latency that otherwise arises from adding the ready'd  
   // goroutines to the end of the run queue.   
   //   
   // Note that while other P's may atomically CAS this to zero,   
   // only the owner P can CAS it to a valid G.   
   runnext guintptr  
  
   // Available G's (status == Gdead)   
   gFree struct {  
      gList  
      n int32  
   }  
  
   sudogcache []*sudog  
   sudogbuf   [128]*sudog  
  
   // Cache of mspan objects from the heap.   
   mspancache struct {  
      // We need an explicit length here because this field is used  
      // in allocation codepaths where write barriers are not allowed,      
      // and eliminating the write barrier/keeping it eliminated from      
      // slice updates is tricky, moreso than just managing the length      
      // ourselves.      
      len int  
      buf [128]*mspan  
   }  
  
   tracebuf traceBufPtr  
  
   // traceSweep indicates the sweep events should be traced.   
   // This is used to defer the sweep start event until a span   
   // has actually been swept.   
   traceSweep bool  
   // traceSwept and traceReclaimed track the number of bytes  
   // swept and reclaimed by sweeping in the current sweep loop.   
   traceSwept, traceReclaimed uintptr  
  
   palloc persistentAlloc // per-P to avoid mutex  
  
   // The when field of the first entry on the timer heap.   
   // This is 0 if the timer heap is empty.  
   timer0When atomic.Int64  
  
   // The earliest known nextwhen field of a timer with   
   // timerModifiedEarlier status. Because the timer may have been   
   // modified again, there need not be any timer with this value.   
   // This is 0 if there are no timerModifiedEarlier timers.  
   timerModifiedEarliest atomic.Int64  
  
   // Per-P GC state   
   gcAssistTime         int64 // Nanoseconds in assistAlloc  
   gcFractionalMarkTime int64 // Nanoseconds in fractional mark worker (atomic)  
  
   // limiterEvent tracks events for the GC CPU limiter.   
   limiterEvent limiterEvent  
  
   // gcMarkWorkerMode is the mode for the next mark worker to run in.   
   // That is, this is used to communicate with the worker goroutine   
   // selected for immediate execution by   
   // gcController.findRunnableGCWorker. When scheduling other goroutines,   
   // this field must be set to gcMarkWorkerNotWorker.  
   gcMarkWorkerMode gcMarkWorkerMode  
   // gcMarkWorkerStartTime is the nanotime() at which the most recent  
   // mark worker started.   
   gcMarkWorkerStartTime int64  
  
   // gcw is this P's GC work buffer cache. The work buffer is   
   // filled by write barriers, drained by mutator assists, and   
   // disposed on certain GC state transitions.   
   gcw gcWork  
  
   // wbBuf is this P's GC write barrier buffer.   
   //   
   // TODO: Consider caching this in the running G.   
   wbBuf wbBuf  
  
   runSafePointFn uint32 // if 1, run sched.safePointFn at next safe point  
  
   // statsSeq is a counter indicating whether this P is currently   
   // writing any stats. Its value is even when not, odd when it is.   
   statsSeq atomic.Uint32  
  
   // Lock for timers. We normally access the timers while running   
   // on this P, but the scheduler can also do it from a different P.   
   timersLock mutex  
  
   // Actions to take at some time. This is used to implement the  
   // standard library's time package.   
   // Must hold timersLock to access.  
   timers []*timer  
  
   // Number of timers in P's heap.   
   numTimers atomic.Uint32  
  
   // Number of timerDeleted timers in P's heap.   
   deletedTimers atomic.Uint32  
  
   // Race context used while executing timer functions.   
   timerRaceCtx uintptr  
  
   // maxStackScanDelta accumulates the amount of stack space held by   
   // live goroutines (i.e. those eligible for stack scanning).   
   // Flushed to gcController.maxStackScan once maxStackScanSlack  
   // or -maxStackScanSlack is reached.  
   maxStackScanDelta int64  
  
   // gc-time statistics about current goroutines  
   // Note that this differs from maxStackScan in that this   
   // accumulates the actual stack observed to be used at GC time (hi - sp),   
   // not an instantaneous measure of the total stack size that might need   
   // to be scanned (hi - lo).   
   scannedStackSize uint64 // stack size of goroutines scanned by this P   
   scannedStacks    uint64 // number of goroutines scanned by this P  
  
   // preempt is set to indicate that this P should be enter the   
   // scheduler ASAP (regardless of what G is running on it).   
   preempt bool  
  
   // pageTraceBuf is a buffer for writing out page allocation/free/scavenge traces.   
   //   
   // Used only if GOEXPERIMENT=pagetrace.   
   pageTraceBuf pageTraceBuf  
  
   // Padding is no longer needed. False sharing is now not a worry because p is large enough  
   // that its size class is an integer multiple of the cache line size (for any of our architectures).
   }
```

#### 部分字段用途

- `id`：`P` 的唯一 `ID`，等于当前 `P` 在 `allp` 中的数组下标
- `staus`： 表示 `P` 的状态
- `link`：是一个没有写屏障的指针，别调度器用来构造链表
- `schedtick`：记录了调度发生的次数，实际上在每一次发生 `goroutine` 切换且不继承时间片的情况下，该字段会加一
- `syscalltick`：每发生一次系统调用就会加一
- `sysmontick`：被监控线程用来存储上一次检查时的调度器时钟滴答，用以实现时间片算法
- `m`：本质上是个指针，反向关联到当前 `P` 绑定的 `M`
- `goidcache`，`goidcacheend`：用来从全局 `sched`，`goidgen` 处申请 goid`，`分配空间，批量申请以减少全局范围的锁竞争
- `runqhead`，`runqtail`，`runq`：当前 `P` 的就绪队列，用一个数组和一头一尾两个下标实现了一个环形队列
- `runnext`：如果不为 `nil`，则指向一个被当前 `G` 准备好（就绪）的 `G`，接下来将会继承当前 `G` 的时间片开始运行。该字段的存在意义在于，假如有一组 `goroutine` 中有生产者和消费者，他们在一个 `channel` 上频繁地等待和唤醒，那么调度器就会把他们作为一个单元来调度。每次使用 `runnext` 比添加到本地 `runq` 尾部能大幅减少延迟
- `gFree`：用来缓存已经退出运行的 `G`，方便再次分配时进行复用
- `preempt`：在 `Go1.14` 版本引入，以支持新的异步抢占机制

#### status 字段的取值及其含义

```go
const (  
   // P status  
  
   // _Pidle means a P is not being used to run user code or the   
   // scheduler. Typically, it's on the idle P list and available   
   // to the scheduler, but it may just be transitioning between   
   // other states.   
   //   
   // The P is owned by the idle list or by whatever is   
   // transitioning its state. Its run queue is empty.  
   _Pidle = iota  
  
   // _Prunning means a P is owned by an M and is being used to   
   // run user code or the scheduler. Only the M that owns this P   
   // is allowed to change the P's status from _Prunning. The M  
   // may transition the P to _Pidle (if it has no more work to   
   // do), _Psyscall (when entering a syscall), or _Pgcstop (to   
   // halt for the GC). The M may also hand ownership of the P  
   // off directly to another M (e.g., to schedule a locked G).   
   _Prunning  
  
   // _Psyscall means a P is not running user code. It has   
   // affinity to an M in a syscall but is not owned by it and   
   // may be stolen by another M. This is similar to _Pidle but   
   // uses lightweight transitions and maintains M affinity.   
   //   
   // Leaving _Psyscall must be done with a CAS, either to steal   
   // or retake the P. Note that there's an ABA hazard: even if   
   // an M successfully CASes its original P back to _Prunning  
   // after a syscall, it must understand the P may have been  
   // used by another M in the interim.   
   _Psyscall  
  
   // _Pgcstop means a P is halted for STW and owned by the M   
   // that stopped the world. The M that stopped the world   
   // continues to use its P, even in _Pgcstop. Transitioning  
   // from _Prunning to _Pgcstop causes an M to release its P and   
   // park.   
   //   
   // The P retains its run queue and startTheWorld will restart   
   // the scheduler on Ps with non-empty run queues.   
   _Pgcstop  
  
   // _Pdead means a P is no longer used (GOMAXPROCS shrank). We   
   // reuse Ps if GOMAXPROCS increases. A dead P is mostly   
   // stripped of its resources, though a few things remain   
   // (e.g., trace buffers).   
   _Pdead  
)
```