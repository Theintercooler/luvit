cmake_minimum_required(VERSION 2.8)
project (luvit C)

set(HAVE_HTTP_PARSER DEFINED HTTP_PARSER_INCLUDE_DIRS)
set(HAVE_YAJL DEFINED YAJL_INCLUDE_DIRS)
set(HAVE_ODBXUV_LUA DEFINED ODBXUVLUA_INCLUDE_DIRS)

if(NOT DEFINED LUAJIT_INCLUDE_DIRS)
    find_package(PkgConfig)
    message(STATUS "Detecting LuaJIT...")
    if(PKG_CONFIG_FOUND)
        message(STATUS "Using pkg-config to detect LuaJIT...")
        pkg_check_modules(LUAJIT luajit)
        if(LUAJIT_FOUND)
            message(STATUS "Found LuaJIT.")
            message(STATUS "include: ${LUAJIT_INCLUDE_DIRS}")
            include_directories(${LUAJIT_INCLUDE_DIRS})
            link_directories(${LUAJIT_LIBRARY_DIRS})
            set(EXTRA_LIBS ${LUAJIT_LIBRARIES})
        else()
            message(FATAL_ERROR "LuaJIT not found.")
        endif()
    else()
        message(STATUS "Using local LuaJIT.")
        set(LUAJIT_LIBRARIES luajit)
    endif()
endif()

option(HAVE_LOCAL_LIBUV 0)
if(${HAVE_LOCAL_LIBUV})
    add_subdirectory(libraries/uv)
endif()

set(LUVIT_TEMP_DIR ${CMAKE_BINARY_DIR}/luvit)
file(MAKE_DIRECTORY ${LUVIT_TEMP_DIR})

file(MAKE_DIRECTORY ${LUVIT_TEMP_DIR}/include)
file(MAKE_DIRECTORY ${LUVIT_TEMP_DIR}/include/luvit)

FILE(GLOB_RECURSE HEADER_FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h")
foreach(HEADER_FILE ${HEADER_FILES})
    FILE(COPY "${HEADER_FILE}" DESTINATION "${LUVIT_TEMP_DIR}/include/luvit/")
endforeach()

set(LUVIT_INCLUDE_DIRS
    ${LUVIT_TEMP_DIR}/include
    ${HTTP_PARSER_INCLUDE_DIRS}
    ${ARES_INCLUDE_DIRS}
    ${UV_INCLUDE_DIRS}
    ${LUAJIT_INCLUDE_DIRS}
    ${YAJL_INCLUDE_DIRS}
    CACHE INTERNAL "Luvit include directories"
)

set(LUVIT_LIBRARY luvit CACHE INTERNAL "Luvit library")

include_directories(
    ${LUVIT_INCLUDE_DIRS}
    ${CMAKE_CURRENT_SOURCE_DIR}/src
)

set(LUVIT_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lconstants.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lenv.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/los.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_debug.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_dns.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_fs.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_fs_watcher.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_handle.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luvit_exports.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luvit_init.c
#   ${CMAKE_CURRENT_SOURCE_DIR}/src/luvit_main.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_misc.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_pipe.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_process.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_stream.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_signal.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_tcp.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_timer.c
#   ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_tls.c
#   ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_tls_conn.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_tty.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_udp.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/luv_zlib.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/utils.c)

if(${HAVE_HTTP_PARSER})
    message("-- luvit: detected http-parser")
    add_definitions(-DHAVE_HTTP_PARSER)
    set(LUVIT_SOURCES ${LUVIT_SOURCES} ${CMAKE_CURRENT_SOURCE_DIR}/src/lhttp_parser.c)
endif()

if(${HAVE_YAJL})
    message("-- luvit: detected YAJL")
    add_definitions(-DHAVE_YAJL)
    set(LUVIT_SOURCES ${LUVIT_SOURCES} ${CMAKE_CURRENT_SOURCE_DIR}/src/lyajl.c)
endif()

if(${HAVE_ODBXUV_LUA})
    message("-- luvit: detected ODBX-uv-lua")
    add_definitions(-DHAVE_ODBXUV_LUA)
endif()
    

add_library(${LUVIT_LIBRARY} STATIC ${LUVIT_SOURCES})
	

target_link_libraries(${LUVIT_LIBRARY}
    ${UV_LIBRARIES}
    ${ZLIB_LIBRARIES}
    ${HTTP_PARSER_LIBRARIES}
    ${YAJL_LIBRARIES}
    ${ODBXUVLUA_LIBRARIES}
)

set(LUVIT_LIBRARIES
	${LUVIT_LIBRARY}
	CACHE INTERNAL "Luvit libraries")

if(DEFINED INSTALL_LUA_DIR)
    install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib/luvit
        DESTINATION ${INSTALL_LUA_DIR}
        PATTERN "*")
endif()

