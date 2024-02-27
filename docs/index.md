## The Introduction

Let's take a look at what it takes to hook up a minimal C++ project to Spack. 
What is Spack? [Spack](spack.readthedocs.io/latest) is an opensource package manager
designed for the high performance computing (HPC) application space.

Spack utilizes an expressive language to describe software application build configurations.

*more on spack specs, combinatorics, concretization, etc*

## The Setup

We're going to begin with a minimal `Hello-World` CMake project with the following code:

``` cmake
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

hosted at [https://github.com/psakievich/hello-spack](https://github.com/psakievich/hello-spack) and create 
a spack package.

We also need to clone spack and get that setup following the directions [here](https://spack.readthedocs.io/en/latest/getting_started.html#installation).

``` terminal
$ git clone -c feature.manyFiles=true https://github.com/spack/spack.git

# Shell support (optional)
# For bash/zsh/sh
$ . spack/share/spack/setup-env.sh
```

## Creating the Package

Now we will create the spack package using the `spack create` command.


``` terminal
# lots to be learned from using --help
$ spack create --help

$ spack create hello-spack --template cmake
```
We can now walk through the suggested modifications based on the `FIXME's` and other comments.
Minimally we need to:
1) add a `git` attribute to point the package to our source
2) add a version using the `main` branch.


