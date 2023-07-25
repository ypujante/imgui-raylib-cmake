cmake_minimum_required(VERSION 3.22)

project(imgui_raylib_cmake LANGUAGES CXX)

set(target "imgui_raylib_cmake")

set(CMAKE_CXX_STANDARD 17)

#######################################################
# Option to turn on/off Google Angle on macOS
#######################################################
option(USE_GOOGLE_ANGLE_MAC_OS "Whether to use Google Angle or not on macOS." ON)

#######################################################
# adding raylib
# Note: raylib is bundled in this project but could
#       easily be fetched instead
#######################################################
#option(CUSTOMIZE_BUILD "Show options for customizing your Raylib library build." ON)
if (APPLE AND USE_GOOGLE_ANGLE_MAC_OS)
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
      MACOSX_BUNDLE_GUI_IDENTIFIER com.acme.raylib_cmake
      MACOSX_BUNDLE_BUNDLE_NAME "RayLib CMake"
      MACOSX_BUNDLE_BUNDLE_VERSION "1.0.0"
      MACOSX_BUNDLE_SHORT_VERSION_STRING "1.0.0"
      MACOSX_BUNDLE_ICON_NAME "AppIcon"
      MACOSX_BUNDLE_COPYRIGHT "© 2023 acme"
      MACOSX_BUNDLE_INFO_PLIST "${CMAKE_SOURCE_DIR}/cmake/MacOSXBundleInfo.plist.in"
      )

  target_link_libraries(${target} PRIVATE "-framework IOKit" "-framework Cocoa" "-framework OpenGL")

  if (USE_GOOGLE_ANGLE_MAC_OS)
    # Where are the angle libraries copied (under the build folder)
    set(GOOGLE_ANGLE_DIR "${CMAKE_CURRENT_BINARY_DIR}/angle/libs")

    # The code assumes Google Chrome is installed in the macOS standard location but can be overriden here
    set(GOOGLE_CHROME_INSTALL_DIR "/Applications/Google Chrome.app" CACHE PATH "Path to Google Chrome")

    macro(find_angle_lib name angle_libs)
      # Locate the library
      file(GLOB_RECURSE libs "${GOOGLE_CHROME_INSTALL_DIR}/Contents/Frameworks/*/${name}")
      if(NOT libs)
        message(FATAL_ERROR "Could not locate ${name}. Are you sure that Google Chrome is installed properly? You can set variable GOOGLE_CHROME_INSTALL_DIR to a different location if required.")
      endif()
      # There may be more than one, pick the latest one
      list(GET libs -1 lib)
      message(STATUS "Found ${name} under ${lib}")
      set(processed_lib "${GOOGLE_ANGLE_DIR}/${name}")
      # process the library
      add_custom_command(
          OUTPUT "${processed_lib}"
          COMMAND ${CMAKE_COMMAND} -E make_directory "${GOOGLE_ANGLE_DIR}"
          COMMAND ${CMAKE_COMMAND} -E copy "${lib}" "${processed_lib}"       # copy the library unde the build folder
          COMMAND install_name_tool -id "@rpath/${name}" "${processed_lib}"  # change the rpath (note that this invalidates the signature)
          COMMAND codesign -s "-" -fv "${processed_lib}"                     # remove the signature
          DEPENDS "${lib}"
      )
      list(APPEND ${angle_libs} "${processed_lib}")
    endmacro()

    set(google_angle_libs "")
    find_angle_lib(libEGL.dylib google_angle_libs)    # libEGL.dylib
    find_angle_lib(libGLESv2.dylib google_angle_libs) # libGLESv2.dylib
    add_custom_target(process_angle_libs DEPENDS "${google_angle_libs}")
    add_dependencies(${target} process_angle_libs)

    set_target_properties(${target} PROPERTIES
        BUILD_RPATH "${GOOGLE_ANGLE_DIR}"               # for debug runs, the rpath is under the build folder
        INSTALL_RPATH "@executable_path/../Frameworks"  # for install, the rpath is relative to the executable (inside the macOS bundle)
        )

    target_link_libraries(${target} PRIVATE "${google_angle_libs}")
    target_compile_definitions(${target} PRIVATE "USE_GOOGLE_ANGLE_MAC_OS")

  endif()

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
  if (USE_GOOGLE_ANGLE_MAC_OS)
    # installs the libraries under Content/Frameworks
    install(
        FILES "${google_angle_libs}"
        DESTINATION "$<TARGET_BUNDLE_DIR_NAME:${target}>/Contents/Frameworks"
    )
    # TODO: if using the Xcode generator, the executable needs to be resigned...
  endif()
endif()
