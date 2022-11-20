# Log4Qt 使用

作者：康林 <kl222@126.com>

文档位置：[https://github.com/KangLin/Documents/blob/master/qt/log4qt.md](https://github.com/KangLin/Documents/blob/master/qt/log4qt.md)

## 介绍

Log4Qt 是 Log4j 的 Qt 版本。

Log4Qt 有三个主要的组件：

- Logger(记录器):日志类别(category)和级别(Level)。
- Appender (输出源):日志要输出的地方。
- Layout(布局):日志以何种形式输出。

### Logger

Logger 在头文件 logger.h 中声明。

分为根 Logger 和非根 Logger(类别)两类。

非根 Logger 用于不同类别(category)的日志。

类别(category)名字格式：

| 格式|分隔符|例子    |
|:---:|:----:|:------:|
|JAVA |  .   |A.B.c   |
|C++  |  ::  |A::B::C |

Logger 中被分为五个级别：DEBUG、INFO、WARN、ERROR和FATAL。这五个级别是有顺序的，DEBUG < INFO < WARN < ERROR < FATAL，分别用来指定这条日志信息的重要程度。Log4Qt 有一个规则：只输出级别不低于设定级别的日志信息，假设 Loggers 级别设定为INFO，则INFO、WARN、ERROR和FATAL级别的日志信息都会输出，而级别比INFO低的DEBUG则不会输出。

#### 使用

##### 得到 Logger

- 得到根 Logger

        Log4Qt::Logger* logger = Log4Qt::Logger::rootLogger();

- 得到非根 Logger。非根 Logger 用于不同类别的日志。

        Log4Qt::Logger* logger = Log4Qt::Logger::logger(category);

  + 类别(category)名字可以使用以下两种格式之一：
  
    |格式 |分隔符|例子    |
    |:---:|:----:|:------:|
    |JAVA |  .   |A.B.c   |
    |C++  |  ::  |A::B::C |

  + 使用 Log4Qt::Logger* logger = Log4Qt::Logger::logger(category); 得 Logger* 。
  它在内部建立与父的关系。例如：
  
        Log4Qt::Logger* qrlogger = Log4Qt::Logger::logger("Qt.RabbitCommon.Logger");
        
     如果 "RabbitCommon" 不存在，则建立一个 "RabbitCommon" Logger, 并与 qrlogger 建立父关系。如果 "Qt" Logger 不存在，则建立一个 "Qt" Logger，并与 "RabbitCommon" Logger 建立父关系。

- 使用宏 LOG4QT_DECLARE_STATIC_LOGGER
  + 得到根Logger
  
        LOG4QT_DECLARE_STATIC_LOGGER(logger,)

  + 得到非根Logger

        LOG4QT_DECLARE_STATIC_LOGGER(logger, "Qt.RabbitCommon.Logger")

##### 输出日志

- C++方式（流式）：

```C++
Log4Qt::Logger* logger = Log4Qt::Logger::rootLogger();
logger->debug() << "Hello rootLogger";
```

- C 语言方式：

```C++
Log4Qt::Logger* logger = Log4Qt::Logger::logger("Qt.RabbitCommon.Logger");
logger->debug("%s", "Hello Qt.RabbitCommon.Logger");
Log4Qt::Logger* main = Log4Qt::Logger::logger("main");
main->debug("Hello main");
```

##### 使用宏

- 普通

```C++
LOG4QT_DECLARE_STATIC_LOGGER(logger, "Qt.RabbitCommon.Logger")
logger()->debug() << "Hello Qt.RabbitCommon.Logger";
```

- 在 Qt 类中使用，必须是继承 QObject 的类
  + 在类的头文件中，类开始处定义：  **LOG4QT_DECLARE_QCLASS_LOGGER** 。
  + 使实现文件中使用用宏 **l4qFatal**、**l4qError**、**l4qWarn**、**l4qInfo**、**l4qDebug**、**l4qTrace** 输出日志
  + 例子：
    - 在类的头文件使用 **LOG4QT_DECLARE_QCLASS_LOGGER** 定义 Logger*
 
        ```C++
        //file: counter.h
        
        #include qobject.h
        #include logger.h
        
        class Counter : public QObject
        {
            Q_OBJECT
            LOG4QT_DECLARE_QCLASS_LOGGER
        public:
            Counter();
            Counter(int preset);
        private:
            int mCount;
        }
        ```
        
   + 在实现文件使用宏 **l4qFatal**、**l4qError**、**l4qWarn**、**l4qInfo**、**l4qDebug**、**l4qTrace**  输出日志
   
        ```C++
        //file: counter.cpp
        
        #include counter.h
        
        Counter::Counter() : mCount(0)
        {}
        
        void Counter::Counter(int preset) :
           mCount(preset)
        {
            if (preset < 0)
            {
                // C 语言方式
                l4qWarn("Invalid negative counter preset %1. Using 0 instead.", preset);
                // 或者使用流式
                l4qWarn() << "Invalid negative counter preset" << preset << ". Using 0 instead."
                mCount = 0;
            }
        }
        ```
        
### Appender

适配器（Appender) 允许把日志输出到不同的地方，如控制台（Console）、文件（Files）等，可以根据天数或者文件大小产生新的文件，可以以流的形式发送到其它地方等等。
#### 常使用的类如下：

|      类名        |        配置名                    |说明   |
|------:|---------------:|:--------------------------------|
|ConsoleAppender  | org.apache.log4j.ConsoleAppender |控制台             |
|FileAppender     | org.apache.log4j.FileAppender    |文件               |
|DailyFileAppender|org.apache.log4j.DailyFileAppender|每天产生一个日志文件|
|DailyRollingFileAppender| org.apache.log4j.DailyRollingFileAppender|每天产生一个日志文件,当文件大小到达指定尺寸的时候产生一个新的文件|
|RollingFileAppender| org.apache.log4j.RollingFileAppender|文件大小到达指定尺寸的时候产生一个新的文件|
|WriterAppender|org.apache.log4j.WriterAppender|将日志信息以流格式发送到任意指定的地方|

所有的注册适配器请详见：[https://github.com/MEONMedical/Log4Qt/blob/master/src/log4qt/helpers/factory.cpp](https://github.com/MEONMedical/Log4Qt/blob/1dc0b05cf7b621026fa40d58d165e765bd8e0750/src/log4qt/helpers/factory.cpp#L433) 中下面函数：

        void Factory::registerDefaultAppenders()
    
#### Logger 设置 Appender
使用 Logger 成员函数 virtual void addAppender(const AppenderSharedPtr &appender); 设置 Appender

### Layout
实现日志输出格式。可以在Appenders的后面附加Layouts来完成这个功能。
Layouts提供四种日志输出样式，如根据HTML样式、自由指定样式、包含日志级别与信息的样式和包含日志时间、线程、类别等信息的样式。
#### 常使用的类如下：

|      类名        |        配置名                    |说明|
|-----:|---------------:|:--------------------------------|
|PatternLayout|org.apache.log4j.PatternLayout|可以灵活地指定布局模式|
|SimpleLayout| org.apache.log4j.SimpleLayout|包含日志信息的级别和信息字符串|
|TTCCLayout|org.apache.log4j.TTCCLayout|包含日志产生的时间、线程、类别等信息|
|XMLLayout|org.apache.log4j.XMLLayout|XML|

所有注册的Layout，请详见：[https://github.com/MEONMedical/Log4Qt/blob/master/src/log4qt/helpers/factory.cpp](https://github.com/MEONMedical/Log4Qt/blob/1dc0b05cf7b621026fa40d58d165e765bd8e0750/src/log4qt/helpers/factory.cpp#L505) 下列函数：

        void Factory::registerDefaultLayouts()
        
- PatternLayout选项：
  + ConversionPattern=%m%n：设定以怎样的格式显示消息。
    - 格式化符号说明：
      - %p：输出日志信息的优先级，即DEBUG，INFO，WARN，ERROR，FATAL。
      - %d：输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，如：%d{yyyy/MM/dd HH:mm:ss,SSS}。
      - %r：输出自应用程序启动到输出该log信息耗费的毫秒数。
      - %t：输出产生该日志事件的线程名。
      - %l：输出日志事件的发生位置，相当于%c.%M(%F:%L)的组合，包括类全名、方法、文件名以及在代码中的行数。例如：test.TestLog4j.main(TestLog4j.java:10)。
      - %c：输出日志信息所属的类目，通常就是所在类的全名。
      - %M：输出产生日志信息的方法名。
      - %F：输出日志消息产生时所在的文件名称。
      - %L:：输出代码中的行号。
      - %m:：输出代码中指定的具体日志信息。
      - %n：输出一个回车换行符，Windows平台为"\r\n"，Unix平台为"\n"。
      - %x：输出和当前线程相关联的NDC(嵌套诊断环境)，尤其用到像java servlets这样的多客户多线程的应用中。
      - %%：输出一个"%“字符。
    - 另外，还可以在%与格式字符之间加上修饰符来控制其最小长度、最大长度、和文本的对齐方式。如：
      1. c：指定输出category的名称，最小的长度是20，如果category的名称长度小于20的话，默认的情况下右对齐。
      2. %-20c：”-"号表示左对齐。
      3. %.30c：指定输出category的名称，最大的长度是30，如果category的名称长度大于30的话，就会将左边多出的字符截掉，但小于30的话也不会补空格。

#### 设置

使用 Appender 成员函数 virtual void setLayout(const LayoutSharedPtr &layout) 设置 layout

### 例子

```C++
include "log4qt/consoleappender.h"
include "log4qt/logger.h"
include "log4qt/ttcclayout.h"

// Create a layout
Log4Qt::LogManager::rootLogger();
TTCCLayout *p_layout = new TTCCLayout();
p_layout->setName(QLatin1String("My Layout"));
p_layout->activateOptions();
// Create an appender
ConsoleAppender *p_appender = new ConsoleAppender(p_layout, ConsoleAppender::STDOUT_TARGET);
p_appender->setName(QLatin1String("My Appender"));
p_appender->activateOptions();
// Set appender on root logger
Log4Qt::Logger::rootLogger()->setAppender(p_appender);
// Request a logger and output "Hello World!"
Log4Qt::Logger::logger(QLatin1String("My Logger"))->info("Hello World!");
```

参见： https://log4qt.sourceforge.net/

## Qt 日志
Log4Qt 可以输出使用 Qt 的日志输出函数产生的日志。
使用 LogManager::setHandleQtMessages(bool handleQtMessages) 打开或关闭输出 Qt 日志函数产生的日志。

- qDebug() 函数参数为空，类似 Log4Qt 的根 Logger 。

        qDebug() << "Hell world";
      
- **QLoggingCategory** 类似 Log4Qt 的非根 Logger。
使用 **qCWarning()**、**qCDebug()**、**qCWarning()**、**qCInfo()**、**qCCritical()** 输出日志。
也可以使用 **qWarning()**、**qDebug()**、**qWarning()**、**qInfo()**、**qCritical()** 输出日志。

        QLoggingCategory Logger("RabbitCommon.Logger");
        qCCritical(Logger) << "Log folder is empty";
        //或者
        qCritical(Logger) << "Log folder is empty";
        
- QLoggingCategory 配置规则：

日志记录规则允许您以灵活的方式启用或禁用类别的日志记录。规则在文本中指定，其中每行必须具有以下格式：

        <category>[.<type>] = true|false

<category\> 是类别的名称，可能使用 * 作为第一个或最后一个字符的通配符，或在两个位置。可选<type>选项必须是 debug、 info、 warning、 或者 critical。不符合此方案的行将被忽略。
规则按文本顺序（从第一个到最后一个）进行评估。也就是说，如果两个规则应用于类别/类型，则稍后出现的规则将应用。
规则可以通过 setFilterRules() 设置过滤器规则：

        QLoggingCategory::setFilterRules("*.debug=false\n"
                                         "driver.usb.debug=true");

日志记录规则从日志记录配置文件的 [规则] 部分自动加载。这些配置文件在 QtProject 配置目录中查找，或在QT_LOGGING_CONF环境变量中显式设置：

    [Rules]
    *.debug=false
    driver.usb.debug=true

日志记录规则也可以在QT_LOGGING_RULES环境变量中指定;多个规则也可以用分号分隔：

         QT_LOGGING_RULES="*.debug=false;driver.usb.debug=true"

由setFilterRules()设置的规则优先于 QtProject 配置目录中指定的规则。反过来，这些规则可以被QT_LOGGING_CONF指定的配置文件中的规则和QT_LOGGING_RULES设置的规则覆盖。
设置顺序如下：

  1. [QLibraryInfo::DataPath]/qtlogging.ini
  2. QtProject/qtlogging.ini
  3. setFilterRules()
  4. QT_LOGGING_CONF
  5. QT_LOGGING_RULES

QtProject/qtlogging.ini 文件在 QStandardPaths::GenericConfigLocation 返回的所有目录中查找。
设置QT_LOGGING_DEBUG环境变量以找出日志记录规则的加载位置。

## 配置文件

### 用下面方法设置配置文件：

        QString szConfFile = RabbitCommon::CDir::Instance()->GetDirConfig(true)
            + QDir::separator() + qApp->applicationName() + ".conf";
        if(!Log4Qt::PropertyConfigurator::configureAndWatch(szConfFile))
            Log4Qt::BasicConfigurator::configure();

### 配置文件说明

#### 变量

格式：

        变量名=值

使用变量：

        ${变量名}

例子：

        #设置储存log文件的根目录
        logpath=log

        log4j.appender.daily.file=${logpath}/root.log

#### 配置 Log4Qt

        log4j.reset=true
        #Log4Qt 库的日志输出级别
        log4j.Debug=WARN
        log4j.threshold=NULL
        #在运行中，是否监视此文件配置的变化
        log4j.watchThisFile=false
        #设置是否监听QDebug输出的字符串
        log4j.handleQtMessages=true
        # QLoggingCategory 过滤规则
        #log4j.qtLogging.filterRules=
        #log4j.qtLogging.messagePattern=
        
#### 配置根 Logger
其语法

        log4j.rootLogger = [level], appenderName1, appenderName2, ...

其中：
- level 控制日志输出的级别：

|级别 | 说明          |
|:---:|:-------------:|
|OFF  |关闭所有日志输出|
|FATAL|               |
|ERROR|               |
|WARN |               |
|INFO |               |
|DEBUG|               |
|TRACE|               |
|ALL  |所有日志均输出  |

- appenderName 是appender的名字，指定输出到哪儿

#### 设置非根logger
语法：

        log4j.logger.categoryName = [level], appenderName1, appenderName2, ...

其中类别名(categoryName)只能使用JAVA格式，因为C++格式分隔符已在配置文件中做为他用。

|     |分隔符|例子    |
|:---:|:----:|:------:|
|JAVA |  .   |A.B.c   |
|C++  |  ::  |A::B::C |


##### 设置非根 Logger 是否输出到其父 Logger 中
非根 Logger 日志默认是同时输出到log4j.rootLogger所有配置的日志中的，如何能只让它们输出到自己指定的日志中呢？用下面配置：

语法：

        log4j.additivity.categoryName = [true/false]
        
- false，只输出到此适配器。
- true，表示Logger会输出到log4j.rootLogger所有配置的日志中
- 默认为true。

类别名(categoryName)应该与非根类别名(categoryName)相同。其中类别名(categoryName)只能使用JAVA格式，因为C++格式分隔符已在配置文件中做为他用。

|     |分隔符|例子    |
|:---:|:----:|:------:|
|JAVA |  .   |A.B.c   |
|C++  |  ::  |A::B::C |

如果不想输出到log4j.rootLogger所有配置的日志，而只是想输出到log4j.rootLogger某一配置（例如：console),则：

        log4j.logger.main = [level], console
        log4j.additivity.main = false

#### 配置 Appender
语法

        log4j.appender.appenderName = className
        log4j.appender.appenderName.option1 = value1
        ...
        log4j.appender.appenderName.optionN = valueN 

className 有以下几种类型：

|               类               |说明           |
|-------------------------------:|:--------------|
|org.apache.log4j.ConsoleAppender|控制台         |
|org.apache.log4j.FileAppender   |文件           |
|org.apache.log4j.DailyRollingFileAppender|每天产生一个日志文件|
|org.apache.log4j.RollingFileAppender|文件大小到达指定尺寸的时候产生一个新的文件|

- ConsoleAppender选项
  + Threshold=WARN：指定日志信息的最低输出级别，默认为DEBUG。
  + ImmediateFlush=true：表示所有消息都会被立即输出，设为false则不输出，默认值是true。
  + Target=System.err：默认值是System.out。
- FileAppender选项
  + Threshold=WARN：指定日志信息的最低输出级别，默认为DEBUG。
  + ImmediateFlush=true：表示所有消息都会被立即输出，设为false则不输出，默认值是true。
  + Append=false：true表示消息增加到指定文件中，false则将消息覆盖指定的文件内容，默认值是true。
  + File=D:/logs/logging.log4j：指定消息输出到logging.log4j文件中。
- DailyRollingFileAppender选项
  + Threshold=WARN #指定日志信息的最低输出级别，默认为DEBUG。
  + ImmediateFlush=true：表示所有消息都会被立即输出，设为false则不输出，默认值是true。
  + Append=false：true表示消息增加到指定文件中，false则将消息覆盖指定的文件内容，默认值是true。
  + File=D:/logs/logging.log4j：指定当前消息输出到logging.log4j文件中。
  + DatePattern=’.'yyyy-MM：每月滚动一次日志文件，即每月产生一个新的日志文件。当前月的日志文件名为logging.log4j，前一个月的日志文件名为logging.log4j.yyyy-MM。
另外，也可以指定按周、天、时、分等来滚动日志文件，对应的格式如下：
    - '.'yyyy-MM：每月
    - '.'yyyy-ww：每周
    - '.'yyyy-MM-dd：每天
    - '.'yyyy-MM-dd-a：每天两次
    - '.'yyyy-MM-dd-HH：每小时
    - '.'yyyy-MM-dd-HH-mm：每分钟
- RollingFileAppender选项
  + Threshold=WARN：指定日志信息的最低输出级别，默认为DEBUG。
  + ImmediateFlush=true：表示所有消息都会被立即输出，设为false则不输出，默认值是true。
  + Append=false：true表示消息增加到指定文件中，false则将消息覆盖指定的文件内容，默认值是true。
  + File=D:/logs/logging.log4j：指定消息输出到logging.log4j文件中。
  + MaxFileSize=100KB：后缀可以是KB, MB 或者GB。在日志文件到达该大小时，将会自动滚动，即将原来的内容移到logging.log4j.1文件中。
  + MaxBackupIndex=2：指定可以产生的滚动文件的最大数，例如，设为2则可以产生logging.log4j.1，logging.log4j.2两个滚动文件和一个logging.log4j文件。

#### 配置 Layout
语法：

        log4j.appender.appenderName.layout = className
        log4j.appender.appenderName.layout.option1 = value1
        …
        log4j.appender.appenderName.layout.optionN = valueN 

其中：className可以是下列值之一：

|        类                    |   说明                           |
|-----------------------------:|:---------------------------------|
|org.apache.log4j.HTMLLayout   |以HTML表格形式布局                 |
|org.apache.log4j.PatternLayout|可以灵活地指定布局模式             |
|org.apache.log4j.SimpleLayout |包含日志信息的级别和信息字符串       |
|org.apache.log4j.TTCCLayout   |包含日志产生的时间、线程、类别等等信息|

- HTMLLayout选项
  + LocationInfo=true：输出java文件名称和行号，默认值是false。
  + Title=My Logging： 默认值是Log4J Log Messages。
- PatternLayout选项：
  + ConversionPattern=%m%n：设定以怎样的格式显示消息。
    - 格式化符号说明：
      - %p：输出日志信息的优先级，即DEBUG，INFO，WARN，ERROR，FATAL。
      - %d：输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，如：%d{yyyy/MM/dd HH:mm:ss,SSS}。
      - %r：输出自应用程序启动到输出该log信息耗费的毫秒数。
      - %t：输出产生该日志事件的线程名。
      - %l：输出日志事件的发生位置，相当于%c.%M(%F:%L)的组合，包括类全名、方法、文件名以及在代码中的行数。例如：test.TestLog4j.main(TestLog4j.java:10)。
      - %c：输出日志信息所属的类目，通常就是所在类的全名。
      - %M：输出产生日志信息的方法名。
      - %F：输出日志消息产生时所在的文件名称。
      - %L:：输出代码中的行号。
      - %m:：输出代码中指定的具体日志信息。
      - %n：输出一个回车换行符，Windows平台为"\r\n"，Unix平台为"\n"。
      - %x：输出和当前线程相关联的NDC(嵌套诊断环境)，尤其用到像java servlets这样的多客户多线程的应用中。
      - %%：输出一个"%“字符。
    - 另外，还可以在%与格式字符之间加上修饰符来控制其最小长度、最大长度、和文本的对齐方式。如：
      1. c：指定输出category的名称，最小的长度是20，如果category的名称长度小于20的话，默认的情况下右对齐。
      2. %-20c：”-"号表示左对齐。
      3. %.30c：指定输出category的名称，最大的长度是30，如果category的名称长度大于30的话，就会将左边多出的字符截掉，但小于30的话也不会补空格。

### 示例

#### 示例一

参见： https://github.com/KangLin/RabbitCommon/blob/master/Src/etc/log4qt.conf

```
#设置储存log文件的根目录
logpath=log

#格式化符号说明：
# %p：输出日志信息的优先级，即DEBUG，INFO，WARN，ERROR，FATAL。
# %d：输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，如：%d{yyyy/MM/dd HH:mm:ss,SSS}。
# %r：输出自应用程序启动到输出该log信息耗费的毫秒数。
# %t：输出产生该日志事件的线程名。
# %l：输出日志事件的发生位置，相当于%c.%M(%F:%L)的组合，包括类全名、方法、文件名以及在代码中的行数。例如：test.TestLog4j.main(TestLog4j.java:10)。
# %c：输出日志信息所属的类目，通常就是所在类的全名。
# %M：输出产生日志信息的方法名。
# %F：输出日志消息产生时所在的文件名称。
# %L:：输出代码中的行号。
# %m:：输出代码中指定的具体日志信息。
# %n：输出一个回车换行符，Windows平台为"\r\n"，Unix平台为"\n"。
# %x：输出和当前线程相关联的NDC(嵌套诊断环境)，尤其用到像java servlets这样的多客户多线程的应用中。
# %%：输出一个"%“字符。
# 另外，还可以在%与格式字符之间加上修饰符来控制其最小长度、最大长度、和文本的对齐方式。如：
# c：指定输出category的名称，最小的长度是20，如果category的名称长度小于20的话，默认的情况下右对齐。
# 2)%-20c：”-"号表示左对齐。
# 3)%.30c：指定输出category的名称，最大的长度是30，如果category的名称长度大于30的话，就会将左边多出的字符截掉，但小于30的话也不会补空格。
logConversionPattern=%d %F:%L [%t] %5p %c - %m%n

log4j.reset=true
log4j.Debug=WARN
log4j.threshold=NULL
#在运行中，是否监视此文件配置的变化
log4j.watchThisFile=false
#设置是否监听QDebug输出的字符串
log4j.handleQtMessages=true
# QLoggingCategory 过滤规则
#log4j.qtLogging.filterRules=
#log4j.qtLogging.messagePattern=

#根 Logger 输出
# log4j.rootLogger 日志输出类别和级别：只输出不低于该级别的日志信息	DEBUG < INFO < WARN < ERROR < FATAL
#设置Log输出的几种输出源（appender）。
#格式：[ level ] , appenderName1, appenderName2, …
# level ：设定日志记录的最低级别，
#        可设的值有 OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL 或者自定义的级别，Log4j建议只使用中间四个级别。
#        通过在这里设定级别，您可以控制应用程序中相应级别的日志信息的开关，比如在这里设定了INFO级别，则应用程序中所有DEBUG级别的日志信息将不会被打印出来。
#        只输出不低于该级别的日志信息	DEBUG < INFO < WARN < ERROR < FATAL
# appenderName：就是指定日志信息要输出到哪里。可以同时指定多个输出目的地，用逗号隔开。本例是：console, daily
log4j.rootLogger=ALL, console, daily

###############################
# 输出到控制台
###############################

# 配置INFO CONSOLE输出到控制台
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.target=System.out
# 配置CONSOLE设置为自定义布局模式
log4j.appender.console.layout=org.apache.log4j.PatternLayout
# 配置logfile为自定义布局模式
log4j.appender.console.layout.ConversionPattern=${logConversionPattern}

###############################
# 输出到日志文件中
###############################

# 配置 logfile 输出到文件中 每日产生文件
log4j.appender.daily=org.apache.log4j.DailyFileAppender
# 输出文件位置此为项目根目录下的logs文件夹中
log4j.appender.daily.file=${logpath}/root.log
#true表示消息增加到指定文件中，false 则将消息覆盖指定的文件内容，默认值是 false
log4j.appender.daily.appendFile=true
# 日期格式
log4j.appender.daily.datePattern=_yyyy_MM_dd
# 立即输出。表示所有消息都会被立即输出，设为 false 则不立即输出，默认值是 true
log4j.appender.daily.immediateFlush=true
# 输入日志等级
log4j.appender.daily.Threshold = DEBUG
# 设置保留天数
log4j.appender.daily.keepDays=10
# 配置logfile为自定义布局模式
log4j.appender.daily.layout=org.apache.log4j.PatternLayout
log4j.appender.daily.layout.ConversionPattern=${logConversionPattern}


###############################
#模块输出到独立的日志
###############################
# 名字以 log4j.logger. 为前缀，后面是 Logger::Logger 参数名。
# 注意：如果名字中有 C++ 名字分隔符 "::" 则用 JAVA 名字分隔符 "." 替换。
# 级别和输出源（appender）格式与 log4j.rootLogger 相同
log4j.logger.main=ALL, main, console
#false，只输出到此适配器。true，表示Logger会在父Logger的appender里输出，默认为true。
# 名字与 log4j.logger 规则相同
log4j.additivity.main=false

log4j.appender.main=org.apache.log4j.DailyFileAppender
# 输出文件位置此为项目根目录下的logs文件夹中
log4j.appender.main.file=${logpath}/main.log
# 配置logfile为自定义布局模式
log4j.appender.main.layout=org.apache.log4j.PatternLayout
log4j.appender.main.layout.ConversionPattern=${logConversionPattern}

###############################
# QLoggingCategory 单独输出到文件
# Category格式： Qt.CategoryName
###############################
# 名字以 log4j.logger. 为前缀，后面是 Logger::Logger 参数名。
# 注意：如果名字中有 C++ 名字分隔符 "::" 则用 JAVA 名字分隔符 "." 替换。
# 级别和输出源（appender）格式与 log4j.rootLogger 相同
log4j.logger.Qt.RabbitCommon.Logger=ALL, RabbitCommonLogger
#false，只输出到此适配器。true，表示Logger会在父Logger的appender里输出，默认为true。
# 名字与 log4j.logger 规则相同
log4j.additivity.Qt.RabbitCommon.Logger=false

log4j.appender.RabbitCommonLogger=org.apache.log4j.DailyFileAppender
# 输出文件位置此为项目根目录下的logs文件夹中
log4j.appender.RabbitCommonLogger.file=${logpath}/RabbitCommonLogger.log
# 配置logfile为自定义布局模式
log4j.appender.RabbitCommonLogger.layout=org.apache.log4j.PatternLayout
log4j.appender.RabbitCommonLogger.layout.ConversionPattern=${logConversionPattern}
```

#### 示例二

```
# 参见：https://blog.csdn.net/qq_43842093/article/details/122810961
log4j.rootLogger=DEBUG,console,dailyFile,im
log4j.additivity.org.apache=true

# 控制台(console)
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.ImmediateFlush=true
log4j.appender.console.Target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 日志文件(logFile)
log4j.appender.logFile=org.apache.log4j.FileAppender
log4j.appender.logFile.Threshold=DEBUG
log4j.appender.logFile.ImmediateFlush=true
log4j.appender.logFile.Append=true
log4j.appender.logFile.File=D:/logs/log.log4j
log4j.appender.logFile.layout=org.apache.log4j.PatternLayout
log4j.appender.logFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 回滚文件(rollingFile)
log4j.appender.rollingFile=org.apache.log4j.RollingFileAppender
log4j.appender.rollingFile.Threshold=DEBUG
log4j.appender.rollingFile.ImmediateFlush=true
log4j.appender.rollingFile.Append=true
log4j.appender.rollingFile.File=D:/logs/log.log4j
log4j.appender.rollingFile.MaxFileSize=200KB
log4j.appender.rollingFile.MaxBackupIndex=50
log4j.appender.rollingFile.layout=org.apache.log4j.PatternLayout
log4j.appender.rollingFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 定期回滚日志文件(dailyFile)
log4j.appender.dailyFile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.dailyFile.Threshold=DEBUG
log4j.appender.dailyFile.ImmediateFlush=true
log4j.appender.dailyFile.Append=true
log4j.appender.dailyFile.File=D:/logs/log.log4j
log4j.appender.dailyFile.DatePattern='.'yyyy-MM-dd
log4j.appender.dailyFile.layout=org.apache.log4j.PatternLayout
log4j.appender.dailyFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 应用于socket
log4j.appender.socket=org.apache.log4j.RollingFileAppender
log4j.appender.socket.RemoteHost=localhost
log4j.appender.socket.Port=5001
log4j.appender.socket.LocationInfo=true
# Set up for Log Factor 5
log4j.appender.socket.layout=org.apache.log4j.PatternLayout
log4j.appender.socket.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# Log Factor 5 Appender
log4j.appender.LF5_APPENDER=org.apache.log4j.lf5.LF5Appender
log4j.appender.LF5_APPENDER.MaxNumberOfRecords=2000
# 发送日志到指定邮件
log4j.appender.mail=org.apache.log4j.net.SMTPAppender
log4j.appender.mail.Threshold=FATAL
log4j.appender.mail.BufferSize=10
log4j.appender.mail.From = xxx@mail.com
log4j.appender.mail.SMTPHost=mail.com
log4j.appender.mail.Subject=Log4J Message
log4j.appender.mail.To= xxx@mail.com
log4j.appender.mail.layout=org.apache.log4j.PatternLayout
log4j.appender.mail.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 应用于数据库
log4j.appender.database=org.apache.log4j.jdbc.JDBCAppender
log4j.appender.database.URL=jdbc:mysql://localhost:3306/test
log4j.appender.database.driver=com.mysql.jdbc.Driver
log4j.appender.database.user=root
log4j.appender.database.password=
log4j.appender.database.sql=INSERT INTO LOG4J (Message) VALUES('=[%-5p] %d(%r) --> [%t] %l: %m %x %n')
log4j.appender.database.layout=org.apache.log4j.PatternLayout
log4j.appender.database.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n

# 自定义Appender
log4j.appender.im = net.cybercorlin.util.logger.appender.IMAppender
log4j.appender.im.host = mail.cybercorlin.net
log4j.appender.im.username = username
log4j.appender.im.password = password
log4j.appender.im.recipient = corlin@cybercorlin.net
log4j.appender.im.layout=org.apache.log4j.PatternLayout
log4j.appender.im.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n

```

