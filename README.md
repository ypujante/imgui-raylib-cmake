Introduction
------------

This is a 100% self-contained "Hello World" project for an ImGui application that uses raylib for the backend, CMake for the build and Google Angle on macOS (to use the Metal API instead of the deprecated OpenGL API)

![macOS](https://github.com/ypujante/imgui-raylib-cmake/releases/download/v1.0.0/macOS.png)

Build
-----

It is a CMake project, so it builds very simply:

```cmake
> mkdir build
> cd build
> cmake ..
> cmake --build . --target imgui_raylib_cmake
```

Run
---

On macOS, you get a bundle that you can simply open
On Windows/Linux, you get an executable that you can simply run

Platforms tested
---------------

* macOS (Ventura 13.4): note that OpenGL is deprecated but this project uses Google Angle to provide a very efficient OpenGL wrapper that delegates to Metal (typical 3-5x speed improvement)
* Windows 11 (uses native OpenGL)
* Linux Ubuntu (uses native OpenGL)

History
-------

It took me a huge amount of time to set this project up (see [discussion](https://discourse.cmake.org/t/embedding-dylib-in-a-macos-bundle/8465/5?u=fry)) and so I decided to share it on github.

Embedded Projects
-----------------

* [ImGui](https://github.com/ocornut/imgui)
* [raylib](https://github.com/raysan5/raylib)
* [rlImGUI](https://github.com/raylib-extras/rlImGui)
* [Google Angle (macOS binaries)](https://chromium.googlesource.com/angle/angle)

Licensing
---------

- Apache 2.0 License. This project can be used according to the terms of the Apache 2.0 license.

