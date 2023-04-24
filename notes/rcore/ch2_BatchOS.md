# 布局图

![[ch2_batch 2023-04-03 15.03.26.excalidraw|600]]


# 一、user

>user是在与os同级目录下的一个library类型的cargo（cargo new user --lib），包含的是**用户库**和**用户程序**  
>user下的很多内容与ch1的libos类似，但libos是直接在0x80200000中加载_entry然后执行rust_main中的内容（包括各种直接使用sbi_call），但batchos是在0x80400000加载lib.rs中的内容，然后执行main（/bin下对应的用户程序），是处在**用户态**的

### 1. syscall.rs

>这里是用户态的syscall，它会调用 `ecall` 来进入内核态调用内核态的 `syscall` 

> `ecall` 的作用：① 从user mode -> supervisor mode; ② 将程序计数器的值保存到SEPC寄存器中；③ 跳转到STVEC寄存器指向的指令

1. 实现了 `syscall()` 函数，其中调用内嵌汇编 `ecall` 以及传参 `in`，通过ecall来trap进内核态，是**用户态转入内核态的入口**
2. 根据 `syscall()` 实现了 `sys_write` 和 `sys_exit`

### 2. console.rs

>这里的 `print!` 和 `println!` 宏调用的 `print` 调用的 `Stdout.write_str` 中**不再是ch1中调用sbi的console_putchar->sbi_call->ecall来直接输出**，而是转化为syscall的 `ecall` 来进入内核调用**内核**的 `Stdout.write_str` 的sbi的 `sbi_rt::legacy::console_putchar`

1. 实现了宏 `print!` 和宏 `println!` 
	* 通过定义结构体Stdout **->** 补充trait Write的 `write_str()` 函数到Stdout中 **->** 实现 `print()` 函数 **->** 定义宏 `print` 和 `println`
	* 函数调用：print!() / println!() **->** print() **->** Stdout.write_fmt **->** Stdout.write_str() **->** write() **->** sys_write() **->** syscall() **->** ecall
	* 参数转变：($fmt $(, $(\$arg)+)?) **->** (fmt::Arguments) **->** (fmt::Arguments) **->** (&str) **->** (&[u8]) **->** (&[u8]) **->** ([fd, ptr, len])

### 3. lib.rs

>用户态的主函数，其所有的内容都会先和内核os一起生成一个bin文件：os的_entry在0x80200000中；而用户态的会先被加到.data段下（这时还没有加到0x80400000中），然后再被内核加载到0x80400000中

1. ·\_start()·：库文件的入口(0x80400000)，清除.bss段和调用对应的用户main函数(user/bin下的)
2. 包装了`pub fn write()` 和 `pub fn exit()` 供外部使用


# 二、Batch OS（kernel）

>操作系统和应用程序需要被放置到 *同一个可执行文件*，应用放置采用“**静态绑定**”的方式，而操作系统加载应用则采用“**动态加载**”的方式

### 1. sbi.rs

* `console_putchar()` 、`console_getchar()`、`shutdown()` 都是**真正发挥作用**的函数，都调用了**SBI call**来进行操作

### 2. console.rs

* 定义了宏 `print!` 和 `println!`（与ch1的一样）

### 3. lang-items.rs

* 提供了 `panic` 的定义（移除了标准可std，就要自己定义）

### 4. batch.rs

>batch OS的核心部分，提供了不同用户apps的切换

1. `KERNEL_STACK` 和 `USER_STACK` 两个全局变量的定义（来自结构体KernelStack和UserStack）
2. `AppManager结构` 用来管理所有的apps
	1. 可变全局变量 `APP_MANAGER` （经过了sync文件夹下定义的结构的包装）
	2. `print_app_info()` ：打印所有apps（.bin）所在的地址（在.data段）
	3. `load_app()` 将一个新的app的内容（.data位置下的.bin文件内容）加载到0x80400000地址，填充AppManager信息
3. `init()` 初始化打印全局变量APP_MANAGER的所有apps的信息
4. **`run_next_app()`** ：加载一个**新的TrapContext**（上下文信息：sp为用户栈的top，sepc为0x80400000）到KERNEL_STACK顶部，并传入内核栈的位置来 *调用__restore回到用户空间*

### 5. link_app.S

* 包含了**所有app的内容(.bin文件)**，同时用一个**数组_num_app**来组织（第一个元素为用户apps数量，后面的为所有apps的起始地址，最后一个元素为最后一个app终止地址）
* 由与os文件夹同级的文件 `build.rs` 生成

## 6. syscall文件夹下

* `mod.rs` 实现了内核syscall的通用接口，在其他文件具体实现

## 7. sync文件夹下

* `mod.rs` 提供了UPSafeCell结构的接口，可以包装一个结构在**全局可变**使用

## 8. trap文件夹下

### ①. context.rs

* 定义了**TrapContext结构**，里面包含了存储上下文信息的必要内容（32个寄存器，sstatus，sepc）；
* 提供了 `app_init_context()` 来初始化一个TrapContext

### ②. trap.S

* `trap.S` 提供了用户态和内核态切换的过渡过程
	1. `__alltraps` ：用户调用**ecall跳转到此处**，切换内核栈并保存上下文到内核栈
	2. `__restore` ：内核的**run_next_app跳转到此处**，从内核栈的TrapContext中去除内容恢复用户态的上下文

### ③. mod.rs

* trap在保存了上下文后跳转到trap_handler处理
	1. `init()` 写入__alltraps的地址到stvec寄存器中，使得ecall可以跳转到__alltraps处
	2. `trap_handler()` __alltraps保存了上下文后会跳转到此处，处理各种syscall、exception、interrupt

### 9. entry.asm

* 在 **\_start中初始化内核的栈**，然后再**call rust_main执行内核**

### 10. linker-qemu.ld

* 链接文件：将 **.text.entry加到开头0x80200000** ；将各个**用户文件.bin加到.data段**（包含了各个段，从_start开始到main）（后面再动态加载到地址0x80400000中运行） 


