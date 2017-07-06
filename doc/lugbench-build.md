---
hash-prefix: 'b23ed2ae_'
menu:
- class: 'documentation button button-green align-right'
  href: '/doc'
  title: Documentation
title: Building LugBench
---

* This will become a table of contents (this text will be scraped).
{:toc}

# LugBench API

## Dependencies for Lugbench API

-   [NodeJS](https://nodejs.org/en/): a JavaScript runtime built on Chrome's V8 JavaScript engine
-   [NPM](https://www.npmjs.com/): a package manager for JavaScript
-   [MongoDB](https://www.mongodb.com/what-is-mongodb): a scalable and flexible document database

## Cloning the repository

First, clone the front-end repository:

    git clone git@github.com:Lugdunum3D/LugBench-API.git

## Prerequisites

### MongoDB

First, you may have to create a local database to test on using the following command:

    mongod --dbpath <wanted_path> --smallfiles

**Note:** `27017` is the default port but you can set it by running `mongod` with the `--port <port_number>` argument.

### Installing dependencies

Using npm, just run:

    npm install

This command will install the dependencies from the `package.json` file.

## Environnement variables

Add the `MONGODB_URI` environment variable to set the MongoDB url, with the port being the port you set in the above step, or the default port, `27017`.

    export MONGODB_URI="mongodb://localhost:27017/lugbench-dev"

Here the name is completely up to you to choose; Mongo will automatically create the database if it doesn't exist yet.

You can also define a custom port for the API to run on by setting the `PORT` environment variable.

## Launch the project

In command line, you can launch the project with:

    npm start

The API will listen on the port 5000 by default. You can then send requests to the server, e.g.:

    GET http://localhost:5000/api/v1/gpus

# LugBench's website (the front-end)

## Dependencies for building LugBench's website

-   [NPM](https://www.npmjs.com/): a package manager for JavaScript
-   [TypeScript](https://www.typescriptlang.org/): TypeScript is a typed superset of JavaScript that compiles to plain JavaScript.
-   [Gulp](http://gulpjs.com/): Gulp is a toolkit for automating painful or time-consuming tasks in your development workflow, so you can stop messing around and build something.
-   [Angular2](http://angular.io/): Angular2 is a JavaScript framework.

## Cloning the repository

First, clone the front-end repository:

    git clone git@github.com:Lugdunum3D/LugBench-Front.git

Then, navigate to the folder `LugBench-Front`

## Launch the project

You will need [NPM](https://www.npmjs.com/) (Node Packet Manager) installed on your computer. NPM will install all the dependances of the project.

In command line, you can launch the project with:

    npm install
    npm run serve

Then start any web browser go to `http://localhost:3000`

## Use NPM scripts

| Command              | Description                                             |
|----------------------|---------------------------------------------------------|
| `npm run build`      | Build an optimized version of your application in /dist |
| `npm run serve`      | Launch a browser sync server on your source files       |
| `npm run serve:dist` | Launch a server on your optimized application           |
| `npm run test`       | Launch your unit tests with Karma                       |
| `npm run test:auto`  | Launch your unit tests with Karma in watch mode         |

# How to build the LugBench application

## Dependencies for the LugBench application

### Introduction

Lugbench depends on many different libraries / projects in order to work properly.
You can find on our [ThirdParty repository](https://github.com/Lugdunum3D/LugBench-ThirdParty "third party lugbench repository") all the compiled versions, ready to use to compile Lugbench and get started quickly.

### List of dependencies

-   [Lugdunum](https://github.com/Lugdunum3D/Lugdunum): Lugdunum is an open-source 3D engine using Vulkan as backend. Lugudunum's goal is to provide a free, modern, cross-platform (mobile and desktop) 3D engine for everyone.
-   [Json *(from nlohmann)*](https://github.com/nlohmann/json) is a header-only Json library for Modern C++.
-   [libcurl](https://curl.haxx.se/libcurl/) is a free and easy-to-use client-side URL transfer library
-   [Restclient-cpp](https://github.com/mrtazz/restclient-cpp) This is a simple REST client for C++. It wraps libcurl for HTTP requests.

**Note:** libcurl and restclient are not needed to build Lugbench on Android.

## Cloning the repository

First, clone Lugbench repository:

    git clone git@github.com:Lugdunum3D/LugBench.git

Now, to build Lugbench, you'll need to either have some dependencies installed, or you can automatically pull them from the `thirdparty` submodule, that regroups their pre-compiled versions to set you up more quickly:

    git submodule update --init --recursive

**Note:** You must first compile the Lugdunum libraries, as shown earlier in this document

## <img src="assets/Tux.svg" width="24px"/> Linux

### General prerequisites

| Target | Toolchain       |
|--------|-----------------|
| Linux  | gcc &gt;= 6     |
| Linux  | clang &gt;= 3.8 |

### Building

The commands below should be distribution independant, hopefully. What we do is create a "build" directory (out-of-source build), `cd` in it and run `cmake` with the appropriate compiler versions and the location of the Lugdunum library.

    mkdir build
    cd build
    cmake
        -DCMAKE_C_COMPILER=gcc-6
        -DCMAKE_CXX_COMPILER=g++-6
        -DLUG_ROOT=PATH_TO_LUGDUNUM_LIBRARY
        ../
    make

**Note:** Of course, CMAKE\_C\_COMPILER and CMAKE\_CXX\_COMPILER can be set to clang and clang++

## <img src="assets/Windows_logo_–_2012_(dark_blue).svg" width="24px"/> Windows

### General prerequisites

| Target     | Toolchain                                                     |
|------------|---------------------------------------------------------------|
| Windows 10 | Visual Studio 2015                                            |
| Windows 10 | [Visual Studio 2017](https://www.visualstudio.com/downloads/) |

### Building

To build Lugbench on Windows, you'll need [CMake](https://cmake.org/download/). CMake will generate a Visual Studio solution that you can then open, and build the project from.

In command line, you can generate the solution with:

    mkdir build
    cd build
    cmake
        -G"Visual Studio 2017 15 Win64"
        -DLUG_ROOT=PATH_TO_LUGDUNUM_LIBRARY
        ../

`LUG_ROOT` designates the location of the Lugdunum library, which is required to build Lugbench. Steps for building the Lugdunum libraries were describes in the first part of this document.

Then, open the generated `Lugbench.sln` with Visual Studio and compile it.

#### Visual studio 2017

With the [recent support of CMake](https://blogs.msdn.microsoft.com/vcblog/2016/10/05/cmake-support-in-visual-studio/) in Visual Studio 2017, building and installing CMake projects is now possible directly within Visual Studio.
Just modify the CMake configuration file called `CMakeSettings.json` to change the install path.

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
      "name": "LUG_ROOT",
      "value": "PATH_TO_LUGDUNUM_LIBRARY"
     }
    ]
  }
 ]
}
```

## <img src="assets/Android_robot.svg" width="24px"/> Android

### General prerequisites

-   You must compile and install [Lugdunum](https://lugdunum3d.github.io) for Android.

**Note:** We suppose that Lugdunum libraries for Android are built in *`ANDROID_NDK/sources/lugdunum`*
In case you specified a different path with `CMAKE_INSTALL_PREFIX`, you must modify the build.gradle accordingly.

### Compiling

Open the folder `Lugbench/android` with Android Studio and let gradle configure the project.

**Note:** If the NDK isn’t configured properly, you’ll have to tell Android Studio where to find it :
*`File > Project Structure > SDK Location > Android NDK Location`*

The project should now be available as a target and be buildable from Android Studio.

## <img src="assets/Apple_logo_black.svg" width="24px"/> Apple macOS & iOS

These platforms are not yet supported, but they might be one day if Apple decides to support Vulkan on their systems.
