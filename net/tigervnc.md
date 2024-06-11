# 如何在 Debian 12 上安装 VNC 服务器
-----------------------------------------

虚拟网络计算 (VNC) 是一种桌面共享协议，允许您使用 VNC 客户端软件远程控制计算机。 VNC 工作在 GUI（图形用户界面）环境中，它使用远程帧缓冲区 (RFB) 协议通过网络传输鼠标和键盘输入的移动。

通常，VNC 用于技术人员控制客户端桌面，或供需要在家中访问办公室桌面的人员使用。可以通过 VPN 网络或使用 SSH 隧道连接安全地使用 VNC。

在本指南中，我将逐步向您展示如何在 Debian 12 服务器上安装 VNC 服务器。

### 先决条件

您需要具备以下条件：

- Debian 12 服务器。
- 具有管理员权限的非 root 用户。

### 安装桌面环境

要开始本教程，您将把桌面环境安装到 Debian 服务器上。这可以通过 APT 或使用任务选择等辅助工具手动完成。 Tasksel 是一个命令行工具，可帮助您通过交互式 shell 安装一组软件包，例如桌面环境和 Web 服务器。

首先，在使用以下命令安装软件包之前更新您的 Debian 存储库。

    sudo apt update

现在运行以下命令来安装taskel。

    sudo apt install tasksel

输入 y 确认安装。

安装tasksel后，执行下面的tasksel命令为您的VNC服务器安装桌面环境。

    sudo tasksel

选择您喜欢的桌面环境，移至“确定”进行确认，然后按 ENTER 开始安装。以下示例将使用 lxqt 作为 VNC 服务器的默认桌面。

### 安装TigerVNC服务器

安装桌面环境后，您就可以安装 VNC 服务器软件包了。在 Debian 上，您可以使用 TigerVNC 创建 VNC 服务器。

执行以下命令将tigervnc-standalone-server软件包安装到您的Debian系统。

    sudo apt install tigervnc-standalone-server

输入 y 并按 ENTER 继续安装。

TigerVNC 服务器安装完成后，查看一些重要的 TigerVNC 配置：

- 目录/etc/tigervnc/：TigerVNC Server的主要配置目录。在此目录中，您应该看到用于存储用户的文件vncserver.users，以及作为将自动加载的主要TigerVNC配置的vncserver-config-mandatory文件。
- ~/.vnc/config：TigerVNC Server 提供了一个 systemd 服务文件，可让您轻松运行 VNC Server 桌面。

### 初始化VNC服务器

现在您已经安装了 TigerVNC，是时候使用 TigerVNC 创建您的第一个 VNC 服务器了。在开始之前，请确保您已准备好非 root 用户。

执行以下命令以登录您的用户。

        su - username

通过执行以下命令初始化 VNC 服务器。通过此操作，您将设置 VNC 服务器密码和仅查看密码（可选）。

        ```bash
        l@lyj:/home$ vncserver 
        perl: warning: Setting locale failed.
        perl: warning: Please check that your locale settings:
	        LANGUAGE = (unset),
        	LC_ALL = (unset),
	        LC_ADDRESS = "zh_CN.UTF-8",
        	LC_NAME = "zh_CN.UTF-8",
        	LC_MONETARY = "zh_CN.UTF-8",
        	LC_PAPER = "zh_CN.UTF-8",
        	LC_IDENTIFICATION = "zh_CN.UTF-8",
        	LC_TELEPHONE = "zh_CN.UTF-8",
        	LC_MEASUREMENT = "zh_CN.UTF-8",
        	LC_TIME = "zh_CN.UTF-8",
        	LC_NUMERIC = "zh_CN.UTF-8",
        	LANG = "en_US.UTF-8"
            are supported and installed on your system.
        perl: warning: Falling back to a fallback locale ("en_US.UTF-8").

        You will require a password to access your desktops.

        Password:
        Verify:
        Would you like to enter a view-only password (y/n)? n
        A view-only password is not used
        /usr/bin/xauth:  file /home/l/.Xauthority does not exist

        New Xtigervnc server 'lyj:1 (l)' on port 5901 for display :1.
        Use xtigervncviewer -SecurityTypes VncAuth -passwd /tmp/tigervnc.SY2VdM/passwd :1 to connect to the VNC server.
        ```

输入 VNC 服务器的新密码，并在询问时重复。然后，键入 n 以禁用仅查看密码，或键入 y 以启用仅查看密码。

现在您已经初始化了 VNC 服务器，VNC 服务器应该在“主机名:x”上运行。主机名是系统主机名，x 是桌面编号。在此示例中，VNC 服务器在 lyj:1 上运行。

### 配置VNC服务器和桌面环境

至此，您已经配置了 VNC 服务器及其密码。接下来，您将配置 VNC 服务器并设置默认桌面环境。

在配置 VNC Server 之前，请通过执行以下命令停止当前的 VNC Server 进程。在以下示例中，我们将停止 VNC 服务器 lyj:1。

        vncserver -kill lyj:1

现在，运行以下命令来检查系统上可用的桌面环境。

        ls /usr/share/xsessions/

在以下输出中，lxqt.desktop 确认 lxqt 可用。

接下来，使用以下 vim 编辑器命令创建新的 VNC 服务器配置 ~/.vnc/config。这是每用户配置，这意味着每个用户可以有不同的配置。

        vim ~/.vnc/config

将以下配置输入到文件中。

        session=lxqt
        geometry=1200x720
        localhost
        alwaysshared

完成后保存并关闭文件。

在此示例中，您将使用以下内容配置 VNC 服务器：

- session=lxqt：将默认会话配置为 lxqt
- Geometry=1200x720：将显示配置为 1200x720
- localhost：仅在本地主机中运行 VNC 服务器
- alwaysshared：始终将传入连接视为共享连接

### 为 TigerVNC 服务器添加用户

现在您已经配置了默认桌面环境和 VNC 服务器，下一个任务是将您的用户添加到 VNC 服务器。

运行以下 vim 编辑器命令以打开 TigerVNC 的文件 /etc/tigervnc/vncserver.users。

        sudo vim /etc/tigervnc/vncserver.users

将以下行添加到文件中。使用此命令，您可以将 VNC 服务器 :1 配置为用户用户名。此列表可以继续存在，具体取决于您将创建的可用 VNC 用户和桌面。

        :1=username

完成后保存并关闭文件。
接下来，运行下面的 systemctl 命令来启动并启用 Tigervncserver@:1.service。服务文件tigervncserver@:1.service意味着您将启动VNC Server桌面:1。

        sudo systemctl start tigervncserver@:1.service
        sudo systemctl enable tigervncserver@:1.service

最后，通过执行以下命令确保 tigervncserver@:1.service 正在运行。

        sudo systemctl status tigervncserver@:1.service

至此，您已经使用 TigerVNC 和 XFCE 作为默认桌面环境完成了 VNC 服务器安装。接下来，您将通过 SSH 隧道安全地连接到 VNC 服务器。

### 通过 SSH 隧道连接到 VNC 服务器

在连接到 VNC 服务器之前，请确保本地计算机上安装了 SSH 客户端和 VNC 查看器。

对于 Windows 用户：您可以使用安装了 SSH 客户端的 PowerShell 和用于 VNC 客户端的 UltraVNC。
对于 Linux 用户：使用终端和 Remmina 远程桌面应用程序。

在本地计算机上打开终端并执行下面的 ssh 命令以创建到 VNC 服务器的 SSH 隧道。当询问时输入您的密码。在此示例中，我们将使用端口 5901 和用户 bob 创建到 VNC 服务器 192.168.5.15 的 SSH 隧道。

        ssh -L 5901:127.0.0.1:5901 -N -f -l bob 192.168.5.15

现在打开 VNC 查看器应用程序并通过端口 5901 连接到本地主机或 127.0.0.1。
