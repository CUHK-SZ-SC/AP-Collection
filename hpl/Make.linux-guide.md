# Guidence of how to write Make.linux

**Environment: `Ubuntu 18.04 LTS, HPL 2.3`**



## Part One - General

```bash
# All
ARCH         = linux
TOPdir       = ../../..    # Because all the compilation location are three level higher

```



## Part Two - MPI

*Check dynamic link status by `ldd bin/linux/xhpl`*.

* `MPICH`

    ```bash
    MPdir        = 
    MPinc        = -I /usr/include/mpich/
    MPlib        = /usr/lib/x86_64-linux-gnu/libmpich.so
    ```

    * Installation: `apt install libmpich-dev`
    * Check: `libmpich.so.0 => /usr/lib/x86_64-linux-gnu/libmpich.so.0`

* `OpenMPI`

    ```bash
    MPdir        = /usr/lib/x86_64-linux-gnu/openmpi
    MPinc        = -I $(MPdir)/include
    MPlib        = $(MPdir)/lib/libmpi.so
    ```
    
    * Installation: `apt install libopenmpi-dev`

    * Check: 

        ```bash
    libmpi.so.20 => /usr/lib/x86_64-linux-gnu/libmpi.so.20
                     => /usr/lib/x86_64-linux-gnu/libmpi.so.20.10.1
                     => /usr/lib/x86_64-linux-gnu/openmpi/lib/libmpi.so.20.10.1
        ```
    
* `Intel MPI`

    ```bash
    MPdir        = /opt/intel/impi/2019.6.166/intel64
    MPinc        = -I$(MPdir)/include
    MPlib        = -L$(MPdir)/lib64/libmpi.a
    ```

    * Installation: `see Intel-MPI-KML-Installation-guide.md`

    * Check: `libmpi.so.12 => /opt/intel/compilers_and_libraries_2020.0.166/linux/mpi/intel64/lib/release/libmpi.so.12`

      


## Part Three - BLAS

*@LAinc should be blank, since HPL use its own `hpl_blas.h`.*

* `netlib-blas`

    ```bash
    LAdir        = /usr/lib/x86_64-linux-gnu/blas
    LAinc        = 
    LAlib        = -Wl,-rpath=$(LAdir)/ $(LAdir)/libblas.so
    ```

    * Installation: `apt install libblas-dev`
    * Check: `libblas.so.3 => /usr/lib/x86_64-linux-gnu/atlas/libblas.so.3`

* `atlas`

    ```bash
    LAdir        = /usr/lib/x86_64-linux-gnu/atlas
    LAinc        = 
    LAlib        = -Wl,-rpath=$(LAdir)/ $(LAdir)/libblas.so
    ```

    * Installation: `apt install libatlas-base-dev`
    * Check: `libblas.so.3 => /usr/lib/x86_64-linux-gnu/blas/libblas.so.3`

* `openblas`

    ```bash
    LAdir        = /usr/lib/x86_64-linux-gnu/openblas
    LAinc        = 
    LAlib        = -Wl,-rpath=$(LAdir)/ $(LAdir)/libblas.so
    ```

    * Installation: `apt install libopenblas-dev`
    * Check: `libblas.so.3 => /usr/lib/x86_64-linux-gnu/openblas/libblas.so.3`

* `Intel MKL`

    ```bash
    LAdir        = /opt/intel/mkl
    LAinc        = -I$(LAdir)/include
    LAlib        = -L$(LAdir)/lib/intel64 \
                   -Wl,--start-group \
                   $(LAdir)/lib/intel64/libmkl_intel_lp64.a \
                   $(LAdir)/lib/intel64/libmkl_intel_thread.a \
                   $(LAdir)/lib/intel64/libmkl_core.a \
                   -Wl,--end-group -lpthread -ldl -lm
    ```

    * Installation: `see Intel-MPI-KML-Installation-guide.md`

    * Check: `Static linked here`

        

## Part Four - Compilation Flags

* `MPICH` or `OpenMPI`

    *`mpicc` is a wrapper of `gcc`*

```bash
HPL_OPTS     = -DHPL_DETAILED_TIMING -DHPL_PROGRESS_REPORT -DHPL_CALL_CBLAS

CC           = mpicc.mpich   # or mpicc.openmpi
CCNOOPT      = $(HPL_DEFS) 
CCFLAGS      = $(HPL_DEFS) -fomit-frame-pointer -O3 -funroll-loops -W -Wall

LINKER       = $(CC)
LINKFLAGS    =
```



*  `Intel MPI` 

    *`mpiicc` is a wrapper of `icc (Intel C++ Compiler)`*

```bash
HPL_OPTS     = -DHPL_DETAILED_TIMING -DHPL_PROGRESS_REPORT -DHPL_CALL_CBLAS

CC       = mpiicc
CCNOOPT  = $(HPL_DEFS)
OMP_DEFS = -qopenmp
CCFLAGS  = $(HPL_DEFS) -O3 -w -ansi-alias -i-static -z noexecstack -z relro -z now -nocompchk -Wall

LINKER       = $(CC)
LINKFLAGS    = $(CCFLAGS) $(OMP_DEFS) -mt_mpi
```



