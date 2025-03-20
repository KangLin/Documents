# Fedora
作者：康林<kl222@126.com>

### dnf 使用

- 显示所有包

      dnf list

- 查看莫个内容，例如：查看 `winpr-makecert` 位于哪个包

      dnf provides */winpr-makecert

- 列出 RPM 包中的文件

      rpm -qpl rabbitremotecontrol-1.0.0-1.fc38.x86_64.rpm

- 查看 RPM 包元信息

      rpm -qpi rabbitremotecontrol-1.0.0-1.fc38.x86_64.rpm

- 查看 RPM 包依赖项

      rpm -qpR rabbitremotecontrol-1.0.0-1.fc38.x86_64.rpm

- 检查已安装的 RPM 内容

      # 列出所有文件
      rpm -ql rabbitremotecontrol

      # 查看包信息
      rpm -qi rabbitremotecontrol

