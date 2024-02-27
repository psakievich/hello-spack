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

Spack has the ability to generate 
[environments](https://spack.readthedocs.io/en/latest/environments.html#environments-spack-yaml)
to help with this issue and allow for specific customization of configuration settings (installation locations,
module names and locations, build parameters, etc).

Let's do a few things now and try to learn a few details.
Each of these commands can be explored with the `--help` argument.

``` console
$ spack env create hello 

$ spack env activate hello # alias is spacktivate hellos

# modify the installation location
$ spack config add config:install_tree:'$env/installs'

# modify the paths for the installations
$ spack config add config:install_tree:projections:all:"'{name}-{version}'"

# see how this affects the configs seen inside the environment
$ spack config blame config

# start setting up the package to build
$ spack add hello-spack

$ spack concretize

$ spack install

$ spack location --install hello-spack

# spack help --all to find out what -E does
$ spack -E location --install hello-spack build_type=Debug

$ spack env deactivate # alias is despacktivate
```

The customization opportunities of environments are why they have gained so much traction.
Environments can be used without activation as follows:

``` console
# can be used without shell support
$ spack -e hello [what commands you want to run with the env active]
```

## Application Development

Now let's do some development of the `hello-spack` project.
Specifically, let's add a string that can be set at compile time to append to the `hello world` message.

First we need to mark our `hello-spack` package for 
[development](https://spack.readthedocs.io/en/latest/environments.html#developing-packages-in-a-spack-environment)
inside our `hello` environment.

``` console
$ spack env activate hello

$ spack develop hello-spack@main
```

We also should reconcretize to make sure the develop status has propagated as desired. 
Look for the `dev_path` attribute in the `hello-spack` spec that is printed after concretization.
This tells you where the source code is that it will be building off of.
``` console
$ spack concretize --force
```

And we can reinstall as well:
``` console
$ spack install
```

Okay now let's change the source code to add in a compile time string option.

``` console
$ spack cd --source hello-spack
```

`CMakeLists.txt`
``` cmake
cmake_minimum_required(VERSION 3.0)

project(HelloWorld)

# Set a default value for the string
set(MY_STRING "default" CACHE STRING "My string")


# Add a compiler flag to set the string
add_compile_definitions(MY_STRING="${MY_STRING}")

add_executable(hello main.cpp)
```

`main.cpp`
``` c++
#include <iostream>
#include <string>

#ifndef MY_STRING
#define MY_STRING "default"
#endif

int main(int argc, char* argv[]) {
    std::cout << ""hello world "  << static_cast<std::string>(MY_STRING) << std::endl;
    return 0;
}
```

Now recompile and run the program:

``` console
$ spack install

$ spack load hello-spack

$ hello
```

We can now make a pull/merge request on these changes or push them directly to the repo if desired.

## Add a Variant

Next let's make that string accessible at compile time through the spack package.


``` console
$ spack edit hello-spack
```

Add the following to the `package.py`.
``` python

    variant("append_string",
            default="package.py",
            description="a string to append to hello world")

    def cmake_args(self):
        args = []
        args.append(self.define("MY_STRING", self.spec.variants["append_string"].value))
        return args
```

In this case we've added a `variant`, is a build option for the package,
and we are going to turn it's value into a CMake argument.

This would be the equivalent to passing `-DMY_STRING=package.py` to CMake at configure time.
There is a lot of options and information on spack package development. 
We won't get into it all here, but the 
[documentation](https://spack.readthedocs.io/en/latest/packaging_guide.html#packaging-guide)
is quite thorough.

Now let's uninstall our old instance of `hello-spack`, re-concretize, re-install, and re-run.

``` console
$ spack info hello-spack

$ spack uninstall hello-spack

# look for the new variant
$ spack concretize --force

$ spack install

$ spack load hello-spack

$ hello
```

Let's see the variant in action by changing it's value.
We will be thorough about our installation management since we are changing the variants.

``` console
$ spack unload hello-spack

$ spack uninstall hello-spack

$ spack change hello-spack append_string="a new world"

$ spack concretize --force

$ spack install

$ spack load hello-spack

$ hello
```

## Other Topics to Explore

- Adding dependencies to the package
