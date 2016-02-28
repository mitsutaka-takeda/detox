project(detox)

cmake_minimum_required(VERSION 3.3)

option(ENABLE_INCLUDE_WHAT_YOU_USE
  "Enable include-what-you-use. Use include-what-you-use to sanitize header-include dependencies.\
Using include-what-you-use can slow the compilation, so if the compilation is slow turn this off."
  ON)
if(ENABLE_INCLUDE_WHAT_YOU_USE)
  find_program(INCLUDE_WHAT_YOU_USE include-what-you-use)
    if(INCLUDE_WHAT_YOU_USE)
      message(STATUS "Enable Include What You Use")
      set(CMAKE_CXX_INCLUDE_WHAT_YOU_USE include-what-you-use)
    else()
      message(WARNING "Could not find includ-what-you-use")
    endif()
endif()

add_executable(
  detox 
  main.cpp
  my_header.cpp
  my_header.h
  )
target_compile_options(
  detox
  PRIVATE
  $<$<CXX_COMPILER_ID:Clang>:-Wall -Weverything -Wno-c++98-compat -Wno-c++98-compat-pedantic>
  $<$<CXX_COMPILER_ID:GCC>:-Wpedantic -Wall -Wextra -Wcast-align -Wcast-qual -Wctor-dtor-privacy -Wdisabled-optimization -Wformat=2 -Winit-self -Wlogical-op -Wmissing-declarations -Wmissing-include-dirs -Wnoexcept -Wold-style-cast -Woverloaded-virtual -Wredundant-decls -Wshadow -Wsign-conversion -Wsign-promo -Wstrict-null-sentinel -Wstrict-overflow=5 -Wswitch-default -Wundef -Werror -Wno-unused>
  $<$<CXX_COMPILER_ID:MSVC>:/W4>
  )

option(FORMAT_FILES_WITH_CLANG_FORMAT_BEFORE_EACH_BUILD
  "If the command clang-format is avilable, format source files before each build.\
Turn this off if the build time is too slow."
  ON)
find_program(CLANG_FORMAT clang-format)
if(CLANG_FORMAT)
  message(STATUS "Enable Clang-Format")
  # detoxのSOURCES属性からソース・ファイルの一覧を取得。
  get_target_property(MY_SOURCES detox SOURCES)
  add_custom_target(
    format-with-clang-format
    COMMAND clang-format -i -style=file ${MY_SOURCES}
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    )
  if(FORMAT_FILES_WITH_CLANG_FORMAT_BEFORE_EACH_BUILD)
    # detoxをビルドする前にclang-formatを実行する。
    add_dependencies(detox format-with-clang-format)
  endif()
endif()
