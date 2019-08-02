---
layout: post
title: Installing gcc without root privilege in CentOS
category: linux
tags: 
  - Configuration
published: true
---

Sometimes one may find the version of the default gcc on CentOS might be out of date. However, without root privelige, the installation of the gcc will be painful.

<!-- more -->

First, you need to download the source from a gcc mirror. For instance, you can go [here](http://www.netgull.com/gcc/releases/) and choose a gcc version whatever you like. I would recommend [gcc-7.4.0](http://www.netgull.com/gcc/releases/gcc-7.4.0/) (released 2018-12-06) since it supoorts most common C++ standards. After donwloading the source, unpack it, and cd into the directory where your version of gcc was unpacked. Then, enter the following commands. You might have to replace all instances of gcc-7.4.0 with your gcc version. Also, this script will install gcc in your ~/software directory. If you want it installed somewhere else, change the prefix option accordingly.  The make command could take around 3 hours, but you can use the parameter -j[n] to allow multiprocessing. With -j60 it will take less than half hour. 

```sh
$ cd $HOME
# You can create a directory for managing installed softwares
$ mkdir software
# Replace all instances of gcc-7.4.0 with your own gcc version
$ wget http://www.netgull.com/gcc/releases/gcc-7.4.0/gcc-7.4.0.tar.gz
$ tar -xzf gcc-7.4.0.tar.gz
$ cd gcc-7.4.0
# Download and install all the prerequisites packages and libraries
$ ./contrib/download_prerequisites 
$ cd ../software
# Create the install directory
$ mkdir gcc-7.4.0
$ cd ../gcc-7.4.0
# Create a directory to store temporary files created in configuration
$ mkdir gcctemp
$ cd gcctemp
# Have to unset library path temporarily to install new gcc
$ unset LIBRARY_PATH LD_LIBRARY_PATH
# Change the prefix option accordingly if you want to install it somewhere else
$ ../configure --prefix=$HOME/software/gcc-7.4.0 --enable-languages=c,c++,fortran,go --disable-multilib

# Notes on the parameters
# --disable-multilib: 
#   This parameter will disable building 32-bit support on 64-bit systems.
#
# --enable-languages=c,c++,fortran,go,objc,obj-c++: 
#   This command identifies which languages to build. 
#   You may modify this command to remove undesired language
#
# Use the parameter -j[2-60] to allow multiprocessing
$ make -j60
$ make install
```

Optionally you can create a direcrtory to store the resource file.

```sh
$ cd $HOME
$ mkdir resource
$ mv gcc-7.4.0 ~/resource/
```

After make and make install you need to set the enviornment in your ~/.bashrc file. 

```sh
$ vi ~/.bashrc
```

Add following lines into your .bashrc file with the insert mode of your vim editor:

```sh
$ export PATH=~/software/gcc-7.4.0/bin:$PATH
$ export LD_LIBRARY_PATH=~/software/gcc-7.4.0/lib:$LD_LIBRARY_PATH
$ export LD_LIBRARY_PATH=~/software/gcc-7.4.0/lib64:$LD_LIBRARY_PATH
```
Save and exit your .bashrc file. Do following commands to check if your gcc is successfully installed.  
```sh
$ source ~/.bashrc
$ g++ --version
```

If it is successfully installed it should print following message:
```sh
g++ (GCC) 7.4.0
Copyright (C) 2017 Free Software Foundation, Inc.
This is free software; see the source for copying conditions. There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

Now gcc and g++ are successfully installed on your own server.

