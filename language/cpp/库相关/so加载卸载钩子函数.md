# so加载卸载钩子函数

动态库加载，卸载都会有初始化函数`_init()`以及卸载函数`_fini()`来完成库的初始化以及资源回收

example:

dep源文件生成一个so，main源文件加载so，通过`my_init`函数在加载动态库时做自定义初始化的工作，通过`my_finit` 函数在卸载动态库做自定义销毁工作

dep.h

```cpp
#pragma once
#include <string>

void func(const std::string &s);
```

dep.cpp

```cpp
#include "dep.h"

#include <cstdio>
#include <iostream>
#include <unordered_map>
#include <vector>

void my_init(void) __attribute__((constructor));  // init section
void my_fini(void) __attribute__((destructor));   // finit section

void func(const std::string &s) {
  std::cout << "dep lib: test func" << std::endl;
  std::cout << s << std::endl;
}

// std::unordered_map<int, std::string> m;
std::vector<std::string> v;

void my_init(void) {
  /* for (int i = 0; i < 3; ++i) {
    // m[i] = std::to_string(i);
    m.emplace(i, std::to_string(i));
  }

  for (auto &[k, v] : m) {
    std::cout << k << " -> " << v << std::endl;
  } */

  for (int i = 0; i < 3; ++i) {
    v.emplace_back(std::to_string(i));
  }
  for (auto &v : v) {
    // std::cout << v << std::endl;
    printf("%s\n", v.c_str());
  }
}
void my_fini(void) {
  // for (auto &[k, v] : m) {
  //   // destroy;
  // }
}
```

main.cpp

```cpp
#include <iostream>
#include "dep.h"

int main(int argc, char** argv) {
  printf("Main IN\n");
  func("Hello");
  printf("Main Quit\n");
}
```

cmake

```cpp
cmake_minimum_required(VERSION 3.20)
project(test_elf)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(dep SHARED dep.cpp)
# target_link_libraries(dep PUBLIC stdc++)

add_executable(test_elf main.cpp)
target_link_libraries(test_elf dep)
```