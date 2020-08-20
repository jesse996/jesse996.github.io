# Qt笔记


[Qt 快速入门系列教程](http://shouce.jb51.net/qt-beginning/)

Qt 如果用 cmake 构建的话，默认是没有 UNICODE 预定义的，也就是说调用的 win32 api 都是 A 版的，而不是 W 版，解决方法是在 CMakeLists.txt 中加入 add_definitions(-DUNICODE -D_UNICODE)

### 设置透明而空间不透明

```cpp
this->setWindowFlags(Qt::FramelessWindowHint);
this->setWindowOpacity(1);
this->setAttribute(Qt::WA_TranslucentBackground);
```

### 关于常用 setWindowFlags 的状态设置：

```cpp
Qt::FramelessWindowHint 窗口无边框
Qt::WindowStaysOnTopHint 窗口置顶
Qt::WindowMaximized 窗口启动最大化
Qt::SubWindow 表示窗口小部件是子窗口
```

