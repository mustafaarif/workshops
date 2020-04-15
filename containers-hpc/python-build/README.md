# Building Python inside the container

There are various options to satisfy Python build requirements.

* Find out if there is any official Python container which provides Python 3.8.2 (recommended)
  * Quick build wit least effort.
* Otherwise, build Python on top of desired OS.
  * Requires effort, but offers flexibility of choosing various optimization paramteres.
## Step 01: Build production ready container
### Option 01 (Build from Python Container)

Explore https://hub.docker.com and identify if Python 3.8.2 tag is available.

Log in to Dev workstation;
```sh
$ sudo su - 

$ mkdir -p ~/container-builds/python && cd ~/container-builds/python

#Replace <tag> with tag found on Docker Hub.
$ singularity build --sandbox py382docker docker://python:<tag>

#At this stage, check the size of the container you have downloaded
$ du -hs py382docker

$ singularity shell --writable py382docker

Singularity> apt-get update

Singularity> apt-get -y install wget git vim nano

Singularity> pip3 --no-cache-dir install tensorflow pandas keras scikit-learn

Singularity> python3 -c 'import tensorflow as tf'

Singularity> exit

# Check the size of the container after customizing your container with requirements
$ du -hs py382docker

# Make your container production ready by converting sandbox to non-writable image file
$ singularity build py382docker.sif py382docker

# Move container to /tmp so that it can be accessed via student account remotely
$ mv py382docker.sif /tmp
$ chown student:student /tmp/py382docker.sif

```

### Option 02 (Build from Source)
Log in to Dev workstation;
```sh

sudo su - 

mkdir -p ~/container-builds/python && cd ~/container-builds/python

singularity build --sandbox py382 docker://ubuntu:18.04

singularity shell --writable py382

apt-get update

apt-get -y install build-essential gfortran wget nano

mkdir -p /tmp/downloads && cd /tmp/downloads

wget https://www.python.org/ftp/python/3.8.2/Python-3.8.2.tar.xz

tar -xvf Python-3.8.2.tar.xz

cd Python-3.8.2 && mkdir build && cd build

../configure --enable-optimizations --with-ensurepip=install

make -j 4

make install

```

## Transfer container to HPC system raad2

```sh
# From your local system where you have VPN connection established, do ssh to raad2 and issue following;
$ mkdir -p ~/opt/python/382
$ cd ~/opt/python/382
$ scp -P <port> student@ml-xx.southcentralus.cloudapp.azure.com:/tmp/py382docker.sif

# Verify version of Tensorflow and Python inside the container
$ singularity exec py382docker.sif python3 -c 'import tensorflow as tf; print(tf.__version__)'

# Run sample application
singularity exec py382docker.sif python3 linearreg.py 
```
