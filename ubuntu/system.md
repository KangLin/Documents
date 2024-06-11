
## Telnet
### 安装openbsd-inetd

```bash
sudo apt-get install openbsd-inetd
```

### 安装telnetd

```bash
sudo apt-get install telnetd
```
 

        sudo vim /etc/inetd.conf  #末尾手动添加 
        telnet stream tcp nowait root /usr/sbin/tcpd /usr/sbin/in.telnetd

 

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

        vim /etc/inetd.conf #去掉  
        telnet stream tcp nowait root /usr/sbin/tcpd /usr/sbin/in.telnetd


### 修改telnet端口：

        /etc/services 修改默认端口 23

## ssh

### 安装

    sudo apt install openssh-server

### 对于具有systemd的系统：

- 允许

```bash
sudo systemctl enable ssh.service
```

- 开始

```bash
sudo systemctl start ssh.service
```

- 查看状态

```bash
sudo systemctl status ssh.service
```

- 停止

```bash
sudo systemctl stop ssh.service
```

### 禁止用户登录
- 打开文件：

        vim /etc/ssh/sshd_config

修改：

        DenyUsers user_name
 
### 允许 root 用户登录

- 打开文件：

        vim /etc/ssh/sshd_config

- 修改 PermitRootLogin 的值：

        PermitRootLogin yes

|参数类别               |允许 ssh 登陆 |登录方式      |交互shell         |
|-----------------------|--------------|--------------|------------------|
|yes	                |允许          |没有限制      |没有限制          |
|prohibit-password      |允许          |除密码以外    |没有限制          |
|without-password(废弃)	|允许          |除密码以外    |没有限制          |
|forced-commands-only	|允许          |仅允许使用密钥|仅允许已授权的命令|
|prohibit-password      |允许          |除密码以外    |没有限制          |
|no                     |不允许        |N/A           |	N/A              |

- 允许无密码登录

        PermitEmptyPasswords yes

### 允许密码登录

        PasswordAuthentication yes

### 使用密钥登录

- 密钥登录的过程

SSH 密钥登录分为以下的步骤。

预备步骤，客户端通过ssh-keygen生成自己的公钥和私钥。

第一步，手动将客户端的公钥放入远程服务器的指定位置。

第二步，客户端向服务器发起 SSH 登录的请求。

第三步，服务器收到用户 SSH 登录的请求，发送一些随机数据给用户，要求用户证明自己的身份。

第四步，客户端收到服务器发来的数据，使用私钥对数据进行签名，然后再发还给服务器。

第五步，服务器收到客户端发来的加密签名后，使用对应的公钥解密，然后跟原始数据比较。如果一致，就允许用户登录。

- 允许公钥登录（默认）

        PubkeyAuthentication yes  # yes表示允许密钥登陆
        AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2 # 指定密钥的文件位置, 必须指定

- 客户端通过 ssh-keygen 生成自己的公钥和私钥

        ssh-keygen

- 在服务器上安装公钥

    老的 OpenSSH ，用户公钥保存在服务器的 ~/.ssh/authorized_keys 文件。你要以哪个用户的身份登录到服务器，密钥就必须保存在该用户主目录的 ~/.ssh/authorized_keys 文件。新的 OpenSSH 需要要配置文件中 AuthorizedKeysFile 指定。只要把公钥添加到这个文件之中，就相当于公钥上传到服务器了。每个公钥占据一行。如果该文件不存在，可以手动创建。 

  - 如果不能使用 ssh 登录服务器，那么用其它方法（例如：使用U盘）在服务器上安装

         $ cd ~/.ssh
         $ cat /media/U/id_rsa.pub >> ~/.ssh/authorized_keys

  - 如果客户端可以使用 ssh 登录服务器

         $ cat ~/.ssh/id_rsa.pub | ssh user@host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

  - ssh-copy-id 命令：自动上传公钥

  OpenSSH 自带一个 ssh-copy-id 命令，可以自动将公钥拷贝到远程服务器的 ~/.ssh/authorized_keys 文件。如果 ~/.ssh/authorized_keys 文件不存在，ssh-copy-id 命令会自动创建该文件。

    用户在本地计算机执行下面的命令，就可以把本地的公钥拷贝到服务器。

         l@l:~$ ssh-copy-id -i ~/.ssh/id_rsa.pub user@host  

### 参考：
- man sshd_config
- [如何禁用SSHD自动启动](https://blog.csdn.net/xqhrs232/article/details/104136472)
- https://askubuntu.com/questions/56753/how-do-i-disable-sshd-from-starting-automatically
- [sshd_config 中 PermitRootLogin 的探讨](https://blog.csdn.net/huigher/article/details/52972013)
- [SSH 如何使用密钥登录服务器](https://cloud.tencent.com/developer/article/1780788)
- [SSH 使用密钥远程登录](https://blog.csdn.net/wzfgd/article/details/114128524)

## 加密文件夹

### 安装 ecryptfs

    l@l:~$ sudo apt install ecryptfs-utils

### 现在让我们开始通过运行 eCryptFS 配置工具来加密某个目录：

    l@l:~$ ecryptfs-setup-private

如果出现下面错误：

    ERROR:  Cannot get ecryptfs version, ecryptfs kernel module not loaded?

加载 ecryptfs 模块:

    l@l:~$ sudo modprobe ecryptfs

然后再执行 ecryptfs-setup-private :

    l@l:~$ ecryptfs-setup-private


它将要求输入登录密码和安装密码。登录密码与您的正常登录密码相同。安装密码用于派生文件加密主密钥。将其留空以自动随机生成一个密码，因为这样更安全。注销并重新登录。

您会注意到，eCryptFS 默认情况下创建了两个目录：您的主目录中的 Private 和 .Private。 ~/.Private目录包含加密数据，您可以在 ~/Private目录中访问相应的解密数据。当您登录时， ~/.Private目录会自动解密并映射到 ~/Private目录，以便您可以访问它。当您注销时， ~/Private 目录会自动卸载，并且 ~/Private 目录中的内容会加密回 ~/.Private 目录中。

eCryptFS 知道您拥有 ~/.Private 目录，并自动将其解密到 ~/Private 目录中，而无需您输入密码，这是通过 eCryptFS PAM 模块 来完成的。

如果您不希望在登录时自动挂载 ~/Private 目录，只需在运行 ecryptfs-setup-private 工具时添加 --noautomount 选项即可。同样，如果您不希望注销后自动卸载 ~/Private 目录，请指定 --noautoumount 选项。但是，您将必须自己手动挂载或卸载 ~/Private 目录：

     $ ecryptfs-mount-private ~/.Private ~/Private
     $ ecryptfs-umount-private ~/Private

### 查看是否加载

    l@l:~$ mount |grep ecryptfs
    /home/l/.Private on /home/l/Private type ecryptfs (rw,nosuid,nodev,relatime,ecryptfs_fnek_sig=ba1fd50362065a83,ecryptfs_sig=42f56088c83f3865,ecryptfs_cipher=aes,ecryptfs_key_bytes=16,ecryptfs_unlink_sigs)

### 参考:
- [如何在 Linux 上使用 eCryptFS 加密文件和目录](https://cn.linux-console.net/?p=9052)
- [eCryptfs，文件系统级加密，在登出时自动为文件加密。通过挂载文件解密和卸载文件加密的方式保护文件](https://blog.csdn.net/ck784101777/article/details/104544731)
