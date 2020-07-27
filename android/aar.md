# Android 归档文件（AAR）
作者：康林(kl222@126.com)

--------------------------

## Android中常见的第三方库包括：
- *.so: C或C++语言的内容打包成的库
- *.jar: JAR（Java Archive，Java 归档文件）是与平台无关的文件格式，它允许将许多文件组合成一个压缩文件。 只包含了class文件与清单文件 ，不包含资源文件，如图片等所有res中的文件。
- *.aar: Android库项目的二进制归档文件，包含所有资源，class以及res资源文件全部包含。

jar 与 aar 区别：jar不包含资源文件，如图片等所有res中的文件。

## 建立 AAR
Android 库（AAR）在结构上与 Android 应用模块（APK）相同。它可以提供构建应用所需的一切内容，包括源代码、资源文件和 Android 清单。不过，Android 库将编译到您可以用作 Android 应用模块依赖项的 Android 归档 (AAR) 文件，而不是在设备上运行的 APK。与 JAR 文件不同，AAR 文件可以包含 Android 资源和一个清单文件，这样，除了 Java 类与方法外，您还可以捆绑布局和可绘制对象等共享资源。

- 直接改 build.gradle 文件，加入下列代码：

        apply plugin: 'com.android.library'

- [使用 android studio](https://www.jianshu.com/p/1777a634db5e)

## 使用
### 以源码方式使用
- 在 Android 应用源码树中加入 AAR 模块
- 导入 AAR 模块

        $ vim settings.gradle
        include ':QtAndroidUtilsModule'

- 修改 build.gradle 加入下列代码：

        $ vim build.gradle
        dependencies {
            implementation project(':QtAndroidUtils/android/QtAndroidUtilsModule')
        }

### 以库方式使用
- 将生成的aar文件拷贝到libs文件夹下
- 在build.gradle文件的Android闭包下添加

        repositories {
            flatDir {
                dirs 'libs'
            }
        }

- 在dependencies中添加

        implementation (name:'QtAndroidUtilsModule-release',ext:'aar')

name为需要引用的aar文件的文件名

## 参见：
[第一篇：Android Studio 打包及引用 AAR（可能是史上最详细的 ）](https://www.jianshu.com/p/1777a634db5e)
[Android中aar文件制作与使用](https://www.jianshu.com/p/e2b574975be0)
[Android中aar与jar的区别](https://www.jianshu.com/p/0a2572a63ed5)

