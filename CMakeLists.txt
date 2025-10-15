cmake_minimum_required(VERSION 3.14)
project(triangle VERSION 0.1 LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

option(USE_GLAD_FETCHCONTENT "Download and build GLAD via FetchContent" ON)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(SOURCE_DIR "${CMAKE_SOURCE_DIR}/src")
set(INCLUDE_DIR "${CMAKE_SOURCE_DIR}/include")

find_package(SDL2 REQUIRED)
if(NOT SDL2_FOUND)
  message(FATAL_ERROR "SDL2 not found. Install libsdl2-dev or equivalent.")
endif()
include_directories(${SDL2_INCLUDE_DIRS})

find_package(OpenGL REQUIRED)
if(NOT OpenGL_FOUND)
  message(FATAL_ERROR "OpenGL not found on this system.")
endif()

find_package(PkgConfig REQUIRED)
pkg_check_modules(WAYLAND QUIET wayland-client wayland-cursor xkbcommon)
pkg_check_modules(LIBDECOR QUIET libdecor-0)

if(WAYLAND_FOUND AND LIBDECOR_FOUND)
  message(STATUS "Wayland and libdecor found - building with Wayland support")
  add_compile_definitions(USE_WAYLAND=1)
else()
  message(STATUS "Wayland/libdecor not found - building with X11 support")
  add_compile_definitions(USE_WAYLAND=0)
endif()

if(USE_GLAD_FETCHCONTENT)
  message(STATUS "FetchContent: GLAD (will be downloaded and built)")
  include(FetchContent)
  FetchContent_Declare(
    glad
    GIT_REPOSITORY https://github.com/Dav1dde/glad.git
    GIT_TAG master
  )
  FetchContent_MakeAvailable(glad)
endif()

set(SOURCES
  ${SOURCE_DIR}/main.cpp
)

add_executable(${PROJECT_NAME} ${SOURCES})

target_include_directories(${PROJECT_NAME} PRIVATE ${INCLUDE_DIR})

target_link_libraries(${PROJECT_NAME} PRIVATE OpenGL::GL ${SDL2_LIBRARIES})

if(TARGET glad)
  target_link_libraries(${PROJECT_NAME} PRIVATE glad)
endif()

if(WAYLAND_FOUND)
  target_link_libraries(${PROJECT_NAME} PRIVATE ${WAYLAND_LIBRARIES})
endif()

if(LIBDECOR_FOUND)
  target_link_libraries(${PROJECT_NAME} PRIVATE ${LIBDECOR_LIBRARIES})
endif()
