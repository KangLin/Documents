# Qt 主题

作者：康林 <kl222@126.com>

------------------------------

- 图标主题规范: https://specifications.freedesktop.org/icon-theme-spec/icon-theme-spec-latest.html
  - 中文翻译：https://glavo.site/translate/2019/09/22/icon-theme-spec/
- 图标命名规范: https://specifications.freedesktop.org/icon-naming-spec/icon-naming-spec-latest.html

### 示例

- RabbitCommon: https://github.com/KangLin/RabbitCommon/tree/master/Src/Resource/icons

这里有一组颜色图标主题。你可以进入其中一个目录，例如：black .

```bash
l@DESKTOP-081RDD4 MSYS /g/Source/RabbitCommon/Src/Resource/icons/black
$ ls
index.theme  svg
```

index.theme

```
[Icon Theme]
Name=black
Comment=black
Inherits=breeze
Directories=svg, png, ico

[svg]
Size=200
MinSize＝16
ManSize=256
Type＝Scalable

[png]
Size=200
MinSize＝16
ManSize=256
Type＝Scalable

[ico]
Size=200
MinSize＝16
ManSize=256
Type＝Scalable
```

**注意**: 作为资源文件时，别名时，心须要包含扩展名

- QIcon::setFallbackThemeName: 设置后备主题
**注意**：必须在主窗口初始化前使用。
