# Linux 内核编译
作者：康林 <kl222@126.com>

## 下载 linux 内核

https://kernel.org/

## 安装依赖库

```bash
sudo apt-get update
sudo apt-get install make gcc
sudo apt-get install flex bison
sudo apt-get install libncurses5-dev #用于menuconfig
```

### 配置内核编译参数

```bash
make menuconfig
```

### 编译内核

```bash
make bzImage
```

### 编译和安装 modules

```bash
make modules
make modules-install
```

### 安装内核

```bash
make install
```
