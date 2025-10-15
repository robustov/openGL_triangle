cmake_minimum_required(VERSION 3.14)
project(triangle_renderer VERSION 0.1 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

option(USE_GLFW_FETCHCONTENT "Download and build GLFW via FetchContent" ON)
option(USE_GLAD_FETCHCONTENT "Download and build GLAD via FetchContent" ON)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include(FetchContent)

if(USE_GLFW_FETCHCONTENT)
  message(STATUS "FetchContent: GLFW (will be downloaded and built)")
  FetchContent_Declare(
    glfw
    GIT_REPOSITORY https://github.com/glfw/glfw.git
    GIT_TAG latest
  )
  FetchContent_MakeAvailable(glfw)
else()
  find_package(PkgConfig REQUIRED)
  pkg_check_modules(GLFW3 QUIET glfw3)
  if(GLFW3_FOUND)
    message(STATUS "Using system glfw3 (found via pkg-config)")
    include_directories(${GLFW3_INCLUDE_DIRS})
    link_libraries(${GLFW3_LIBRARIES})
  else()
    message(FATAL_ERROR "GLFW not found. Either enable USE_GLFW_FETCHCONTENT or install libglfw3-dev.")
  endif()
endif()

if(USE_GLAD_FETCHCONTENT)
  message(STATUS "FetchContent: GLAD (will be downloaded and built)")
  FetchContent_Declare(
    glad
    GIT_REPOSITORY https://github.com/Dav1dde/glad.git
    GIT_TAG master
  )
  FetchContent_MakeAvailable(glad)
endif()

find_package(OpenGL REQUIRED)
if(NOT OpenGL_FOUND)
  message(FATAL_ERROR "OpenGL not found on this system.")
endif()

set(SRC
  src/main.cpp
)

add_executable(triangle ${SRC})

target_include_directories(triangle PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)

target_link_libraries(triangle PRIVATE OpenGL::GL)

if(TARGET glfw)
  target_link_libraries(triangle PRIVATE glfw)
endif()

if(TARGET glad)
  target_link_libraries(triangle PRIVATE glad)
endif()


