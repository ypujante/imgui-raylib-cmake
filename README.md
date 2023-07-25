Introduction
------------

This is a "Hello World" project for an ImGui application that uses raylib for the backend, CMake for the build and Google Angle on macOS (to use the Metal API instead of the deprecated OpenGL API)

![macOS](https://github.com/ypujante/imgui-raylib-cmake/releases/download/v2.0.0/macOS.png)

Dependencies
------------

On macOS, due to the difficulty in handling security, this project no longer includes the Google Angle libraries but simply depends on Google Chrome being installed on the machine. CMake will automatically find the libraries and applies the changes necessary to make it work (using `install_name_tool` and `codesign`)

Build
-----

It is a CMake project, so it builds very simply:

```cmake
> mkdir build
> cd build
> cmake ..
> cmake --build . --target imgui_raylib_cmake
```

> #### Note
> On macOS, CMake automatically copies and modifies the Google Chrome Angle libraries to link with

Run
---

* On macOS, you get a bundle that you can simply open
* On Windows/Linux, you get an executable that you can simply run

Platforms tested
---------------

* macOS (Ventura 13.4): note that OpenGL is deprecated but this project uses Google Angle to provide a very efficient OpenGL wrapper that delegates to Metal
* Windows 11 (uses native OpenGL)
* Linux Ubuntu (uses native OpenGL)

History
-------

It took me a huge amount of time to set this project up (see [discussion](https://discourse.cmake.org/t/embedding-dylib-in-a-macos-bundle/8465/5?u=fry) as well as this [issue](https://github.com/raysan5/raylib/issues/3179)), and so I decided to share it on github.

Embedded Projects
-----------------

* [ImGui](https://github.com/ocornut/imgui)
* [raylib](https://github.com/raysan5/raylib)
* [rlImGUI](https://github.com/raylib-extras/rlImGui)

Licensing
---------

- Apache 2.0 License. This project can be used according to the terms of the Apache 2.0 license.

