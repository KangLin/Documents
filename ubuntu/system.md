
## Telnet
### 安装openbsd-inetd

```bash
sudo apt-get install openbsd-inetd
```

### 安装telnetd

```bash
sudo apt-get install telnetd
```
 

sudo vim /etc/inetd.conf  末尾手动添加 telnet stream tcp nowait root /usr/sbin/tcpd /usr/sbin/in.telnetd

 

### 重启openbsd-inetd

```bash
sudo /etc/init.d/openbsd-inetd restart
```
        如下输出，说明服务启动成功：
        * Restarting internet superserver inetd [ OK ]

### 查看telnet运行状态

```bash
sudo netstat -a | grep telnet
```

        如下输出，说明telnet服务运行正常：
        tcp 0 0 *:telnet *:* LISTEN

 

### 关闭服务：

```bash
sudo /etc/init.d/openbsd-inetd stop
```

/etc/inetd.conf 去掉  telnet stream tcp nowait root /usr/sbin/tcpd /usr/sbin/in.telnetd


### 修改telnet端口：

/etc/services 修改默认端口 23

## ssh

### 对于具有systemd的系统：

```bash
sudo systemctl disable ssh.service
```

然后

```bash
sudo systemctl stop ssh.service
```

参考：
- https://blog.csdn.net/xqhrs232/article/details/104136472
- https://askubuntu.com/questions/56753/how-do-i-disable-sshd-from-starting-automatically

