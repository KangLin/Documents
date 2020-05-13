# Qt

作者：康林(kl222@126.com)
-------------------------

- [获得 qt 二进制执行程序](#获得-qt-二进制执行程序)
- [qt install](#qt-install)
- [qt 插件](#qt-插件)
- [开源qt第三方库](qt_third.md)

### 获得 qt 二进制执行程序
- 系统自带的
  + ubuntu
    - 基本的

        sudo apt-get install qttools5-dev qttools5-dev-tools \
             qtbase5-dev qtbase5-dev-tools

    - 模块
      + libqt5serialport5-dev 
      + qtmultimedia5-dev
      + qtwebengine5-dev
      + libqt5webkit5-dev
      + qtdeclarative5-dev
      + qtlocation5-dev
      + qtpositioning5-dev
      + qtpim5-dev
      + qtquickcontrols2-5-dev
      + qtscript5-dev
      + libqt5texttospeech5-dev
      + libqt5webchannel5-dev
  + [launchpad](https://launchpad.net/~beineri)
- [qt官网](http://download.qt.io/official_releases/qt/)

### qt install
- 应用默认安装前缀。
  + qnx 安装到 /tmp 
  + android 安装到 / 
  + 其它安装到编译目录下的 install 目录下
- 启动 desktop 文件。安装到 /usr/share/applications/ 目录下
- 启动图标。安装到 /usr/share/pixmaps 目录下
- 例如：
  + qmake

        isEmpty(PREFIX) : !isEmpty(INSTALL_ROOT) : PREFIX=$$INSTALL_ROOT
        isEmpty(PREFIX) {
            qnx : PREFIX = /tmp
            else : android : PREFIX = /.
            else : PREFIX = $$OUT_PWD/install
        }
        
        !android : !macx : unix {
            # install icons
            icon128.target = icon128
            icon128.files = App/Resource/png/SerialPortAssistant.png
            icon128.path = $${PREFIX}/share/pixmaps
            icon128.CONFIG = directory no_check_exist
        
            # install desktop file
            DESKTOP_FILE.target = DESKTOP_FILE
            DESKTOP_FILE.files = $$PWD/debian/SerialPortAssistant.desktop
            DESKTOP_FILE.path = $$system_path($${PREFIX})/share/applications
            DESKTOP_FILE.CONFIG += directory no_check_exist
            INSTALLS += DESKTOP_FILE icon128
        }

### qt 在　android　启动时白屏

	#if (QT_VERSION > QT_VERSION_CHECK(5,6,0))
	    QApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
	#endif

	android启动时透明

	#if defined(Q_OS_ANDROID) && QT_VERSION >= QT_VERSION_CHECK(5, 7, 0)
	    QtAndroid::hideSplashScreen();
	#endif

### qt 插件

qt 插件分为高层插件与低层插件。

- 高层插件是由Qt已经定义好接口的插件。用户只需要实现接口就可以了。
- 低层插件是由用户自己定义接口并实现。

#### qt低层插件
- 插件扩展应用程序
  1. 定义接口
  2. 用宏
Q_DECLARE_INTERFACE() 向qt元对象系统声明接口
  3. 在应用程序中用 
QPluginLoader 加载插件
  4. 用 qobject_cast() 测试插件是否是指定接口的实现。并返回接口实例

- 实现一个插件
  1. 从 QObject 和接口继承声明一个插件类
  2. 用 
 Q_INTERFACES() 向qt元对象系统声明接口
  3. 用 
 Q_PLUGIN_METADATA() 宏导出插件
  4. 用 pro 文件或 CMake 建立子项目
