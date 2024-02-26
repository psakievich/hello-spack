# Hello-Spack

Let's take a look at what it takes to hook up a minimal C++ project to Spack. 

We're going to begin with a minimal `Hello-World` CMake project with the following:

``` Cmake
# CMakeLists.txt

cmake_minimum_required(VERSION 3.0)

project(HelloSpack)

add_executable(hello main.cpp)
```

and 

``` c++
#include <iostream>

int main(int argc, char* argv[]){
  std::cout<<"hello world"<<std::endl;
  return 0;
}
```
