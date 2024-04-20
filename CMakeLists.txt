##
## CMake support for libiconv
##

cmake_minimum_required(VERSION 3.0.0)
project(Libiconv VERSION 1.15 )

#
# Options
#
set(BUILD_SHARED_LIBS ON CACHE BOOL "Build shared libraries, Always ON since this code is LGPL" FORCE)

# Config file
configure_file(
  ${Libiconv_SOURCE_DIR}/config.h.in
  ${Libiconv_BINARY_DIR}/config.h
)
include_directories(${Libiconv_SOURCE_DIR} ${Libiconv_BINARY_DIR})

if (CMAKE_SYSTEM_NAME MATCHES "Linux")
elseif (CMAKE_SYSTEM_NAME MATCHES "Windows")
elseif (CMAKE_SYSTEM_NAME MATCHES "FreeBSD")
else ()
    MESSAGE(STATUS "other platform: ${CMAKE_SYSTEM_NAME}")
endif (CMAKE_SYSTEM_NAME MATCHES "Linux")


if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    # using Clang
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    # using GCC
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Intel")
    # using Intel C++
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
    # using Visual Studio C++
endif()

#cmake.exe -G "MinGW Makefiles"


if (CMAKE_SYSTEM_NAME MATCHES "Linux")
    MESSAGE(STATUS "current platform: Linux ")
elseif (CMAKE_SYSTEM_NAME MATCHES "Windows")
    MESSAGE(STATUS "current platform: Windows")
elseif (CMAKE_SYSTEM_NAME MATCHES "Darwin")
    MESSAGE(STATUS "current platform: Mac OS X")
else ()
    MESSAGE(STATUS "other platform: ${CMAKE_SYSTEM_NAME}")
endif (CMAKE_SYSTEM_NAME MATCHES "Linux")

cmake_host_system_information(RESULT PRETTY_NAME QUERY DISTRIB_PRETTY_NAME)
message(STATUS "${PRETTY_NAME}")

cmake_host_system_information(RESULT DISTRO QUERY DISTRIB_INFO)
foreach(VAR IN LISTS DISTRO)
    message(STATUS "${VAR}=`${${VAR}}`")
endforeach()

if(MSVC)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(CMAKE_DEBUG_POSTFIX "d")
    elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
    endif()
    #set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /utf-8")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /fexec-charset=utf-8 /source-charset=utf-8 /execution-charset=utf-8")
    #set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /fexec-charset=utf-8 /source-charset:utf-8 /execution-charset:utf-8")
    #set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -fexec-charset=GBK")
    #set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -fexec-charset=GBK")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fexec-charset=utf-8")
endif()
#link_directories(${CMAKE_BINARY_DIR} )

message("CMAKE_C_COMPILER = ${CMAKE_C_COMPILER}")
message("CMAKE_C_COMPILER_VERSION = ${CMAKE_C_COMPILER_VERSION}")
message("CMAKE_CXX_PLATFORM_ID = ${CMAKE_CXX_PLATFORM_ID}")
message("CMAKE_CXX_COMPILER_TARGET = ${CMAKE_CXX_COMPILER_TARGET}")
message("CMAKE_CXX_COMPILER_AR = ${CMAKE_CXX_COMPILER_AR}")
message("CMAKE_C_COMPILER_ABI = ${CMAKE_C_COMPILER_ABI}")
message("CMAKE_C_COMPILER_TARGET = ${CMAKE_C_COMPILER_TARGET}")
message("CMAKE_HOST_SYSTEM = ${CMAKE_HOST_SYSTEM}")
message("CMAKE_HOST_SYSTEM_NAME = ${CMAKE_HOST_SYSTEM_NAME}")
message("CMAKE_VS_PLATFORM_NAME = ${CMAKE_VS_PLATFORM_NAME}")
message("CMAKE_VS_PLATFORM_TOOLSET = ${CMAKE_VS_PLATFORM_TOOLSET}")
message("CMAKE_CXX_COMPILER_VERSION = ${CMAKE_CXX_COMPILER_VERSION}")
#message("CMAKE_CXX_COMPILER_TARGET= ${CMAKE_CXX_COMPILER_TARGET}")
message("CMAKE_CXX_COMPILER_ABI= ${CMAKE_CXX_COMPILER_ABI}")
message("CMAKE_CXX_COMPILER_EXTERNAL_TOOLCHAIN= ${CMAKE_CXX_COMPILER_EXTERNAL_TOOLCHAIN}")
message("CMAKE_CXX_COMPILER_ID= ${CMAKE_CXX_COMPILER_ID}")
message("CMAKE_CXX_COMPILER_ARCHITECTURE_ID= ${CMAKE_CXX_COMPILER_ARCHITECTURE_ID}")
#message("CMAKE_HOST_SYSTEM_NAME= ${CMAKE_HOST_SYSTEM_NAME}")
message("CMAKE_Fortran_FORMAT= ${CMAKE_Fortran_FORMAT}")
message("CMAKE_Fortran_COMPILER= ${CMAKE_Fortran_COMPILER}")
message("CMAKE_Fortran_COMPILER_VERSION= ${CMAKE_Fortran_COMPILER_VERSION}")
message("CMAKE_Fortran_COMPILER_ID= ${CMAKE_Fortran_COMPILER_ID}")


# 改变安装路径可以使用下面的语句，Unix: "/usr/local", Windows: "c:/Program Files/${PROJECT_NAME}" 为默认值，所有的DESTINATION均以此目录为根目录
set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/../${PROJECT_NAME}_${PROJECT_VERSION}_${CMAKE_CXX_COMPILER_VERSION}_${CMAKE_Fortran_COMPILER_ID}_${CMAKE_Fortran_COMPILER_VERSION}_CXX_${CMAKE_CXX_STANDARD}_${CMAKE_SYSTEM_PROCESSOR}_install)

set(OUTPUT_DIR   ${CMAKE_BINARY_DIR}/../${PROJECT_NAME}_${PROJECT_VERSION}_${CMAKE_CXX_COMPILER_ID}_${CMAKE_CXX_COMPILER_VERSION}_${CMAKE_Fortran_COMPILER_ID}_${CMAKE_Fortran_COMPILER_VERSION}_CXX_${CMAKE_CXX_STANDARD}_${CMAKE_SYSTEM_PROCESSOR}_bin )
#set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${OUTPUT_DIR})
#set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${OUTPUT_DIR})
#set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${OUTPUT_DIR})

# install
include(CMakePackageConfigHelpers)

# aux_source_directory(./  sourceAll )
# file(GLOB_RECURSE  ./*.cpp  all_source_files )

macro(header_injection_hack)
  include(CMakeParseArguments)
  set(options)
  set(oneValueArgs LIBRARY_NAME INPUT OUTPUT)
  set(multiValueArgs)
  cmake_parse_arguments(hack
    "${options}" "${oneValueArgs}" "${multiValueArgs}" ${ARGN}
  )
  string(TOUPPER ${hack_LIBRARY_NAME} hack_library_name_upper)
  set(HAVE_VISIBILITY
  "WIN32
    #if defined(${hack_LIBRARY_NAME}_EXPORTS)
      #define ${hack_library_name_upper}_DLL_EXPORTED __declspec(dllexport)
    #else
      #define ${hack_library_name_upper}_DLL_EXPORTED __declspec(dllimport)
    #endif
  #elif true"
  )
  configure_file(
    ${hack_INPUT}
    ${hack_OUTPUT}
    )
endmacro()
#
# Build libcharset
#
set(libname libcharset)
set(libcharset_source_dir ${Libiconv_SOURCE_DIR}/libcharset)

if(WIN32)
  add_definitions(-DLIBDIR)
endif()

# Hack: to be able to configure the localcharset.h.build.in correctly, we
# inject code in the variable have visibility
header_injection_hack(
  LIBRARY_NAME ${libname}
  INPUT ${Libiconv_SOURCE_DIR}/libcharset/include/localcharset.h.build.in
  OUTPUT ${Libiconv_BINARY_DIR}/localcharset.h
  )

set(${libname}_sources
  ${libcharset_source_dir}/lib/relocatable.c
  ${libcharset_source_dir}/lib/relocatable.h
  ${libcharset_source_dir}/lib/localcharset.c
  ${Libiconv_BINARY_DIR}/localcharset.h
)

add_library(${libname} ${${libname}_sources})
list(APPEND Libiconv_TARGETS ${libname})
list(APPEND Libiconv_headers ${Libiconv_BINARY_DIR}/localcharset.h)

#
# Build libiconv
#
set(libname libiconv)

# Hack: same hack than for libcharset
set(HAVE_WCHAR_T "HAVE_WCHAR_T")
set(USE_MBSTATE_T "USE_MBSTATE_T")
set(BROKEN_WCHAR_H "BROKEN_WCHAR_H")

header_injection_hack(
  LIBRARY_NAME ${libname}
  INPUT ${Libiconv_SOURCE_DIR}/include/iconv.h.build.in
  OUTPUT ${Libiconv_BINARY_DIR}/iconv.h
  )

list(APPEND simple_header_config
  uniwidth
  unitypes
  )
foreach(header ${simple_header_config})
  configure_file(
    ${Libiconv_SOURCE_DIR}/srclib/${header}.in.h
    ${Libiconv_BINARY_DIR}/${header}.h
    )
endforeach()

set(${libname}_sources
  ${Libiconv_SOURCE_DIR}/lib/iconv.c
  ${Libiconv_BINARY_DIR}/iconv.h
)

add_library(${libname} ${${libname}_sources})
target_link_libraries(${libname} libcharset)
list(APPEND Libiconv_TARGETS ${libname})
list(APPEND Libiconv_headers ${Libiconv_BINARY_DIR}/iconv.h)

#
# Export targets
#
install(TARGETS ${Libiconv_TARGETS}
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)

install(FILES ${Libiconv_headers} DESTINATION include)

set(Libiconv_TARGETS_FILE ${Libiconv_BINARY_DIR}/libiconvTargets.cmake)
export(TARGETS ${Libiconv_TARGETS} FILE ${Libiconv_TARGETS_FILE})

configure_file(
  ${Libiconv_SOURCE_DIR}/LibiconvConfig.cmake.in
  ${Libiconv_BINARY_DIR}/libiconvConfig.cmake
  @ONLY
)

#####

configure_file(
  ${Libiconv_SOURCE_DIR}/libcharsetConfig.cmake.in
  ${Libiconv_BINARY_DIR}/libcharsetConfig.cmake
  @ONLY
)

#//--- libcharset ---------------------------------------------
# 写入库的版本信息

write_basic_package_version_file(
    libcharsetConfigVersion.cmake
    VERSION ${PACKAGE_VERSION}
    COMPATIBILITY AnyNewerVersion
)

# 将库文件，可执行文件，头文件安装到指定目录
install(TARGETS libcharset
    EXPORT libcharsetTargets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    PUBLIC_HEADER DESTINATION include
)

# 将库的相关信息输出为MyMathConfig.cmake，供find_package查找
install(EXPORT libcharsetTargets
    FILE libcharsetTargets.cmake
    NAMESPACE libcharset::
    DESTINATION lib/cmake/libcharset
)

#configure_file( iconConfig.cmake.in  icon/iconConfig.cmake @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/libcharsetConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/libcharsetConfigVersion.cmake"
    DESTINATION lib/cmake/libcharset
)


message( "{libiconv_INCLUDE_DIRS} = ${libiconv_INCLUDE_DIRS}"  )
message( "{INTERFACE_INCLUDE_DIRECTORIES} = ${INTERFACE_INCLUDE_DIRECTORIES}"  )




#//--- Libiconv ---------------------------------------------
# 写入库的版本信息

write_basic_package_version_file(
    libiconvConfigVersion.cmake
    VERSION ${PACKAGE_VERSION}
    COMPATIBILITY AnyNewerVersion
)

# 将库文件，可执行文件，头文件安装到指定目录
install(TARGETS libiconv
    EXPORT libiconvTargets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    PUBLIC_HEADER DESTINATION include
)

# 将库的相关信息输出为MyMathConfig.cmake，供find_package查找
install(EXPORT libiconvTargets
    FILE libiconvTargets.cmake
    NAMESPACE libiconv::
    DESTINATION lib/cmake/libiconv
)

#configure_file( iconConfig.cmake.in  icon/iconConfig.cmake @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/libiconvConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/libiconvConfigVersion.cmake"
    DESTINATION lib/cmake/libiconv
)


#message( "{libiconv_INCLUDE_DIRS} = ${libiconv_INCLUDE_DIRS}"  )
#message( "{INTERFACE_INCLUDE_DIRECTORIES} = ${INTERFACE_INCLUDE_DIRECTORIES}"  )



