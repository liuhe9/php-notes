mac下ab测试出现

```
apr_socket_recv: Connection reset by peer (54)
```

解决办法

```
# 查看当前内核和进程能打开的文件描述符限制
$ sysctl -A | grep kern.maxfiles
kern.maxfiles: 12288             # 系统级的限制
kern.maxfilesperproc: 10240    # 内核级的限制

# 通过sysctl命令热更改 系统重启后失效
$ sysctl -w kern.maxfilesperproc=20480

# 通过配置文件永久更改 重启生效
# 在/etc/sysctl.conf中写入
kern.maxfiles=20480000 kern.maxfilesperproc=20480000
$ ulimit -S -n
10240
$ ulimit -S -n 20480000
$ ulimit -S -n
20480000
```



