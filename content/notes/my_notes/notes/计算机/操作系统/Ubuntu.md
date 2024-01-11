# Ubuntu

#linux #ubuntu #ssh

## 开启 SSH 服务

检查是否有 `ssh`
```shell
ps -e | grep ssh
```

更新软件列表和软件
```shell
sudo apt-get update
sudo apt-get upgrade
```

安装 `ssh`
```shell
sudo apt-get install ssh
```

启动 `ssh`
```shell
sudo /etc/int.d/ssh start
```

再次检查是否有 `ssh`，当结果里看到 `sshd` 即为 `ssh` 服务端已启动

