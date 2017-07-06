---
hash-prefix: '84c644a9_'
menu:
- class: 'documentation button button-green align-right'
  href: '/doc'
  title: Documentation
title: Building Lugdunum
---

* This will become a table of contents (this text will be scraped).
{:toc}

# Dependencies for Lugdunum

## Introduction

Lugdunum depends on many different libraries / projects in order to work properly.
You can find [here](https://github.com/Lugdunum3D/Lugdunum-ThirdParty "third party lugdunum repository") compiled versions, ready to use to compile Lugdunum and get started quickly.

## List of dependencies

-   [Assimp](http://assimp.org/): *Open Asset Import Library (short name: Assimp) is a portable Open Source library to import various well-known 3D model formats in a uniform manner.*
-   [Fmt](http://fmtlib.net/latest/index.html): *fmt (formerly cppformat) is an open-source formatting library. It can be used as a safe alternative to printf or as a fast alternative to C++ IOStreams.*
-   [Vulkan](https://www.khronos.org/vulkan/): *Vulkan is a new generation graphics and compute API that provides high-efficiency, cross-platform access to modern GPUs used in a wide variety of devices from PCs and consoles to mobile phones and embedded platforms.*

## Optional dependencies

All our code is covered by different tests through the Googletest / Googlemock framework.
You can find the sources of the framework at the [following link](https://github.com/google/googletest) or in our [third party repository](https://github.com/Lugdunum3D/Lugdunum-ThirdParty)

## How it works

All the dependencies can be found in our [ThirdParty repository](https://github.com/Lugdunum3D/Lugdunum-ThirdParty), which is added as a submodule of the main repository.

It is planned to add an utility script to update and compile all the dependencies at once, but as of now, it is still a manual task.

# How to build Lugdunum

## Cloning the repository

First, clone the 3D engine repository:

    git clone git@github.com:Lugdunum3D/Lugdunum.git

Now, to build Lugdunum, you'll need to either have some dependencies installed, or you can automatically pull them from the `thirdparty` submodule, that regroups their pre-compiled versions to set you up more quickly:

    git submodule update --init --recursive

## <img src="assets/Tux.svg" width="24px"/> Linux

### General prerequisites

| Target | Toolchain       |
|--------|-----------------|
| Linux  | gcc &gt;= 6     |
| Linux  | clang &gt;= 3.8 |

### Distribution specific prerequisites

#### <img src="assets/logo-ubuntu_cof-orange-hex.svg" width="16px"/> Ubuntu

These instructions were tested for Ubuntu 16.04 LTS. A recent version of GCC (at least the version 6) is needed to compile Lugdunum. You can add the correct repository on an Ubuntu machine with the following commands:

``` bash
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt update
```

You can now install the dependencies needed to build Lugdunum: gcc6, CMake (the tool we use to build Lugdunum), the development version of the X11 libraries:

``` bash
sudo apt install gcc-6 cmake libxrandr-dev
```

There is not yet a Vulkan SDK package on Ubuntu, so you'll have to download and install it yourself. A very complete documentation is already available [on the LunarG website](https://vulkan.lunarg.com/doc/sdk/latest/linux/getting_started.html), so we won't get into details here. Just make sure you have the `VULKAN_SDK` environment variable set, [as described here](https://vulkan.lunarg.com/doc/sdk/latest/linux/getting_started.html#user-content-set-up-the-runtime-environment), with the `x86_64` architecture.

#### <img src="assets/Archlinux-icon-crystal-64.svg" width="16px"/> Arch Linux

On Arch Linux, nice people packaged the Vulkan SDK and provided it at [vulkan-validation-layers](https://www.archlinux.org/packages/extra/i686/vulkan-validation-layers/), and it depends on all the right things to make things easier. So all you have to do is:

    pacman -S vulkan-validation-layers base-devel cmake

### Building

The commands below should be distribution independant, hopefully. What we do is create a "build" directory (out-of-source build), `cd` in it and run `cmake` with the appropriate compiler versions.

    mkdir build
    cd build
    cmake ../ -DCMAKE_C_COMPILER=gcc-6 -DCMAKE_CXX_COMPILER=g++-6
    make

## <img src="assets/Windows_logo_–_2012_(dark_blue).svg" width="24px"/> Windows

### General prerequisites

| Target     | Toolchain                                                     |
|------------|---------------------------------------------------------------|
| Windows 10 | Visual Studio 2015                                            |
| Windows 10 | [Visual Studio 2017](https://www.visualstudio.com/downloads/) |

### Building

To build Lugdunum on Windows, you'll need [CMake](https://cmake.org/download/). CMake will generate a Visual Studio solution that you can then open, and build.

In command line, you can generate the solution with:

    mkdir build
    cd build
    cmake
        -G"Visual Studio 15 2017 Win64"
        -DCMAKE_INSTALL_PREFIX="Path/To/Install"
        ../

**Caution:** As Windows doesn't have a default path to install libraries, CMAKE\_INSTALL\_PREFIX is mandatory

Then, open the generated `Lugdunum.sln` with Visual Studio and compile it.

#### Visual studio 2017

With the [recent support of CMake](https://blogs.msdn.microsoft.com/vcblog/2016/10/05/cmake-support-in-visual-studio/) in Visual Studio 2017, building and installing CMake projects is now possible directly within Visual Studio.
Just modify the CMake configuration file `CMakeSettings.json` to change the install path.

``` json
{
  "configurations": [
   {
    "name": "my-config",
    "generator": "Visual Studio 15 2017",
    "buildRoot": "${env.LOCALAPPDATA}\\CMakeBuild\\${workspaceHash}\\build\\${name}",
    "cmakeCommandArgs": "",
    "variables": [
     {
      "name": "CMAKE_INSTALL_PREFIX",
      "value": "Path/To/Install"
     }
    ]
  }
 ]
}
```

## <img src="assets/Android_robot.svg" width="24px"/> Android

| Target  | Toolchain                                |
|---------|------------------------------------------|
| Android | NDK &gt;= r14 + clang + Gradle &gt;= 2.2 |

### General prerequisites

-   [Android NDK r14+](https://developer.android.com/ndk/index.html)
-   [Android Studio 2.2+](https://developer.android.com/studio/index.html)

Please note that [arm64-v8a](https://developer.android.com/ndk/guides/abis.html#arm64-v8a) is the only supported ABI and that we only support Android N (android-24) and up.

### About the Android NDK

As the gcc toolchain is now deprecated by Android's developers, the clang toolchain will be the only one supported in this project. Please note that we're also using [Unified Headers](https://github.com/android-ndk/ndk/wiki/Changelog-r14) from Android NDK 14.

### Compiling

The following commands should work on a Linux environment, and should give you an idea of what’s necessary to build Lugdunum for Android in another environment.

For better understanding of Android NDK CMake variables, visit [official NDK documentation](https://developer.android.com/ndk/guides/cmake.html#cmake-variables)

    mkdir build
    cd build
    ~/Android/Sdk/cmake/3.6.3155560/bin/cmake \
        -G "Android Gradle - Unix Makefiles" \
        -DANDROID=true \
        -DANDROID_PLATFORM=android-24 \
        -DANDROID_STL=c++_shared \
        -DCMAKE_BUILD_TYPE=Debug \
        -DCMAKE_TOOLCHAIN_FILE=PATH_TO_ANDROID_NDK/build/cmake/android.toolchain.cmake \
        -DANDROID_UNIFIED_HEADERS=ON
        ../
    make install

**Note:** Here the CMake path might be different of the one displayed in the command above, please double-check before executing the command and/or filing a bug report.

### Samples

Open the folder `samples/compiler/android` with Android Studio, let gradle configure the project.
If the NDK isn’t configured properly, you’ll have to tell Android Studio where to find it:

*`File > Project Structure > SDK Location > Android NDK Location`*

Let the gradle configure and sync the project.

The samples should now be available as targets and be buildable from Android Studio.

## <img src="assets/Apple_logo_black.svg" width="24px"/> Apple macOS & iOS

These platforms are not yet supported, but they might be one day if Apple decides to release Vulkan on their systems.
