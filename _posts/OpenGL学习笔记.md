# OpenGL 学习笔记

[toc]

主要根据[这个教程](https://learnopengl-cn.github.io/)，也可以在[这里](https://learnopengl.com/)找到它的英文原版。

环境 Windows10，开发工具 Visual Studio 2019，OpenGL>=3.3。



## 一些会用到的第三方库

### GLFW

GLFW是一个专门针对OpenGL的C语言库，它提供了一些渲染物体所需的最低限度的接口。它允许用户创建OpenGL上下文，节省了我们书写操作系统相关代码的时间。

在[这里](http://www.cmake.org/cmake/resources/software.html)下载CMake，在[这里](http://www.glfw.org/download.html)下载GLFW的源码，编译得到 **glfw3.lib** ，并在源码中找到它的 **include** 文件夹，在项目里把他们添加进依赖里就行。

### GLAD

因为OpenGL只是一个标准/规范，具体的实现是由驱动开发商针对特定显卡实现的。由于OpenGL驱动版本众多，它大多数函数的位置都无法在编译时确定下来，需要在运行时查询。所以任务就落在了开发者身上，开发者需要在运行时获取函数地址并将其保存在一个函数指针中供以后使用。而取得地址的方法[因平台而异](https://www.khronos.org/opengl/wiki/Load_OpenGL_Functions)。于是我们需要 **GLAD** 来帮助我们简化这一过程。

可以在[这里](http://glad.dav1d.de/)配置并下载到对应的 **GLAD** ，Language设置为**C/C++**，在API选项中，选择**3.3**以上的OpenGL(gl)版本（我们的教程中将使用3.3版本，但更新的版本也能正常工作）。之后将模式(Profile)设置为**Core**，并且保证**生成加载器**(Generate a loader)的选项是选中的。





