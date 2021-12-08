---
title: 'Fix GCC 11 Missing Headers on Mac OS Monterey'
date: 2021-12-08T19:10:30+08:00
tags:
- development
- cpp
- gcc
---

Encountered this issue trying to compile a simple "Hello World" program with `gcc-11` from `homebrew`.
The compiler complained about `fatal error: _stdio.h: No such file or directory`. After some investigation,
this was caused by wrong building configuration for `gcc`:

```
$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc-11
COLLECT_LTO_WRAPPER=/opt/homebrew/Cellar/gcc/11.2.0_3/libexec/gcc/aarch64-apple-darwin21/11/lto-wrapper
Target: aarch64-apple-darwin21
Configured with: ../configure --prefix=/opt/homebrew/Cellar/gcc/11.2.0_3 --libdir=/opt/homebrew/Cellar/gcc/11.2.0_3/lib/gcc/11 --disable-nls --enable-checking=release --with-gcc-major-version-only --enable-languages=c,c++,objc,obj-c++,fortran --program-suffix=-11 --with-gmp=/opt/homebrew/opt/gmp --with-mpfr=/opt/homebrew/opt/mpfr --with-mpc=/opt/homebrew/opt/libmpc --with-isl=/opt/homebrew/opt/isl --with-zstd=/opt/homebrew/opt/zstd --with-pkgversion='Homebrew GCC 11.2.0_3' --with-bugurl=https://github.com/Homebrew/homebrew-core/issues --build=aarch64-apple-darwin21 --with-system-zlib --disable-multilib --with-native-system-header-dir=/usr/include --with-sysroot=/Library/Developer/CommandLineTools/SDKs/MacOSX12.sdk
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 11.2.0 (Homebrew GCC 11.2.0_3)
```

The `gcc` from `homebrew` is assuming `--with-sysroot=/Library/Developer/CommandLineTools/SDKs/MacOSX12.sdk`, but it doesn't exists on Mac OS Monterey.
Instead the path is `/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk`. As we now know
the root cause of this issue, fixing it will be easy:

```
sudo ln -s  /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk /Library/Developer/CommandLineTools/SDKs/MacOSX12.sdk
```
