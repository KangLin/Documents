# Linux 下的开发工具

## 动态库工具

- ldd: 查看库依赖工具
- readelf：查看 elf 文件
  - `readelf -d`: 可查看`rpath`

- macOS 下的相同工具

| Linux    | macOS   |
|:--------:|:-------:|
|ldd       |otool -L |
|readelf -d|otool -l |
