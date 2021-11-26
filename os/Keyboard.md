# 键盘

作者：康林 <kl222@126.com>

### 模拟键盘按下

开发有时候有这种需求：要求由程序模拟键盘按下。

#### 在MacOS下，可以用 CGEventPost 方式模拟键盘按下。


    CGEventRef key = CGEventCreateKeyboardEvent(NULL, kVK_UpArrow, true);
    CGEventPost(kCGHIDEventTap, key);
    CFRelease(key);


#### 在windows下，有 keybd_event 模拟键盘按下。

    keybd_event(VK_UP,0,0,0); 
    keybd_event(VK_UP,0,KEYEVENTF_KEYUP,0);

#### 在Linux下，可以使用Xlib提供的接口去模拟键盘按下。

##### 需要用到的头文件有

    #include <X11/Xlib.h>
    #include <X11/extensions/XTest.h> //XTestFakeKeyEvent
    #include <X11/keysym.h> //KeySym KeyCode XKeysymToKeycode


##### 需要用到的库有两个：   libX11.so libXst.so 

##### 完整代码：

    void ClickKey()
    {
        Display* p_display = XOpenDisplay( NULL );
        KeySym keysym = XK_Down;
        KeyCode keycode = NoSymbol;
 
        keycode = XKeysymToKeycode( p_display , keysym );
   
        XTestFakeKeyEvent( p_display , keycode , True  , 0 ); // 键盘按下event
        XTestFakeKeyEvent( p_display , keycode , False , 0 ); // 键盘释放event
        XFlush( p_display );
 
        XCloseDisplay( p_display );  
    }


