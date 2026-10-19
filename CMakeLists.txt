cmake_minimum_required(VERSION 3.16)
project(boxplorer2 C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

find_program(CCACHE ccache)
if (CCACHE)
    set(CMAKE_C_COMPILER_LAUNCHER   ${CCACHE})
    set(CMAKE_CXX_COMPILER_LAUNCHER ${CCACHE})
endif()

# SDL2 submodule, linked statically
set(SDL_SHARED OFF CACHE BOOL "" FORCE)
set(SDL_STATIC ON  CACHE BOOL "" FORCE)
set(SDL_TEST   OFF CACHE BOOL "" FORCE)
add_subdirectory(SDL2 EXCLUDE_FROM_ALL)

find_package(OpenGL REQUIRED)

add_executable(boxplorer2
    boxplorer2.cc
    camera.cc
    gfx.cc
    glsl.cc
    hmd.cc
    interpolate.cc
    shader.cc
    shader_manager.cc
    shader_procs.cc
    TGA.cc
    uniforms.cc
    utils.cc
    AntTweakBar/src/LoadOGL.cpp
    AntTweakBar/src/LoadOGLCore.cpp
    AntTweakBar/src/TwBar.cpp
    AntTweakBar/src/TwColors.cpp
    AntTweakBar/src/TwEventSDL.c
    AntTweakBar/src/TwEventSDL12.c
    AntTweakBar/src/TwEventSDL13.c
    AntTweakBar/src/TwEventSDL20.c
    AntTweakBar/src/TwFonts.cpp
    AntTweakBar/src/TwMgr.cpp
    AntTweakBar/src/TwOpenGL.cpp
    AntTweakBar/src/TwOpenGLCore.cpp
    AntTweakBar/src/TwPrecomp.cpp
)

target_include_directories(boxplorer2 PRIVATE . include AntTweakBar/include AntTweakBar/src)

# BOX_NO_HMD selects the headset-less HMD stub, so no LibOVR is needed
target_compile_definitions(boxplorer2 PRIVATE TW_STATIC TW_NO_LIB_PRAGMA BOX_NO_HMD)

target_link_libraries(boxplorer2 PRIVATE SDL2::SDL2main SDL2::SDL2-static OpenGL::GL ${CMAKE_DL_LIBS})

if (MSVC)
    # the sources auto-link SDL2.lib and SDL2main.lib by pragma, the targets above provide them
    target_link_options(boxplorer2 PRIVATE /NODEFAULTLIB:SDL2.lib /NODEFAULTLIB:SDL2main.lib)
endif()

# the binary loads setup.inc and the default effect shaders from include/ next to itself
add_custom_command(TARGET boxplorer2 POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/include $<TARGET_FILE_DIR:boxplorer2>/include)
