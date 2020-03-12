---
title: 任务0.1：编译GCC
date: 2020-03-12 20:16:20
tags: [env, gcc, compile]
---

金老师给我们的附加任务是从源代码编译一套GCC。

在[GNU网站](https://gcc.gnu.org/install/index.html)上有完整的关于如何从源代码编译GCC的指导。GNU网站上将编译和安装GCC的过程划分为6个步骤，我们只需要完成前4个步骤（即到Build这一步）就算是完成任务了。

我打算在云主机中编译GCC。总的来说，这不算是一件难事。

```sh
# 在编译前确认系统信息。
$ uname -r
3.10.0-1062.12.1.el7.x86_64

# 建议使用较新版本的GCC。
$ gcc --version
gcc (GCC) 8.3.1 20190311 (Red Hat 8.3.1-3)
```

## 取得GCC的源代码

这一部分对应指南中的[第2步](https://gcc.gnu.org/install/download.html)。

我跳过了指南中的[第1步](https://gcc.gnu.org/install/prerequisites.html)，因为第1步描述的是编译GCC所需要安装的软件。这些软件如果缺失的话，可以在“配置”这一步骤中检测出来。

GCC的源代码可以从GNU的[官方Git仓库](https://gcc.gnu.org/git.html)中克隆下来，或者从[镜像网站](https://gcc.gnu.org/mirrors.html)下载。这里我选择的是速度较快的日本镜像站[ftp.tsukuba.wide.ad.jp](http://ftp.tsukuba.wide.ad.jp/software/gcc/)，下载的版本是9.2.0。

```sh
cd ~
mkdir -p esit-workspace/task0.1
cd esit-workspace/task0.1

wget http://ftp.tsukuba.wide.ad.jp/software/gcc/releases/gcc-9.2.0/gcc-9.2.0.tar.gz
# 下载时如果速度太慢或失败，可以现在网络条件好的地方下载好，然后使用`scp`上传至服务器。

# 从压缩包中解压源代码。
tar -xf gcc-9.2.0.tar.gz
```

这样，GCC的源代码就准备好了。

## 配置

这一部分对应指南中的[第3步](https://gcc.gnu.org/install/configure.html)。

```sh
$ pwd
/home/lightyears/esit-workspace/task0.1
$ ls
gcc-9.2.0 gcc-9.2.0.tar.gz

# 根据指南的要求，新建一个目录用来装编译产物。
$ mkdir gcc-9.2.0-build
gcc-9.2.0  gcc-9.2.0-build  gcc-9.2.0.tar.gz

# 运行配置脚本
$ cd gcc-9.2.0-build
$ ../gcc-9.2.0/configure
configure: error: Building GCC requires GMP 4.2+, MPFR 2.4.0+ and MPC 0.8.0+.
Try the --with-gmp, --with-mpfr and/or --with-mpc options to specify
their locations.  Source code for these libraries can be found at
their respective hosting sites as well as at
ftp://gcc.gnu.org/pub/gcc/infrastructure/.  See also
http://gcc.gnu.org/install/prerequisites.html for additional info.  If
you obtained GMP, MPFR and/or MPC from a vendor distribution package,
make sure that you have installed both the libraries and the header
files.  They may be located in separate packages.
```

`configure`在运行的过程中报错，看上去应该是必须的一些软件没有安装。依次执行`yum search/info GMP`、`yum search/info MPFR`和`yum search MPC`、`yum info libmpc`，发现应该是MPC包没有安装，`sudo yum install libmpc`将需要的包装好，再次执行配置。

```sh
$ ../gcc-9.2.0/configure
configure: error: I suspect your system does not have 32-bit development libraries (libc and headers). If you have them, rerun configure with --enable-multilib. If you do not have them, and want to build a 64-bit-only compiler, rerun configure with --disable-multilib.
```

由于不需要进行编译32位的程序，因此可以使用`--disable-multilib`重新配置。

```sh
$ ../gcc-9.2.0/configure
# 程序运行后退出，MakeFile已经生成好了。
```

到这里，这一步骤就顺利结束了。

## 编译

这一部分对应指南中的[第4步](https://gcc.gnu.org/install/build.html)。

由于编译过程会持续很久，这时可以使用`screen`这样的终端管理器。这样即使断开SSH连接，编译过程也继续运行。

```sh
screen -S compile-gcc # 新建一个窗口提供给编译gcc使用。
make -j 2 # 然后等待。
# 可随时断开SSH连接。
```

```sh
# 当重新通过SSH连接到主机时，使用screen恢复之前的会话。
screen -r compile-gcc
```

todo
