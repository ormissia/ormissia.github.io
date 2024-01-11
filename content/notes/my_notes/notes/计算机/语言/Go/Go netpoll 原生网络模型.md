# Go netpoll 原生网络模型

#go #golang #tcp #网络 #网络模型 #netpoll #epoll

## 一个简单的 `TCP` 服务器

```go
func main() {  
   l, err := net.Listen("tcp4", ":8000")  
   if err != nil {  
      log.Fatal(err)  
   }  
   defer func() {  
      _ = l.Close()  
   }()  
   log.Println("server listen port: 8000")  
  
   var id int  
   for {  
      conn, err := l.Accept()  
      if err != nil {  
         log.Fatal(err)  
      }  
      id++  
  
      go func(id int, conn net.Conn) {  
         defer func() {  
            _ = conn.Close()  
         }()  
         var packet [0xFFF]byte  
         for {  
            n, err := conn.Read(packet[:])  
            if err != nil {  
               return  
            }  
            _, _ = conn.Write(packet[:n])  
         }  
      }(id, conn)  
   }  
}
```