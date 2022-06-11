---
published: false
title: "1"
categories: [LLVM]
toc: true
classes: []
excerpt: "1"
---

<!--

### 1. 编译并安装 LLVM 源码

LLVM version 12.0.1

cmake version 3.18.1

以管理员身份运行 PowerShell。
    

```powershell
cd llvm-project-llvmorg-12.0.1
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD=host -DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi" -DCMAKE_INSTALL_PREFIX="D:/LLVM12/" -G "Visual Studio 16 2019" -A x64 -Thost=x64 ..\llvm
```

`-DCMAKE_BUILD_TYPE` 指定编译配置类型，Release 体积比较小。

`-DLLVM_TARGETS_TO_BUILD` 编译本机平台。

`-DLLVM_ENABLE_PROJECTS` 同时编译 clang 等子项目。

`-DCMAKE_INSTALL_PREFIX` 配置安装路径

`-G` Generator，`-A` Architecture，`-Thost` 选择 Host 的构建工具版本。
    
> 不过需要注意的是，windows 下 `-DCMAKE_BUILD_TYPE=Release` 并不生效，需要在 `build` 的时候手动指定。



```powershell
cmake --build . --config Release --target install
```

直接编译并安装到指定目录。



将 `<install dir>/bin` 和 `<install dir>/bin/llvm-config.exe` 加入到环境变量中。



### 2. 配置项目

-->

### 1. 编译并安装 LLVM 源码

LLVM version 12.0.1

cmake version 3.18.1

```shell
cd llvm-project-llvmorg-12.0.1
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD=host -DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi" -G "Ninja" ../llvm
```

`-DCMAKE_BUILD_TYPE` 指定编译配置类型，Release 体积比较小。

`-DLLVM_TARGETS_TO_BUILD` 编译本机平台。

`-DLLVM_ENABLE_PROJECTS` 同时编译 clang 等子项目。

`-DCMAKE_INSTALL_PREFIX` 配置安装路径，不填写就使用默认的 `\usr\local\`

`-G` Generator，`-A` Architecture，`-Thost` 选择 Host 的构建工具版本。



```powershell
sudo cmake --build . --config Release --target install
```

直接编译并安装到指定目录。



