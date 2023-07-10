cmake_minimum_required(VERSION 3.19)

project(imgui)

set(imgui_CPP_SRC_DIR "${CMAKE_CURRENT_LIST_DIR}")

set(imgui_BUILD_SOURCES
    "${imgui_CPP_SRC_DIR}/imgui.cpp"
    "${imgui_CPP_SRC_DIR}/imgui_demo.cpp"
    "${imgui_CPP_SRC_DIR}/imgui_draw.cpp"
    "${imgui_CPP_SRC_DIR}/imgui_tables.cpp"
    "${imgui_CPP_SRC_DIR}/imgui_widgets.cpp"
    "${imgui_CPP_SRC_DIR}/misc/cpp/imgui_stdlib.cpp"
    )

add_library(imgui STATIC "${imgui_BUILD_SOURCES}")
target_compile_definitions(imgui PUBLIC IMGUI_USER_CONFIG="imgui_user_config.h")
target_compile_definitions(imgui PRIVATE $<$<CONFIG:Release>:IMGUI_DISABLE_DEMO_WINDOWS> $<$<CONFIG:Release>:IMGUI_DISABLE_DEBUG_TOOLS>)
target_include_directories(imgui PUBLIC "${imgui_CPP_SRC_DIR}")

add_executable("binary_to_compressed_c" "${imgui_CPP_SRC_DIR}/misc/fonts/binary_to_compressed_c.cpp")