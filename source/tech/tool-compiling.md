# 常用工具编译方法

## GCC源码编译和升级安装(v7.3.0)

下载gcc-7.3.0的源码，并下载依赖的gmp、mpc、mpfr，解压到gcc-7.3.0的根目录下

```
cd gcc-7.3.0
tar -xjf gmp-6.1.0.tar.bz2 && ln -s gmp-6.1.0 gmp
tar -xzf mpc-1.0.3.tar.gz && ln -s mpc-1.0.3 mpc
tar -xjf mpfr-3.1.4.tar.bz2 && ln -s mpfr-3.1.4 mpfr
mkdir build
cd build
../configure --disable-multilib --enable-languages=c,c++ --enable-checking=release --prefix=/usr/local/gcc-7.3.0
make -j 8
make install
export PATH=/usr/local/bin:$PATH
```

若gmp/mpc/mpfr不存在，编译时会报错
```
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

## GLIBC编译和安装(v2.18)

```
wget http://mirrors.ustc.edu.cn/gnu/libc/glibc-2.18.tar.gz
tar -zxvf glibc-2.18.tar.gz
cd glibc-2.18
mkdir build
cd build
../configure --prefix=/usr
make -j4
sudo make install
```

## MP4Box编译

```
./configure --static-mp4box --enable-debug
```

## X265编译

cmake增加-pg编译和链接选项，为了做gprof profiling，需要使用静态编译，动态编译无法分析动态库的函数耗时
```
cmake -DCMAKE_INSTALL_PREFIX=../bin/debug/ -DCMAKE_BUILD_TYPE=Debug -DENABLE_SHARED=OFF -DCMAKE_C_FLAGS=-pg -DCMAKE_CXX_FLAGS=-pg -DCMAKE_EXE_LINKER_FLAGS=-pg -DCMAKE_SHARED_LINKER_FLAGS=-pg -G "Unix Makefiles" ../source/
```

