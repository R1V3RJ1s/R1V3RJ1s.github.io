---
layout: post
title: Installing gcc without root privilege in CentOS
category:
  - Server
date: 2019-08-03 15:19:53
tags: 
  - CentOS
  - Bash
published: true
copyright: true
---

Sometimes one may find the version of the default gcc on CentOS might be out of date. However, without root privelige, the installation of the gcc will be painful.

<!-- more -->
### Main Steps

First, you need to download the source from a gcc mirror. For instance, you can go [here](http://www.netgull.com/gcc/releases/) and choose a gcc version whatever you like. I would recommend [gcc-7.4.0](http://www.netgull.com/gcc/releases/gcc-7.4.0/) (released 2018-12-06) since it supoorts most common C++ standards. After donwloading the source, unpack it, and cd into the directory where your version of gcc was unpacked. Then, enter the following commands. You might have to replace all instances of gcc-7.4.0 with your gcc version. Also, this script will install gcc in your `~/software` directory. If you want it installed somewhere else, change the prefix option accordingly. The whole step may take around 4 hours but you can allow multiprocessing in the building steps to shorten the duration.

```sh
$ cd $HOME # pwd: ~
# You can create a directory for managing installed softwares
$ mkdir software
# Replace all instances of gcc-7.4.0 with your own gcc version
$ wget http://www.netgull.com/gcc/releases/gcc-7.4.0/gcc-7.4.0.tar.gz
$ tar -xzf gcc-7.4.0.tar.gz
$ cd gcc-7.4.0 # pwd: ~/gcc-7.4.0
```

If you do not have all the prerequisites packages and libraries on your system or you are not sure about it, GCC can help you download and install them through `download_prerequisites`.

```sh
$ ./contrib/download_prerequisites 
```

After download all the prerequisites we can head to the configuration step.

```sh
$ cd ../software # pwd: ~/software
# Create the install directory
$ mkdir gcc7
$ cd ../gcc7 # pwd: ~/software/gcc7
# Create a directory to store temporary files created in configuration
$ mkdir gcctemp
$ cd gcctemp # pwd: ~/software/gcc7/gcctemp
# Have to unset library path temporarily to install new gcc
$ unset LIBRARY_PATH LD_LIBRARY_PATH
# Configuration
$ ../configure --prefix=$HOME/software/gcc-7.4.0 --enable-languages=c,c++,fortran,go --disable-multilib
```

The `--prefix` parameters specifies the installation path. The `--disable-multilib` parameter will disable building 32-bit support on 64-bit systems. The `--enable-languages` parameter identifies which languages to build. You can modify this command to remove undesired language. Potential languages you can choose to build or to remove are `c`, `c++`, `fortran`, `go`, `objc`(Objective-C) and `obj-c++`(Objective-C++). Usually we only need `c,c++,fortran,go` these four.

Once the configuration is complete we can go to the building and installing step. The `make` command could take around 3 hours, but you can use the parameter `-j[n]` to allow multiprocessing. With `-j60` it will take less than half hour. 

```sh
# Use the parameter -j[n] to allow multiprocessing
$ make -j60
$ make install
```

Optionally you can create a direcrtory to store the resource file.

```sh
$ cd $HOME # pwd: ~
$ mkdir resource
$ mv gcc-7.4.0 ~/resource/
```

After make and make install you need to set the enviornment in your `.bashrc` file. 

```sh
$ vi ~/.bashrc
```

Add following lines into your `.bashrc` file with the insert mode of your vim editor:

```sh
$ export PATH=$HOME/software/gcc-7.4.0/bin:$PATH
# If your server has a 32-bit system or 
# you do not add the --disable-multilib parameter in the previous step 
# you might need to adjust the $LD_LIBRARY_PATH to $GCC_PATH/lib instead of $GCC_PATH/lib64
$ export LD_LIBRARY_PATH=$HOME/software/gcc-7.4.0/lib64:$LD_LIBRARY_PATH
```
Save and exit your `.bashrc` file. Do following commands to check if your gcc is successfully installed.  
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

### Troubleshooting

1. Sometimes if you do `make` or `make install` in gcc download directory directly may cause error, so a better way is to create a new folder called `gcctemp`, and do the congfiguration, building and installation step in this temporary folder.
2. Remember to unset `LIBRARY_PATH` and `LD_LIBRARY_PATH` before configuration, or it will raise `LIBRARY_PATH shouldn't contain the current directory when building gcc` ERROR.
3. In the configuration step, if you want to refer your `root` path in `--prefix` parameter you should use `$HOME` instead of `~`, or it will cause error.
4. If you are installing gcc on a 64-bit system remember to add --disable-multilib flag, or it will cause error.
5. You need to add `fortran` in your  `--enable-languages` parameter if you want to install R in the future, or it will cause error.
6. For the `-j[n]` parameter in the `make` step, do not let `n` greater than the processor number you have in your system, or it may cause erorr. You can use `lscpu` command to check how many processors you have in your system. 