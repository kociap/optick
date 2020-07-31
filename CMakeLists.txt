# Windows x64 only

cmake_minimum_required(VERSION 3.13)
project(Optick LANGUAGES CXX)

string(FIND "${CMAKE_CXX_COMPILER}" "clang++" OPTICK_COMPILER_CLANGPP)
if(OPTICK_COMPILER_CLANGPP GREATER -1)
  set(OPTICK_COMPILER_CLANGPP ON)
else()
  set(OPTICK_COMPILER_CLANGPP OFF)
endif()

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set(CMAKE_CXX_STANDARD 11)

# Sources
file(GLOB OPTICK_SRC "src/*.*")
source_group("OptickCore" FILES ${OPTICK_SRC})

# Enabled
option(OPTICK_ENABLED "Enable profiling with Optick" ON)
if(NOT OPTICK_ENABLED)
    # add dummy target as a replacement
    add_library(OptickCore STATIC ${OPTICK_SRC})
    target_include_directories(OptickCore PUBLIC "src")
    target_compile_definitions(OptickCore PUBLIC USE_OPTICK=0)
    return()
endif()

# Options
option(OPTICK_USE_VULKAN "Built-in support for Vulkan" OFF)
option(OPTICK_BUILD_GUI_APP "Build Optick gui viewer app" OFF)

# OptickCore
add_library(OptickCore SHARED ${OPTICK_SRC})
target_include_directories(OptickCore PUBLIC "src")
target_compile_definitions(OptickCore PUBLIC OPTICK_EXPORTS=1)
target_compile_definitions(OptickCore PRIVATE _SILENCE_ALL_CXX17_DEPRECATION_WARNINGS _CRT_SECURE_NO_WARNINGS)

# We won't be using dx
target_compile_definitions(OptickCore PRIVATE OPTICK_ENABLE_GPU_D3D12=0)

if(OPTICK_USE_VULKAN)
    target_include_directories(OptickCore PRIVATE "$(VULKAN_SDK)/Include")
    target_compile_definitions(OptickCore PRIVATE OPTICK_ENABLE_GPU_VULKAN=1)
    target_compile_definitions(OptickCore PRIVATE OPTICK_ENABLE_GPU=1)
else()
    target_compile_definitions(OptickCore PRIVATE OPTICK_ENABLE_GPU_VULKAN=0)
    target_compile_definitions(OptickCore PRIVATE OPTICK_ENABLE_GPU=0)
endif()

if(OPTICK_COMPILER_CLANGPP)
    target_compile_options(OptickCore PRIVATE -Wno-gnu-zero-variadic-macro-arguments)
endif()

set(EXTRA_LIBS ${EXTRA_LIBS} OptickCore)

# Gui App
if(MSVC AND OPTICK_BUILD_GUI_APP)
    add_subdirectory(gui)
endif()
