# MPI

MPI (Message Passing Interface) is used for parallel communication between processes. By communicating across processes, our programs can change from serial execution to parallel.

## Installation

The OpenMPI implementation previously used is buggy. Instead, you will compile OpenMPI from source. First, get the files:

```bash
wget https://download.open-mpi.org/release/open-mpi/v5.0/openmpi-5.0.3.tar.gz
tar -xzf openmpi-5.0.3.tar.gz
cd openmpi-5.0.3
```

Then load the OpenPMIx dependency and configure the compilation:

```bash
ml openpmix
./configure --prefix=${HOME}/opt/openmpi
```

Your OpenMPI build is now configured to install into ```${HOME}/opt/openmpi```. Now compile and install it:

```bash
make -j$(nproc)
make install
```

Confirm that the executables are in ```.../bin``` and the libraries in ```.../lib```. However, running the following will give an error:

```bash
which mpicc
```

This is because your environment variables haven't been updated to include your OpenMPI. Update the ```PATH``` and ```LD_LIBRARY_PATH``` variables as follows:

```bash
export PATH=${HOME}/opt/openmpi/bin:${PATH}
export LD_LIBRARY_PATH=${HOME}/opt/openmpi/lib
which mpicc
```

You should now see that the compiler is found successfully.

## Hello world Hello world

We will first try using MPI to print "Hello world" to the terminal multiple times. Try to print a sentence out multiple times using MPI.

```bash
mpirun -n 2 echo "Hello world"
```

You should see your message appear on screen twice. Try increasing 2 to a higher number, however, and you will receive an error message. Try it for yourself and understand the reason why it fails.

## Two heads are better than one

The previous "Hello world" processes were only sent to the CPU on your current node. However, MPI is capable of sending processes to other connected nodes as well.<br>
First, confirm the name of the node you are currently on:

```bash
hostname
```

Next, you will try to get the names of all the compute nodes in the cluster:

```bash
mpirun -H com1,com2 hostname
```

This should print out each of the node's names. Take note that they may not necessarily be in order; this is an issue common in parallel computing.

# The Final Task

Your team is required to run the HPL benchmark across as much of the cluster as you are allowed. Use anything provided on the system. Research anything and everything that can give you an edge over the other teams.<br>
Before the deadline, you will need to upload to ```.../task5/``` in your repo:
- The executable ```xhpl``` which you used to get your best score
- The textfile ```HPL.dat``` which was used to get your best score
- The makefile which compiled your benchmark
- An output file called ```HPL.out``` which contains the result
- A bash script which loads the necessary dependencies from lmod and runs your benchmark. This will be used to confirm your score independently, so ensure that it works as intended.
- Screenshots and ```proof.md``` of the commands and their output from above uploaded to your fork<br>
Teams will be ranked according to their score. Whenever you get a new best result, upload at least the ```HPL.dat``` file to your repo. At certain hour marks until the deadline, a temporary ranking will be posted for everyone to see. How often you hold higher positions will increase your overall score, which will be revealed at the end.<br>
The scoring formula scales with time, so those who get first place early on but ended in last place might lose to someone who was initially last place but ended in first place.