![[Pasted image 20230427103029.png]]

![[ch5_Process 2023-04-27 10.30.45.excalidraw|600]]
# 一、用户空间 User Sapce

## 1. 用户初始程序 initproc

用户初始程序会先调用 `fork()` 创建**子进程来 `exec()`** "user_shell" 来提供shell与用户交互; 而**该初始程序会不断等待所有的子进程退出**(处于死循环)

```rust
// user/src/bin/initproc.rs
#[no_mangle]
fn main() -> i32 {
    if fork() == 0 {
        exec("user_shell\0");
    } else {
        loop {
            let mut exit_code: i32 = 0;
            let pid = wait(&mut exit_code);
            if pid == -1 {
                yield_();
                continue;
            }
            println!(
                "[initproc] Released a zombie process, pid={}, exit_code={}",
                pid,
                exit_code,
            );
        }
    }
    0
}
```

## 2. shell 程序

> initproc初始化程序会调用 `fork()` 和 `exec()` 来执行shell来读取用户输出执行对应的文件（所以这些**文件都已经提前编译好同内核一起加载到一个可执行文件**中，在内核的.data段里面，shell只是选择地将他们提出来调用）

shell 程序通过不断 `getchar()` 从屏幕中读取字符，根据字符类型来处理（普通字符就记录在字符串中，回车/换行就 `fork()` 和 `exec()` 执行对应子程序, 然后shell进程就一直等待子进程结束）

```rust
// user/src/bin/user_shell.rs
const LF: u8 = 0x0au8;    // 换行
const CR: u8 = 0x0du8;    // 回车
const DL: u8 = 0x7fu8;    // 删除
const BS: u8 = 0x08u8;    // 退格

#[no_mangle]
pub fn main() -> i32 {
    println!("Rust user shell");
    let mut line: String = String::new();
    print!(">> ");
    loop {
        let c = getchar();
        match c {
	        // 执行文件
            LF | CR => {
                println!("");
                if !line.is_empty() {
                    line.push('\0');
                    let pid = fork();
                    if pid == 0 {
                        // child process
                        if exec(line.as_str()) == -1 {
                            println!("Error when executing!");
                            return -4;
                        }
                        unreachable!();
                    } else {
                        let mut exit_code: i32 = 0;
                        let exit_pid = waitpid(pid as usize, &mut exit_code);
                        assert_eq!(pid, exit_pid);
                        println!(
                            "Shell: Process {} exited with code {}",
                            pid, exit_code
                        );
                    }
                    line.clear();
                }
                print!(">> ");
            }
            // 回退一格
            BS | DL => {
                if !line.is_empty() {
                    print!("{}", BS as char);
                    print!(" ");
                    print!("{}", BS as char);
                    line.pop();
                }
            }
            // 继续读入文件名
            _ => {
                print!("{}", c as char);
                line.push(c as char);
            }
        }
    }
}
```

# 二、系统调用

> **用户态** `*()` -> `sys_*()` -> `syscall()` -> `ecall` 进入**内核态**

## 1. fork

**用户态** `fork()` -> `sys_fork()` -> `syscall()` -> `ecall` 进入**内核态**

* 创建一个状态几乎相同（用户态空间的所有内容、寄存器的值都相同，除了a0返回值的寄存器不同）的子进程，但他们的地址空间是独立的两份
* 为新进程**提供地址空间**

## 2. wait / waitpid

> **等待**子进程退出并**回收**其剩下所有资源

* `wait()` 是当**任意**一个子进程退出后就返回exit_pid, 否则`yield_()`切换
* `waitpid()` 是**指定**的子进程退出后就返回exit_pid, 否则`yield_()`切换
* `wait()` 和 `waitpid()` 都是调用 `sys_waitpid()` ，不过前者是传-1作为pid参数，代表任意子进程，后者是传特定pid作为参数，指定该子进程

* 进程 `exit` 退出后**回收部分资源**，而不能直接将所有资源全部回收（如果进程exit在内核中执行的时候，会用到内核栈等资源，如果直接回收放置的物理页帧，会导致系统调用出问题），需要等待父进程 `waitpid` 来**回收剩下的资源**

## 3. exec

* 通过替换文件内容来执行不同的文件

## 4. read / write

* 通过文件 *描述符fd* 和 *用户空间的buffer* 来读取/写入文件

* `getchar()` 向STDIN读取一个字符


# 三、进程标识符 Pid 与 内核栈

## 1. Pid分配器PidAllocator

> pid的分配类似于物理页号的分配，都是一个数值被包装起来，被一个全局分配器分配

* PidHandle <-> PhysPageNum
* PidAllocator <-> StackFrameAllocator
* 但物理页帧的分配多了一层中间的FrameTracker对PhysPageNum的包装，于是Drop自动回收是对**FrameTracker的自动回收**，分配用到的也是FrameTracker；而Pid中Drop自动回收是对**PidHandle的自动回收**，分配用到的是PidHandle

* PidHandle结构的值即为pid

```rust
// os/src/task/pid.rs

// pid的所有权被PidHandle拥有
pub struct PidHandle(pub usize);

// pid的分配
pub fn pid_alloc() -> PidHandle {
    PID_ALLOCATOR.exclusive_access().alloc()
}

// 自动回收pid
impl Drop for PidHandle {
    fn drop(&mut self) {
        PID_ALLOCATOR.exclusive_access().dealloc(self.0);
    }
}
```

* pid**分配器PidAllocator**

```rust
// os/src/task/pid.rs
// 类比于StackFrameAllocator
struct PidAllocator {
    current: usize,
    recycled: Vec<usize>,
}

impl PidAllocator {
    pub fn new() -> Self {
        PidAllocator {
            current: 0,
            recycled: Vec::new(),
        }
    }
    // 分配一个PidHandle
    pub fn alloc(&mut self) -> PidHandle {
        if let Some(pid) = self.recycled.pop() {
            PidHandle(pid)
        } else {
            self.current += 1;
            PidHandle(self.current - 1)
        }
    }
    // 回收一个Pid: usize(会被PidHandle自动调用Drop回收)
    pub fn dealloc(&mut self, pid: usize) {
        assert!(pid < self.current);
        assert!(
            self.recycled.iter().find(|ppid| **ppid == pid).is_none(),
            "pid {} has been deallocated!", pid
        );
        self.recycled.push(pid);
    }
}

lazy_static! {
	// 全局PID_ALLOCATOR分配器
    static ref PID_ALLOCATOR : UPSafeCell<PidAllocator> = unsafe {
        UPSafeCell::new(PidAllocator::new())
    };
}
```

## 2. 内核栈

> 之前的内核栈都是直接根据**app的顺序编号**来分配对应的内核栈的，现在的内核栈是根据**pid的编号**来在对应的位置分配的；
> 且之前的内核栈一直存在直到内核空间回收，现在内核栈的所有权在KernelStack(pid)中，KernelStack回收就直接回收对应的内核栈

* 在创建内核栈的时候除了**分配一页物理页帧并映射**到内核空间中之外，还会**创建一个结构KernelStack**{ pid: usize }来管理着对应的内核栈的所有权，当KernelStack回收时会调用 *Drop* 来调用对应的函数回收内核空间中的该内核栈

```rust
// os/src/task/pid.rs
// 创建内核栈时会返回的结构，管理着内核栈的所有权
pub struct KernelStack {
    pid: usize,
}

// 返回对应pid的内核栈的虚拟地址(高地址空间)的栈顶和栈底(直接就是顶部和底部)
pub fn kernel_stack_position(app_id: usize) -> (usize, usize) {
    let top = TRAMPOLINE - app_id * (KERNEL_STACK_SIZE + PAGE_SIZE);
    let bottom = top - KERNEL_STACK_SIZE;
    (bottom, top)
}

impl KernelStack {
	// 分配一个新的内核栈: 物理页帧+映射+KernelStack
    pub fn new(pid_handle: &PidHandle) -> Self {
        let pid = pid_handle.0;
        let (kernel_stack_bottom, kernel_stack_top) = kernel_stack_position(pid);
        KERNEL_SPACE
            .exclusive_access()
            .insert_framed_area(
                kernel_stack_bottom.into(),
                kernel_stack_top.into(),
                MapPermission::R | MapPermission::W,
            );
        KernelStack {
            pid: pid_handle.0,
        }
    }
    // 自动回收物理页帧和映射关系
    fn drop(&mut self) {
        let (kernel_stack_bottom, _) = kernel_stack_position(self.pid);
        let kernel_stack_bottom_va: VirtAddr = kernel_stack_bottom.into();
        KERNEL_SPACE
            .exclusive_access()
 .remove_area_with_start_vpn(kernel_stack_bottom_va.into());
    }
    // 将一个T类型的结构放到栈顶
    pub fn push_on_top<T>(&self, value: T) -> *mut T where
        T: Sized, {
        let kernel_stack_top = self.get_top();
        let ptr_mut = (kernel_stack_top - core::mem::size_of::<T>()) as *mut T;
        unsafe { *ptr_mut = value; }
        ptr_mut
    }
    // 获得栈顶(最顶部)
    pub fn get_top(&self) -> usize {
        let (_, kernel_stack_top) = kernel_stack_position(self.pid);
        kernel_stack_top
    }
}
```



# 四、进程控制块PCB 与 任务管理器TaskManager

> 前面章节中的任务管理器 `TaskManager` 不仅负责管理所有的任务，还维护着 CPU 当前在执行哪个任务，这里将其拆分成两个管理模块——**任务管理器TaskManager**和**处理器管理结构Processor**

## 1. TCB (PCB)

> 这里用TaskControlBlock的名字代替ProcessControlBlock，让其承担PCB的功能，相对于第四章TCB又多了些内容

* TCB中的所有内容包含了一个**应用所对应的所有东西的所有权**，包括pid、内核栈、应用地址空间
* 所有的TCB都被放到**内核堆**中，所以才可以用Weak和Arc来指向对应的TCB

```rust
// os/src/task/task.rs
pub struct TaskControlBlock {
    // immutable不可变的
    // 1.管理着pid的所有权
    pub pid: PidHandle,
    // 2.管理着kernel_stack内核栈的所有权
    pub kernel_stack: KernelStack,
    // mutable可变的(需要用UPSafeCell)
    inner: UPSafeCell<TaskControlBlockInner>,
}

pub struct TaskControlBlockInner {
    pub trap_cx_ppn: PhysPageNum,
    pub base_size: usize,
    pub task_cx: TaskContext,
    pub task_status: TaskStatus,
    // 3.管理着应用的页表页帧和实际物理页帧的所有权
    pub memory_set: MemorySet,
    // 这里用Weak指向父进程,不会增加引用计数
    pub parent: Option<Weak<TaskControlBlock>>,
    // 这里用Arc指向子进程，会增加引用计数
    // 当子进程结束时其对应TCB以及资源还未完全Drop，需要等到父进程释放后其TCB引用计数才为0才能完全释放
    pub children: Vec<Arc<TaskControlBlock>>,
    // 进程的退出码
    pub exit_code: i32,
}
```

## 2. TaskManager

> 任务管理器自身仅负责管理所有任务(进程)，不包含当前CPU执行的管理

* TaskManager中仅包含**一个元素为TCB的引用的双端队列**，从原来的直接包含很多内容(包括TCB)占用大量空间到现在只包含一个简单的双端队列
* 因为TCB需要经常放入/取出，直接移动会产生大量开销，而现在用智能指针可以减少开销

```rust
// os/src/task/manager.rs
// 任务管理器
pub struct TaskManager {
    ready_queue: VecDeque<Arc<TaskControlBlock>>,
}

/// A simple FIFO scheduler.
impl TaskManager {
    pub fn new() -> Self {
        Self { ready_queue: VecDeque::new(), }
    }
    // 将任务加到队列中
    pub fn add(&mut self, task: Arc<TaskControlBlock>) {
        self.ready_queue.push_back(task);
    }
    // 取出队列中的第一个任务
    pub fn fetch(&mut self) -> Option<Arc<TaskControlBlock>> {
        self.ready_queue.pop_front()
    }
}

lazy_static! {
	// 全局任务管理器TASK_MANAGER
    pub static ref TASK_MANAGER: UPSafeCell<TaskManager> = unsafe {
        UPSafeCell::new(TaskManager::new())
    };
}

// 加入新的任务TCB
pub fn add_task(task: Arc<TaskControlBlock>) {
    TASK_MANAGER.exclusive_access().add(task);
}

// 取出第一个任务
pub fn fetch_task() -> Option<Arc<TaskControlBlock>> {
    TASK_MANAGER.exclusive_access().fetch()
}
```

# 五、处理器管理结构 Processor

> 处理器管理结构 `Processor` 负责从任务管理器 `TaskManager` 中分出去的**维护 CPU 状态**的职责

## 1. Processor

* `Processor` 是描述 **CPU 执行状态** 的数据结构。在单核CPU环境下，我们仅创建单个 `Processor` 的全局实例 `PROCESSOR`

```rust
// os/src/task/processor.rs
pub struct Processor {
	// 当前的TCB指针
    current: Option<Arc<TaskControlBlock>>,
    // 当前处理器上的idle控制流的任务上下文(这里的上下文指的是在该处理器初始化的时候，在内核中还未进入用户空间的一个控制流，其他的TCB中的TaskContext都是对应的应用的控制流)
    idle_task_cx: TaskContext,
}

impl Processor {
    pub fn new() -> Self {
        Self {
            current: None,
            idle_task_cx: TaskContext::zero_init(),
        }
    }
}

lazy_static! {
	// 管理一个CPU的全局PROCESSOR
    pub static ref PROCESSOR: UPSafeCell<Processor> = unsafe {
        UPSafeCell::new(Processor::new())
    };
}

// ------------------ 与当前任务有关-----------------------
impl Processor {
	// 取出当前任务TCB的引用(取出后Processor.current里面为None)
    pub fn take_current(&mut self) -> Option<Arc<TaskControlBlock>> {
        self.current.take()
    }
    // 复制一份引用，不会改变Processor.current
    pub fn current(&self) -> Option<Arc<TaskControlBlock>> {
        self.current.as_ref().map(|task| Arc::clone(task))
    }
}

pub fn take_current_task() -> Option<Arc<TaskControlBlock>> {
    PROCESSOR.exclusive_access().take_current()
}

pub fn current_task() -> Option<Arc<TaskControlBlock>> {
    PROCESSOR.exclusive_access().current()
}

pub fn current_user_token() -> usize {
    let task = current_task().unwrap();
    let token = task.inner_exclusive_access().get_user_token();
    token
}

pub fn current_trap_cx() -> &'static mut TrapContext {
current_task().unwrap().inner_exclusive_access().get_trap_cx()
}
```

## 2. 任务调度的 idle 控制流

> 每个Processor中都有一个对应的与TCB中应用的TaskContext不同的**内核任务调度控制流**，该控制流负责调度所有应用

* `run_tasks()` 内核任务调度控制流调度进程
* 每次切换任务的时候都会根据 `schedule()` 切换上下文跳转到这里
	1. 在进入 `run_tasks()` 之前(也在`schedule()`之前)就已经从Processor中取出来当前 current 任务
	2. 进入后先 `fetch_task()` 从TaskManager中取出了一个task（使得该task不会被TaskManager引用），并将其加入到processor.current中（对于**初始进程INITPROC**除了在 *全局变量* 中引用就是在 *这里* 被引用；**其他进程**会被 *父进程引用* 和*processor.current或者TaskManager引用* ） ^fetchTask230427
	3. 再调用 `__switch()` 切换上下文

```rust
// os/src/task/processor.rs
// 该处理器在内核初始化完成之后会调用run_tasks()函数进入idle控制流
pub fn run_tasks() {
    loop {
        let mut processor = PROCESSOR.exclusive_access();
        // 2.从TaskManager中取出task
        if let Some(task) = fetch_task() {
            let idle_task_cx_ptr = processor.get_idle_task_cx_ptr();
            // access coming task TCB exclusively
            let mut task_inner = task.inner_exclusive_access();
            let next_task_cx_ptr = &task_inner.task_cx as *const TaskContext;
            task_inner.task_status = TaskStatus::Running;
            // stop exclusively accessing coming task TCB manually
            drop(task_inner);
            processor.current = Some(task);
            // stop exclusively accessing processor manually
            drop(processor);
            unsafe {
	            // 将当前上下文(内核调度控制流)保存到processor中，
	            // 将下一个TCB的TaskContext的内容取出切换到应用的控制流中
                __switch(
	                // idle_task_cx开始的值不重要,因为会在这里保存
                    idle_task_cx_ptr,
                    next_task_cx_ptr,
                );
            }
        }
    }
}

impl Processor {
	// 获得当前处理器的idle控制流
    fn get_idle_task_cx_ptr(&mut self) -> *mut TaskContext {
        &mut self.idle_task_cx as *mut _
    }
```

* `schedule()` 当前应用进程控制流(包括其内核控制流)切换到内核调度控制流

```rust
// os/src/task/processor.rs
pub fn schedule(switched_task_cx_ptr: *mut TaskContext) {
    let mut processor = PROCESSOR.exclusive_access();
    let idle_task_cx_ptr = processor.get_idle_task_cx_ptr();
    drop(processor);
    unsafe {
	    //保存上下文，取出调度控制流
        __switch(
            switched_task_cx_ptr,
            idle_task_cx_ptr,
        );
    }
}
```

* 之前的任务切换(调度)在内核初始化后使用 `__switch(tmp_cx_ptr, next_task_cx_ptr)` 来 *丢弃当前内核控制流，加载下一个应用控制流*；并在任务切换时使用的是直接 `__switch(now_task_cx_ptr, next_task_cx_ptr)` *保存当前应用控制流，加载下一个应用控制流*  (这也使得原初始内核用的栈在切换后不会再用，用的是对应内核栈)
* 现在将之前丢弃的内核调度控制流保存下来到处理器管理器Processor中，**任务切换时会回到该控制流**(用回原内核启动时用的栈)
	* 这样的话，调度相关的数据**不会出现在进程内核栈**上，也使得调度机制对于换出进程的Trap执行流是不可见的，它在决定换出的时候只需调用schedule而无需操心调度的事情。从而**各执行流的分工更加明确**了，虽然带来了更大的开销。



# 六、进程的生成过程

## 1. 初始进程的创建

* **初始进程的TCB**的创建（一个全局引用）

```rust
// os/src/task/mod.rs
lazy_static! {
	// 根据initproc的ELF文件创建一个初始进程的TCB的全局引用INITPROC
    pub static ref INITPROC: Arc<TaskControlBlock> = Arc::new(TaskControlBlock::new(get_app_data_by_name("initproc").unwrap())
    );
}

// 将INITPROC的引用加入到TASK_MANAGER任务管理器中
pub fn add_initproc() {
    add_task(INITPROC.clone());
}
```

* **TCB的创建**(与第四章的类似, 多了pid和一些TCB的成员) ^tcb230426

```rust
// os/src/task/task.rs

impl TaskControlBlock {
	pub fn new(elf_data: &[u8]) -> Self {
	    // 1.from_elf()创建一个完整的用户空间(包括program/trampoline/TrapContext/user stack)
	    let (memory_set, user_sp, entry_point) = MemorySet::from_elf(elf_data);
	    let trap_cx_ppn = memory_set
	        .translate(VirtAddr::from(TRAP_CONTEXT).into())
	        .unwrap()
	        .ppn();
	    // 2.alloc a pid and a kernel stack in kernel space
	    let pid_handle = pid_alloc();
	    let kernel_stack = KernelStack::new(&pid_handle);
	    let kernel_stack_top = kernel_stack.get_top();
	    // 3.创建TCB(里面的TaskContext的返回地址是trap_return)
	    let task_control_block = Self {
	        pid: pid_handle,
	        kernel_stack,
	        inner: unsafe { UPSafeCell::new(TaskControlBlockInner {
	            trap_cx_ppn,
	            base_size: user_sp,
	            task_cx: TaskContext::goto_trap_return(kernel_stack_top),
	            task_status: TaskStatus::Ready,
	            memory_set,
	            parent: None,
	            children: Vec::new(),
	            exit_code: 0,
	        })},
	    };
	    // 4.prepare TrapContext in user space
	    let trap_cx = task_control_block.inner_exclusive_access().get_trap_cx();
	    *trap_cx = TrapContext::app_init_context(
	        entry_point,
	        user_sp,
	        KERNEL_SPACE.exclusive_access().token(),
	        kernel_stack_top,
	        trap_handler as usize,
	    );
	    task_control_block
	}
}
```

## 2. 进程的切换

* 当应用调用 `sys_yield()`主动交出使用权、本轮时间片用尽或内核因某些原因无法继续处理的时候会调用 `suspend_current_and_run_next`

```rust
// os/src/syscall/process.rs
pub fn sys_yield() -> isize {
    suspend_current_and_run_next();
    0
}

// os/src/trap/mod.rs
#[no_mangle]
pub fn trap_handler() -> ! {
    set_kernel_trap_entry();
    let scause = scause::read();
    let stval = stval::read();
    match scause.cause() {
        Trap::Interrupt(Interrupt::SupervisorTimer) => {
            set_next_trigger();
            suspend_current_and_run_next();
        }
        ...
    }
    trap_return();
}

// -------------------suspend_current_and_run_next---------
// os/src/task/mod.rs
pub fn suspend_current_and_run_next() {
    // 1.从PROCESS中取出当前任务
    let task = take_current_task().unwrap();

    // ---- access current TCB exclusively
    let mut task_inner = task.inner_exclusive_access();
    let task_cx_ptr = &mut task_inner.task_cx as *mut TaskContext;
    // 2.Change status to Ready
    task_inner.task_status = TaskStatus::Ready;
    drop(task_inner);
    // ---- stop exclusively accessing current PCB

    // 3.push back to ready queue.
    add_task(task);
    // 4.jump to scheduling cycle
    schedule(task_cx_ptr);
}
```

## 3. 进程的生成

> 在内核中**手动生成**的进程只有初始进程 `initproc` ，余下所有的进程都是它**直接或间接 fork 出来**的。当一个子进程被 fork 出来之后，它可以调用 `exec()` 系统调用来加载并执行另一个可执行文件。因此， `fork()/exec()` 两个系统调用提供了进程的生成机制。

### 1. fork()复制父进程的内容

对于 `fork()` 生成子进程，需要做的和[[#^tcb230426|TCB的创建]]的几乎一样，但是是**根据已有的TCB来创建**的

1. **用户地址空间**的复制
	* memory_set：trampoline/  TrapContext/user stack/sections
2. **内核相关内容**的复制/修改
	1. pid
	2. 内核栈
	3. TCB（复制得到子进程的TCB，填充当前父进程的TCB）

```rust
// os/src/mm/memory_set.rs
impl MapArea {
	// 从另一个MapArea逻辑段复制除了物理页帧的所有内容
    pub fn from_another(another: &MapArea) -> Self {
        Self {
            vpn_range: VPNRange::new(
                another.vpn_range.get_start(),
                another.vpn_range.get_end()
            ),
            data_frames: BTreeMap::new(),
            map_type: another.map_type,
            map_perm: another.map_perm,
        }
    }
}

impl MemorySet {
	// 1.用户地址空间的复制(包括映射和数据填充)
    pub fn from_existed_user(user_space: &MemorySet) -> MemorySet {
        let mut memory_set = Self::new_bare();
        // map trampoline
        memory_set.map_trampoline();
        // copy data sections/trap_context/user_stack
        for area in user_space.areas.iter() {
            let new_area = MapArea::from_another(area);
            memory_set.push(new_area, None);
            // copy data from another space
            for vpn in area.vpn_range {
                let src_ppn = user_space.translate(vpn).unwrap().ppn();
                let dst_ppn = memory_set.translate(vpn).unwrap().ppn();
                dst_ppn.get_bytes_array().copy_from_slice(src_ppn.get_bytes_array());
            }
        }
        memory_set
    }
}
```

* `TaskContext` 使得当切换到子进程时可以**直接 return 到trap_return中返回到用户空间**；`TrapContext` 与父进程的几乎一样(返回值不同)使得子进程进入用户态和其父进程回到用户态的那一瞬间 **CPU 的状态是完全相同的**（除了返回值）
* 子进程时直接返回到 `trap_return()` 然后返回用户空间；父进程时顺着下去执行完 `fork()`，回到 `trap_handler()`再返回到 `__restore()` 的 

```rust
// os/src/task/task.rs

impl TaskControlBlock {
    pub fn fork(self: &Arc<TaskControlBlock>) -> Arc<TaskControlBlock> {
        // ---- access parent PCB exclusively
        let mut parent_inner = self.inner_exclusive_access();
        // 1.copy user space(include trap context)
        let memory_set = MemorySet::from_existed_user(
            &parent_inner.memory_set
        );
        let trap_cx_ppn = memory_set
            .translate(VirtAddr::from(TRAP_CONTEXT).into())
            .unwrap()
            .ppn();
        //2.(1&2)alloc a pid and a kernel stack in kernel space
        let pid_handle = pid_alloc();
        let kernel_stack = KernelStack::new(&pid_handle);
        let kernel_stack_top = kernel_stack.get_top();
        // 2.(3)创建TCB
        let task_control_block = Arc::new(TaskControlBlock {
            pid: pid_handle,
            kernel_stack,
            inner: unsafe { UPSafeCell::new(TaskControlBlockInner {
                trap_cx_ppn,
                base_size: parent_inner.base_size,
                task_cx: TaskContext::goto_trap_return(kernel_stack_top),
                task_status: TaskStatus::Ready,
                memory_set,
                parent: Some(Arc::downgrade(self)),
                children: Vec::new(),
                exit_code: 0,
            })},
        });
        // 2.(3)修改父进程TCB: add child
        parent_inner.children.push(task_control_block.clone());
        // 2.(3)修改子进程TCB: modify kernel_sp in trap_cx
        // **** access children PCB exclusively
        let trap_cx = task_control_block.inner_exclusive_access().get_trap_cx();
        trap_cx.kernel_sp = kernel_stack_top;
        // return
        task_control_block
        // ---- stop exclusively accessing parent/children PCB automatically
    }
}
```

### 2. 修改进程部分内容使得子进程区别于父进程

* `sys_fork()` 依然时父进程在执行，除了创建了一个新的子进程TCB之外，还会将**子进程TrapContext中的返回值设置为0**

```rust
// os/src/syscall/process.rs
// sys_fork()才是完整地fork了子进程
pub fn sys_fork() -> isize {
    let current_task = current_task().unwrap();
    // 创建新的TCB(即上面两段复制代码)
    let new_task = current_task.fork();
    let new_pid = new_task.pid.0;
    // modify trap context of new_task, because it returns immediately after switching
    let trap_cx = new_task.inner_exclusive_access().get_trap_cx();
    // we do not have to move to next instruction since we have done it before
    // for child process, fork returns 0
    trap_cx.x[10] = 0;  //x[10] is a0 reg
    // add new task to scheduler
    add_task(new_task);
    new_pid as isize
}

// os/src/trap/mod.rs

#[no_mangle]
pub fn trap_handler() -> ! {
    set_kernel_trap_entry();
    let scause = scause::read();
    let stval = stval::read();
    match scause.cause() {
        Trap::Exception(Exception::UserEnvCall) => {
            // jump to next instruction anyway ^cx230426
            let mut cx = current_trap_cx();
            // 再sys_fork之前+4使得子进程也可以回到ecall之后的指令
            cx.sepc += 4;
            // get system call return value
            let result = syscall(cx.x[17], [cx.x[10], cx.x[11], cx.x[12]]);
            // cx is changed during sys_exec, so we have to call it again
            cx = current_trap_cx();
            // 修改当前父进程的返回值
            cx.x[10] = result as usize;
        }
    ...
}
```

### 3. exec() 切换新应用

> `exec` 系统调用使得一个进程能够**加载一个新应用的 ELF 可执行文件**中的代码和数据替换原有的应用地址空间中的内容，并开始执行

1. 根据ELF文件生成新的地址空间并替换到原来的TCB中，使得原有的地址空间生命周期结束，释放里面所有的物理页帧
2. 修改内核的TCB的一些信息
3. 修改新的地址空间的Trap上下文（返回用户空间的地址、用户栈）

```rust
// os/src/task/task.rs

impl TaskControlBlock {
    pub fn exec(&self, elf_data: &[u8]) {
        // 1.memory_set with elf program headers/trampoline/trap context/user stack
        let (memory_set, user_sp, entry_point) = MemorySet::from_elf(elf_data);
        let trap_cx_ppn = memory_set
            .translate(VirtAddr::from(TRAP_CONTEXT).into())
            .unwrap()
            .ppn();

        // **** access inner exclusively
        let mut inner = self.inner_exclusive_access();
        // 2.修改TCB的一些信息
        // substitute memory_set
        inner.memory_set = memory_set;
        // update trap_cx ppn
        inner.trap_cx_ppn = trap_cx_ppn;
        // initialize trap_cx
        let trap_cx = inner.get_trap_cx();
        // 3.修改TrapContext使得返回到对应的应用
        *trap_cx = TrapContext::app_init_context(
            entry_point,
            user_sp,
            KERNEL_SPACE.exclusive_access().token(),
            self.kernel_stack.get_top(),
            trap_handler as usize,
        );
        // **** stop exclusively accessing inner automatically
    }
}
```

* `sys_exec()` 传入应用名字字符串在用户地址空间中的虚拟地址，所以要先根据用户页表**翻译得到名字字符串**，再**得到数据地址**，将数据地址传入到 **`exec()` 修改当前TCB**；
* 若成功则会返回到新的应用的起始地址执行，此时 `sys_exec()` 的返回值没有用，若失败会返回到用户空间中调用`exec()`的地方，此时会得到返回值-1

```rust
// os/src/mm/page_table.rs
// 根据用户页表得到用户虚拟地址对应的物理地址(内核虚拟地址直接映射)
pub fn translated_str(token: usize, ptr: *const u8) -> String {
    let page_table = PageTable::from_token(token);
    let mut string = String::new();
    let mut va = ptr as usize;
    loop {
        let ch: u8 = *(page_table.translate_va(VirtAddr::from(va)).unwrap().get_mut());
        if ch == 0 {
            break;
        } else {
            string.push(ch as char);
            va += 1;
        }
    }
    string
}

// os/src/syscall/process.rs
// 执行sys_exec()的整个过程
pub fn sys_exec(path: *const u8) -> isize {
    let token = current_user_token();
    // 1.翻译名字用户虚拟地址得到物理地址
    let path = translated_str(token, path);
    // 2.获得ELF文件的实际物理地址
    if let Some(data) = get_app_data_by_name(path.as_str()) {
        let task = current_task().unwrap();
        // 3.修改当前TCB
        task.exec(data);
        0
    } else {
        -1
    }
}
```

* 因为 `sys_exec()` 会切换新的用户地址空间，其里面的**TrapContext也改变**了，导致返回到 `trap_handler()` 的时候，在调用 `sys_call()` 之前得到的cx: TrapContext会**失效**，所以在后面要 *重新获得* 一次cx


# 七、进程资源的回收

## 1. 进程的退出

> 应用调用 `sys_exit()` 系统调用主动退出 或者 出错内核终止后，会在内核调用 `exit_current_and_run_next()` 

* `exit_current_and_run_next()` 带有一个退出码作为参数传到TCB中
	1. 从Processor.current中取出task ^takeCurrentTask230427
	2. 修改当前TCB状态为 Zombie
	3. 记录下TCB的exit_code为传入的参数
	4. 将当前进程所有子进程的父进程设置为INITPROC初始进程的子进程，清除子进程Vec列表
	5. 清除当前进程的memory_set的实际物理页帧（充分利用空闲页帧）,页表页帧还未清除
	6. 设置TaskContext为0
	7. `schedule()` 调度下一个应用

```rust
// os/src/mm/memory_set.rs

impl MemorySet {
    pub fn recycle_data_pages(&mut self) {
        self.areas.clear();
    }
}

// os/src/task/mod.rs

pub fn exit_current_and_run_next(exit_code: i32) {
    // 1.take from Processor
    let task = take_current_task().unwrap();
    // **** access current TCB exclusively
    let mut inner = task.inner_exclusive_access();
    // 2.Change status to Zombie
    inner.task_status = TaskStatus::Zombie;
    // 3.Record exit code
    inner.exit_code = exit_code;
    // do not move to its parent but under initproc

	// 4.修改子进程的父进程为INITPROC
    // ++++++ access initproc TCB exclusively
    {
        let mut initproc_inner = INITPROC.inner_exclusive_access();
        for child in inner.children.iter() {
            child.inner_exclusive_access().parent = Some(Arc::downgrade(&INITPROC));
            initproc_inner.children.push(child.clone());
        }
    }
    // ++++++ stop exclusively accessing parent PCB

	// 4.清除子进程列表
    inner.children.clear();
    // 5.deallocate user space
    inner.memory_set.recycle_data_pages();
    drop(inner);
    // **** stop exclusively accessing current PCB
    // drop task manually to maintain rc correctly
    drop(task);
    // 6.we do not have to save task context
    let mut _unused = TaskContext::zero_init();
    // 7.调度
    schedule(&mut _unused as *mut _);
}
```

## 2.父进程回收子进程资源

* 子进程 `exit()` 退出时已经处理了一些内容，剩下的要父进程 `sys_waitpid()` 来回收（pid为对应的子进程pid，-1代表任意）
	1. 查找是否有该子进程，没有就返回-1
	2. 有该子进程，查看其是否是Zombie，不是的话就返回-2
	3. 将该子进程从当前进程的子进程列表中移除并放到临时变量中，结尾时会drop
	4. 将exit_code的内容写回用户空间对应位置
	5. 返回子进程pid
	6. 最后3中的临时变量释放，**子进程引用数为0，释放TCB**（fork()创建子进程时加入了加入了**父进程的子进程列表**, 加入了TaskManager的TCB列表; 当子进程 `exit()` 时首先会将子进程[[#^fetchTask230427|从TaskManager中取出来到processor.current中]]，并[[#^takeCurrentTask230427|在 `exit` 中从current中除去]]，这时就只剩下父进程一个引用了, 而且被取出来放到局部变量后最终会被释放）

```rust
// os/src/syscall/process.rs

/// If there is not a child process whose pid is same as given, return -1.
/// Else if there is a child process but it is still running, return -2.
pub fn sys_waitpid(pid: isize, exit_code_ptr: *mut i32) -> isize {
    let task = current_task().unwrap();
    // find a child process

    // ---- access current TCB exclusively
    let mut inner = task.inner_exclusive_access();
    // 1.查找是否有该子进程，没有就返回-1
    if inner.children
        .iter()
        .find(|p| {pid == -1 || pid as usize == p.getpid()})
        .is_none() {
        return -1;
        // ---- stop exclusively accessing current PCB
    }
    // 2.有该子进程，查看其是否是Zombie，不是的话就返回-2
    let pair = inner.children
        .iter()
        .enumerate()
        .find(|(_, p)| {
            // ++++ temporarily access child PCB exclusively
            p.inner_exclusive_access().is_zombie() && (pid == -1 || pid as usize == p.getpid())
            // ++++ stop exclusively accessing child PCB
        });
    // 3.将该子进程从当前进程的子进程列表中拿出来，结尾时会drop
    if let Some((idx, _)) = pair {
        let child = inner.children.remove(idx);
        // confirm that child will be deallocated after removing from children list(子进程只有一个引用了)
        assert_eq!(Arc::strong_count(&child), 1);
        let found_pid = child.getpid();
        // ++++ temporarily access child TCB exclusively
        let exit_code = child.inner_exclusive_access().exit_code;
        // ++++ stop exclusively accessing child PCB
        // 4.将exit_code的内容写回用户空间
        *translated_refmut(inner.memory_set.token(), exit_code_ptr) = exit_code;
        // 5.返回子进程pid
        found_pid as isize
    } else {
        -2
    }
    // ---- stop exclusively accessing current PCB automatically
}

// user/src/lib.rs
pub fn wait(exit_code: &mut i32) -> isize {
    loop {
        match sys_waitpid(-1, exit_code as *mut _) {
            -2 => { yield_(); }
            // -1 or a real pid
            exit_pid => return exit_pid,
        }
    }
}
```