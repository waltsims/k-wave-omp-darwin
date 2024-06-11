## Overview

k-Wave is an open source MATLAB toolbox designed for the time-domain simulation
of propagating acoustic waves in 1D, 2D, or 3D. The toolbox has a wide range of
functionality, but at its heart is an advanced numerical model that can account
for both linear or nonlinear wave propagation, an arbitrary distribution of
heterogeneous material parameters, and power law acoustic absorption.
See the k-Wave website (http://www.k-wave.org) for further details.

This project is a part of the k-Wave toolbox accelerating 2D/3D simulations
using an optimized C++ implementation to run small to moderate grid sizes (e.g.,
128x128 to 10,000x10,000 in 2D or 64x64x64 to 512x512x512 in 3D) on systems
with shared memory. 2D simulations can be carried out in both normal and
axisymmetric coordinate systems.

This copy of the software has been adapted for compilation on Apple M1 processors.


## Repository structure

    .
    +--Containers    - Matrix and output stream containers
    +--Data          - Small test data
    +--GetoptWin64   - Windows version of the getopt routine
    +--Hdf5          - HDF5 classes (file access)
    +--KSpaceSolver  - Solver classes with all the kernels
    +--Logger        - Logger class for reporting progress and errors
    +--MatrixClasses - Matrix classes holding simulation data
    +--OutputStreams - Output streams for sampling data
    +--Parameters    - Parameters of the simulation
    +--Utils         - Utility routines
    Changelog.md     - Change log
    License.md       - License file
    Makefile         - GNU Makefile
    Readme.md        - Read me
    Doxyfile         - Doxygen documentation file
    header_bg.png    - Doxygen logo
    main.cpp         - Main file of the project


## Compilation

The source codes of `kspaceFirstOrder-OMP` are written using the C++-11 standard
and use the OpenMP 4.0, FFTW 3.3.8 or MKL 11, and HDF5 1.10.x libraries.

Before compiling the code, it is necessary to install a C++ compiler and several
libraries.

First [install x-code](https://developer.apple.com/xcode/). X-Code installs the Apple Clang compiler for M1.

The code also relies on several libraries that are to be installed before
compiling. A recommended package manager on Mac is [Homebrew](https://brew.sh/):

 1. HDF5 library - Mandatory I/O library, version 1.8.x,
         https://portal.hdfgroup.org/display/support/HDF5+1.8.21,
         or version 1.10.x.
         https://www.hdfgroup.org/downloads/hdf5/source-code/.
 1. FFTW library - Optional library for FFT, version 3.3.x,
         http://www.fftw.org/.
 1. MKL library  - Optional library for FFT, version 2018 or higher
         http://software.intel.com/en-us/intel-composer-xe/.
 1. OpenMP - Manditory library for shared memory parallel execution.
 1. zlib - Manditory library for compression with HD5F.

To install these libraries with Homebrew, run the following command:
```bash
brew install hdf5 fftw zlib libomp
```

 3. Select how to link the libraries. Static linking is preferred as it may be
    a bit faster, however, on some systems (e.g, HPC clusters) it may be better
    to use dynamic linking and use the system specific libraries at runtime.
    ```bash
     LINKING = STATIC
    #LINKING = DYNAMIC
    ```

 6. Close the makefile and compile the source code by typing:
    ```bash
    make -j
    ```
    If you want to clean the distribution, type:
    ```bash
    make clean
    ```

## Usage

The C++ codes offers a lot of parameters and output flags to be used. For more
information, please type:

```bash
./kspaceFirstOrder-OMP --help
```
