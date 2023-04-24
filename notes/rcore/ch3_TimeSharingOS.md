> **多道程序OS**：将多个应用程序 *加载到内存不同位置*，一个app执行完后才到另一个；  
> **多道程序与协作式调度OS**：不同应用程序可以 *主动让出*（在等待I/O时）CPU，让给另一个app执行；  
> **分时与抢占式OS**：每个应用程序分成 *多个时间片*，时钟中断来 *抢占切换* app

![[Pasted image 20230410113820.png]]


![[ch3_TimeSharingOS 2023-04-09 22.12.53.excalidraw|600]]


# 时钟中断

### 中断屏蔽

如果这个Trap属于异常，那么绝对不会被屏蔽；如果是中断，设中断前CPU特权级为A，中断需要在特权级B处理，有如下几种情况：

1.  B高于A，CPU原有的指令执行**被抢占**，Trap到B去处理中断；
2.  B与A相同，设此特权级为x，此时当且仅当xstatus.xie和xie的中断对应使能位均为1该中断才不会被屏蔽，并会在特权级x处理；
3.  B低于A，该中断**被屏蔽**

### 中断嵌套

默认情况下，当中断产生并进入某个特权级之后，在中断处理的过程中**同特权级的中断都会被屏蔽**。中断产生后，硬件会完成如下事务：

-   当中断发生时，`sstatus.sie` 字段会被保存在 `sstatus.spie` 字段中，同时把 `sstatus.sie` 字段置零，这样软件在进行后续的中断处理过程中，所有 S 特权级的中断都会被屏蔽；
-   当软件执行中断处理完毕后，会执行 `sret` 指令返回到被中断打断的地方继续执行，硬件会把 `sstatus.sie` 字段恢复为 `sstatus.spie` 字段内的值。

也就是说，如果不去手动设置 `sstatus` CSR ，在只考虑 S 特权级中断的情况下，是 *不会* 出现 **嵌套中断** (Nested Interrupt) 的

### 内核中断

* 我们的内核不会被 S 特权级中断所打断，这是因为 CPU 在 S 特权级时， `sstatus.sie` 总为 0 。

# 一、user

* user下的linker.ld的内容还是ch2下的内容，但是在生成app的.bin执行文件时，会通过**build.py来动态修改linker.ld的基地址**(0x80400000)来 *加上不同的偏移*，再调用cargo build生成不同基地址的.bin文件，最后再将linker.ld文件的基地址改回0x80400000
* user下其他内容和ch2类似（多了sys_yield、sys_get_time）

### print!

* 用户态下：print!() -> write() -> sys_write() -> syscall()（ecall）（接下来进入内核态）-> sys_write() -> print!() -> console_putchar() -> sbi_rt::legacy::console_putchar()
* 内核态下：print!() -> console_putchar() -> sbi_rt::legacy::console_putchar()

# 二、TimeSharingOS

### config.rs

* 各种常量的定义（用户栈大小、内核栈大小、APP相关等）

### console.rs

* `print!()` 和 `println!()` 宏的实现（会调用console_putchar() -> sbi_rt::legacy::console_putchar()

### loader.rs

1. `KernelStack`  和 `UserStack` 结构体 -> 全局变量 `KERNEL_STACK` 和 `USER_STACK` （二者都是有MAX_APP_NUM个KERNEL_STACK_SIZE大小的数组）
2. KernelStack : : `push_context()` 将TrapContext的内容放到内核栈顶
3. `load_apps()` 将所有apps加载到对应的base_address + app_id \* APP_SIZE_LIMIT
4. `init_app_cx()` 将对应app_id的TrapContext放入对应的内核栈中

### timer.rs

1. `get_time()` 返回当前时钟数目
2. `get_time_ms()` 返回当前时间（ms）
3. `set_next_trigger()` 设置下一次中断发生

### sbi.rs

>sbi的调用有两种实现方式，一种是使用sbi_rt::对应的内容；另一种是包装一个sbi_call()来调用ecall到sbi中使用

1. `console_putchar()` 输出字符
2. `set_timer()` 设置下一个时钟中断发生的时间
3. `shutdown()` 使用sbi_rt的系统退出

### link_app.S

* 包含了**所有app的内容(.bin文件)**，同时用一个**数组_num_app**来组织（第一个元素为用户apps数量，后面的为所有apps的起始地址，最后一个元素为最后一个app终止地址）
* 由与os文件夹同级的文件 `build.rs` 生成

### linker-qemu.ld

* 内核的链接文件（同ch2）

### main.rs

1. `clear_bss()` 清除.bss段的内容
2. `trap::init()` 初始化让stvec指向__alltraps
3. `loader::load_apps()` 将所有的apps加载到对应的内存位置
4. `task::run_first_task()` 调用\_\_switch切换到第一个app的TaskContext

## syscall文件夹下（各种系统调用）

### ① mod.rs

* 内核 `syscall()` 通用接口，调用了各种sys_xxx

### ② fs.rs

* `sys_write()` ：如果是写到屏幕就调用 `print!宏` ，否则就 `panic!`

### ③ process.rs

1. `sys_exit()`: 调用task下的 `exit_current_and_run_next()` 函数
2. `sys_yield()`: 调用task下的 `suspend_current_and_run_next()`函数

## task文件夹下

### ① task.rs

* 包含了结构 `TaskControlBlock` 和 枚举 `TaskStatus` 的定义

### TCB

>**用户和内核**之间的切换是通过 *内核栈里面的TrapContext* 来存储上下文进行切换的；  
>**不同内核**(用户进程)之间的切换是通过在内核里面的 *TCB* (TaskControlBlock)来存储上下文，并通过__switch来切换内核栈（以及寄存器、ra）来切换的。

* 在ch2中，第一次进入用户态的时候是通过：在内核栈中存储用户态的各种信息（ra、sp、寄存器），并通过传入内核栈地址来调用__restore来恢复上下文回到用户态中
* 因为这里回到用户态除了包括ch2的TrapContext恢复，还包括内核之间的切换，所以要先__switch再__restore；
	1. 在不同app执行期间切换就如上所述
	2. 但第一次进入app时就需要正确初始化TCB中的内容：将ra <- \_\_restore，sp <- 对应的内核栈；这样__switch后就会跳转到__restore执行，就实现了同ch2的直接跳到__restore一样


### ② context.rs

* 包含了结构 `TaskContext` 的定义，补充了方法 `zero_init()` 返回内容都是0的TaskContext，`goto_restire()` 返回ra: \_\_restore, sp: kstack_ptr, s: \[0; 12]的TaskContext

### ③ switch.rs 和 switch.S

* switch.rs：内嵌了switch.S，该文件只是包装了switch.S汇编函数为一个rust函数：\_\_switch(current_task_cx_ptr, next_task_cx_ptr)
* switch.S：实现了`__switch` 汇编函数：保存当前TC到current_task_cx_ptr，取出next_task_cx_ptr到寄存器
* 注意：这里 `__switch` 返回后**直接返回到next_task_cx_ptr中的ra寄存器**位置
* `__switch` 是一个用汇编代码写的特殊函数，它不会被 Rust/C 编译器处理，所以我们需要在 `__switch` 中手动编写保存 `s0~s11` 的汇编代码。 不用保存其它寄存器是因为：其它寄存器中，属于调用者保存的寄存器是由编译器在高级语言编写的调用函数中自动生成的代码来完成保存的；还有一些寄存器属于临时寄存器，不需要保存和恢复

### ④ mod.rs

1. `TaskManager` 结构（num_app和inner(可变的TaskManagerInner)）、`TaskManagerInner`（tasks(TCB数组)，current_task）
2. 全局变量**TASK_MANAGER**：初始化所有的TCB的task_cx为goto_restore
3. TaskManager下的函数：
	1. `run_first_task()`：用一个临时TaskContext作为current和tasks\[0]作为next来__switch切换到第一个app的内核
	2. `mark_current_suspended()` 和 `mark_current_exited()` 分别设置当前task状态为 'Ready' 和 'Exited'
	3. `find_next_task()`：找到下一个 'Ready' 的task
	4. `run_next_task()`：若找到下一个task，修改inner信息并__switch到下一个task；否则所有task完成，exit_success()
4. 全局函数：
	1. `run_first_task()`：执行第一个app，TASKMANAGER.1
	2. `suspend_current_and_run_next()`：暂停当前并切换下一个，TASKMANAGER.2&4
	3. `exit_current_and_run_next()`：结束当前并切换下一个，TASKMANAGER.2&4

## 8. trap文件夹下

### ①. context.rs

* 定义了 `TrapContext` 结构，里面包含了存储上下文信息的必要内容（32个寄存器，sstatus，sepc）；
* 提供了 `app_init_context()` 来初始化一个TrapContext

### ②. trap.S

* `trap.S` 提供了用户态和内核态切换的过渡过程
	1. `__alltraps` ：用户调用**ecall跳转到此处**，切换内核栈并保存上下文到内核栈
	2. `__restore` ：内核的**run_next_app跳转到此处**，从内核栈的TrapContext中去除内容恢复用户态的上下文

### ③. mod.rs

* trap在保存了上下文后跳转到trap_handler处理
	1. `init()` 写入__alltraps的地址到stvec寄存器中，使得ecall可以跳转到__alltraps处
	2. `trap_handler()` __alltraps保存了上下文后会跳转到此处，处理各种syscall、exception、interrupt
	3. `enable_timer_interrupt()` 设置sie::set_stimer()，使得 S 特权级时钟中断不会被屏蔽