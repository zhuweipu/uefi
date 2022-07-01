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


## 编译build工具

edk2 自构建了一套自动化编译工具build

```bash
cd edk2
make -C BaseTools
```

## 编译运行模拟器

**依赖**
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

**编译**

编译生成的中间文件和目标文件均在Build目录下，Build目录地址依赖WORKSPACE环境变量


使用build命令指定编译EmulatorPkg
```bash
build -p EmulatorPkg/EmulatorPkg.dsc -t GCC5 -a X64
```

-t指定编译器，虽然是GCC5，但是使用GCC 11.2可行

或者使用脚本编译，此时WORKSPACE只能设置为edk2目录

```bash
EmulatorPkg/build.sh

# 32 bits
EmulatorPkg/build.sh -a IA32
```
**命令行运行**

```bash
# 必须进入此目录，因为依赖../FV/FV_RECOVERY.fd
cd Build/emulatorX64/DEBUG_GCC5/X64
./Host
```

**脚本运行**

```bash
EmulatorPkg/build.sh run

# 32 bits 
EmulatorPkg/build.sh -a IA32 run
```

build.sh run直接启动GDB命令

```bash
/usr/bin/gdb $BUILD_ROOT_ARCH/Host -q -cd=$BUILD_ROOT_ARCH -x $WORKSPACE/EmulatorPkg/Unix/GdbRun.s
```

在main函数打断点
```bash
b main
c
tui enable
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

### qemu编译运行Ovmf

```bash
./OvmfPkg/build.sh
./OvmfPkg/build.sh qemu
```

qemu参数执行命令
```bash
qemu-system-x86_64 -drive if=pflash,format=raw,file=/home/uefi/uefi-traning/edk2-ws/edk2/Build/OvmfX64/DEBUG_GCC5/QEMU/bios.bin
```

使用官方[wiki](https://github.com/tianocore/tianocore.github.io/wiki/How-to-run-OVMF)教程运行会遇到一些问题
```bash
qemu-system-x86_64 -L . -hda fat:hda-contents

WARNING: Image format was not specified for 'json:{"fat-type": 0, "dir": "hda-contents", "driver": "vvfat", "floppy": false, "rw": false}' and probing guessed raw.
         Automatically detecting the format is dangerous for raw images, write operations on block 0 will be restricted.
         Specify the 'raw' format explicitly to remove the restrictions.
qemu-system-x86_64: Block node is read-only
```

```bash
qemu-system-x86_64 -pflash bios.bin -hda fat:hda-contents -net none
WARNING: Image format was not specified for 'bios.bin' and probing guessed raw.
         Automatically detecting the format is dangerous for raw images, write operations on block 0 will be restricted.
         Specify the 'raw' format explicitly to remove the restrictions.
WARNING: Image format was not specified for 'json:{"fat-type": 0, "dir": "hda-contents", "driver": "vvfat", "floppy": false, "rw": false}' and probing guessed raw.
         Automatically detecting the format is dangerous for raw images, write operations on block 0 will be restricted.
         Specify the 'raw' format explicitly to remove the restrictions.
qemu-system-x86_64: Block node is read-only
```

### qemu 源码级调试

官方教程[wiki](https://github.com/tianocore/tianocore.github.io/wiki/How-to-debug-OVMF-with-QEMU-using-GDB)比较麻烦

参考教程[Debugging OVMF with GDB](https://retrage.github.io/2019/12/05/debugging-ovmf-en.html)脚本自动生成gdb symbol地址

编写Makefile，启动参数将OVMF_CODE.fd指定为只读，将OVMF_VARS.fd指定为可读写，指定image文件夹为存储设备，执行生成的debug log信息在debug.log文件中。

Makefile脚本
```bash
LOG=debug.log
OVMFBASE=edk2/Build/OvmfX64/DEBUG_GCC5/
OVMFCODE=$(OVMFBASE)/FV/OVMF_CODE.fd
OVMFVARS=$(OVMFBASE)/FV/OVMF_VARS.fd
QEMU=qemu-system-x86_64

QEMUFLAGS=-drive format=raw,file=fat:rw:image \
          -drive if=pflash,format=raw,readonly,file=$(OVMFCODE) \
          -drive if=pflash,format=raw,file=$(OVMFVARS) \
          -debugcon file:$(LOG) -global isa-debugcon.iobase=0x402 \
          -serial stdio \
          -nographic \
          -nodefaults

run:
    $(QEMU) $(QEMUFLAGS)

debug:
    $(QEMU) $(QEMUFLAGS) -s -S

.PHONY: run debug

```

要进行源码级调试，需要符号表等调试信息，对应在Build/OvmfX64/DEBUG_GCC5/X64目录中的debug文件中。

```bash
file DevicePathDxe.debug 
DevicePathDxe.debug: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, with debug_info, not stripped
```

通过gdb命令add-symbol-file将调试信息加载至对应地址即可。

但是对应地址应该如何计算，实际加载和运行的程序是efi文件
```
file DevicePathDxe.efi 
DevicePathDxe.efi: MS-DOS executable PE32+ executable (EFI boot service driver) x86-64, for MS Windows
```

调试信息打印在debug.log文件中，说明了每个efi文件加载的地址。

efi是PE格式文件，通过[peinfo](https://github.com/retrage/peinfo)Portable Executable Header Viewer解析其头文件信息得到代码段偏移地址，然后加上efi加载地址即可得到调试信息应该加载的地址。

调试信息地址生成脚本
```bash
#!/bin/bash

LOG="debug.log"
BUILD="edk2/Build/OvmfX64/DEBUG_GCC5/X64"
SEARCHPATHS="edk2/Build/OvmfX64/DEBUG_GCC5/X64"
PEINFO="peinfo/peinfo"

cat ${LOG} | grep Loading | grep -i efi | while read LINE; do
  BASE="`echo ${LINE} | cut -d " " -f4`"
  NAME="`echo ${LINE} | cut -d " " -f6 | tr -d "[:cntrl:]"`"
  EFIFILE="`find ${SEARCHPATHS} -name ${NAME} -maxdepth 1 -type f`"
  ADDR="`${PEINFO} ${EFIFILE} \
        | grep -A 5 text | grep VirtualAddress | cut -d " " -f2`"
  TEXT="`python -c "print(hex(${BASE} + ${ADDR}))"`"
  SYMS="`echo ${NAME} | sed -e "s/\.efi/\.debug/g"`"
  SYMFILE="`find ${SEARCHPATHS} -name ${SYMS} -maxdepth 1 -type f`"
  echo "add-symbol-file ${SYMFILE} ${TEXT}"
done
```


```bash
make run
gen_symbol_offsets.sh > gdbscript
make debug
(gdb) source gdbscript
(gdb) b CoreHandleProtocol
(gdb) target remote localhost:1234
(gdb) c
(gdb) tui enable
```


### compile_command.json

编写生成json文件的函数
edk2/BaseTools/Source/Python/edk2_compile_commands.py

```python
import glob
import os
import json
from pprint import pprint
from filelock import FileLock
from time import sleep

def update_compile_commands_file(TargetDict, AutoGenObject, Macros):
    # process only compilation, not linkage or whatever else
    #  pprint(TargetDict)
    if not TargetDict['cmd'].startswith('"$(CC)"'):
        return

    # Need to fild compiler and his flags for this project
    cc = ''
    cc_flags = ''
    for p in glob.glob(os.path.join(Macros['BIN_DIR'], 'TOOLS_DEF.*')):
        #  pprint(p)
        try:
            content = open(p, 'r').read()
            done = {'CC_PATH': False, 'CC_FLAGS': False}
            for line in content.splitlines():
                if line.startswith('CC_FLAGS = '):
                    done['CC_FLAGS'] = True
                    cc_flags = line[len('CC_FLAGS = '):]
                    if all(v == True for v in done.values()):
                        break
                elif line.startswith('CC_PATH = '):
                    done['CC_PATH'] = True
                    cc = line[len('CC_PATH = '):]
                    if all(v == True for v in done.values()):
                        break
            if all(v == True for v in done.values()):
                break
        except:
            continue
    if not cc or not cc_flags:
        print ('Error: cc or cc_flags is not defined!\n')
        exit(1)

    #  project_path    = AutoGenObject.Macros['PLATFORM_DIR']
    project_path    = AutoGenObject.Macros['WORKSPACE']
    #  pprint(project_path)
    directory_field = Macros['BIN_DIR']
    command_field   = TargetDict['cmd'].replace('$(CC)', cc, 1).replace(
                          '$(CC_FLAGS)', cc_flags, 1
                      ).replace(
                          '$(INC)', '-I' + ' -I'.join(
                              AutoGenObject.IncludePathList
                          ), 1
                      )
    file_field      = TargetDict['cmd'].split()[-1]

    return common_update_compile_commands_file(
        project_path, directory_field, command_field, file_field
    )

# spec.url: https://clang.llvm.org/docs/JSONCompilationDatabase.html
# project_path    - path to compile_commands.json, path to project root
# directory_field - "directory" from spec., not really used here, because nearly all paths are absolute
# command_field   - "command" from spec.
# file_field      - "file" from spec.
def common_update_compile_commands_file (
    project_path, directory_field, command_field, file_field
):
    result_path = os.path.join(project_path, 'compile_commands.json')
    with FileLock("lockfile.txt"):
        content = []
        try:
            f = open(result_path, 'r')
            candidate_content_raw = f.read()
            f.close()
            candidate_content = json.loads(candidate_content_raw)
            content = candidate_content
        except IOError:
            print ('Can\'t read {}, creating new..'.format(result_path))
        except ValueError:
            # ValueError is inherited by json.decoder.JSONDecodeError
            print ('Error at parsing {}, creating new..'.format(result_path))
            exit(1);
        except:
            print ('Unexpected error')
            exit(1);

        obj = {
            'directory': directory_field,
            'command':   command_field,
            'file':      file_field
        }
        try:
            i = next(i for i,entry in enumerate(content) if entry['file'] == file_field)
            content[i] = obj
        except StopIteration:
            content.append(obj)

        f = open(result_path, 'w')
        # Save to file
        json.dump(content, f, indent=2)
        f.close()
```

使用filelock实现多线程，因此需要安装

```bash
pip3 install filelock
```


在edk2/BaseTools/Source/Python/AutoGen/GenMake.py调用

```python
from edk2_compile_commands import update_compile_commands_file

if self._AutoGenObject.BuildRuleFamily == TAB_COMPILER_MSFT and Type == TAB_C_CODE_FILE:
                    T, CmdTarget, CmdTargetDict, CmdCppDict = self.ParserCCodeFile(T, Type, CmdSumDict, CmdTargetDict,
                                                                                   CmdCppDict, DependencyDict, RespFile,
                                                                                   ToolsDef, resp_file_number)
                    resp_file_number += 1
                    TargetDict = {"target": self.PlaceMacro(T.Target.Path, self.Macros), "cmd": "\n\t".join(T.Commands),"deps": CCodeDeps}
                    CmdLine = self._BUILD_TARGET_TEMPLATE.Replace(TargetDict).rstrip().replace('\t$(OBJLIST', '$(OBJLIST')
                    if T.Commands:
                        CmdLine = '%s%s' %(CmdLine, TAB_LINE_BREAK)
                    if CCodeDeps or CmdLine:
                        self.BuildTargetList.append(CmdLine)
                else:
                    TargetDict = {"target": self.PlaceMacro(T.Target.Path, self.Macros), "cmd": "\n\t".join(T.Commands),"deps": Deps}

                    # 调用生成compile_command.json的函数
                    update_compile_commands_file(TargetDict, self._AutoGenObject, self.Macros)

                    self.BuildTargetList.append(self._BUILD_TARGET_TEMPLATE.Replace(TargetDict))

                    # Add a Makefile rule for targets generating multiple files.
                    # The main output is a prerequisite for the other output files.
                    for i in T.Outputs[1:]:
                        AnnexeTargetDict = {"target": self.PlaceMacro(i.Path, self.Macros), "cmd": "", "deps": self.PlaceMacro(T.Target.Path, self.Macros)}
                        self.BuildTargetList.append(self._BUILD_TARGET_TEMPLATE.Replace(AnnexeTargetDict))

```

重新编译生成目标文件即可看到根目录下的compile_command.json文件
