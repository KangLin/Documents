## 工具
康林 (kl222@126.com)

-----------------------------------------------

#### 库查看工具
##### Windows

- dumpbin： MSVC 自带，需要在 MSVC 工具控制台中运行
  + 查看动态库导出函数

```
dumpbin /exports a.dll
```

  + 查看静态库导出函数

```
dumpbin /LINKERMEMBER os.lib
```

##### Linux

```
nm a.lib
```