# 常用命令合集

## find

找出/home下不是以 .txt 结尾的文件

```bash
find /home ! -name "*.txt"
```

## lsof

lsof : list oen files

**List all IPv4 network files**

`sudo lsof -i4`

**List all open sockets**

`lsof -i`

 **List all listening ports**

`lsof -Pnl +M -i4`

**Find which program is using the port 80**

`lsof -i TCP:80`

**List all connections to a specific host**

`lsof -i@192.168.1.5`

**List all files/network connections a command is using**

`lsof -c <command-name>`

**List all files a process has open**

`lsof -p <pid>`

