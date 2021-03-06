project(vobsub2srt)

cmake_minimum_required(VERSION 2.6.4 FATAL_ERROR)

set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/CMakeModules)

if(NOT CMAKE_BUILD_TYPE)
    set(
        CMAKE_BUILD_TYPE
        Debug
        CACHE
        STRING
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
        FORCE)
endif()

message(STATUS "Source: ${CMAKE_SOURCE_DIR}")
message(STATUS "Binary: ${CMAKE_BINARY_DIR}")
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")

if(${CMAKE_BINARY_DIR} STREQUAL ${CMAKE_SOURCE_DIR})
    message(FATAL_ERROR "In-source builds are not permitted. Make a separate folder for building:\nmkdir build; cd build; cmake ..\nBefore that, remove the files that cmake just created:\nrm -rf CMakeCache.txt CMakeFiles")
endif()

set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)

set(INSTALL_EXECUTABLES_PATH ${CMAKE_INSTALL_PREFIX}/bin)
set(INSTALL_LIBRARIES_PATH ${CMAKE_INSTALL_PREFIX}/lib)
set(INSTALL_HEADERS_PATH ${CMAKE_INSTALL_PREFIX}/include)
set(INSTALL_ETC_PATH ${CMAKE_INSTALL_PREFIX}/etc)
set(INSTALL_PC_PATH ${CMAKE_INSTALL_PREFIX}/lib/pkgconfig)

add_definitions("-DINSTALL_PREFIX=\"${CMAKE_INSTALL_PREFIX}\"")

include(CheckIncludeFile)
include(CheckCCompilerFlag)
include(CheckCSourceCompiles)
include(CheckCSourceRuns)
include(CheckCXXCompilerFlag)
include(CheckCXXSourceCompiles)
include(CheckCXXSourceRuns)

set(CMAKE_C_FLAGS "-std=gnu99")
set(CMAKE_CXX_FLAGS "-ansi -pedantic-errors -Wall -Wextra")

set(CMAKE_CXX_FLAGS_RELEASE "-O3 -mtune=native -march=native -DNDEBUG -fomit-frame-pointer -ffast-math") # TODO -Ofast GCC 4.6
set(CMAKE_C_FLAGS_RELEASE ${CMAKE_CXX_FLAGS_RELEASE})
set(CMAKE_CXX_FLAGS_DEBUG "-g3 -DDEBUG")
set(CMAKE_C_FLAGS_DEBUG ${CMAKE_CXX_FLAGS_DEBUG})
# TODO RelWithDebInfo MinSizeRel? maybe move -march=native -ffast-math etc. to a different build type

find_package(Libavutil)
find_package(Tesseract)

add_subdirectory(mplayer)
add_subdirectory(src)
add_subdirectory(doc)

#### Package Generation
execute_process (
  COMMAND          /usr/bin/dpkg --print-architecture
  OUTPUT_VARIABLE  CPACK_DEBIAN_PACKAGE_ARCHITECTURE
  RESULT_VARIABLE  EXECUTE_RESULT
  OUTPUT_STRIP_TRAILING_WHITESPACE
  ERROR_QUIET
)
if(EXECUTE_RESULT)
  message(STATUS "dpkg not found: No package generation.")
else()
  message(STATUS "Debian architecture: ${CPACK_DEBIAN_PACKAGE_ARCHITECTURE}")
  set(CPACK_GENERATOR "DEB")
  set(CPACK_PACKAGE_NAME "vobsub2srt")
  set(CPACK_PACKAGE_VERSION "0.0git-current")
  set(CPACK_PACKAGE_CONTACT "Rüdiger Sonderfeld <ruediger@c-plusplus.de>")
  set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Converts VobSub subtitles (.idx/.srt format) into .srt subtitles.")
  set(CPACK_PACKAGE_DESCRIPTION "VobSub2SRT is a simple command line program to convert .idx / .sub subtitles into .srt text subtitles by using OCR. It is based on code from the MPlayer project - a really really great movie player. Tesseract is used as OCR software.")

  set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/COPYING")
  set(CPACK_RESOURCE_FILE_README "${CMAKE_SOURCE_DIR}/README.org")
  set(CPACK_STRIP_FILES TRUE)

  set(CPACK_DEBIAN_PACKAGE_SECTION "video")
  set(CPACK_DEBIAN_PACKAGE_DEPENDS "tesseract-ocr")

  include(CPack)
endif()
