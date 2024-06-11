# Ubuntu

作者：康林(kl222@126.com)

-------------------------

### [Ubuntu 文档](https://www.debian.org/doc/)
- [软件包制做](https://www.debian.org/doc/manuals/packaging-tutorial/packaging-tutorial.en.pdf)
- [Debian 新维护者手册](https://www.debian.org/doc/manuals/maint-guide/index.zh-cn.html)
- [Debian 维护者指南](https://www.debian.org/doc/manuals/debmake-doc/index.zh-cn.html)

- [dh_shlibdeps](http://www.man7.org/linux/man-pages/man1/dh_shlibdeps.1.html)

- [mentors](https://mentors.debian.net)
- [维护者简介：我的软件包将如何进入 Debian?](https://mentors.debian.net/intro-maintainers/)
- [发布软件到debian源.md](https://x.hacking8.com/post-386.html)
- [打包、发布](package.md)

### [Ubuntu 系统](system.md)

### Ubuntu 系统设置

Ubuntu 系统的用户桌面默认目录设置位置：

        ~/.config/user-dirs.dirs

语言设置：

        ~/.config/user-dirs.locale

### 加密文件夹

#### eCryptfs的安装

        # Debian，Ubuntu或其衍生版：
        $ sudo apt-get install ecryptfs-utils
        # CentOS， RHEL or Fedora：
        $ yum install ecryptfs-utils
        # Arch Linux：
        $ sudo pacman -S ecryptfs-utils

#### 使用eCryptfs
1. 运行 eCryptFS 配置工具来加密某个目录：

        $ ecryptfs-setup-private

它将要求输入登录密码和安装密码。登录密码与您的正常登录密码相同。安装密码用于派生文件加密主密钥。将其留空以生成一个，因为这样更安全。注销并重新登录。

您会注意到，eCryptFS 默认情况下创建了两个目录：您的主目录中的 Private 和 .Private。 ~/.Private目录包含加密数据，您可以在~/Private目录中访问相应的解密数据。当您登录时，~/.Private目录会自动解密并映射到~/Private目录，以便您可以访问它。当您注销时，~/Private 目录会自动卸载，并且 ~/Private 目录中的内容会加密回 ~/.Private 目录中。

eCryptFS 知道您拥有 ~/.Private 目录，并自动将其解密到 ~/Private 目录中，而无需您输入密码，这是通过 eCryptFS PAM 模块 来完成的。

如果您不希望在登录时自动挂载 ~/Private 目录，只需在运行 ecryptfs-setup-private 工具时添加 --noautomount 选项即可。同样，如果您不希望注销后自动卸载 ~/Private 目录，请指定 --noautoumount 选项。但是，您将必须自己手动挂载或卸载 ~/Private 目录：

        $ ecryptfs-mount-private ~/.Private ~/Private
        $ ecryptfs-umount-private ~/Private

2. 加载eCryptFS内核模块(可选,一般会自动加载):

        $ sudo modprobe ecryptfs

3. 您可以通过运行以下命令来验证 .Private 文件夹是否已挂载：

        $ mount | grep ecryptfs

#### 参考：
- https://cn.linux-console.net/?p=9052
- https://blog.csdn.net/u013316945/article/details/132657776
- https://blog.csdn.net/feelabclihu/article/details/121433912
