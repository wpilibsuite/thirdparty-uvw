# Disable in-source builds to prevent source tree corruption.
if(" ${CMAKE_SOURCE_DIR}" STREQUAL " ${CMAKE_BINARY_DIR}")
  message(FATAL_ERROR "
FATAL: In-source builds are not allowed.
       You should create a separate directory for build files.
")
endif()

project(thirdparty-uvw)
cmake_minimum_required(VERSION 3.3.0)

INCLUDE(CPack)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT AND MSVC)
    set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install" CACHE PATH "Default install dir on windows" FORCE)
endif()

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

SET(CMAKE_SKIP_BUILD_RPATH  FALSE)

# when building, don't use the install RPATH already
# (but later on when installing)
SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)

SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/thirdparty-uvw/lib")

# add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

# the RPATH to be used when installing, but only if it's not a system directory
LIST(FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES "${CMAKE_INSTALL_PREFIX}/thirdparty-uvw/lib" isSystemDir)
IF("${isSystemDir}" STREQUAL "-1")
   SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/thirdparty-uvw/lib")
ENDIF("${isSystemDir}" STREQUAL "-1")

set ( thirdparty_dest thirdparty-uvw )
set ( include_dest thirdparty-uvw/include )
set ( main_lib_dest thirdparty-uvw/lib )

if (MSVC)
    set (thirdparty_uvw_config_dir ${thirdparty_dest})
else()
    set (thirdparty_uvw_config_dir share/thirdparty-uvw)
endif()

option(BUILD_SHARED_LIBS "build with shared libs" ON)

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

file(GLOB uv_native_src libuv/src/*.c)

file(GLOB uv_windows_src libuv/src/win/*.c)

set(uv_unix_src
    libuv/src/unix/async.c
    libuv/src/unix/atomic-ops.h
    libuv/src/unix/core.c
    libuv/src/unix/dl.c
    libuv/src/unix/fs.c
    libuv/src/unix/getaddrinfo.c
    libuv/src/unix/getnameinfo.c
    libuv/src/unix/internal.h
    libuv/src/unix/loop-watcher.c
    libuv/src/unix/loop.c
    libuv/src/unix/pipe.c
    libuv/src/unix/poll.c
    libuv/src/unix/process.c
    libuv/src/unix/signal.c
    libuv/src/unix/spinlock.h
    libuv/src/unix/stream.c
    libuv/src/unix/tcp.c
    libuv/src/unix/thread.c
    libuv/src/unix/timer.c
    libuv/src/unix/tty.c
    libuv/src/unix/udp.c
)

set(uv_darwin_src
    libuv/src/unix/bsd-ifaddrs.c
    libuv/src/unix/darwin.c
    libuv/src/unix/darwin-proctitle.c
    libuv/src/unix/fsevents.c
    libuv/src/unix/kqueue.c
    libuv/src/unix/proctitle.c
)

set(uv_linux_src
    libuv/src/unix/linux-core.c
    libuv/src/unix/linux-inotify.c
    libuv/src/unix/linux-syscalls.c
    libuv/src/unix/linux-syscalls.h
    libuv/src/unix/procfs-exepath.c
    libuv/src/unix/proctitle.c
    libuv/src/unix/sysinfo-loadavg.c
    libuv/src/unix/sysinfo-memory.c
)

add_library(uv ${uv_native_src})

set_property(TARGET uv PROPERTY FOLDER "libraries")

if(NOT MSVC)
    target_sources(uv PRIVATE ${uv_unix_src})
    target_sources(uv PRIVATE ${uv_linux_src})
    target_compile_options(uv PUBLIC -Wall -pedantic -Wextra -Wno-unused-parameter)
    target_compile_options(uv PRIVATE -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64 -D_GNU_SOURCE)
else()
    target_sources(uv PRIVATE ${uv_windows_src})
    target_include_directories(uv PRIVATE include)
    target_compile_options(uv PUBLIC -DNOMINMAX)
    target_compile_options(uv PRIVATE -D_CRT_SECURE_NO_WARNINGS /FIpragmas.h)
    if(BUILD_SHARED_LIBS)
        target_compile_options(uv PRIVATE -DBUILDING_UV_SHARED)
    endif()
endif()

target_link_libraries(uv Threads::Threads)
target_include_directories(uv PRIVATE libuv/src)

target_include_directories(uv PUBLIC
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/libuv/include>
                            $<INSTALL_INTERFACE:${include_dest}/uv>)

add_library(uvw INTERFACE)

if(NOT MSVC)
    target_compile_options(uvw INTERFACE -std=c++14 -Wall -pedantic -Wextra -Wno-unused-parameter)
else()
    target_compile_options(uvw INTERFACE -DNOMINMAX)
endif()

target_link_libraries(uvw INTERFACE uv Threads::Threads)

target_include_directories(uvw INTERFACE
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/uvw/src>
                            $<INSTALL_INTERFACE:${include_dest}/uvw>)

install(TARGETS uv EXPORT uv DESTINATION "${main_lib_dest}")
install(TARGETS uvw EXPORT uvw DESTINATION "${main_lib_dest}")
install(DIRECTORY uvw/src/ DESTINATION "${include_dest}/uvw")
install(DIRECTORY libuv/include/ DESTINATION "${include_dest}/uv")

install(FILES thirdparty-uvw-config.cmake DESTINATION ${thirdparty_uvw_config_dir})
install(EXPORT uv DESTINATION ${thirdparty_uvw_config_dir})
install(EXPORT uvw DESTINATION ${thirdparty_uvw_config_dir})
