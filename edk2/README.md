# EDK2

## 环境配置

[Using EDK II with Native GCC](https://github.com/tianocore/tianocore.github.io/wiki/Using-EDK-II-with-Native-GCC)

LSB (Linux Standard Base) and Distribution information

```bash
$ lsb_release  -a

No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 22.04 LTS
Release:	22.04
Codename:	jammy
```

- build-essential - Informational list of build-essential packages including gcc g++ make libc (x86 or x64 is based on your system)

- uuid-dev - a tool to generate Universally Unique ID(uuid) number 

- iasl - Intel ASL compiler/decompiler (also provided by acpica-tools)

- git - support for git revision control system 

- nasm - General-purpose x86 assembler

- python-is-python3 - Ubuntu 22.04 python command is 'python3' but edk2 tools use 'python'

```bash
sudo apt install build-essential uuid-dev iasl git  nasm  python-is-python3 
```

## 下载源码

[edk2 github](https://github.com/tianocore/edk2)

```bash
git clone https://github.com/tianocore/edk2.git
cd edk2
git submodule update --init # --recursive is not recommended
```

Note: When cloning submodule repos, '--recursive' option is not recommended. EDK II itself will not use any code/feature from submodules in above submodules. So using '--recursive' adds a dependency on being able to reach servers we do not actually want any code from, as well as needlessly downloading code we will not use.

If there's update for submodules, use following git commands to get the latest submodules code.

```bash
cd edk2
git pull
git submodule update
```

## 编译模拟器

[EmulatorPkg wiki](https://github.com/tianocore/tianocore.github.io/wiki/EmulatorPkg)

the extra build dependencies required to build EmulatorPkg,

```bash
sudo apt install libx11-dev libxext-dev
```

For building IA32 on X64 Ubuntu 22.04 LTS, some X86 libs are needed [provided by Platform CI](https://github.com/tianocore/edk2/blob/master/EmulatorPkg/PlatformCI/ReadMe.md).

```bash
sudo dpkg --add-architecture i386
sudo apt update

sudo apt install libc6-dev:i386 lib32gcc-7-dev libx11-dev:i386 libxext-dev:i386 
# or sudo apt install gcc-multilib libx11-dev:i386 libxext-dev:i386
```

```bash
EmulatorPkg/build.sh
EmulatorPkg/build.sh run
```


32 bits 
```bash
EmulatorPkg/build.sh -a IA32
EmulatorPkg/build.sh -a IA32 run
```
### 编译问题

1. fatal error: X11/Xlib.h: No such file or directory

```bash
sudo apt install libx11-dev
```

2. fatal error: X11/extensions/XShm.h: No such file or directory

```bash
sudo apt install libxext-dev
```

3. 32 bits emulator bits/libc-header-start.h: No such file or directory

```bash 
sudo apt-get install gcc-multilib    
```    

4. 32 bits emulator
/usr/bin/ld: skipping incompatible /usr/lib/x86_64-linux-gnu/libXext.so when searching for -lXext    
/usr/bin/ld: cannot find -lXext: No such file or directory    
/usr/bin/ld: skipping incompatible /usr/lib/x86_64-linux-gnu/libX11.so when searching for -lX11    
/usr/bin/ld: cannot find -lX11: No such file or directory    
collect2: error: ld returned 1 exit status

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt-get install libx11-dev:i386
sudo apt install libxext-dev:i386
```

