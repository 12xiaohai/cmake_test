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


