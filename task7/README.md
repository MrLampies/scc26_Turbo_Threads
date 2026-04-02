# Getting Started with OpenMX

OpenMX can be a little complex, try and do steps 1,2 below use step 3 Common errors to try and build openmx. Optimization up until MPI wappers are advanced topics. Only do these to improve on openMX run.

# Contents

1. [Install openMX](#install-openmx)
1. [makefile](#makefile)
1. [Common errors](#common-errors)
1. [Optimization](#optimization)
1. [OpenMP flags](#openmp-flags)
1. [Mpi ranks vs openMP](#openmp-vs-mpi-ranks)
    1. [openMPI example](#openmpi-example)
    1. [intelMPI example](#intelmpi-example)
1. [MPI wrappers](#mpi-wrapper)

## Task

1. Build openmx and upload ```openmx``` executable and ```makefile``` as proof to task7.
1. Run ```1-Methane.dat``` using 4 cores, upload ```Methane.out```.
1. Run ```2-NVC.dat``` using max allowed resoures and upload ```NVC.out```. 
1. Visualize results using [viewer](https://www.openmx-square.org/viewer/) and ```met.tden.cube``` output file. Upload as very small video to task7.
1. Visualize results using [viewer](https://www.openmx-square.org/viewer/) and ```nvc.sden.cube``` output file. Upload as very small video to task7.

## Install openMX

Here are instructions to setup openMX.

As well as a source folder with a new source used for ISC VIrtual competition.

- Needs some MPI like openMPI or intelMPI

- Needs maths like intel oneApi MKL (maths kernel library) with intel fftw

- Or instead of intel use [ fftw , openBlas and Scalapack  ] or [ fftw , Blacs, lapack , scalapack ]

Installation TIPS :

> https://openmx-square.org/openmx_man3.9/node14.html

```bash
# Get openMX
wget https://www.openmx-square.org/openmx3.9.tar.gz
tar xfp openmx3.9.tar.gz
cd openmx3.9

# Load modules
ml scalapack

# Edit makefile more info below
vim makefile

# Build openMX
make -j$(nproc)

# Install openmx to work
make install

# If successful, OpenMX executable file will be created in openmx3.9/work/openmx
cd ../work

# Use mpirun or slurm to run job
mpirun -np 2 ./openmx Methane.dat
```

### MakeFile

#### GNU

- Bare minimum

Enter the location to fftw3 library in ```FFTROOT=```

```
FFTROOT=nfs/home/software/fftw3/3.3.10/openMPI-5.0.7-gcc-14.2.0-vectorized
CC = mpicc -O3 -fopenmp -fcommon -Wno-error=implicit-function-declaration -I/$(FFTROOT)/include
FC = mpif90 -O3 -fopenmp -fallow-argument-mismatch
LIB= -lfftw3 -lmpi -lmpi_mpifh -lopenblas -lscalapack -lgfortran
```

- Notes on libs

If you use modules to load libraries like (fftw3 , openBLAS and scalapack), you can just load the lib directly e.g.

> LIB = -lfftw3 -lmpi -lmpi_mpifh -lopenblas -lscalapack -lgfortran

`tip` - The libraries has to be loaded in the order above,

- additional

```
# try additional CC and FC flags
# -mavx2 -mavx512bw -mavx512dq -mavx512vl -mavx512cd
CC = mpicc -O3 -mavx2 -mavx512bw -mavx512dq -mavx512vl -mavx512cd -fopenmp #...
FC = mpif90 -O3 -mavx2 -mavx512bw -mavx512dq -mavx512vl -mavx512cd -fopenmp #...
```

#### Common errors

`Pro tip!` : Add and remove flags as you build and it will complete eventually.

`Pro tip!` : Try `make clean` if a error persists or after making a flag change

`Pro tip!` : openmx CC and FC wants `-O3` optimization!!

1. File `not found` or `cant open` .o or .mod file errors!

- openMP FLAG

```
Intel = -openmp
Intel = -qopenmp
Intel = -fiopenmp          
GNU   = -fopenmp
CLANG = -fopenmp         
PGI = -mp -Dnosse
```

If the openMP flag is correct then just **retry** `make install` until it works!!!!

2. elpa.o: In function `elpa1_mp_tridiag_real_':
    elpa1.f90:(.text+0x2b7): undefined reference to `mpi_comm_rank_'

- Cannot find MPI Fortran library

```
LIB = ... -lmpi_f77 -lmpi_f90 -lifcore
```

3. undefined reference to `__kmpc_flush'

- OpenMP library to LIB 

```
LIB = ... -liomp5 -lpthread
```

4. Missing fortran library

- fortran library

```
# Intel
-lifcore

# GNU
-lgfortran

# PGI
-lpgf90
```

5. Cluster_DFT_Col_DMmu.c:456:3: error: call to undeclared function 'Cblacs_barrier'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]

```bash
# Add to CC
CC = -Wno-error=implicit-function-declaration
```

6. Error: Type mismatch in argument ‘vmat_s’ at (1); passed `COMPLEX(8) to REAL(8)`

[Full guide](https://www.openmx-square.org/forum/patio.cgi?mode=view&no=2704)

```
FC = -fallow-argument-mismatch
```

7. To avoid errors about `"multiple definition"` of global variables:
openmx_common.o:(.bss+0x44fe8): multiple definition of `ratv'
openmx.o:(.bss+0x45d48): first defined here

```
CC = -fcommon

# must do a clean after this is added
make clean
```

8. undefined reference to `Spherical_Bessel2'

[instructions](https://www.openmx-square.org/forum/patio.cgi?mode=view&no=3153)

```
CC = -Dkcomp
```

9. /usr/bin/ld: cannot find /include: No such file or directory

> There might be a space between a `variable` and `/include` 

```
${FFTROOT} /include
```

> Or a space after a variable declaration

```
GCCROOT=nfs/home/software/gcc/gcc-13.3.0/gcc-13.3.0-install | <-- space here
```

10. cblacs error

```bash
# add openblas include to CC or FC
FC = -I/${LBSROOT}/include
```

11. extra libs

```
LIB = -lm -ldl
```

12. `seg error` or The calculation was terminated due to the illegal SCF calculation.

> check build chain all the same c,c++,fc flags

in makefile

```
# check paths
LBSROOT=path-to/openblas/0.3.28/gcc-11.5.0
MPIROOT=path-to/openMPI/5.0.6/gcc-11.5.0
FFTROOT=path-to/fftw3/3.3.10/gcc-11.5.0

# They may need these headers/includes for special build flags
CC = -I/$(FFTROOT)/include -I/$(LBSROOT)/include -I/$(MPIROOT)/include
FC = -I/$(FFTROOT)/include -I/$(LBSROOT)/include -I/$(MPIROOT)/include

# this headers removed a illegal SCF error

# Remember to rebuild!

# Also, it wants O3 optimization!
CC = -O3
FC = -O3

```

### Optimization

1. Make sure openMPI can use hyperthreading if it gives a performance bonus, otherwise disable hyperthreading.

**MPI Ranks**: These are individual processes that communicate with each other using MPI. Each rank typically runs on a separate core or processor. EACH `process` is assigned a unigue `rank number`.

**OpenMP Threads**: Within each MPI rank, you can use OpenMP to `create multiple threads`. These threads share the memory space of the MPI rank and can run concurrently on different cores.

By varying the number of `OpenMP threads` **per** `MPI rank`, you can optimize the performance of your application. This approach can be beneficial in several scenarios:

*Memory Bandwidth* and *Cache Efficiency*: On nodes with many cores, using multiple OpenMP threads per MPI rank can help alleviate memory bandwidth and cache size limitations.

**GPU** Utilization: When running applications on `GPUs`, using `fewer MPI ranks` and **more** `OpenMP threads` can better utilize the computational power of the CPU for tasks that still run on the CPU side.

To implement this, you can set the number of OpenMP threads using the `OMP_NUM_THREADS` environment variable and then run your application with mpirun. 

For example (bad, just for learning):

```bash
# This command sets 4 OpenMP threads per MPI rank and runs the application with 2 MPI ranks
export OMP_NUM_THREADS=4
mpirun -np 2 ./openmx Methane.dat

# Real example for 32 processes x 4 openMP threads per MPI = 128 processes for the 128 logical cores [hyper threading enabled]
export OMP_NUM_THREADS=4
mpirun -np 32 --map-by socket:PE=4 --bind-to hwthread --use-hwthread-cpus --oversubscribe --report-bindings ./openmx 2-NVC.dat -nt 4

# Recommended add
export OMP_PROC_BIND=close

# It is clear from this what numbers is for what, just ignore the -nt 4 as this is specific to openmx to enable openMP for 4 parallel threads

# Use this for 128 core on 1 node
export OMP_PROC_BIND=close
export OMP_NUM_THREADS=4
mpirun -np 32 --map-by socket:PE=4 --bind-to hwthread --use-hwthread-cpus --oversubscribe --report-bindings ./openmx 2-NVC.dat -nt 4
```

### OpenMP flags

```
Intel = -openmp
Intel = -qopenmp
Intel = -fiopenmp          
GNU   = -fopenmp
CLANG = -fopenmp         
PGI = -mp -Dnosse
```

### Compiler Flags

#### Intel

```bash
# Min
-xHost
# Min++
-axAVX -axSSE4.2 -xSSE2
# Favourite
-mavx2 -mavx512bw -mavx512dq -mavx512vl -mavx512cd
# Others
-O3 -fopenmp -march=native -mtune=native
# Safe
-xHOST -fp-model precise

For AVX-512, here are some specific vectorization flags you can use when compiling programs:

-mavx512f: Enables the foundational AVX-512 instructions 1.
-mavx512bw: Enables AVX-512 byte and word instructions 1.
-mavx512dq: Enables AVX-512 doubleword and quadword instructions 1.
-mavx512vl: Enables AVX-512 vector length extensions for 128-bit and 256-bit vectors 1.
-mavx512cd: Enables AVX-512 conflict detection instructions 1.
-qopt-zmm-usage: Controls the usage of ZMM registers for AVX-512 2.
-mprefer-vector-width-512: Prefers 512-bit vector width for AVX-512 2.
```

#### OpenMP vs MPI ranks

It all depends on the mpi you are using.

Use [openMPI manual](https://docs.open-mpi.org/en/v5.0.x/man-openmpi/man1/mpirun.1.html) for guidance on how to properly use `mpirun`

below is breakdown of keypoints

```bash
# Command use
mpirun [options] ./app [app input]

# [options]

-np X   # create X processes

--map-by [options]   # This is how processes are distributed through cluster/system

    # map options
    slot / core / numa / node / ppr:N
    # ppr:N   -> map N Processes Per Resource (core/slot/numa/node)
    pe=n
    # pe=n    -> n cpu's to each process, processing elements (cores)
    # slots   -> cpu's / cores

--bind-to [options]     # Bind processes to 

    # bind options
    none / package / numa / core
    :overload-allowed

--rank-by [options]     # How processes are place/ordered

    slot / node / fill / span

--report-bindings

-x OMP_NUM_THREADS=2 

# To pass lib to all hosts
-x LD_LIBRARY_PATH

-H host1,host2

--hostfile hosts.txt

    hosts.txt
    com1 slot=64
    com2 slot=64
```

##### openMPI example

```bash
# If 4 nodes with 256 total cores
# 4 openmp threads , 64 mpi ranks
# 16 processes per node , 4 cpu's pre process (processing elements)
export OMP_NUM_THREADS=4
mpirun -np 64 --map-by ppr:16:node:pe=4 ./openmx 2-NVC.dat -nt 4
```

- Full Example

```bash
# Full example 

mpirun -np 128 -x OMP_NUM_THREADS=2 --hostfile hosts.txt -x LD_LIBRARY_PATH --map-by ppr:32:node:pe=2 ./openmx 2-NVC.dat -nt 2
```

##### intelMPI example

```bash
# n    -> number of processes
# ppn  -> processes per node
# On 4 nodes with 256 total cores
# 4 openmp threads, 64 mpi ranks, 16 processes per node
mpiexec -n 64 -ppn 16 -genv OMP_NUM_THREADS=4 -genv I_MPI_PIN_DOMAIN=socket -genv I_MPI_PIN_ORDER=compact ./openmx 2-NVC.dat -nt 4

```

### MPI wrapper

To use the Intel oneAPI compilers (like icx) with MPI, you can use the Intel MPI Library with specific options to wrap the LLVM-based compilers. Here’s how you can do it:

> Using mpiicc with -cc=icx: The mpiicc wrapper can be configured to use icx by specifying the -cc option. Similarly, mpiicpc and mpiifort can be used for C++ and Fortran compilers respectively.

```bash
mpiicc -cc=icx sample.c
mpiicpc -cxx=icpx sample.cpp
mpiifort -fc=ifx sample.f90
```

or

Environment Variables: You can also set environment variables to specify the compilers:

```bash
export I_MPI_CC=icx
export I_MPI_CXX=icpx
export I_MPI_F90=ifx
```