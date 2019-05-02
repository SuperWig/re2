# Copyright 2015 The RE2 Authors.  All Rights Reserved.
# Use of this source code is governed by a BSD-style
# license that can be found in the LICENSE file.

# Old enough to support Ubuntu Xenial.
cmake_minimum_required(VERSION 3.1)

project(RE2 CXX)
include(CTest)

option(BUILD_SHARED_LIBS "build shared libraries" OFF)
option(USEPCRE "use PCRE in tests and benchmarks" OFF)

# CMake seems to have no way to enable/disable testing per subproject,
# so we provide an option similar to BUILD_TESTING, but just for RE2.
option(RE2_BUILD_TESTING "enable testing for RE2" ON)

set(RE2_SOURCES
    re2/bitstate.cc
    re2/compile.cc
    re2/dfa.cc
    re2/filtered_re2.cc
    re2/mimics_pcre.cc
    re2/nfa.cc
    re2/onepass.cc
    re2/parse.cc
    re2/perl_groups.cc
    re2/prefilter.cc
    re2/prefilter_tree.cc
    re2/prog.cc
    re2/re2.cc
    re2/regexp.cc
    re2/set.cc
    re2/simplify.cc
    re2/stringpiece.cc
    re2/tostring.cc
    re2/unicode_casefold.cc
    re2/unicode_groups.cc
    util/rune.cc
    util/strutil.cc
    )

add_library(re2 ${RE2_SOURCES})
add_library(re2::re2 ALIAS re2)

target_include_directories(re2 PUBLIC
  $<INSTALL_INTERFACE:include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>)

target_compile_features(re2
  PUBLIC
    cxx_explicit_conversions cxx_rvalue_references cxx_variadic_templates
  PRIVATE
    cxx_defaulted_functions cxx_right_angle_brackets cxx_static_assert cxx_nonstatic_member_init
)

if(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
  if(MSVC_VERSION LESS 1900)
    message(FATAL_ERROR "you need Visual Studio 2015 or later")
  endif()
  if(BUILD_SHARED_LIBS)
    # See http://www.kitware.com/blog/home/post/939 for details.
    cmake_minimum_required(VERSION 3.4)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
  endif()
  # CMake defaults to /W3, but some users like /W4 (or /Wall) and /WX,
  # so we disable various warnings that aren't particularly helpful.
  target_compile_options(re2 PRIVATE /wd4100 /wd4201 /wd4456 /wd4457 /wd4702 /wd4815)
  # Without a byte order mark (BOM), Visual Studio assumes that the source
  # file is encoded using the current user code page, so we specify UTF-8.
  target_compile_options(re2 PRIVATE /utf-8)
endif()

if(WIN32)
  target_compile_definitions(re2 PRIVATE -DUNICODE -D_UNICODE -DSTRICT -DNOMINMAX)
  target_compile_definitions(re2 PRIVATE -D_CRT_SECURE_NO_WARNINGS -D_SCL_SECURE_NO_WARNINGS)
endif()

if(RE2_BUILD_TESTING)
  add_subdirectory(re2/testing)
endif()

set(RE2_HEADERS
    re2/filtered_re2.h
    re2/re2.h
    re2/set.h
    re2/stringpiece.h
    )

install(FILES ${RE2_HEADERS} DESTINATION include/re2)
install(TARGETS re2 EXPORT re2Config ARCHIVE DESTINATION lib LIBRARY DESTINATION lib RUNTIME DESTINATION bin INCLUDES DESTINATION include)
install(EXPORT re2Config DESTINATION lib/cmake/re2 NAMESPACE re2::)
