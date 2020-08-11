# Cmake笔记


### 文件结构

```cpp
helloworld
  |--CMakeLists.txt
  |--helloworld.cpp
```

### CMakeLists.txt

```cpp
# CMake 最低版本号要求
cmake_minimum_required(VERSION 3.0)

# 指定项目名称，其实就是给变量PROJECT_NAME赋值
project(HelloWorld)

# 查找指定目录下的所有源文件 并存放到指定变量名SRC中
aux_source_directory(. SRC)

# 指定生成目标
add_executable(${PROJECT_NAME} ${SRC})
```

-   **\*`aux_source_directory`**这个函数在添加源码文件时，是不会把头文件添加进去的\*

```cpp
# CMake 最低版本号要求
cmake_minimum_required(VERSION 3.0)

# 项目名称
project(CMakeFile)

# 查找指定目录下的所有.cpp与.h文件 并存放到指定变量名SC_FILES中
FILE(GLOB SC_FILES "*.cpp" "*.h")

# 指定生成目标
add_executable(${PROJECT_NAME} ${SC_FILES})
```

从第三方获取一些功能的源码文件，如 md5

```cpp
CMakeFile
  |--common
  |  |--md5
  |     |--md5.cpp
  |     |--md5.h
  |--CMakeLists.txt
  |--main.cpp
  |--stdafx.h
```

```cpp
# CMake 最低版本号要求
cmake_minimum_required(VERSION 3.0)

# 项目名称
project(CMakeFile)

# 设置md5代码文件的路径
set(MD5_FILE "./common/md5/md5.cpp" "./common/md5/md5.h")

# 查找指定目录下的所有.cpp与.h文件 并存放到指定变量名SC_FILES中
FILE(GLOB SC_FILES "*.cpp" "*.h")

# 对md5的源码分组到md5组里
source_group(md5 FILES ${MD5_FILE})

# 指定生成目标
add_executable(${PROJECT_NAME} ${SC_FILES} ${MD5_FILE})
```

[一. 初识 cmake - cmake 实践 1.0.0 documentation](https://cmake.readthedocs.io/en/latest/1.html)

