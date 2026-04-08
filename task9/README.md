# HemelB

Find hemelb repo [here](https://github.com/hemelb-codes/hemelb)

1. [Prerequisites](#prerequisites)
1. [HemelB](#hemelb)
      1. [HemelB Module](#hemelb-module)
1. [HemelB RUN](#hemelb-run)
1. [Common errors](#common-errors)
1. [Additional Help](#additional-help)
      1. [GKLib](#gklib)
      1. [METIS](#metis)
      1. [ParMETIS](#parmetis)

## Prerequisites

### CMake

```bash
sudo dnf install cmake
```

### development tools (similar to build-essential)

```bash
sudo dnf groupinstall "Development Tools"
```

### TinyXML

```bash
sudo yum install tinyxml-devel
```

## HemelB

 ```bash
git clone https://github.com/hemelb-codes/hemelb.git
cd hemelb

mkdir build
cd build

# load modules (GCC / MPI / ParMETIS)

# Config CMakeLists.txt in hemel root folder for settings

ccmake ..
# press `c` to config
# press `e` to exit
# Maybe press `c` to config again
# press g to generate a Unix MakeFile and exit back

# Important setting to change
# ParMetis tarball : https://karypis.github.io/glaros/files/sw/parmetis/parmetis-4.0.3.tar.gz
# install dir : ~/bin

make -j$(nproc)

# If you missed something or on error just repeat
# from ccmake .. to make -j$(nproc)
 ```
 
  - Another way if ccmake dont work

```bash
# Set all the paths below remember ..
# check .a and .so extentions and include dirs
cmake -B build \
  -DCMAKE_INSTALL_PREFIX=/path \
  -DCMAKE_BUILD_TYPE=Release \
  -DMETIS_INCLUDE_DIR=/path-to-/metis/include \
  -DMETIS_LIBRARY=/path-to-/metis/lib/libmetis.a \
  -DParMETIS_INCLUDE_DIR=/path-to-/parmetis/include \
  -DParMETIS_LIBRARY=/path-to-/parmetis/lib/libparmetis.a \
  -DCMAKE_C_COMPILER=gcc \
  ..

cmake --build build
cmake --install build
```

### Hemelb Module

Create a module file for easy loading and running

```bash
mkdir /home/user/modulefiles/load
vim /home/user/modulefiles/load/hemelb
```

```bash
#%Module1.0
##
## HemelB modulefile
##
proc ModulesHelp { } {
   puts stderr "This module loads HemelB"
}
module-whatis "Loads HemelB"

# Load your pre-req modules e.g. gcc and mpi
module load openmpi/5.0.7-gcc-14.3.0-source-omp-native

# Set the installation prefix
set root /home/allen/fail/soft/hemelb

# Set environment variables
prepend-path PATH $root/bin
```

---

## HemelB RUN

For testing in the `hemel` build folder a `share/hemelb/resources/` is created with some resources

```bash
# Load modules
ml load/hemelb

# Make a working folder
mkdir ~/work
cd ~/work

# Copy over a test benchmark
cp /path-to-hemel/share/hemelb/resources/large_cylinder.xml .
cp /path-to-hemel/share/hemelb/resources/large_cylinder.gmy .

# Create output folder
mkdir output

# Add output properties to end of xml
vim large_cylinder.xml
# FIND below

# run
mpirun -n 4 hemelb -in large_cylinder.xml -out output/large_cylinder_test

# 2nd run
mpirun -n 4 hemelb -in large_cylinder.xml -out output/large_cylinder_test2
```

- Add to large_cylinder.xml just before `</hemelbsettings>`

```bash
<properties>
  <propertyoutput file="whole.xtr" period="100">
    <geometry type="whole" />
    <field type="velocity" />
    <field type="pressure" />
  </propertyoutput>
</properties>
```


## Common Errors

1. -- Install configuration: ""
CMake Error at cmake_install.cmake:62 (file):
  file cannot create directory: /usr/local/lib64/cmake/Catch2.  Maybe need
  administrative privileges.

```bash
# the -DCMAKE_INSTALL_PREFIX=/path didn't load correct
# remove build config and try again
rm -rf build
mkdir hemelb_build_dir && cd hemelb_build_dir
ccmake ..
 ```

2. on cTemplate error : bootstrap_build/dependencies/ctemplate-prefix/src/ctemplate/src/htmlparser

```bash
python dependencies/ctemplate-prefix/src/ctemplate/src/htmlparser/generate_fsm.py dependencies/ctemplate-prefix/src/ctemplate/src/htmlparser/htmlparser_fsm.config > dependencies/ctemplate-prefix/src/ctemplate/src/htmlparser/htmlparser_fsm.h

python2 dependencies/ctemplate-prefix/src/ctemplate/src/htmlparser/generate_fsm.py dependencies/ctemplate-prefix/src/ctemplate/src/htmlparser/jsparser_fsm.config > dependencies/ctemplate-prefix/src/ctemplate/src/htmlparser/jsparser_fsm.h

python dependencies/ctemplate-prefix/src/ctemplate/src/htmlparser/generate_fsm.py dependencies/ctemplate-prefix/src/ctemplate/src/tests/statemachine_test_fsm.config > dependencies/ctemplate-prefix/src/ctemplate/src/tests/statemachine_test_fsm.h

make -j$(nproc)
```

3. hemelb-v0.8/hemelb_build_dir/hemelb-prefix/src/hemelb/util/static_assert.h:81:1: error: extended character § is not valid in an identifier
   81 | §

```bash
# remove § from line 81
vi hemelb-prefix/src/hemelb/util/static_assert.h
```

## Additional Help

### GKLib

 - ! Please note install GKLib, Metis and ParMetis in same [path]

repo [here](https://github.com/KarypisLab/GKlib)

- load needed modules e.g. correct gcc

```bash
# Instructions
git clone https://github.com/KarypisLab/GKlib.git
cd GKlib

# Set PATH
make config cc=gcc prefix=/path
make 
make install
```

- Additional flags

```bash
# Add to make line
cc=[compiler]     - The C compiler to use [default: gcc]
prefix=[PATH]     - Set the installation prefix [default: ~/local]
openmp=set        - To build a version with OpenMP support
```

### METIS

repo [here](https://github.com/KarypisLab/METIS) check the site for more flags to config build

```bash
git clone https://github.com/KarypisLab/METIS.git
cd METIS

# Set gklib and install path
make config cc=gcc prefix=/path

make install
```

- Additional Flags

```
cc=[compiler]     - The C compiler to use [default is determined by CMake]
shared=1          - Build a shared library instead of a static one [off by default]
prefix=[PATH]     - Set the installation prefix [~/local by default]
gklib_path=[PATH] - Set the prefix path where GKlib has been installed. You can skip
                    this if GKlib's installation prefix is the same as that of METIS.
i64=1             - Sets to 64 bits the width of the datatype that will store information
                    about the vertices and their adjacency lists. 
r64=1             - Sets to 64 bits the width of the datatype that will store information 
                    about floating point numbers.
```

- If you didnt use the same [path] set `gklib_path=/path` in the make line otherwise you can leave it out

#### On error /usr/bin/ld: cannot find -lGKlib

```bash 
# View the content of gklib installed path
ls software/gklib

# See something like
# bin  include  lib  lib64

#Copy the lib from lib64 to lib
cp software/gklib/lib64/libGKlib.a software/gklib/lib/libGKlib.a

#Or create a link in lib
ln -s /path/to/original /path/to/shortcut
ln -s lib64/libGKlib.a lib/libGKlib.a

# Retry make or make install
make install
```

### ParMETIS

repo [here](https://github.com/KarypisLab/ParMETIS)

```bash
git clone https://github.com/KarypisLab/ParMETIS.git
cd ParMETIS

# Set path to GKLib and Metis as well as ParMetis
make config cc=mpicc prefix=/path
make install
```

- Additional flags

```bash
cc=[compiler]     - The C compiler to use [default is determined by CMake]
shared=1          - Build a shared library instead of a static one [off by default]
prefix=[PATH]     - Set the installation prefix [~/local by default]
gklib_path=[PATH] - Set the prefix path where GKlib has been installed. You can skip
                    this if GKlib's installation prefix is the same as that of ParMETIS.
metis_path=[PATH] - Set the prefix path where METIS has been installed. You can skip
                    this if METIS' installation prefix is the same as that of ParMETIS.
```

- If you didnt use the same [path] set `gklib_path=/path` and `metis_path=/path` in the make line otherwise you can leave it out