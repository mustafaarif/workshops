# Building Gromacs Container with MPI Support

GROMACS is a versatile package to perform molecular dynamics, i.e. simulate the Newtonian equations of motion for systems with hundreds to millions of particles.

This example demostrates building Gromacs Package with MPI support inside a container and porting it to HPC system "raad2


## Step 01
Explore Gromacs website and identify package requirements.
http://manual.gromacs.org/documentation/2020/install-guide

## Step 02
Prepare container with pre-reqs

```sh
$ sudo su - 

$ mkdir -p ~/container-builds/gromacs2020 && cd ~/container-builds/gromacs2020

$ singularity build --sandbox gromacs2020 docker://ubuntu:18.04

$ singularity shell --writable gromacs2020

$ apt-get update

$ export DEBIAN_FRONTEND=noninteractive

$ apt-get -y install build-essential wget nano git locales-all tzdata

$ ln -fs /usr/share/zoneinfo/Asia/Qatar /etc/localtime
```

## Step 03
Install MPICH inside the container
```sh
$ export MPICH_VERSION=3.3

$ export MPICH_URL="http://www.mpich.org/static/downloads/$MPICH_VERSION/mpich-$MPICH_VERSION.tar.gz"

$ mkdir -p /tmp/downloads

$ cd /tmp/downloads && wget -O mpich-$MPICH_VERSION.tar.gz $MPICH_URL && tar xzf mpich-$MPICH_VERSION.tar.gz

$ cd /tmp/downloads/mpich-$MPICH_VERSION

$ ./configure
# Did config failed? Are we missing any package? Hint: apt-get install gfortran

$ make -j 4

$ make install
```
## Step 04
Download and build Gromacs
```sh
mkdir -p /tmp/downloads
cd /tmp/downloads && wget http://ftp.gromacs.org/pub/gromacs/gromacs-2020.tar.gz
tar -xvzf /tmp/downloads/gromacs-2020.tar.gz
cd /tmp/downloads/gromacs-2020
mkdir build && cd build
cmake .. -DGMX_BUILD_OWN_FFTW=ON -DCMAKE_C_COMPILER=mpicc -DCMAKE_CXX_COMPILER=mpicxx -DGMX_MPI=on
make -j 4
make install
```
## Setup environment variables inside the container
Gromacs supplies a bash script which should be exectued before launching Gromacs simulation.
However when you are running singularity on raad2 on multi-node it is not possible to interactively set environment variables. Therefore we need to identify which variables are required and set them in container in advance.
```sh
cd /.singularity.d/
touch 99-gromacs2020.sh
```



