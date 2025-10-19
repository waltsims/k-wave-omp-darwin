cmake_minimum_required(VERSION 3.20)
project(kspaceFirstOrderOMP LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_EXTENSIONS OFF)

set(HOMEBREW_PREFIX /opt/homebrew CACHE PATH "Homebrew prefix")
if(NOT EXISTS "${HOMEBREW_PREFIX}/opt/hdf5")
    set(HOMEBREW_PREFIX /usr/local CACHE PATH "Homebrew prefix" FORCE)
endif()

add_executable(kspaceFirstOrder-OMP main.cpp)
foreach(dir IN ITEMS Containers Hdf5 KSpaceSolver Logger MatrixClasses OutputStreams Parameters)
    add_subdirectory(${dir})
endforeach()

set(_brew_includes
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${HOMEBREW_PREFIX}/opt/hdf5/include
    ${HOMEBREW_PREFIX}/opt/fftw/include
    ${HOMEBREW_PREFIX}/opt/zlib/include
    ${HOMEBREW_PREFIX}/opt/libomp/include
)
set(_brew_libdirs
    ${HOMEBREW_PREFIX}/opt/hdf5/lib
    ${HOMEBREW_PREFIX}/opt/fftw/lib
    ${HOMEBREW_PREFIX}/opt/zlib/lib
    ${HOMEBREW_PREFIX}/opt/libomp/lib
)

target_include_directories(kspaceFirstOrder-OMP PRIVATE ${_brew_includes})
target_link_directories(kspaceFirstOrder-OMP PRIVATE ${_brew_libdirs})
target_link_libraries(kspaceFirstOrder-OMP PRIVATE fftw3f fftw3f_omp hdf5 hdf5_hl z omp m)
target_compile_options(kspaceFirstOrder-OMP PRIVATE -O3 -ffast-math -fassociative-math -g -Wall -Xpreprocessor -fopenmp -m64 -mtune=native)
target_link_options(kspaceFirstOrder-OMP PRIVATE -Xpreprocessor -fopenmp -m64 -mtune=native)
set_target_properties(kspaceFirstOrder-OMP PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}" BUILD_RPATH "${_brew_libdirs}" INSTALL_RPATH "${_brew_libdirs}")
