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

install(TARGETS hello DESTINATION bin)
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

# other option but lose some features that need shell support
# export PATH=`pwd`/spack/bin/spack:${PATH}

# sandbox things if you are already using spack
$ export SPACK_DISABLE_LOCAL_CONFIG=1
$ export SPACK_USER_CACHE_PATH=$SPACK_ROOT/.cache

# bootstrap
$ spack bootstrap now
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

1. add a `git` attribute to point the package to our source
2. add a version using the `main` branch.

Now let's run `spack info hello-spack` and digest what is there.

## Installing the Package

Now let's take a look at some concretizations of `hello-spack`.

``` terminal
$ spack solve hello-spack

$ spack solve hello-spack build_type=Debug
```

Notice the build depencencies coming from `CMake` and `gmake`?
If we know these are on the system already we can tell spack to look for them with

``` terminal
$ spack external find cmake gmake
```

Let's see the impact on concretization with the added externals:

``` terminal
$ spack external find cmake gmake
```
We can install with:

``` terminal
$ spack install hello-spack
```

And we can see what all has been installed with:
``` terminal
$ spack find
```

And use it:
``` terminal
$ spack load hello-spack

$ hello
hello world

$ spack unload hello-spack
```

## The Case for Environments

What will happen if we also install a debug version?
``` terminal
$ spack install hello-spack build_type=Debug

$ spack find -vL hello-spack

$ spack load hello-spack
```

So if we install multiple version of the same spec we have to get very specific,
and we have to start worrying about collisions, stale installations etc.

Spack has the ability to generate [environments](https://spack.readthedocs.io/en/latest/environments.html#environments-spack-yaml)
to help with this issue and allow for specific customization of configuration settings (installation locations, module names and locations, build parameters, etc).


