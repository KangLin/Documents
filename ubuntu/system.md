
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

### 添加二次验证
- [安装 TOTP PAM 模块](#安装-TOTP-PAM-模块)
- [为用户生成密钥](#为用户生成密钥)
- [为SSH远程登录添加二次验证](#为SSH远程登录添加二次验证)

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

它将要求输入登录密码和安装密码。登录密码与您的正常登录密码相同。安装密码用于生成文件加解密的主密钥。
（安装密码由登录密码加密后保存在 ~/.ecryptfs/wrapped-passphrase 中。) 将其留空以自动随机生成一个密码，因为这样更安全。注销并重新登录。

您会注意到，eCryptFS 默认情况下创建了两个目录：您的主目录中的 Private 和 .Private。
~/.Private目录包含加密数据，您可以在 ~/Private目录中访问相应的解密数据。
当您登录时， ~/.Private目录会自动解密并映射到 ~/Private目录，以便您可以访问它。
当您注销时， ~/Private 目录会自动卸载，并且 ~/Private 目录中的内容会加密回 ~/.Private 目录中。

eCryptFS 知道您拥有 ~/.Private 目录，并自动将其解密到 ~/Private 目录中，而无需您输入密码，这是通过 eCryptFS PAM 模块 来完成的。

如果您不希望在登录时自动挂载 ~/Private 目录，只需在运行 ecryptfs-setup-private 工具时添加 --noautomount 选项即可。
同样，如果您不希望注销后自动卸载 ~/Private 目录，请指定 --noautoumount 选项。但是，您将必须自己手动挂载或卸载 ~/Private 目录：

    $ ecryptfs-mount-private ~/.Private ~/Private
    $ ecryptfs-umount-private ~/Private

### 配置位置

配置放在 ~/.ecryptfs 目录下，你可以看到相关的文件：

    mint@mint-H310M-BATTLE-GENERAL-D4-G10b:~/.ecryptfs$ ls
    auto-mount  auto-umount  Private.mnt  Private.sig  wrapped-passphrase

- wrapped-passphrase：保存用于文件加解密的主密钥（此密钥被登录密码进行加密）
- Private.mnt：保存 Private 的位置。默认为: ~/Private

### 查看是否加载

    l@l:~$ mount |grep ecryptfs
    /home/l/.Private on /home/l/Private type ecryptfs (rw,nosuid,nodev,relatime,ecryptfs_fnek_sig=ba1fd50362065a83,ecryptfs_sig=42f56088c83f3865,ecryptfs_cipher=aes,ecryptfs_key_bytes=16,ecryptfs_unlink_sigs)

### 多个操作系统多用户使用相同的 eCryptFS
例如： 在一台主机上有两个操作系统 Ubuntu(用户ubuntu)、 Linuxmint(用户mint)
- 安装 Ubuntu,并安装设置好 eCryptFS。
  - 设置目录权限

        ubuntu@ubuntu:~$ chmod a+rw ~/.ecryptfs
        ubuntu@ubuntu:~$ chmod a+rw ~/.Private
        ubuntu@ubuntu:~$ chmod a+rw ~/Private

- 安装 Linuxmint,并安装 eCryptFS。
  - 在主目录下链接 ~/.ecryptfs

        mint@mint-H310M-BATTLE-GENERAL-D4-G10b:~$ ln -s /home/ubuntu/.ecryptfs ~/.ecryptfs
        mint@mint-H310M-BATTLE-GENERAL-D4-G10b:~$ ln -s /home/ubuntu/.Private ~/.Private
        mint@mint-H310M-BATTLE-GENERAL-D4-G10b:~$ ln -s /home/ubuntu/Private ~/Private

  - 修改当前用户密码与 ubuntu 用户的密码相同
这样，就可以两操作系统使用相同的 eCryptFS 了。

### 同一系统上，多用户使用相同的 eCryptFS
国为 eCryptFs 加载 ~/.Private 后，就解密到 ~/Private 中。
如果此户未注销，同时其它用户登录，因为其它用户有 ~/Private 的读写权限。
那么其它用户就能使用它。

    ubuntu@ubuntu:~$ chmod 777 ~/Private

如果你不想其它用户使用它，那么你需要设置 ~/Private 的权限，只有当前用户有权限：

    ubuntu@ubuntu:~$ chmod 700 ~/Private

### 参考:
- [如何在 Linux 上使用 eCryptFS 加密文件和目录](https://cn.linux-console.net/?p=9052)
- [eCryptfs，文件系统级加密，在登出时自动为文件加密。通过挂载文件解密和卸载文件加密的方式保护文件](https://blog.csdn.net/ck784101777/article/details/104544731)

## 登录添加二次验证

- [Initiative for Open Authentication (OATH)](https://www.openauthentication.org/)
  - TOTP(Time-based One-Time Password):
    Time-synchronized OTP used by many authenticator apps and services.
    [RFC 6238](https://rfcinfo.com/rfc-6238/)

### 安装 TOTP PAM 模块
在终端执行：

    sudo apt update
    sudo apt install libpam-google-authenticator -y

### 为用户生成密钥
切换到需要开启验证的用户（不要直接用 root 操作），运行初始化命令：

    google-authenticator

终端会显示一个巨大的二维码和密钥。
用手机上的 Google Authenticator、Authy 或 Microsoft Authenticator 扫码绑定。
务必抄下屏幕上显示的 5 个紧急备用码，手机丢失时全靠它登录。
后续提示建议选择：基于时间（y）、禁止重复使用（y）、开启限速（y）。

配置保存在 `~/.google_authenticator`

### 为本地图形界面/终端登录添加二次验证

- 修改 PAM 配置。根据你的需求修改：
  - 所有的验证都添加二次验证

        sudo vim /etc/pam.d/common-auth

  - 只有图形登录需要添加二次验证。根据使用的桌面，选择相应的 gdm (图形桌面管理器)

        sudo vim /etc/pam.d/gdm-password  # GNOME 桌面使用 gdm

  - 只有 TTY 登录需要添加二次验证

        sudo vim /etc/pam.d/login         # TTY 登录

  - 只有 SSH 登录添加二次验证

        sudo vim /etc/pam.d/sshd          # SSH 登录

- 在文件末尾添加（注意顺序，通常加在 pam_unix.so 之后）：

      auth required pam_google_authenticator.so

风险提示：修改此文件会影响所有认证方式（包括 sudo）。
建议先在 common-auth 里使用 nullok 参数测试，或者确保你有 TTY 备用登录方式。
并务必带上 nullok 参数：

    auth required pam_google_authenticator.so nullok

参见：[重置用户密码](#重置用户密码)

- 生效
保存后，注销当前用户或重启，再次登录时就会要求输入验证码。

关键安全提醒
备用码是生命线：生成的紧急备用码必须离线保存（写在纸上或存进离线密码管理器）。
一旦手机丢失且没有备用码，你将无法登录系统。

时间同步：TOTP 依赖时间。确保 Ubuntu 系统时间准确（timedatectl 查看），否则验证码会一直错误。

- 本地登录特有的风险与对策
与 SSH 不同，本地物理机没有“另开一个 SSH 会话测试”的便利。你需要额外做两件事：

第一，保留一个不受影响的 root 终端会话。 修改 common-auth 时，
不要关闭当前已经用 sudo -i 或 su - 获得的 root shell。
如果配置出错导致新登录全部失败，你还可以在这个已打开的会话里用 sudo nano 把配置改回去。


### 为SSH远程登录添加二次验证

- 配置 PAM 与 SSH
  - 编辑 SSH 的 PAM 文件：

        sudo vim /etc/pam.d/sshd

  - 在文件最顶部（@include common-auth 之前）添加一行：

        auth required pam_google_authenticator.so

- 编辑 SSH 主配置：

      vim /etc/ssh/sshd_config

  - 确保以下配置生效（新版 Ubuntu 用 KbdInteractiveAuthentication 替代了 ChallengeResponseAuthentication）：
  
        UsePAM yes
        KbdInteractiveAuthentication yes

注意：如果你原本使用密钥登录且不想被动态码打断，需要确保 AuthenticationMethods 包含了 keyboard-interactive，
或者暂时保留 PasswordAuthentication yes 来测试。

- 重启并测试

      sudo systemctl restart ssh

不要关闭当前窗口，新开一个终端尝试 SSH 连接。如果成功，系统会先询问密码，再询问 Verification code

## 重置用户密码
### 恢复模式。 
  - 进入 GRUB 菜单并选择恢复模式

    重启电脑，在启动过程中长按 Shift 键（UEFI 机型可尝试反复按 Esc）调出 GRUB 菜单。
选择 “Advanced options for Ubuntu”，然后选中带有 “(recovery mode)” 字样的内核版本并回车。

  - 获取可写权限并重置密码

    在恢复菜单中，选择 “root” 或 “Drop to root shell prompt” 进入 root 命令行。
此时文件系统通常是只读的，需要先重新挂载为可写：

        mount -o remount,rw /

  - 然后使用 passwd 命令重置你忘记密码的那个账户（将 你的用户名 替换为实际用户名）：
  
        passwd 你的用户名

    按提示输入两次新密码（输入时屏幕不会显示字符，这是正常的）。
  - 重启生效
    密码修改成功后，输入 reboot -f 强制重启，或者选择菜单中的 resume 正常引导。
- Live USB 救援

### Live USB 救援（如果恢复模式进不去或 GRUB 被锁）
- 挂载原系统分区
  从 Live USB 启动并选择“Try Ubuntu”，打开终端。
  先用 lsblk 或 fdisk -l 找到你原系统的根分区（例如 /dev/sda2 或 /dev/nvme0n1p2）。
  然后挂载它：

      sudo mount /dev/sda2 /mnt   # 请替换为实际的根分区

- 切换环境并重置密码
  切换到挂载好的原系统环境中，然后执行重置：

      $ sudo chroot /mnt
      # passwd 你的用户名
      # exit

-  完成后重启并拔掉 U 盘即可。
