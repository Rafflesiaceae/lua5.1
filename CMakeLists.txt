cmake_minimum_required(VERSION 3.1...4.0)
project(Lua VERSION 5.1.5 LANGUAGES C)

# Keep the library separate from the interpreter and bytecode compiler.
# Omitting the library type lets the parent choose via BUILD_SHARED_LIBS.
add_library(
    lua
    src/lapi.c
    src/lcode.c
    src/ldebug.c
    src/ldo.c
    src/ldump.c
    src/lfunc.c
    src/lgc.c
    src/llex.c
    src/lmem.c
    src/lobject.c
    src/lopcodes.c
    src/lparser.c
    src/lstate.c
    src/lstring.c
    src/ltable.c
    src/ltm.c
    src/lundump.c
    src/lvm.c
    src/lzio.c
    src/lauxlib.c
    src/lbaselib.c
    src/ldblib.c
    src/liolib.c
    src/lmathlib.c
    src/loslib.c
    src/ltablib.c
    src/lstrlib.c
    src/loadlib.c
    src/linit.c
)
add_library(Lua::Lua ALIAS lua)

# Lua uses ANSI C89/C90. Keep the language settings local to this target;
# the parent can override these properties for its toolchain if needed.
set_target_properties(
    lua
    PROPERTIES C_STANDARD 90 C_STANDARD_REQUIRED ON C_EXTENSIONS OFF
)

# Expose the C headers and the upstream C++ wrapper to build-tree consumers.
# Installation layout and export rules belong to the parent project.
target_include_directories(
    lua
    PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>"
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/etc>"
)

# Some platforms provide math in the C runtime; others need a separate libm.
include(CheckLibraryExists)
check_library_exists(m pow "" LUA_HAVE_LIBM)
if(LUA_HAVE_LIBM)
    target_link_libraries(lua PRIVATE m)
endif()

# Lua's headers select dllexport while building and dllimport in consumers.
get_target_property(lua_library_type lua TYPE)
if(WIN32 AND lua_library_type STREQUAL "SHARED_LIBRARY")
    target_compile_definitions(lua PUBLIC LUA_BUILD_AS_DLL)
endif()
