
# 问题集锦
## mac下ab测试出现 
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
$ sysctl -w kern.maxfiles=204800
$ sysctl -w kern.maxfilesperproc=204800

# 通过配置文件永久更改 重启生效
# 在/etc/sysctl.conf中写入
kern.maxfiles=204800 kern.maxfilesperproc=204800
$ ulimit -S -n
10240
$ ulimit -S -n 204800
$ ulimit -S -n
204800
```
# PHP大神博客
+ [Laruence](http://www.laruence.com/)
+ [韩天峰](http://rango.swoole.com/)
