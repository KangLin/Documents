# Qt

作者：康林(kl222@126.com)
-------------------------

### qt install
- 安装前缀。
  + qnx 安装到 /tmp 
  + android 安装到 / 
  + unix 安装到 /usr
  + 其它安装到编译目录下的 install 目录下
- 启动 desktop 文件。安装到 /usr/share/applications/ 目录下
- 启动图标。安装到 /usr/share/pixmaps 目录下
- 例如：
  + qmake

        isEmpty(PREFIX) : !isEmpty(INSTALL_ROOT) : PREFIX=$$INSTALL_ROOT
        isEmpty(PREFIX) {
            qnx : PREFIX = /tmp
            else : android : PREFIX = /.
            else : unix : PREFIX = /usr
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


