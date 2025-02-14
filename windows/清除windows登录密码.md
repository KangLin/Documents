## 清除windows登录密码

@[TOC](清除windows登录密码)

-------------------------------------------------------------------------------

作者：康林(kl222@126.com)

-------------------------------------------------------------------------------

# 原理

因为登录界面中的轻松使用中的工具是以管理员账户启动的，所以它们具有管理员权限。我们只需要替换登录界面中的轻松使用中的工具为 cmd.exe 。有了具有管理员权限的 cmd.exe 以后，你想做啥就做啥 :) 
此方法适用所WINDOWS版本。

## 替换登录界面中的工具
### 登录界面中的轻松使用中的工具一般有放大镜（c:\Windows\System32\Magnify.exe）、屏幕键盘、讲述人等。可以使用下列方法之一完成：

- 用包含了 LINUX 或 WINPE 的启动 U 盘，启动系统，然后替换放大镜程序为 cmd.exe  
现在的 LINUX 或 WINPE  启动 U盘都是图形界面的，所以很容易进行替换操作。 
- 取下系统盘，放到其它机器上，在其它机器上的系统中进行替换。

#### 命令行下的操作：

- linux
  
```
mount /dev/sdb1 /X                  #加载原来的C盘到挂载点 /X
cd /X/Windows/System32              # X 为在新系统中原来 C 盘的挂载点（mount）
cp Magnify.exe Magnify.exe.bak
cp cmd.exe Magnify.exe
```
- windows
    
```
    cd X:\Windows\System32          # X 为在新系统中原来C盘的盘符
    copy Magnify.exe Magnify.exe.bak
    copy cmd.exe Magnify.exe
```

#### 图形界面操作

### 替换后重启系统，点登录界面中的轻松使用工具中的放大镜，会显示命令行控制台。

### 清除登录密码或设置密码

在命令行控制台中输入下列命令，会出现修改用户对话框。

```
control userpasswords2
```

清除或修改完成密码后，就可以重新登录了。

### 增加用户
在命令行控制台中增加新用户：

例如：增加用户 admin，密码 123，并增加到管理组。

```
net user admin 123 /add
net localgroup administrators admin /add 
```

#### 切换用户

windows10在登录界面按 ALT＋F4，切换用户。选择刚才加入的 admin,然后输入密码登录。

## 其它
### windows 10 修改密码
清除登录密码或设置密码

```
control userpasswords2
```

### 增加或删除用户
打开“控制面板->用户”进行设置

### [如何在win10中设置免密码登录](https://jingyan.baidu.com/article/37bce2be3061e25103f3a24c.html)
按win键弹出设置菜单，点击设置按钮，进入设置界面。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/04a6707d9b57b68ccdf1d0a9fd08acf0.png)
在Windows设置窗口，点击“帐户”即可进入，相关密码设置。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3ae3b5579a846d1552a207179959354d.png)
点击左侧登录选项按钮，即可设置相关登录密码问题。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/05d4d0db1c02bec905c4c82289a44dd2.png)
点击密码按钮，则可进入密码设置选项，可以重新设置密码。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/938dc070a5b4c6317e1817171c06ff5c.png)
显示此帐户没有密码，点击“添加”则可以添加一个密码。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d14801ed50bfcde794a7558d6f3da5f8.png)
在新密码和确认密码位置处不输入任何东西，点击下一步即可创建空密码。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eee9ce643991a0c03489175321fc18ba.png)


### 安全问题
本方法能清除掉目前所有 WINDOWS 版本（windows xp 到 windows 10）的登录密码。所以，如果用于非授权用途，账户所有人是会立即发现密码失效。所以并不会导致严重的安全风险。
当然，如果你对安全有极高的要求，建议使用文件加密。或者，使用物理隔离（其它人接触不到此电脑）。

