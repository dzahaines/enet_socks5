cmake_minimum_required(VERSION 3.20)

project(enet_socks5 LANGUAGES C)

include(CheckFunctionExists)
include(CheckStructHasMember)
include(CheckTypeSize)

check_function_exists("fcntl"             HAS_FCNTL)
check_function_exists("poll"              HAS_POLL)
check_function_exists("getaddrinfo"       HAS_GETADDRINFO)
check_function_exists("getnameinfo"       HAS_GETNAMEINFO)
check_function_exists("gethostbyname_r"   HAS_GETHOSTBYNAME_R)
check_function_exists("gethostbyaddr_r"   HAS_GETHOSTBYADDR_R)
check_function_exists("inet_pton"         HAS_INET_PTON)
check_function_exists("inet_ntop"         HAS_INET_NTOP)
check_struct_has_member("struct msghdr" "msg_flags"
                        "sys/types.h;sys/socket.h" HAS_MSGHDR_FLAGS)

set(CMAKE_EXTRA_INCLUDE_FILES "sys/types.h" "sys/socket.h")
check_type_size("socklen_t" HAS_SOCKLEN_T BUILTIN_TYPES_ONLY)
unset(CMAKE_EXTRA_INCLUDE_FILES)

if (MSVC)
    add_compile_options(/W3)
else()
    add_compile_options(-Wno-error)
endif()

set(feature_defs)
foreach(feature
        HAS_FCNTL
        HAS_POLL
        HAS_GETNAMEINFO
        HAS_GETADDRINFO
        HAS_GETHOSTBYNAME_R
        HAS_GETHOSTBYADDR_R
        HAS_INET_PTON
        HAS_INET_NTOP
        HAS_MSGHDR_FLAGS
        HAS_SOCKLEN_T)
    if (${feature})
        list(APPEND feature_defs ${feature}=1)
    endif()
endforeach()

file(GLOB SOURCE_FILES CONFIGURE_DEPENDS
     *.c
     *.cpp
     src/*.c
     src/*.cpp
     src/**/*.c
     src/**/*.cpp)

file(GLOB_RECURSE HEADER_FILES CONFIGURE_DEPENDS
     ${PROJECT_SOURCE_DIR}/include/enet/*.h)

source_group(TREE ${PROJECT_SOURCE_DIR}/include FILES ${HEADER_FILES})
source_group(TREE ${PROJECT_SOURCE_DIR}            FILES ${SOURCE_FILES})

add_library(enet_socks5 SHARED ${SOURCE_FILES} ${HEADER_FILES})

target_include_directories(enet_socks5
    PUBLIC ${PROJECT_SOURCE_DIR}/include)

target_compile_definitions(enet_socks5
    PRIVATE ${feature_defs})

if (WIN32)
    target_compile_definitions(enet_socks5
        PRIVATE ENET_DLL=1 ENET_BUILDING_LIB)
    target_link_libraries(enet_socks5
        PRIVATE winmm ws2_32)
endif()

add_library(enet ALIAS enet_socks5)

include(GNUInstallDirs)

install(TARGETS enet_socks5
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/enet
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
