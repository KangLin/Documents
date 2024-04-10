## 工具
康林（kl222@126.com)

--------------------------------------

#### 查看库内容
##### Windows
- dumpbin: MSVC工具自带。使用前启动MSVC控制台
  + 查看 DLL 导出函数

```
dumpbin /exports os.lib
```

  + 查看静态库

```
dumpbin /LINKERMEMBER os.lib
```

##### linux

```
nm a.lib
```

- 查看程序格式

```
readelf -d a.dll
```
