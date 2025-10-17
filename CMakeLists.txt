cmake_minimum_required(VERSION 3.20)

project(kspaceFirstOrderOMP VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(SOURCES
    main.cpp
    Containers/MatrixContainer.cpp
    Containers/OutputStreamContainer.cpp
    Hdf5/Hdf5File.cpp
    Hdf5/Hdf5FileHeader.cpp
    KSpaceSolver/KSpaceFirstOrderSolver.cpp
    Logger/Logger.cpp
    MatrixClasses/BaseFloatMatrix.cpp
    MatrixClasses/BaseIndexMatrix.cpp
    MatrixClasses/ComplexMatrix.cpp
    MatrixClasses/FftwComplexMatrix.cpp
    MatrixClasses/FftwRealMatrix.cpp
    MatrixClasses/IndexMatrix.cpp
    MatrixClasses/RealMatrix.cpp
    OutputStreams/BaseOutputStream.cpp
    OutputStreams/CuboidOutputStream.cpp
    OutputStreams/IndexOutputStream.cpp
    OutputStreams/WholeDomainOutputStream.cpp
    Parameters/CommandLineParameters.cpp
    Parameters/Parameters.cpp
)

add_executable(kspaceFirstOrder-OMP ${SOURCES})

target_include_directories(kspaceFirstOrder-OMP
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}
)

find_package(OpenMP QUIET COMPONENTS CXX)

if(OpenMP_CXX_FOUND)
    target_link_libraries(kspaceFirstOrder-OMP PRIVATE OpenMP::OpenMP_CXX)
else()
    if(APPLE)
        set(_libomp_hint_paths
            $ENV{OpenMP_ROOT}
            $ENV{LIBOMP_ROOT}
            $ENV{LIBOMP_DIR}
            /opt/homebrew/opt/libomp
            /usr/local/opt/libomp
            /opt/local/libomp
        )

        find_path(LIBOMP_INCLUDE_DIR omp.h
            HINTS ${_libomp_hint_paths}
            PATH_SUFFIXES include
        )

        find_library(LIBOMP_LIBRARY
            NAMES omp
            HINTS ${_libomp_hint_paths}
            PATH_SUFFIXES lib
        )

        if(NOT LIBOMP_INCLUDE_DIR OR NOT LIBOMP_LIBRARY)
            message(FATAL_ERROR "Failed to locate libomp. Install it (brew install libomp) and/or set LIBOMP_ROOT or OpenMP_ROOT.")
        endif()

        target_include_directories(kspaceFirstOrder-OMP PRIVATE ${LIBOMP_INCLUDE_DIR})
        target_link_libraries(kspaceFirstOrder-OMP PRIVATE ${LIBOMP_LIBRARY})
        target_compile_options(kspaceFirstOrder-OMP PRIVATE -Xpreprocessor -fopenmp)
        target_link_options(kspaceFirstOrder-OMP PRIVATE -Xpreprocessor -fopenmp)
    else()
        message(FATAL_ERROR "OpenMP support not found. Provide OpenMP toolchain or install the relevant runtime.")
    endif()
endif()

find_package(HDF5 REQUIRED COMPONENTS C HL)

set(_hdf5_targets "")
foreach(_candidate
        HDF5::HDF5_C
        HDF5::HDF5_HL
        HDF5::HDF5
        hdf5::hdf5
        hdf5::hdf5_hl)
    if(TARGET ${_candidate})
        list(APPEND _hdf5_targets ${_candidate})
    endif()
endforeach()
list(REMOVE_DUPLICATES _hdf5_targets)

if(_hdf5_targets)
    target_link_libraries(kspaceFirstOrder-OMP PRIVATE ${_hdf5_targets})
else()
    message(WARNING "Falling back to legacy HDF5 variables; consider upgrading CMake or HDF5 package.")
    target_include_directories(kspaceFirstOrder-OMP PRIVATE ${HDF5_INCLUDE_DIRS})
    set(_legacy_hdf5_libs
        ${HDF5_LIBRARIES}
        ${HDF5_C_LIBRARIES}
        ${HDF5_HL_LIBRARIES}
        ${HDF5_C_HL_LIBRARIES}
    )
    list(REMOVE_DUPLICATES _legacy_hdf5_libs)
    target_link_libraries(kspaceFirstOrder-OMP PRIVATE ${_legacy_hdf5_libs})
endif()

find_package(ZLIB REQUIRED)
target_link_libraries(kspaceFirstOrder-OMP PRIVATE ZLIB::ZLIB)

find_path(FFTW_INCLUDE_DIR fftw3.h
    HINTS
        ENV FFTW_ROOT
        ENV FFT_DIR
        /opt/homebrew/opt/fftw
        /usr/local/opt/fftw
        /opt/local
    PATH_SUFFIXES include
)

if(NOT FFTW_INCLUDE_DIR)
    message(FATAL_ERROR "Failed to locate FFTW headers; set FFTW_ROOT or FFT_DIR")
endif()

find_library(FFTW_LIB
    NAMES fftw3f
    HINTS
        ENV FFTW_ROOT
        ENV FFT_DIR
        /opt/homebrew/opt/fftw
        /usr/local/opt/fftw
        /opt/local
    PATH_SUFFIXES lib
)

if(NOT FFTW_LIB)
    message(FATAL_ERROR "Failed to locate FFTW float library (fftw3f)")
endif()

find_library(FFTW_OMP_LIB
    NAMES fftw3f_omp fftw3f_threads
    HINTS
        ENV FFTW_ROOT
        ENV FFT_DIR
        /opt/homebrew/opt/fftw
        /usr/local/opt/fftw
        /opt/local
    PATH_SUFFIXES lib
)

if(NOT FFTW_OMP_LIB)
    message(FATAL_ERROR "Failed to locate FFTW OpenMP/threaded float library (fftw3f_omp or fftw3f_threads)")
endif()

target_include_directories(kspaceFirstOrder-OMP PRIVATE ${FFTW_INCLUDE_DIR})
target_link_libraries(kspaceFirstOrder-OMP PRIVATE ${FFTW_LIB} ${FFTW_OMP_LIB})

if(APPLE)
    set(CPU_FLAGS -m64 -mtune=native)
    if(CMAKE_SYSTEM_PROCESSOR STREQUAL "arm64")
        list(APPEND CPU_FLAGS -mcpu=apple-m1)
    endif()

    target_compile_options(kspaceFirstOrder-OMP PRIVATE ${CPU_FLAGS})
    target_link_options(kspaceFirstOrder-OMP PRIVATE ${CPU_FLAGS})
endif()

target_compile_options(kspaceFirstOrder-OMP
    PRIVATE
        -O3
        -ffast-math
        -fassociative-math
        -g
        -Wall
)

target_link_options(kspaceFirstOrder-OMP
    PRIVATE
        -O3
        -ffast-math
        -fassociative-math
        -g
)

find_library(M_LIB m)
if(M_LIB)
    target_link_libraries(kspaceFirstOrder-OMP PRIVATE ${M_LIB})
endif()

set_target_properties(kspaceFirstOrder-OMP PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}"
)

if(APPLE)
    set(_rpath_dirs "")
    foreach(_lib ${FFTW_LIB} ${FFTW_OMP_LIB})
        get_filename_component(_lib_dir "${_lib}" DIRECTORY)
        list(APPEND _rpath_dirs "${_lib_dir}")
    endforeach()
    foreach(_lib ${HDF5_LIBRARIES} ${HDF5_C_LIBRARIES} ${HDF5_HL_LIBRARIES} ${HDF5_C_HL_LIBRARIES})
        if(IS_ABSOLUTE "${_lib}")
            get_filename_component(_lib_dir "${_lib}" DIRECTORY)
            list(APPEND _rpath_dirs "${_lib_dir}")
        endif()
    endforeach()
    if(LIBOMP_LIBRARY)
        get_filename_component(_lib_dir "${LIBOMP_LIBRARY}" DIRECTORY)
        list(APPEND _rpath_dirs "${_lib_dir}")
    endif()
    if(ZLIB_LIBRARY)
        get_filename_component(_lib_dir "${ZLIB_LIBRARY}" DIRECTORY)
        list(APPEND _rpath_dirs "${_lib_dir}")
    endif()
    list(REMOVE_DUPLICATES _rpath_dirs)
    if(_rpath_dirs)
        set_property(TARGET kspaceFirstOrder-OMP PROPERTY BUILD_RPATH "${_rpath_dirs}")
        set_property(TARGET kspaceFirstOrder-OMP PROPERTY INSTALL_RPATH "${_rpath_dirs}")
    endif()
endif()
