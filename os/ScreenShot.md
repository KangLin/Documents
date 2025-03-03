# 截屏
作者： 康林 <kl222@126.com>

## Windows

 Windows中接入多个显示器时，可设置为复制和扩展屏。
 1. 设置为复制屏幕时，多个显示器的分辨率是一样的，位置为0~分辨率值
 2. 设置为扩展屏幕时，显示器之间的关系比较复杂些。首先Windows系统会识别一个主显示器，
 
 这个可以在屏幕分辨率中更改。多个显示器之间的位置关系也可以再屏幕分辨率中更改。
 其中主显示器的位置为(0,0)到(width,height)，其他显示器位置由与主显示器的位置关系决定，
 在主显示器左上，则为负数，用0减去长宽；在右下，则由主显示器的分辨率加上长宽。
 其中驱动或用mouse_event处理时也是一样，主显示器为0~65535，
 其他显示器根据主显示器的相对位置确定。
 
    -----------------------------------------------------------------------------
    |                                       Virtural Screen                     |
    |   ------------                                                            |
    |   |          |                                                            |
    |   |  Screen1 |     (0,0)                                                  |
    |   |          |    /                                                       |
    |   ------------   /                                                        |
    |   ------------  ------------                                              |
    |   |          |  |          |                                              |
    |   |  Screen2 |  |  Primary |                                              |
    |   |          |  |  Screen3 |                                              |
    |   ------------  ------------                                              |
    |                                                                           |
    |                                                                           |
    |                                                                           |
    |                                                                           |
    |                                                                           |
    |                                                                           |
    -----------------------------------------------------------------------------
 
### 截屏

- 得到屏幕 DC， 以下几种方法一样：

      // The following methods can all get the desktop
      //m_DC = CreateDCA("DISPLAY",NULL,NULL,NULL ); // Primary screen
      //m_DC = GetWindowDC(NULL);
      m_DC = GetDC(GetDesktopWindow()); // Multi-screen;


- 得到屏幕的大小
  + 物理屏幕
  
        GetDeviceCaps(m_DC, HORZRES); // Primary screen pixel
        GetDeviceCaps(m_DC, VERTRES); // Primary screen pixel
    
  + 虚拟屏幕
  
        GetSystemMetrics(SM_CXVIRTUALSCREEN);
        GetSystemMetrics(SM_CYVIRTUALSCREEN);
    
- 代码：

https://github.com/KangLin/RabbitRemoteControl/blob/master/Service/DesktopWindows.h  
https://github.com/KangLin/RabbitRemoteControl/blob/master/Service/DesktopWindows.cpp

## linux - X 窗口(WINDOW)
### X 窗口（WINDOW）系统介绍

X 窗口(WINDOW)系统支持一个或多个包含重叠窗口或子窗口的屏幕。屏幕(Screen)是物理监视器和硬件，可以是彩色、灰度或单色。每个显示器(Display)或工作站可以有多个屏幕。单个 X 服务器可以为任意数量的屏幕提供显示服务。为单个用户提供一个键盘和一个指针（通常是鼠标）的一组屏幕称为显示器(Display)。

X 服务器中的所有窗口都按严格的层次结构排列。每个层次结构的顶部是一个根窗口，它覆盖了每个显示屏幕。每个根窗口都被子窗口部分或完全覆盖。除根窗口外，所有窗口都有父窗口。每个应用程序通常至少有一个窗口。子窗口又可能有自己的子窗口。这样，应用程序可以在每个屏幕上创建任意深度的树。 X 为窗口提供图形、文本和光栅操作。

子窗口可以大于其父窗口。也就是说，子窗口的一部分或全部可以扩展到父窗口的边界之外，但所有输出到窗口的内容都被其父窗口剪掉了。如果一个窗口的多个子窗口具有重叠位置，则其中一个子窗口被认为在其他窗口之上或高于其他子窗口，从而遮挡了它们。除非窗口有后备存储，否则输出到其他窗口覆盖的区域会被窗口系统抑制。如果一个窗口被第二个窗口遮挡，则第二个窗口仅遮挡第二个窗口的那些也是第一个窗口的祖先的祖先。

窗口(Window)的边框(border)宽度为 0 个或多个像素，可以是您喜欢的任何图案（像素图）或纯色。一个窗口通常但不总是有一个背景图案，当它被揭开时，它会被窗口系统重新绘制。子窗口遮挡了父窗口，父窗口中的图形操作通常被子窗口裁剪。

每个窗口和像素图都有自己的坐标系。坐标系具有水平 X 轴和垂直 Y 轴，原点 [0, 0] 位于左上角。就像素而言，坐标是整数，并且与像素中心重合。对于窗口，原点位于左上角内侧的边框内。

X 不保证保留 windows 的内容。当部分或全部窗口被隐藏，然后又回到屏幕上时，其内容可能会丢失。然后服务器向客户端程序发送一个 Expose 事件，通知它需要重新绘制部分或全部窗口。程序必须准备好按需重新生成窗口的内容。

X 还提供图形对象的屏幕外存储，称为像素图。单平面（深度 1）像素图有时称为位图。像素图可以在大多数图形功能中与窗口互换使用，并在各种图形操作中用于定义图案或平铺。 Windows 和像素图一起称为可绘制对象。

Xlib 中的大多数函数只是将请求添加到输出缓冲区。这些请求稍后在 X 服务器上异步执行。返回存储在服务器中的信息值的函数在收到显式回复或发生错误之前不会返回（即它们阻塞）。您可以提供一个错误处理程序，在报告错误时将调用该处理程序。

如果客户端不希望异步执行请求，它可以跟随请求调用 XSync，它会阻塞直到所有先前缓冲的异步事件都已发送并执行。作为一个重要的副作用，Xlib 中的输出缓冲区总是通过调用任何从服务器返回值或等待输入的函数来刷新。

许多 Xlib 函数将返回一个整数资源 ID，它允许您引用存储在 X 服务器上的对象。这些可以是 Window、Font、Pixmap、Colormap、Cursor 和 GContext 类型，如文件 <X11/X.h> 中所定义。这些资源由请求创建，并在请求或连接关闭时销毁（或释放）。大多数这些资源可能在应用程序之间共享，实际上，窗口是由窗口管理器程序显式操作的。字体和光标在多个屏幕之间自动共享。字体根据需要加载和卸载，并由多个客户端共享。字体通常缓存在服务器中。 Xlib 不支持在应用程序之间共享图形上下文。

客户端程序被通知事件。事件可能是请求的副作用（例如，重新堆叠窗口生成 Expose 事件）或完全异步（例如，来自键盘）。客户端程序要求获知事件。因为其他应用程序可以向您的应用程序发送事件，所以程序必须准备好处理（或忽略）所有类型的事件。

输入事件（例如，按下的键或移动的指针）从服务器异步到达并排队，直到它们被显式调用（例如，XNextEvent 或 XWindowEvent）请求。 此外，一些库函数（例如 XRaiseWindow）会生成 Expose 和 ConfigureRequest 事件。 这些事件也是异步到达的，但是客户端可能希望在调用一个可以导致服务器生成事件的函数之后通过调用 XSync 来显式地等待它们。 

### 截屏

- 打开设备和初始化 Image

      int CDisplayXLib::Open()
      {
          int nRet = 0;
          /* Connect to a local display.
           * it defaults to the value of the DISPLAY environment variable. */
          m_pDisplay = XOpenDisplay(NULL);
          do{
              if(!m_pDisplay)
              {
                  nRet = -1;
              }

              if (strstr(ServerVendor(m_pDisplay), "X.Org")) {
                  int vendrel = VendorRelease(m_pDisplay);

                  QString version("X.Org version: ");
                  version += QString::number(vendrel / 10000000);
                  version += "." + QString::number((vendrel /   100000) % 100),
                          version += "." + QString::number((vendrel /     1000) % 100);
                  if (vendrel % 1000) {
                      version += "." + QString::number(vendrel % 1000);
                  }
                  LOG_MODEL_DEBUG("CDisplayXLib", version.toStdString().c_str());
              }

              m_RootWindow = DefaultRootWindow(m_pDisplay); // RootWindow(dsp,0);/* Refer to the root window */
              if(0 == m_RootWindow)
              {
                  LOG_MODEL_ERROR("CDisplayXLib", "cannot get root window");
                  nRet = -2;
                  break;
              }
          }while(0);

          if(nRet)
          {   
              Close();
              return nRet;
          }

          // Initial QImage and XImage
          Visual* vis = DefaultVisual(m_pDisplay, DefaultScreen(m_pDisplay));
          if (vis && vis->c_class == TrueColor) {
              m_pImage = XCreateImage(m_pDisplay, vis,
                                  DefaultDepth(m_pDisplay, DefaultScreen(m_pDisplay)),
                                  ZPixmap, 0, 0, Width(), Height(),
                                  BitmapPad(m_pDisplay), 0);
              if(m_pImage)
              {
                  m_pImage->data = (char *)malloc(m_pImage->bytes_per_line * m_pImage->height);
                  if (m_pImage->data) 
                  {
                      m_Format = GetFormat(m_pImage);
                      if(QImage::Format_Invalid != GetFormat())
                          m_Desktop = QImage((uchar*)m_pImage->data,
                                           Width(), Height(), GetFormat());
                  }

                  if(!m_pImage->data || QImage::Format_Invalid == GetFormat())
                  {
                      DestroyImage(m_pImage);
                      m_pImage = NULL;
                  }
              }
          }

          return nRet;
      }

- 截屏

      QImage CDisplayXLib::GetDisplay(int x, int y, int width, int height)
      {
          if(!m_pDisplay || 0 == m_RootWindow)
              return QImage();

          if(-1 > width)
              width = Width();
          if(-1 > height)
              height = Height();
          XImage *pImage = XGetImage(m_pDisplay, m_RootWindow, x, y, width, height,
                                     AllPlanes, ZPixmap);
          if(QImage::Format_Invalid == GetFormat())
              m_Format = GetFormat(pImage);
          if(QImage::Format_Invalid != GetFormat())
          {
              QImage img((uchar*)pImage->data,
                         pImage->width, pImage->height,
                         GetFormat(),
                         DestroyImage, pImage);
              return img;
          }
          else
              DestroyImage(pImage);

          return QImage();
      }

- 代码：

https://github.com/KangLin/RabbitRemoteControl/blob/master/Service/DisplayXLib.h  
https://github.com/KangLin/RabbitRemoteControl/blob/master/Service/DisplayXLib.cpp

### 参考

- [xlib documents](https://www.x.org/releases/current/doc/libX11/libX11/libX11.html)

