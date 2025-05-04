+++
date = '2025-05-04T17:10:13+08:00'
draft = false
title = 'Linux如何通过崩溃信息找到代码行'
tags = ["Linux","Debug"]
+++

崩溃信息中显示了SIGSEGV信号（信号11，表示段错误），但只有内存地址，没有具体行号。以下是几种定位代码行的方法：

### 1. 使用addr2line工具

```bash
# 使用addr2line将地址转换为源代码行号
addr2line -e ./RecoderCabinet 0x4081dc 0x4260f7 0x408e20
```

### 2. 使用GDB反向调试

```bash
# 加载程序到GDB
gdb ./RecoderCabinet

# 在GDB中查询地址
(gdb) info line *0x408e20
(gdb) list *0x408e20
```

### 3. 使用objdump分析

```bash
# 反汇编可执行文件
objdump -d -S ./RecoderCabinet > disassembly.txt

# 然后在输出文件中搜索地址0x408e20
```

### 4. 重新编译带调试信息的程序

如果可能，重新编译程序并添加调试信息：

```bash
# 使用gcc/g++编译
g++ -g -O0 -o RecoderCabinet src/main.cpp ...其他源文件...

# 使用cmake
cmake -DCMAKE_BUILD_TYPE=Debug ..
make
```

### 5. 为现有程序附加调试符号

如果有原始源代码但没有调试信息的程序：

```bash
# 创建单独的调试符号文件
objcopy --only-keep-debug ./RecoderCabinet RecoderCabinet.debug
objcopy --add-gnu-debuglink=RecoderCabinet.debug ./RecoderCabinet
```

从堆栈信息看，崩溃发生在main函数内（`main+0x6fe`），可能与设备借用或归还功能相关。将调试信息与源代码对照分析，就能找到确切的崩溃位置。