# 一、启动项目的步骤

1. cargo build --release
2. rust-objcopy --strip-all target/riscv64gc-unknown-none-elf/release/os -O binary target/riscv64gc-unknown-none-elf/release/os.bin  
   （去掉元数据）
3. qemu-system-riscv64 \
    -machine virt \
    -nographic \
    -bios ../bootloader/rustsbi-qemu.bin \
    -device loader,file=target/riscv64gc-unknown-none-elf/release/os.bin,addr=0x80200000  
    （启动）
4. Ctrl+x+a  
   （退出qemu）

## 1. 准备工作

1. .config文件配置（os目录下新建.cargo目录）：  
```
#os/.cargo/config  
[build]    
target = "riscv64gc-unknown-none-elf" 
```
2. 参考”第一章 内核第一条指令（实践篇）[准备工作](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/4first-instruction-in-kernel2.html)

## 2. qemu启动

```shell
qemu-system-riscv64 \
    -machine virt \
    -nographic \
    -bios ../bootloader/rustsbi-qemu.bin \
    -device loader,file=target/riscv64gc-unknown-none-elf/release/os.bin,addr=0x80200000

#`-machine virt` 表示将模拟的 64 位 RISC-V 计算机设置为名为 `virt` 的虚拟计算机
#`-nographic` 表示模拟器不需要提供图形界面，而只需要对外输出字符流
#`-bios` 可以设置 Qemu 模拟器开机时用来初始化的引导加载程序（bootloader），这里我们使用预编译好的 `rustsbi-qemu.bin` ，它需要被放在与 `os` 同级的 `bootloader` 目录下,该目录可以从每一章的代码分支中获得
#`-device` 中的 `loader` 属性可以在 Qemu 模拟器开机之前将一个宿主机上的文件载入到 Qemu 的物理内存的指定位置中， `file` 和 `addr` 属性分别可以设置待载入文件的路径以及将文件载入到的 Qemu 物理内存上的物理地址。注意这里我们载入的文件带有 `.bin` 后缀，它并不是上一节中我们移除标准库依赖后构建得到的内核可执行文件，而是还要进行加工处理得到内核镜像
```

## 3. GDB启动

```shell
#终端1
qemu-system-riscv64 \
    -machine virt \
    -nographic \
    -bios ../bootloader/rustsbi-qemu.bin \
    -device loader,file=target/riscv64gc-unknown-none-elf/release/os.bin,addr=0x80200000 \
    -s -S
#多加了-s -S
#`-s` 可以使 Qemu 监听本地 TCP 端口 1234 等待 GDB 客户端连接，而 `-S` 可以使 Qemu 在收到 GDB 的请求后再开始运行

#终端2
riscv64-unknown-elf-gdb \
    -ex 'file target/riscv64gc-unknown-none-elf/release/os' \
    -ex 'set arch riscv:rv64' \
    -ex 'target remote localhost:1234'
#接着就可以gdb调试
```

# 二、分析生成的elf文件

```shell
#1.查看文件格式
$ file target/riscv64gc-unknown-none-elf/debug/os
#输出：target/riscv64gc-unknown-none-elf/debug/os: ELF 64-bit LSB executable, UCB RISC-V, ......

#2.文件头信息
$ rust-readobj -h target/riscv64gc-unknown-none-elf/debug/os
#输出信息：File: target/riscv64gc-unknown-none-elf/debug/os
		   #Format: elf64-littleriscv
		   #Arch: riscv64
		   #AddressSize: 64bit
		   #......
		   #Type: Executable (0x2)
		   #Machine: EM_RISCV (0xF3)
		   #Version: 1
		   #Entry: 0x0
		   #......
		   #}

#3.反汇编导出汇编程序
$ rust-objdump -S target/riscv64gc-unknown-none-elf/debug/os
   #输出饭汇编文件:target/riscv64gc-unknown-none-elf/debug/os:       file format elf64-littleriscv...
```

# 三、注意事项

1. 当用到**子模块**mod（**同级模块**——同一文件夹下其他文件——可以直接use互相使用）的**函数/结构体**的时候，要先`mod 模块名称;`来**声明要用哪个模块**，再用`use crate::模块所在位置::{要用的东西};`“来**使用**，这里用`crate::`是为了让无论当前mod被加载到哪个mod都可以正确找到其所使用的mod的路径；  
2. 当用到**宏**时，需要在main.rs中在需要用到宏的mod`mod 模块名称;`上面加上`#[macro_use]`才能使用宏（一个#[macro_use]下面可连续接多个mod声明）
3. 在宏里面，若用到了其他函数/结构等，需要加上$crate::模块所在位置::{用到的东西}，因为虽然宏是在当前文件的当前模块中定义，但**宏可以在其他模块中展开**
