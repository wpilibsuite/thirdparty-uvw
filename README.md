# WPILib Third Pary Library - uvw

WPILib has some occasional need for 3rd party libraries. Some of these libraries, don't have good cmake builds, or cmake builds that work correctly for all platforms. For these packages, we are providing a CMake build that properly handles the setup and install of these libraries in a format the WPILib CMake build can correctly consume. This is the instructions for `uvw` and `libuv`

## Prerequisites
For this library, all that is needed is CMake 3.3.0 or greater, and a compiler for your native system.

## Build Options
We provide the following build options, in addition to the standard CMake build type options

* BUILD_SHARED_LIBS (ON Default)
  * Enable this option to build a shared library. We recommend this matches whichever option you are planning on building WPILib with.

## Build Setup
The build does not allow in source builds. Because the `build` directory is used by gradle, we recommend a `buildcmake` directory in the root. This folder is included in the gitignore.

Once you have a build folder, run cmake configuration in that build directory with the following command.

```
cmake path/to/thirdparty-uvw/root
```

If you want to change any of the options, add `-DOPTIONHERE=VALUE` to the cmake command. This will check for any dependencies. If everything works properly this will succeed. If not, please check out the troubleshooting section for help.

If you want, you can also use `ccmake` in order to visually set these properties as well.
https://cmake.org/cmake/help/v3.0/manual/ccmake.1.html
Here is the link to the documentation for that program.

## Building

Once you have cmake setup, run `make` from the directory you configured cmake in. This will build the library. If you have a multicore system, we recommend running make with multiple jobs. The usual rule of thumb is 1.5x the number of cores you have. To run a multiple job build, run the following command with x being the number of jobs you want.

```
make -jx
```

## Installing

After build, in order for the WPILib build to find it, you will need to install the library. Run the following command to install it.

```
sudo make install
```

## Using the installed libraries
The WPILib build already knows about the library so nothing extra will be required to use it.

If you want to use the build in an independent project, create a new folder to contain your project. Add the following code below to a `CMakeLists.txt` file in that directory.

```
project (uvtest)

find_package(thirdparty-uvw REQUIRED)

cmake_minimum_required(VERSION 3.3.0)

add_executable(uvtest main.cpp)
target_link_libraries(uvtest uvw uv)

```

Add a `main.cpp` file to contain your code, and create a build folder. Move into the build folder, and run

```
cmake /path/to/folder/containing/CMakeLists
```

After that, run `make`. That will create your executable. Then you should be able to run `./my_vision_app` to run your application.
