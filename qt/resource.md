## qt 资源

作 者：康 林(kl222@126.com)

--------------------------------------

qt 资源包括图片资源、翻译资源等。

### 添加资源文件

在项目视图中，在要添加资源的目录上右键。点 "Add New ..."，选择 Qt 资源文件。
**注意**:资源文件名建议使用 ${PROJECT_NAME}_RESOURCE.qrc ; 如下图:


![](image/NewResourceFile.png)


### 添加资源

在刚才建立的资源文件上双击，或者使用资源编辑器打开。向里面添加资源。资源文件实际上是一个文件文件，你也可以直接编辑。

![](image/EditResource.png)

### 使用
#### 加入项目

- pro: 把资源文件加到变量 RESOURCES 中
- cmake: 把资源文件直接加入到 add_library 或 add_executable 中

#### 初始化资源

在程序开始的时候，需要用 Q_INIT_RESOURCE 初始化资源，在程序结束时，用 Q_CLEANUP_RESOURCE 清理资源。如果是程序，Qt 会自动调用 Q_INIT_RESOURCE 初始化资源。但如果是库，则需要手工调用 Q_INIT_RESOURCE 初始化资源。

#### 访问资源

在代码中，资源可以用 ":/" 象普通文件一样访问。

#### 翻译资源

- pro

[Translations.pri](https://github.com/KangLin/RabbitCommon/blob/master/pri/Translations.pri)


使用：
把此文件包含进入工程文件中(.pro)

        TRANSLATIONS_DIR =
        TRANSLATIONS_NAME = 
        include(../pri/Translations.pri) 

在代码中加载翻译资源

        Q_INIT_RESOURCE(${TRANSLATIONS_NAME}); //初始化资源
        QTranslator translator;
        translator.load(CRabbitCommonGlobalDir::Instance()->GetDirTranslations()
                   + "/" + qApp->applicationName() + "_" + QLocale::system().name() + ".qm");
        qApp->installTranslator(&translator);

debug 翻译资源做为资源文件嵌入程序

其它系统发行模式下，做为文件放在程序的安装目录 Translations 目录下  
程序的安装目录：


        AppRoot 
          |- bin
          |    |- App.exe
          |- lib
          |      
          |- translations
              |- ${TRANSLATIONS_NAME}_zh_CN.qm
              |- ${TRANSLATIONS_NAME}_zh_TW.qm


Android 翻译安装目录:


        assets
          |- translations
              |- ${TRANSLATIONS_NAME}_zh_CN.qm
              |- ${TRANSLATIONS_NAME}_zh_TW.qm


源码目录：


        SourceRoot 
          |- App
          |   |- Resource
          |         |-Translations
          |               |- ${TRANSLATIONS_NAME}_zh_CN.ts
          |               |- ${TRANSLATIONS_NAME}_zh_TW.ts
          |- pri
          |      |- Translations.pri
          |      |- lrelease.pri
          |- Src
              |- Resource
                   |-Translations
                         |- ${TRANSLATIONS_NAME}_zh_CN.ts
                         |- ${TRANSLATIONS_NAME}_zh_TW.ts

- cmake

[Translations.cmake](https://github.com/KangLin/RabbitCommon/blob/master/cmake/Translations.cmake)

1. 在 CMakeLists.txt加入下列：  
1.1. 设置 TRANSLATIONS_NAME 为目标名  
1.2. 包含本文件  
1.3. 把 TRANSLATIONS_RESOURCE_FILES 加入到 add_executable 或 add_library 中  
    例如： 在 add_executable(${PROJECT_NAME} ${TRANSLATIONS_RESOURCE_FILES})  
1.4. 增加目标依赖（可选，默认会自动执行）： 

      `add_dependencies(${TRANSLATIONS_NAME} translations_${TRANSLATIONS_NAME})`

      例子：

        set(TRANSLATIONS_NAME ${PROJECT_NAME})
        include(${CMAKE_SOURCE_DIR}/cmake/Translations.cmake)
        add_executable(${PROJECT_NAME} ${TRANSLATIONS_RESOURCE_FILES})
        add_dependencies(${TRANSLATIONS_NAME} translations_${TRANSLATIONS_NAME})


2. 在源码 main 函数中加入, 如果是 DEBUG，需要加入宏定义 _DEBUG . 必须使用 -DCMAKE_BUILD_TYPE=Debug

        Q_INIT_RESOURCE(${TRANSLATIONS_NAME}); //初始化资源
        QTranslator translator;
        translator.load(RabbitCommon::CDir::Instance()->GetDirTranslations()
                   + "/" + qApp->applicationName() + "_" + QLocale::system().name() + ".qm");
        qApp->installTranslator(&translator);


debug 翻译资源做为资源文件嵌入程序

android 翻译资源放在 assets 中


        Android:
          assets                                      GetDirApplicationInstallRoot()  (Only read)
            |- translations                           GetDirTranslations()
            |        |- ${TRANSLATIONS_NAME}_zh_CN.ts
            |        |- ${TRANSLATIONS_NAME}_zh_TW.ts
 
其它系统发行模式下，做为文件放在程序的安装目录 Translations 目录下
程序的安装目录：

        AppRoot |
                |- bin
                |   |- App.exe
                |- lib
                |
                |- translations
                      |- ${TRANSLATIONS_NAME}_zh_CN.qm
                      |- ${TRANSLATIONS_NAME}_zh_TW.qm

 源码目录：

        SourceRoot |
                   |- App
                   |   |- Resource
                   |        |-Translations
                   |             |- ${TRANSLATIONS_NAME}_zh_CN.ts
                   |             |- ${TRANSLATIONS_NAME}_zh_TW.ts
                   |- cmake
                   |   |- Translations.cmake
                   |- Src
                       |- Resource
                           |-Translations
                                 |- ${TRANSLATIONS_NAME}_zh_CN.ts
                                 |- ${TRANSLATIONS_NAME}_zh_TW.ts


