# CentOS开启防火墙及开放指定端口

查看防火墙状态

```Plain Text
systemctl status firewalld
```


启动防火墙

```Plain Text
systemctl start firewalld
```


停止某项服务，这里举例停止防火墙

```Plain Text
systemctl stop firewalld
```


查看防火墙已经开放的端口

```Plain Text
firewall-cmd --list-port
```


添加开放指定防火墙

```Plain Text
firewall-cmd --zone=public --add-port=这里是需要开启的端口号/tcp --permanent
```


移除开放指定防火墙

```Plain Text
firewall-cmd --zone=public --remove-port=这里是需要关闭的端口号/tcp --permanent
```


重新加载防火墙

```Plain Text
firewall-cmd --reload
```
