# Android 调试
作者：康林（kl222@126.com)

### adb 使用
#### 开启无线调试
1. 连接USB数据线，打开usb调试，使用windows的“运行”命令行方式：（此方法需配置adb环境变量，也可直接进入adb工具目录执行 \android-sdk\platform-tools\）

    - 启动 android 设备上的 TCPIP 调试服务

            adb tcpip 5555  #端口号

    - 连接 Android 设备

            adb connect 192.168.1.199  #Android设备IP地址

    - 切回 USB 调试

            adb usb  #使用回usb调试

2. 无需数据线，且可解决部分机器在方法1时出现的“unable to connect to 192.168.1.199:5555”错误
在android设备上安装 “终端模拟器”等类似shell命令工具，使用下面命令（需root权限）：

    - TCP/IP方式：

            setprop service.adb.tcp.port 5555
            stop adbd
            start adbd
     
    - usb方式：

            setprop service.adb.tcp.port -1
            stop adbd
            start adbd