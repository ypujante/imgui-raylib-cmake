cmake_minimum_required(VERSION 3.22)

project(imgui_raylib_cmake LANGUAGES CXX)

set(target "imgui_raylib_cmake")

set(CMAKE_CXX_STANDARD 17)

#######################################################
# adding raylib
#######################################################
option(CUSTOMIZE_BUILD "Show options for customizing your Raylib library build." ON)
if (APPLE)
  set(OPENGL_VERSION "ES 2.0" CACHE STRING "Forcing ES 2.0 to use angle")
endif()
add_subdirectory("${CMAKE_SOURCE_DIR}/external/raysan5/raylib" raylib EXCLUDE_FROM_ALL)

#######################################################
# adding imgui
#######################################################
add_subdirectory("${CMAKE_SOURCE_DIR}/external/ocornut/imgui" imgui EXCLUDE_FROM_ALL)

#######################################################
# adding rlImGui (imgui backend)
#######################################################
add_subdirectory("${CMAKE_SOURCE_DIR}/external/raylib-extras/rlImGui" rlImGui EXCLUDE_FROM_ALL)

#######################################################
# Main executable
#######################################################
add_executable(${target} MACOSX_BUNDLE main.cpp)
target_link_libraries(${target} PUBLIC raylib imgui rlImGui)

#######################################################
# on macOS => create a bundle (.app) and use Google angle
#######################################################
if (APPLE)
  set_target_properties(${target} PROPERTIES
      BUNDLE True
      MACOSX_BUNDLE_GUI_IDENTIFIER com.acme.imgui_raylib_cmake
      MACOSX_BUNDLE_BUNDLE_NAME "ImGui RayLib CMake"
      MACOSX_BUNDLE_BUNDLE_VERSION "1.0.0"
      MACOSX_BUNDLE_SHORT_VERSION_STRING "1.0.0"
      MACOSX_BUNDLE_ICON_NAME "AppIcon"
      MACOSX_BUNDLE_COPYRIGHT "© 2023 acme"
      MACOSX_BUNDLE_INFO_PLIST "${CMAKE_SOURCE_DIR}/cmake/MacOSXBundleInfo.plist.in"
      INSTALL_RPATH "@executable_path/../Frameworks"
      )
  target_link_libraries(${target} PRIVATE "-framework IOKit" "-framework Cocoa" "-framework OpenGL")

  # link with google angle which implements the old/deprecated macOS OpenGL API to use metal
  set(GOOGLE_ANGLE_DIR "${CMAKE_CURRENT_LIST_DIR}/external/google/angle")
  list(APPEND google_angle_libs "${GOOGLE_ANGLE_DIR}/macOS/libEGL.dylib" "${GOOGLE_ANGLE_DIR}/macOS/libGLESv2.dylib")
  target_link_libraries(${target} PRIVATE "${google_angle_libs}")
endif()

#######################################################
# Install
#######################################################
set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install")

install(
    TARGETS "${target}"
    DESTINATION "."
)

if (APPLE)
  # installs the libraries under Content/Frameworks
  install(
      FILES "${google_angle_libs}"
      DESTINATION "$<TARGET_BUNDLE_DIR_NAME:${target}>/Contents/Frameworks"
  )

  # TODO: if using the Xcode generator, the executable needs to be resigned...

endif()
