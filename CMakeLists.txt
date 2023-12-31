cmake_minimum_required(VERSION 3.16)

file(GLOB_RECURSE C10_CUDA_SIMULATOR_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
message(STATUS "C10_CUDA_SIMULATOR_SRCS: ${C10_CUDA_SIMULATOR_SRCS}")

set(CMAKE_BUILD_TYPE RelWithDebInfo)
set(CMAKE_CXX_STANDARD 17)

if(NOT TORCH_INSTALL_LIB_DIR)
    set(TORCH_INSTALL_LIB_DIR lib)
endif()
message(STATUS "TORCH_INSTALL_LIB_DIR: ${TORCH_INSTALL_LIB_DIR}")

# python
find_package(Python COMPONENTS Interpreter Development)
if (NOT ${Python_FOUND})
    message(SEND_ERROR "PYTHON not found")
endif()
message(STATUS "PYTHON found: ${Python_FOUND}")
message(STATUS "PYTHON include: ${Python_INCLUDE_DIRS}")
message(STATUS "PYTHON libraries: ${Python_LIBRARIES}")

# libunwind
set(LIBUNWIND_LIBRARIES /usr/lib/x86_64-linux-gnu)
find_library(LIBUNWIND_LIB NAMES unwind PATHS ${LIBUNWIND_LIBRARIES})
if (NOT LIBUNWIND_LIB)
    message(SEND_ERROR "LIBUNWIND not found")
endif()
message(STATUS "LIBUNWIND found: ${LIBUNWIND_LIB}")

# cuda-sanitizer
set(CUDA_DIR /usr/local/cuda)
if(NOT DEFINED ENV{OLD_CUDA_VERSION})
    set(SANITIZER_DIR ${CUDA_DIR}/compute-sanitizer)
else()
    set(SANITIZER_DIR ${CUDA_DIR}/extras/Sanitizer)
endif()

find_library(SANITIZER_LIB NAMES sanitizer-public PATHS ${SANITIZER_DIR})
if (NOT SANITIZER_LIB)
    message(SEND_ERROR "SANITIZER not found")
endif()
message(STATUS "SANITIZER found: ${SANITIZER_LIB}")

set(PYBIND11_DIR ${CMAKE_CURRENT_SOURCE_DIR}/pybind11)
add_subdirectory(${PYBIND11_DIR})

add_library(allocatorsim SHARED ${C10_CUDA_SIMULATOR_SRCS})
target_include_directories(allocatorsim PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include
                            ${Python_INCLUDE_DIRS}
                            ${PYBIND11_DIR}/include
                            ${SANITIZER_DIR}/include
                            ${CUDA_DIR}/include)

target_link_libraries(allocatorsim ${Python_LIBRARIES} ${SANITIZER_LIB} ${LIBUNWIND_LIB} stdc++fs)

install(TARGETS allocatorsim DESTINATION ${TORCH_INSTALL_LIB_DIR})