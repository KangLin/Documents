# 截屏
作者： 康林 <kl222@126.com>

## Windows

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

void DestroyImage(void *pImage)
{
    LOG_MODEL_DEBUG("CDisplayXLib", "void DestroyImage(void *info)");
    XDestroyImage(static_cast<XImage*>(pImage));
}

QImage CScreenXLib::GetScreen(int index)
{
    Window desktop = 0;
    Display* dsp = NULL;
    QImage img;

    dsp = XOpenDisplay(NULL);/* Connect to a local display */
    if(NULL == dsp)
    {
        LOG_MODEL_ERROR("CScreenXLib","Cannot connect to local display");
        return img;
    }

    do{
        desktop = DefaultRootWindow(dsp); // RootWindow(dsp,0);/* Refer to the root window */
        if(0 == desktop)
        {
            LOG_MODEL_ERROR("CScreenXLib", "cannot get root window");
            break;
        }
        
        /* Retrive the width and the height of the screen */
        int width = DisplayWidth(dsp, DefaultScreen(dsp));
        int height = DisplayHeight(dsp, DefaultScreen(dsp));
        
        /* Get the Image of the root window */
        XImage* pImage = XGetImage(dsp, desktop, 0, 0, width, height, AllPlanes, ZPixmap);
        img = QImage((uchar*)pImage->data,
                     pImage->width, pImage->height,
                     GetFormat(pImage),
                     myImageCleanupHandler, pImage);
    } while(0);
    XCloseDisplay(dsp);
    return img;
}

### 参考

- [xlib documents](https://www.x.org/releases/current/doc/libX11/libX11/libX11.html)

