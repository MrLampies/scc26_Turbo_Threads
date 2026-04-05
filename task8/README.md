# MILC guide

1. [Get MILC](#get-milc)
    
    i.  [Edit Makefile](#edit-the-makefile)

    ii. [Build MILC](#build-milc)

    iii. [Clean](#clean)

2. [Benchmark](#benchmark)

    i.  [Edit batch files](#edit-batch-file)

    ii. [Run benchmark](#run-benchmark)

3. [Additional](#additionals)

    - OpenMPI commands

4. [Build errors](#build-error)

5. [Tips](#tips)

## Get MILC

Get repo from github

```
git clone https://github.com/milc-qcd/milc_qcd.git
```

Checkout development branch

```
cd milc_qcd
git checkout develop
```

Copy Makefile to RHMC build

```
cd ks_imp_rhmc
cp ../Makefile .
```

### Edit the Makefile

```
nano Makefile
```

Important properties

>COMPILER=gcc or intel

>ARCH=  

```
clx (OpenStack)
spr (Competition)
hsw (lab)   [ haswell -> Ivy bridge ~ Sandy Bridge ] 
```

```
#List for reference
EPYC: AMD's server processors, known for high performance and energy efficiency.
HSW: Haswell, an Intel microarchitecture.
SKX: Skylake, an Intel microarchitecture.
CLX: Cascade Lake, an Intel microarchitecture.
ICX: Ice Lake, an Intel microarchitecture.
SPR: Sapphire Rapids, an Intel microarchitecture.
KNL: Knights Landing, an Intel Xeon Phi processor.
POW8: Power8, an IBM microprocessor.
POW9: Power9, an IBM microprocessor.
```

NO QUDA (gpu) stuff, Double percision, openMP (OMP) , want openMP

>WANTQUDA ?= false

>MPP ?= true

>PRECISION ?= 2

>OMP ?= true

> OFFLOAD ?= openmp

### Build milc

Make sure openMPI is loaded

>ml

Is it available

>ml av

Load correct openMPI version (openMPI or intel MPI)

>ml openmpi

**Build**

```
make su3_rhmd_hisq 2>&1 | tee make_logfile.log
```

### Clean

To clean for a **new build** or to **retry** after *errors*

```
make clean
rm ../libraries/*.o
```

## Benchmark

Check filenames online at <https://portal.nersc.gov/project/m888/apex>

Get the apex MILC

```
cd ~
wget https://portal.nersc.gov/project/m888/apex/MILC_160413.tgz
tar xvzf MILC_160413.tgz
```

Head into the medium benchmark and check requirements

```
cd MILC-apex/benchmarks/medium
ls -al
```

**Copy the correct lattices, build and libraries over to MILK-apex**

```
cp -r ~/milc_qcd/ks_imp_rhmc/ ~/MILC-apex/
cp -r ~/milc_qcd/libraries/ ~/MILC-apex/
cd ../lattices/
wget https://portal.nersc.gov/project/m888/apex/MILC_lattices/36x36x36x72.chklat
```

### Edit batch file

```
cd /home/cput/MILC-apex/benchmarks/medium/
nano run_medium.sh
```

*Info on the batch file can be found in the **[Slurm](https://github.com/CPUT-HPC-Club/lessons/blob/main/slurm/README.md)** lessons*

### RUN Benchmark

Manual run 

> ./run_medium.sh

Background run

>sbatch run_medium.sh

*Info on the batch file can be found in the **[Slurm](https://github.com/CPUT-HPC-Club/lessons/blob/main/slurm/README.md)** lessons*

## Additionals

check OpenMPI and GCC versions

```
mpiexec --version       #recent version at least
mpirun --version
mpiicpc -v              #Intel MPI compiler
mpicc --version         #OpenMPI compiler
gcc --version           #GNU compiler
```

Compiling a C++ file with Intel MPI

>mpiicpc -o xxxx.cpp filename -lmpi

Compiling a c++ file with OpenMPI

>mpiCC xxxx.cpp -o filename

Running a mpi file

>mpirun -np 4 filename

Testing mpirun (do 2 mpiruns)

> mpirun -np 2 hostname

Testing srun (change n from 4 to 16 to see if 16 cores is available)

> srun -n4 -l /bin/hostname

## Build Error

1. Sometime a -xCORE-AVX2 error will ocure 

    >   cc -c -O3 -g -xCORE-AVX2 -DFAST -DMILC_PRECISION=2

    - replace with -march=core-avx2

    >   cc -c -O3 -g -march=core-avx2 -DFAST -DMILC_PRECISION=2

2. Errors in the libraries, check

    > ../libraries/Make_vanilla

3. With intel check compiler
    - mpicc
        > which mpicc

    - mpicxx
        > which mpicxx

    - Usually this modules must be loaded
        ```
        ml tbb/latest compiler-rt/latest compiler-intel-llvm/latest
        ```
    
    - Verify
        ```
        which icx
        which icxp
        ```

## TIPS

1. Make a backup of your Makefile

> cp Makefile Makefile.bak

2. Inside Makefile

    -   with 
        > ARCH_FLAG = -xCORE-AVX512 -qopt-zmm-usage=high
    -   Remove
        > -xCORE-AVX512
    -   Replace
        > -mtune=cascadelake -march=cascadelake
    - Such that
        > ARCH_FLAG = -mtune=cascadelake -march=cascadelake -qopt-zmm-usage=high

    - OFFLOAD setting
        > OFFLOAD ?= openmp
    - OMP
        > OMP ?= true

3. Always clean / sanitize

> make clean && rm ../libraries/*.o

4. Output to log files

> make su3_rhmd_hisq 2>&1 | tee make_i201h260_skx.log

5. Look at Make_template in **build** and **libraries** loactions

    - to fix errors

6. Benchmark download

```
#download benchmark

curl -O https://portal.nersc.gov/project/m888/apex/MILC_lattices/36x36x36x72.chklat
curl -O https://portal.nersc.gov/project/m888/apex/MILC_160413.tgz
tar xvzf MILC_160413.tgz

#config build script

vim build_medium.sh
vim milc_in.sh
```

7. With Intel

    - Check mpicc is intel

        > which mpicxx

    - Common intel dir

        > ~/software/Intel/oneAPI/2024.0.0/HPCKit/mpi/2021.11/bin/mpicxx

    - Load modules

        > ml tbb/latest compiler-rt/latest intel_ipp_intel64/latest intel_ippcp_intel61/latest mkl/latest mpi/latest ccl/latest oclfpga compiler/latest

8. Run

    >   ./run_medium

    - When setting up run_medium.sh

        -   Make total cores = (Total core incl threads)
        -   Threads per core = (Acctual threads per core) *get devided out*

9. Info on MILC

The quirks that are inside protons and neutrons
and
the interaction between that strong necular force that binds, that keeps the proton as a proton

quantom chromo dynamics