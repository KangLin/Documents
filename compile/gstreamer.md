## GStreamer
作者：康林 (kl222@126.com)

### 编译

- 下载 gst-build ：

    git clone https://github.com/GStreamer/gst-build.git

- 安装工具：
  - 查看 gst-build 的 README.md 中的说明
  - flex
  - bison

- 编译

        cd gst-build
        meson build
        ninja -C build
