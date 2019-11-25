## 前提
syberh_qrcode模块如何引入so文件，这里我们以`libzxing.so`为例

# syberh_qrcode.pri文件
首先把`libzxing.so`文件放在根目录下的lib目录里面

```C++
// 引入动态库(-L 表示目录, -l表示库的名字)
LIBS += -L$$PWD/lib -lzxing

// libFile.files 是库在项目中的目录
// libFile.path 是库在打包后的安装目录（打包后的根目录下的libs目录）
// INSTALLS 表示打包的时候把libFile打包进去
libFile.files = $$PWD/lib/*
libFile.path = $$INSTALL_DIR/libs
INSTALLS += libFile

// QMAKE_LFLAGS的好处在于，能够选择不同路径下的依赖库
// $$LIB_DIR路径是在app/syberos.pri中声明的
QMAKE_LFLAGS += -Wl,-rpath=$$LIB_DIR/

// 用来在IDE中做展示用的
DISTFILES += \
    $$PWD/lib/libzxing.so
```

# 如何在syberos.pri中引入
``` C++
// 引入lib库(so文件)需要添加这个变量，安装到手机以后lib库位于/data/app-libs目录下， 根据appid来分开的
LIB_DIR = /data/app-libs/com.syberos.demo
```