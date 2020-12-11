## 基础知识
作者：康林

### 码表
- [键盘扫描码](https://baike.baidu.com/item/%E9%94%AE%E7%9B%98%E6%89%AB%E6%8F%8F%E7%A0%81/7360710?fr=aladdin)
键盘扫描码是键盘硬件产生的编码。由驱动程序解析成虚拟键盘码。

/* @msdn{cc240584} says:
 * "... (a scancode is an 8-bit value specifying a key location on the keyboard).
 * The server accepts a scancode value and translates it into the correct character depending on the
 * language locale and keyboard layout used in the session." The 8-bit value is later called
 * "keyCode" The extended flag is for all practical an important 9th bit with a strange encoding -
 * not just a modifier.
 */

- [虚拟键盘码](https://docs.microsoft.com/zh-cn/windows/win32/inputdev/virtual-key-codes?redirectedfrom=MSDN)
- [ASCII 码](https://baike.baidu.com/item/ASCII/309296?fromtitle=ascii%E7%A0%81&fromid=99077&fr=aladdin)
