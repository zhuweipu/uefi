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

For building IA32 on X64 Ubuntu 22.04 LTS, some X86 libs are needed.

```bash
sudo dpkg --add-architecture i386
sudo apt update

sudo apt install libc6-dev:i386 lib32gcc-7-dev libx11-dev:i386 libxext-dev:i386 
# or sudo apt install gcc-multilib libx11-dev:i386 libxext-dev:i386

```

