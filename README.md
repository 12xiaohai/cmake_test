# cmake_test
> vscode + cmake + git 创建**简单**的工程项目模板

## 准备工作

### github上新建一个仓库

1. 仓库名字
2. 添加描述 描述会被自动添加到 README.md 文件里
3. 选择私有或公开
4. 添加REAMDE.md
5. 选择忽略文本的语言为 C++ 或者 cmake
6. 选择许可证为MIT License
7. 创建即可

### 克隆仓库到本地

一般 HTTPS 即可 把仓库的代码地址复制下来终端使用 git clone 命令克隆下载到本地用vscode打开

## 构建方式之分目录构建

### 创建必要文件和目录

写代码前要创建必要的目录用于管理代码 可以用 tree /f 命令查看项目树

![image-20250418170428044](..\image\image-20250418170428044.png)

* include 用于放头文件
* src 用于放源文件
* CMakeLists.txt 用于管理项目
* main.cpp 主函数 一般为项目的可执行文件

### 选择工具包

ctrl +shift + p 输入cmake 选择 CMake:Build 选择构建工具 我这里选择 GCC

![image-20250418170143116](image\image-20250418170143116.png)

退出代码为0代表构建成功

### 写代码

我们写一个hello.h放在include目录下，hello.cpp放在src目录下，main.cpp放在根目录下

代码内容就是输出打印 Hello World! 

### 编写 CMakeLists.txt 文件

这个是比较重要的 因为所有的目录之间的关系都需要 CMakeLists.txt 来配置

```cmake
cmake_minimum_required(VERSION 3.10)
project(CmakeTest)
set(CMAKE_CXX_STANDARD 17)

# 1、变量和目录设置
# 把动态库和可执行文件都指定到bin目录下 windows平台
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)
# linux平台
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)

# 2、动态库
# 源文件打包到变量SRC_LIST中 
aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src SRC_LIST)
# 生成动态库文件 SHARED表示生成共享库文件 库名为common
add_library(common SHARED ${SRC_LIST})
# 包含动态库common的头文件路径
target_include_directories(common PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)

# 3、可执行文件
# 生成可执行文件
add_executable(main main.cpp)
# 生成的可执行文件包含头文件路径
target_include_directories(main PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)

# 4、链接动态库
# 可执行文件也要链接动态库common
target_link_libraries(main PUBLIC common)

```

上面就是编写的大概步骤 基本已经适合绝大多数小项目

> 动态库是什么意思呢？在windows平台下其实就是.dll 的文件，是 .exe文件运行的必备文件

### 解读 CMakeLists.txt 文件

这个 `CMakeLists.txt` 文件的工作原理和设计逻辑可以分为几个关键部分来理解。我会逐步解释每个指令的作用，以及为什么需要这样写。

---

#### **1. 基础配置**
**`cmake_minimum_required(VERSION 3.10)`**

• **作用**：指定 CMake 的最低版本要求（这里是 3.10）。
• **为什么需要**：确保 CMake 的功能兼容性，避免因版本过低导致语法错误。

**`project(CmakeTest)`**

• **作用**：定义项目名称（`CmakeTest`）。
• **为什么需要**：
  • 设置项目名称，影响后续变量（如 `PROJECT_NAME`）。
  • 隐式定义 `CMAKE_PROJECT_NAME` 和 `PROJECT_SOURCE_DIR` 等变量。

**`set(CMAKE_CXX_STANDARD 17)`**

• **作用**：指定 C++ 标准为 C++17。
• **为什么需要**：确保编译器使用 C++17 标准，避免因版本差异导致语法错误。

---

#### **2. 输出目录设置**
**Windows 平台**

```cmake
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)
```
• **作用**：指定可执行文件（`.exe`）和动态库（`.dll`）的输出目录为 `./bin`。
• **为什么需要**：
  • Windows 下，可执行文件和动态库通常放在同一目录（`bin`）。
  • 避免构建产物散落在 `build` 目录中，方便管理。

**Linux 平台**

```cmake
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)
```
• **作用**：指定动态库（`.so`）的输出目录为 `./bin`。
• **为什么需要**：
  • Linux 下，动态库通常放在 `lib` 或 `bin` 目录。
  • 这里统一放在 `bin` 目录，保持和 Windows 一致。

---

#### **3. 动态库的构建**
**`aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src SRC_LIST)`**

• **作用**：自动扫描 `src/` 目录下的所有 `.cpp` 文件，并存入变量 `SRC_LIST`。
• **为什么需要**：
  • 避免手动列出所有源文件，简化维护。
  • **缺点**：如果 `src/` 目录下有无关文件（如临时文件），可能会被错误包含。

**`add_library(common SHARED ${SRC_LIST})`**

• **作用**：将 `SRC_LIST` 中的源文件编译成动态库 `libcommon.so`（Linux）或 `common.dll`（Windows）。
• **为什么需要**：
  • `SHARED` 表示生成动态库（`.so`/`.dll`），而不是静态库（`.a`/`.lib`）。
  • 动态库可以在运行时加载，节省内存（多个程序可共享同一个库）。

**`target_include_directories(common PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)`**

• **作用**：指定 `common` 库的头文件搜索路径为 `include/`。
• **为什么需要**：
  • `PUBLIC` 表示不仅 `common` 库本身需要这个路径，**依赖 `common` 的目标（如 `main`）也会自动继承该路径**。
  • 确保编译器能找到 `#include "common.h"` 这样的头文件。

---

#### **4. 可执行文件的构建**
**`add_executable(main main.cpp)`**

• **作用**：将 `main.cpp` 编译成可执行文件 `main`（Linux）或 `main.exe`（Windows）。
• **为什么需要**：定义可执行文件的入口。

**`target_include_directories(main PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)`**

• **作用**：指定 `main` 的头文件搜索路径（和 `common` 一致）。
• **为什么需要**：
  • 如果 `main.cpp` 也使用了 `include/` 下的头文件，需要确保编译器能找到它们。
  • 如果 `common` 已经 `PUBLIC` 包含了 `include/`，这里可以省略（因为 `main` 链接了 `common`，会自动继承）。

**`target_link_libraries(main PUBLIC common)`**

• **作用**：让 `main` 链接 `common` 动态库。
• **为什么需要**：
  • 确保 `main` 能调用 `common` 中的函数。
  • `PUBLIC` 表示如果其他目标依赖 `main`，它们也会自动链接 `common`（适用于库的传递依赖）。

---

#### **5. 整体工作原理**
1. **配置阶段**（`cmake ..`）：
   • 解析 `CMakeLists.txt`，生成构建规则（如 Makefile 或 Visual Studio 项目）。
2. **编译阶段**（`make` 或 `cmake --build`）：
   • 先编译 `src/*.cpp` 成动态库 `common`。
   • 再编译 `main.cpp` 并链接 `common`，生成可执行文件 `main`。
3. **运行阶段**：
   • 在 Windows 下，`main.exe` 和 `common.dll` 必须放在同一目录（`bin/`）。
   • 在 Linux 下，`main` 需要能通过 `LD_LIBRARY_PATH` 找到 `libcommon.so`。

---

### 为什么这样设计？
1. **模块化**：
   • 动态库 `common` 可以复用于其他项目。
2. **跨平台**：
   • 通过 `CMAKE_RUNTIME_OUTPUT_DIRECTORY` 和 `CMAKE_LIBRARY_OUTPUT_DIRECTORY` 适配不同操作系统。
3. **依赖管理**：
   • `target_include_directories` + `target_link_libraries` 确保头文件和库的正确传递。
4. **可维护性**：
   • 清晰的目录结构（`src/`、`include/`、`bin/`）便于扩展。

---

## 一键构建

构建完之后我们可以借助 cmake 插件 点击生成 然后就会一键构建我们的项目

或者 cd到build目录下 输入 cmake .. 然后输入 make 同样可以

![image-20250418173127010](image\image-20250418173127010.png)

![image-20250418173158857](image\image-20250418173158857.png)

同样退出代码为 0 代表构建成功

点击运行即可自动执行 bin 目录下的main.exe文件

![image-20250418173323822](image\image-20250418173323822.png)

## 推送代码

写好代码后即可把本地的代码推送到github上去 在此之前选择git bash 终端

![image-20250418173743490](image\image-20250418173743490.png)

### git 命令介绍

```bash
git status
```

`git status` 是 Git 版本控制系统中一个**最常用**的命令，用于查看当前仓库的状态，包括：

- **哪些文件被修改了**
- **哪些文件已暂存（准备提交）**
- **哪些文件未被跟踪（新文件）**
- **当前分支信息**
- **是否有需要拉取（pull）或推送（push）的变更**

```bash
git remote -v
```

`git remote -v` 是 Git 中用于**查看远程仓库信息**的命令，它会列出当前项目配置的所有远程仓库名称及其对应的 URL（包括 `fetch` 和 `push` 地址）。

```bash
git branch -v
```

`git branch -v` 是 Git 中用于**查看本地分支及其最新提交信息**的命令。它会列出所有本地分支，并显示每个分支的最后一次提交的**提交哈希（缩写）**和**提交信息摘要**。

```bash
git add .
```

`git add .` 是 Git 中一个**核心命令**，用于将当前目录及其子目录下的**所有新增或修改的文件**添加到 Git 的暂存区（Staging Area），为后续提交（`git commit`）做准备。

```bash
git commit -m "提交信息"
```

它会将 `git add` 暂存的变更永久记录到版本历史中，并允许你通过 `-m` 参数直接附加提交信息。

```bash
git push
```

`git push` 是 Git 中用于**将本地提交推送到远程仓库**的核心命令

**总结：git add . -> git commit -m "提交信息" -> git push**

> 在推送之前最好把cmake的构建过程的文件给添加到忽略文件里 包括 build 和 bin文件夹

![image-20250418175108461](image\image-20250418175108461.png)

刷新仓库

![image-20250418175216527](image\image-20250418175216527.png)

## 完结
