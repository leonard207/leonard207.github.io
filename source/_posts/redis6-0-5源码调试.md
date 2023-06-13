---
title: redis6.0.5源码调试
date: 2020-07-15 14:40:30
tags: redis
---

环境：Windows 10 + WSL 2 + Clion

在WSL 2中安装Ubuntu

在WSL 2 中的Ubuntu下执行

https://github.com/JetBrains/clion-wsl/blob/master/ubuntu_setup_env.sh


~~~shell
git clone -b 6.0 https://github.com/antirez/redis.git
~~~


## 四、修改CMakeLists.txt文件

我们需要修改5个CMakeLists.txt文件

~~~shell
./CMakeLists.txt
./deps/CMakeLists.txt
./deps/linenoise/CMakeLists.txt
./deps/lua/CMakeLists.txt
./src/modules/CMakeLists.txt
~~~



./CMakeLists.txt修改如下：

~~~shell
cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
project(redis VERSION 6.0)

if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type defined; defaulting to 'Debug'")
    set(CMAKE_BUILD_TYPE "Debug" CACHE STRING
            "The type of build. Possible values are: Debug, Release, RelWithDebInfo and MinSizeRel.")
endif()

message(STATUS "Host is: ${CMAKE_HOST_SYSTEM}.  Build target is: ${CMAKE_SYSTEM}")
get_filename_component(REDIS_ROOT "${CMAKE_CURRENT_SOURCE_DIR}" ABSOLUTE)
message(STATUS "Project root directory is: ${REDIS_ROOT}")

# Just for debugging when handling a new platform.
if (false)
    message("C++ compiler supports these language features:")
    foreach(i ${CMAKE_CXX_COMPILE_FEATURES})
        message("  ${i}")
    endforeach()
endif()

message(STATUS "Generating release.h...")
execute_process(
        COMMAND sh -c ./mkreleasehdr.sh
        WORKING_DIRECTORY ${REDIS_ROOT}/src/
)

add_subdirectory(deps)
add_subdirectory(src/modules)

set(SRC_SERVER_TMP
        src/crcspeed.c
        src/crcspeed.h
        src/sha256.c
        src/sha256.h
        src/connection.c
        src/connection.h
        src/acl.c
        src/timeout.c
        src/tracking.c
        src/tls.c
        src/adlist.c
        src/ae.c
        src/anet.c
        src/dict.c
        src/sds.c
        src/zmalloc.c
        src/lzf_c.c
        src/lzf_d.c
        src/pqsort.c
        src/zipmap.c
        src/sha1.c
        src/ziplist.c
        src/release.c
        src/networking.c
        src/util.c
        src/object.c
        src/db.c
        src/replication.c
        src/rdb.c
        src/t_string.c
        src/t_list.c
        src/t_set.c
        src/t_zset.c
        src/evict.c
        src/defrag.c
        src/module.c
        src/quicklist.c
        src/expire.c
        src/childinfo.c
        src/redis-check-aof.c
        src/redis-check-rdb.c
        src/lazyfree.c
        src/geohash.c
        src/rax.c
        src/geohash_helper.c
        src/siphash.c
        src/geo.c
        src/t_hash.c
        src/config.c
        src/aof.c
        src/pubsub.c
        src/multi.c
        src/debug.c
        src/sort.c
        src/intset.c
        src/syncio.c
        src/cluster.c
        src/crc16.c
        src/endianconv.c
        src/slowlog.c
        src/scripting.c
        src/bio.c
        src/rio.c
        src/rand.c
        src/memtest.c
        src/crc64.c
        src/bitops.c
        src/sentinel.c
        src/notify.c
        src/setproctitle.c
        src/blocked.c
        src/hyperloglog.c
        src/latency.c
        src/sparkline.c
        src/t_stream.c
        src/lolwut.c
        src/lolwut.h
        src/lolwut5.c
        src/lolwut6.c
        src/listpack.c
        src/localtime.c
        src/gopher.c
        )

set(SRC_SERVER src/server.c ${SRC_SERVER_TMP})

set(SRC_CLI
        src/anet.c src/sds.c src/adlist.c src/redis-cli.c src/zmalloc.c src/release.c src/anet.c src/ae.c src/crc64.c
        )
if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    # better not to work with jemalloc
endif()

add_executable(redis-server ${SRC_SERVER})
add_executable(redis-cli ${SRC_CLI})

set_property(TARGET redis-server PROPERTY C_STANDARD 99)
set_property(TARGET redis-server PROPERTY CXX_STANDARD 11)
set_property(TARGET redis-server PROPERTY CXX_STANDARD_REQUIRED ON)

set_property(TARGET redis-cli PROPERTY C_STANDARD 99)
set_property(TARGET redis-cli PROPERTY CXX_STANDARD 11)
set_property(TARGET redis-cli PROPERTY CXX_STANDARD_REQUIRED ON)

target_include_directories(redis-server
        PRIVATE ${REDIS_ROOT}/deps/hiredis
        PRIVATE ${REDIS_ROOT}/deps/linenoise
        PRIVATE ${REDIS_ROOT}/deps/lua/src
        )

target_include_directories(redis-cli
        PRIVATE ${REDIS_ROOT}/deps/hiredis
        PRIVATE ${REDIS_ROOT}/deps/linenoise
        PRIVATE ${REDIS_ROOT}/deps/lua/src
        )

target_link_libraries(redis-server
        PRIVATE pthread
        PRIVATE m
        PRIVATE lua
        PRIVATE linenoise
        PRIVATE hiredis
        )

target_link_libraries(redis-cli
        PRIVATE pthread
        PRIVATE m
        PRIVATE linenoise
        PRIVATE hiredis
        )

link_directories(deps/hiredis/ deps/linenoise/ diredeps/lua/src)
~~~



./deps/CMakeLists.txt修改如下：

~~~shell
add_subdirectory(hiredis)
add_subdirectory(linenoise)
add_subdirectory(lua)
~~~



./deps/linenoise/CMakeLists.txt修改如下：

~~~shell
add_library(linenoise linenoise.c)
~~~



./deps/lua/CMakeLists.txt修改如下：

~~~shell
set(LUA_SRC
        src/lauxlib.c
        src/liolib.c
        src/lopcodes.c
        src/lstate.c
        src/lobject.c
        src/print.c
        src/lmathlib.c
        src/loadlib.c
        src/lvm.c
        src/lfunc.c
        src/lstrlib.c
        src/lua.c
        src/linit.c
        src/lstring.c
        src/lundump.c
        src/luac.c
        src/ltable.c
        src/ldump.c
        src/loslib.c
        src/lgc.c
        src/lzio.c
        src/ldblib.c
        src/strbuf.c
        src/lmem.c
        src/lcode.c
        src/ltablib.c
        src/lua_struct.c
        src/lapi.c
        src/lbaselib.c
        src/lua_cmsgpack.c
        src/ldebug.c
        src/lparser.c
        src/lua_cjson.c
        src/fpconv.c
        src/lua_bit.c
        src/llex.c
        src/ltm.c
        src/ldo.c
        )

add_library(lua STATIC ${LUA_SRC})
~~~



./src/modules/CMakeLists.txt修改如下：

~~~shell
cmake_minimum_required(VERSION 3.9)
set(CMAKE_BUILD_TYPE "Debug")
add_library(helloworld SHARED helloworld.c)
set_target_properties(helloworld PROPERTIES PREFIX "" SUFFIX ".so")


add_library(hellotype SHARED hellotype.c)
set_target_properties(hellotype PROPERTIES PREFIX "" SUFFIX ".so")


add_library(helloblock SHARED helloblock.c)
set_target_properties(helloblock PROPERTIES PREFIX "" SUFFIX ".so")


add_library(testmodule SHARED testmodule.c)
set_target_properties(testmodule PROPERTIES PREFIX "" SUFFIX ".so")
~~~






