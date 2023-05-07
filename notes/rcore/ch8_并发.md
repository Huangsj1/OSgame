![[ch8_并发 2023-05-06 14.27.21.excalidraw]]

![[ch8_并发 2023-05-07 11.08.59.excalidraw|480]]

# 用户多线程

> 用户多线程实际上是在内核到用户空间的**一条控制流**中，模仿着内核多线程的模式，将该控制流分给不同的用户线程调度使用
> 用户态下的线程切换需要线程主动让出

## 1. 使用方式

```rust
// 多线程基本执行环境的代码
...
// 多线程应用的主体代码
fn main() {
	// 1.创建基本执行环境
    let mut runtime = Runtime::new();
    // 2.基本执行环境的初始化
    runtime.init();
    // 3.新建用户线程
    runtime.spawn(|| {
        println!("TASK 1 STARTING");
        let id = 1;
        for i in 0..10 {
            println!("task: {} counter: {}", id, i);
            // 5.线程调度切换
            yield_task();
        }
        println!("TASK 1 FINISHED");
    });
    runtime.spawn(|| {
        println!("TASK 2 STARTING");
        let id = 2;
        for i in 0..15 {
            println!("task: {} counter: {}", id, i);
            // 5.线程调度切换
            yield_task();
        }
        println!("TASK 2 FINISHED");
    });
    // 4.主线程调度切换线程
    runtime.run();
}
```

## 2. 线程结构

线程的结构包含：

1. 线程ID
2. 线程的状态
3. 上下文（各种寄存器）
4. 栈

```rust
//线程控制块
struct Task {
	id: usize,           // 线程ID
	stack: Vec<u8>,      // 栈
	ctx: TaskContext,    // 当前指令指针(PC)和通用寄存器集合
	state: State,        // 执行状态
}

struct TaskContext {
	// 15 u64
	x1: u64,  //ra: return address，在将要执行的环境nx1中执行完后要返回的地方
	x2: u64,  //sp
	x8: u64,  //s0,fp
	x9: u64,  //s1
	x18: u64, //x18-27: s2-11
	x19: u64,
	...
	x27: u64,
	nx1: u64, //new return address, 将要执行的环境，即马上返回的地方
}

enum State {
	Available, // 初始态：线程空闲，可被分配一个任务去执行
	Running,   // 运行态：线程正在执行
	Ready,     // 就绪态：线程已准备好，可恢复执行
}

impl Task {
	// 线程Task的初始化
	fn new(id: usize) -> Self {
		Task {
			id,
			stack: vec![0_u8; DEFAULT_STACK_SIZE],
			ctx: TaskContext::default(),
			state: State::Available,
		}
	}
}
```

## 3. 线程运行环境

线程的运行环境 `Runtime` 负责管理线程，包括初始化、调度、终止

```rust
pub struct Runtime {
    tasks: Vec<Task>,
    current: usize,
}

impl Runtime {
	// 新建运行环境
	pub fn new() -> Self {
		// 1.This will be our base task, which will be initialized in the `running` state
		let base_task = Task {
			id: 0,
			stack: vec![0_u8; DEFAULT_STACK_SIZE],
			ctx: TaskContext::default(),
			state: State::Running,
		};

		// 2.We initialize the rest of our tasks.
		let mut tasks = vec![base_task];
		let mut available_tasks: Vec<Task> = (1..MAX_TASKS).map(|i| Task::new(i)).collect();
		tasks.append(&mut available_tasks);

		Runtime {
			tasks,
			current: 0,
		}
	}

	// 运行环境初始化(赋值全局变量)
	pub fn init(&self) {
		unsafe {
			let r_ptr: *const Runtime = self;
			RUNTIME = r_ptr as usize;
		}
	}
}
```

## 4. 线程创建

通过 `runtime.spawn()` 创建一个新的线程添加到队列里面

```rust
impl Runtime {
	pub fn spawn(&mut self, f: fn()) {
		// 1.先找到队列中一个空的位置Available
		let available = self
			.tasks
			.iter_mut()
			.find(|t| t.state == State::Available)
			.expect("no available task.");

		let size = available.stack.len();
		unsafe {
			// 2.设置栈指针为栈顶
			let s_ptr = available.stack.as_mut_ptr().offset(size as isize);
			let s_ptr = (s_ptr as usize & !7) as *mut u8;
			// 3.TaskContext的x1值为将来执行完后需要返回的地方
			available.ctx.x1 = guard as u64;  
			// 4.TaskContext的nx1值为将要(马上)要执行的地方
			available.ctx.nx1 = f as u64;   
			available.ctx.x2 = s_ptr.offset(32) as u64; 
		}
		// 5.设置状态Ready
		available.state = State::Ready;
	}
}

// 终止当前线程(子线程最终会跳到的地方)
fn guard() {
	unsafe {
		let rt_ptr = RUNTIME as *mut Runtime;
		(*rt_ptr).t_return();
	};
}

fn t_return(&mut self) {
	if self.current != 0 {
		// 从队列中删除
		self.tasks[self.current].state = State::Available;
		// 线程调度切换
		self.t_yield();
	}
}
```

## 5. 线程调度切换

应用调用 `yield_task()` 来调用 `runtime.t_tield()` 来切换线程

```rust
// 1.其他线程的函数接口
pub fn yield_task() {
	unsafe {
		let rt_ptr = RUNTIME as *mut Runtime;
		(*rt_ptr).t_yield();
	};
}

// 2.主线程的切换函数
impl Runtime {
   pub fn run(&mut self) -> ! {
		while self.t_yield() {}
		std::process::exit(0);
	}
}

impl Runtime {
	fn t_yield(&mut self) -> bool {
		let mut pos = self.current;
		// 1.找到一个Ready的线程(如果只剩下自己就直接返回)
		while self.tasks[pos].state != State::Ready {
			pos += 1;
			if pos == self.tasks.len() {
				pos = 0;
			}
			if pos == self.current {
				return false;
			}
		}
		// 2.将当前线程设置为Ready
		if self.tasks[self.current].state != State::Available {
			self.tasks[self.current].state = State::Ready;
		}
		// 3.设置下一个线程Running
		self.tasks[pos].state = State::Running;
		let old_pos = self.current;
		self.current = pos;

		unsafe {
			// 4.调用switch()切换上下文
			switch(&mut self.tasks[old_pos].ctx, &self.tasks[pos].ctx);
		}
		self.tasks.len() > 0
	}
}
```

```rust
#[naked]
#[inline(never)]
unsafe fn switch(old: *mut TaskContext, new: *const TaskContext) {
	// a0: old, a1: new
	llvm_asm!("
		// if comment below lines: sd x1..., ld x1..., TASK2 can not finish, and will segment fault
		// 1.保存当前线程上下文
		// 这里x1直接就是当前的返回地址x1(ra)
		sd x1, 0x00(a0)
		sd x2, 0x08(a0)
		sd x8, 0x10(a0)
		sd x9, 0x18(a0)
		sd x18, 0x20(a0) # sd x18..x27
		...
		sd x27, 0x68(a0)
		// 这里的nx1也是当前的返回地址
		sd x1, 0x70(a0)

		// 2.恢复下一个线程的上下文到寄存器中
		// 如果下一个线程是新的，那么这个x1(ra)的值就为guard()线程终止时去的地方
		// 如果下一个线程是旧的，那么这个x1(ra)的值就为上面保存到switch()返回后的地址
		ld x1, 0x00(a1)
		ld x2, 0x08(a1)
		ld x8, 0x10(a1)
		ld x9, 0x18(a1)
		ld x18, 0x20(a1) #ld x18..x27
		...
		ld x27, 0x68(a1)
		// 如果下一个线程是新的,那么这里t0就是新线程开始执行的位置
		// 如果下一个线程是旧的,那么这里t0就是下一个线程的上次切换的的地方(返回到原来的地址，也是同上面x1一样)
		ld t0, 0x70(a1)

		// 跳转到t0的地址
		// 如果下一个线程是新的，那么就会跳转到新线程开始位置执行，并且其返回寄存器x1(ra)为guard()，之后这个线程就是一个普通线程
		// 如果下一个线程是旧的，那么这里就会跳到该线程上次执行切换的switch()的后面，且回去后调用者会恢复x1(ra)为正常的返回值
		jr t0
	"
	:    :    :    : "volatile", "alignstack"
	);
}
```

# 内核多线程

> 进程的主要目的是隔离，线程的主要目的是共享资源
> 本章中如果创建了子进程就不再创建多线程，创建了多线程就不再创建子进程

## 1. 应用的使用

应用程序主线程通过 `thread_create()` 传入函数和参数来创建子线程；主线程可以通过`waittid()` 等待子线程的退出（退出时需要子进程调用修改后的 `exit()` 来回收用户态资源）并回收子线程内核态资源

```rust
// user/src/lib.rs
pub fn thread_create(entry: usize, arg: usize) -> isize {
    sys_thread_create(entry, arg)
}

/// 功能：当前进程创建一个新的线程
/// 参数：entry 表示线程的入口函数地址，arg 表示传给线程入口函数参数
/// 返回值：创建的线程的 TID
/// syscall ID: 1000
pub fn sys_thread_create(entry: usize, arg: usize) -> isize;

// --------------------waittid------------------------------
pub fn waittid(tid: usize) -> isize {
    loop {
        match sys_waittid(tid) {
            -2 => { yield_(); }
            exit_code => return exit_code,
        }
    }
}

/// 功能：等待当前进程内的一个指定线程退出
/// 参数：tid 表示指定线程的 TID
/// 返回值：如果线程不存在，返回-1；如果线程还没退出，返回-2；其他情况下，返回结束线程的退出码
/// syscall ID: 1002
pub fn sys_waittid(tid: usize) -> i32;
```

## 2. 线程有关的核心数据结构

1. 将原来的 `TaskControlBlock` 只是管理进程改成：`ProcessorControlBlock` 进程控制块 (线程共享的一些资源) + `TaskControlBlock` 线程控制块（包含线程的信息以及独占的资源）
2. 将 `KernelStack` 改为线程独占的资源，增加 `TaskUserRes` 为线程独占的用户态资源，`RecycleAllocator` 为各类资源的通用分配器
3. CPU调度单位仍为 `TaskControlBlock`，但实际从进程变到了线程，而任务管理器 `TaskManager` 和处理器管理结构 `Processor` 基本不变

### 1. 通用资源的分配器

使用通用资源分配器 `RecycleAllocator` 作为多种不同资源分配的核心部分：

1. 进程描述符 `PidHandle`
2. 线程独占的资源组 `TaskUserRes` （包括线程描述符 `tid`）
3. 线程独占的内核栈 `KernelStack`

```rust
// os/src/task/id.rs
pub struct RecycleAllocator {
    current: usize,
    recycled: Vec<usize>,
}

impl RecycleAllocator {
    pub fn new() -> Self {
        RecycleAllocator {
            current: 0,
            recycled: Vec::new(),
        }
    }
    pub fn alloc(&mut self) -> usize {
        if let Some(id) = self.recycled.pop() {
            id
        } else {
            self.current += 1;
            self.current - 1
        }
    }
    pub fn dealloc(&mut self, id: usize) {
        assert!(id < self.current);
        assert!(
            !self.recycled.iter().any(|i| *i == id),
            "id {} has been deallocated!",
            id
        );
        self.recycled.push(id);
    }
}
```

1. 进程描述符 `pid` 的分配

```rust
// os/src/task/id.rs
lazy_static! {
    static ref PID_ALLOCATOR: UPSafeCell<RecycleAllocator> =
        unsafe { UPSafeCell::new(RecycleAllocator::new()) };
}

pub fn pid_alloc() -> PidHandle {
    PidHandle(PID_ALLOCATOR.exclusive_access().alloc())
}

impl Drop for PidHandle {
    fn drop(&mut self) {
        PID_ALLOCATOR.exclusive_access().dealloc(self.0);
    }
}
```

2. 线程独占资源 `TaskUserRes`

线程描述符 `tid` 的分配

```rust
// os/src/task/process.rs
pub struct ProcessControlBlock {
    // immutable
    pub pid: PidHandle,
    // mutable
    inner: UPSafeCell<ProcessControlBlockInner>,
}

pub struct ProcessControlBlockInner {
    ...
    pub task_res_allocator: RecycleAllocator,
    ...
}
// 1.tid
impl ProcessControlBlockInner {
    pub fn alloc_tid(&mut self) -> usize {
        self.task_res_allocator.alloc()
    }

    pub fn dealloc_tid(&mut self, tid: usize) {
        self.task_res_allocator.dealloc(tid)
    }
}
```

用户栈和线程的 `TrapContext` 都可以根据 `tid` 计算得到对应位置

![[Pasted image 20230506161057.png|500]]

```rust
// os/src/config.rs
pub const TRAMPOLINE: usize = usize::MAX - PAGE_SIZE + 1;
pub const TRAP_CONTEXT_BASE: usize = TRAMPOLINE - PAGE_SIZE;

// os/src/task/id.rs
// 2.TrapContxt
fn trap_cx_bottom_from_tid(tid: usize) -> usize {
    TRAP_CONTEXT_BASE - tid * PAGE_SIZE
}
// 3.ustack
fn ustack_bottom_from_tid(ustack_base: usize, tid: usize) -> usize {
    ustack_base + tid * (PAGE_SIZE + USER_STACK_SIZE)
}
```

所以可以将 `tid` 、用户栈、`TrapContext` 三个资源打包起来管理形成 `TaskUserRes` 的线程资源集合

```rust
// os/src/task/id.rs
pub struct TaskUserRes {
    pub tid: usize,
    pub ustack_base: usize,
    // 因为用户栈和 `TrapContext` 的资源都属于物理页帧，需要在 `TaskControlBlock` 的 `memory_set` 中进行管理，所以要有弱指针`process`
    pub process: Weak<ProcessControlBlock>,
}

impl TaskUserRes {
	// 只有alloc_user_res是True的时候才会分配用户栈和TrapContext实际物理页帧
	// 因为若是创建子进程的话，也会创建子进程的主线程，就不需要再重新复制一遍memory_set中的用户栈和TrapContext了
    pub fn new(
        process: Arc<ProcessControlBlock>,
        ustack_base: usize,
        alloc_user_res: bool,
    ) -> Self {
	    // 1.alloc tid
        let tid = process.inner_exclusive_access().alloc_tid();
        let task_user_res = Self {
            tid,
            ustack_base,
            process: Arc::downgrade(&process),
        };
        if alloc_user_res {
            task_user_res.alloc_user_res();
        }
        task_user_res
    }

    /// 在进程地址空间中实际映射线程的用户栈和 Trap 上下文。
    pub fn alloc_user_res(&self) {
        let process = self.process.upgrade().unwrap();
        let mut process_inner = process.inner_exclusive_access();
        // 2.alloc user stack
        let ustack_bottom = ustack_bottom_from_tid(self.ustack_base, self.tid);
        let ustack_top = ustack_bottom + USER_STACK_SIZE;
        process_inner.memory_set.insert_framed_area(
            ustack_bottom.into(),
            ustack_top.into(),
            MapPermission::R | MapPermission::W | MapPermission::U,
        );
        // 3.alloc trap_cx
        let trap_cx_bottom = trap_cx_bottom_from_tid(self.tid);
        let trap_cx_top = trap_cx_bottom + PAGE_SIZE;
        process_inner.memory_set.insert_framed_area(
            trap_cx_bottom.into(),
            trap_cx_top.into(),
            MapPermission::R | MapPermission::W,
        );
    }
}
```

线程退出的时候，`TaskUserRes` 也会一起和 `TaskControlBlock` 一起回收

```rust
// os/src/task/id.rs
impl TaskUserRes {
	// 回收ustack 和 TrapContext
    fn dealloc_user_res(&self) {
        // dealloc tid
        let process = self.process.upgrade().unwrap();
        let mut process_inner = process.inner_exclusive_access();
        // 2.dealloc ustack manually
        let ustack_bottom_va: VirtAddr = ustack_bottom_from_tid(self.ustack_base, self.tid).into();
        process_inner
            .memory_set.remove_area_with_start_vpn(ustack_bottom_va.into());
        // 3.dealloc trap_cx manually
        let trap_cx_bottom_va: VirtAddr = trap_cx_bottom_from_tid(self.tid).into();
        process_inner
            .memory_set
            .remove_area_with_start_vpn(trap_cx_bottom_va.into());
    }
    // 1.回收tid
    pub fn dealloc_tid(&self) {
        let process = self.process.upgrade().unwrap();
        let mut process_inner = process.inner_exclusive_access();
        process_inner.dealloc_tid(self.tid);
    }
}

impl Drop for TaskUserRes {
    fn drop(&mut self) {
        self.dealloc_tid();
        self.dealloc_user_res();
    }
}
```

3. 内核栈的分配与回收

```rust
// os/src/task/id.rs
// 内核栈资源分配器
lazy_static! {
    static ref KSTACK_ALLOCATOR: UPSafeCell<RecycleAllocator> =
        unsafe { UPSafeCell::new(RecycleAllocator::new()) };
}
// 内核栈结构
pub struct KernelStack(pub usize);

/// Return (bottom, top) of a kernel stack in kernel space.
pub fn kernel_stack_position(kstack_id: usize) -> (usize, usize) {
    let top = TRAMPOLINE - kstack_id * (KERNEL_STACK_SIZE + PAGE_SIZE);
    let bottom = top - KERNEL_STACK_SIZE;
    (bottom, top)
}

pub fn kstack_alloc() -> KernelStack {
    let kstack_id = KSTACK_ALLOCATOR.exclusive_access().alloc();
    let (kstack_bottom, kstack_top) = kernel_stack_position(kstack_id);
    KERNEL_SPACE.exclusive_access().insert_framed_area(
        kstack_bottom.into(),
        kstack_top.into(),
        MapPermission::R | MapPermission::W,
    );
    KernelStack(kstack_id)
}

impl Drop for KernelStack {
    fn drop(&mut self) {
        let (kernel_stack_bottom, _) = kernel_stack_position(self.0);
        let kernel_stack_bottom_va: VirtAddr = kernel_stack_bottom.into();
        KERNEL_SPACE
            .exclusive_access()
.remove_area_with_start_vpn(kernel_stack_bottom_va.into());
        KSTACK_ALLOCATOR.exclusive_access().dealloc(self.0);
    }
}
```

### 2. 进程和线程控制块

线程控制块 `TaskControlBlock`

```rust
// os/src/task/task.rs
pub struct TaskControlBlock {
    // immutable 
    // 指向进程控制块弱引用
    pub process: Weak<ProcessControlBlock>,
    // 内核栈
    pub kstack: KernelStack,
    // mutable
    inner: UPSafeCell<TaskControlBlockInner>,
}

pub struct TaskControlBlockInner {
    pub res: Option<TaskUserRes>,
    // TrapContext被TaskUserRes所管理(所有权)
    pub trap_cx_ppn: PhysPageNum,
    pub task_cx: TaskContext,
    pub task_status: TaskStatus,
    pub exit_code: Option<i32>,
}

impl TaskControlBlock {
    pub fn new(
        process: Arc<ProcessControlBlock>,
        ustack_base: usize,
        alloc_user_res: bool,
    ) -> Self {
        let res = TaskUserRes::new(Arc::clone(&process), ustack_base, alloc_user_res);
        let trap_cx_ppn = res.trap_cx_ppn();
        let kstack = kstack_alloc();
        let kstack_top = kstack.get_top();
        Self {
            process: Arc::downgrade(&process),
            kstack,
            inner: unsafe {
                UPSafeCell::new(TaskControlBlockInner {
                    res: Some(res),
                    trap_cx_ppn,
                    task_cx: TaskContext::goto_trap_return(kstack_top),
                    task_status: TaskStatus::Ready,
                    exit_code: None,
                })
            },
        }
    }
}
```

进程控制块 `ProcessControlBlock` 保存进程内线程共享的资源

```rust
// os/src/task/process.rs
pub struct ProcessControlBlock {
    // immutable
    pub pid: PidHandle,
    // mutable
    inner: UPSafeCell<ProcessControlBlockInner>,
}

pub struct ProcessControlBlockInner {
    pub is_zombie: bool,
    pub memory_set: MemorySet,
    pub parent: Option<Weak<ProcessControlBlock>>,
    pub children: Vec<Arc<ProcessControlBlock>>,
    pub exit_code: i32,
    pub fd_table: Vec<Option<Arc<dyn File + Send + Sync>>>,
    pub signals: SignalFlags,
    // 所有线程
    pub tasks: Vec<Option<Arc<TaskControlBlock>>>,
    // 为进程内的线程分配tid
    pub task_res_allocator: RecycleAllocator,
    ... // 其他同步互斥相关资源
}
```

### 3. 任务管理器和处理器管理结构

```rust
// os/src/task/manager.rs
pub struct TaskManager {
    ready_queue: VecDeque<Arc<TaskControlBlock>>,
}
/// 全局变量：
/// 1. 全局任务管理器 TASK_MANAGER
/// 2. 全局 PID-进程控制块映射 PID2TCB

/// 将线程加入就绪队列
pub fn add_task(task: Arc<TaskControlBlock>);
/// 将线程移除出就绪队列
pub fn remove_task(task: Arc<TaskControlBlock>);
/// 从就绪队列中选出一个线程分配 CPU 资源
pub fn fetch_task() -> Option<Arc<TaskControlBlock>>;
/// 根据 PID 查询进程控制块
pub fn pid2process(pid: usize) -> Option<Arc<ProcessControlBlock>>;
/// 增加一对 PID-进程控制块映射
pub fn insert_into_pid2process(pid: usize, process: Arc<ProcessControlBlock>);
/// 删除一对 PID-进程控制块映射
pub fn remove_from_pid2process(pid: usize);

// os/src/task/processor.rs---------------------------------
/// 全局变量：当前处理器管理结构 PROCESSOR

/// CPU 的调度主循环
pub fn run_tasks();
/// 取出当前处理器正在执行的线程
pub fn take_current_task() -> Option<Arc<TaskControlBlock>>;
/// 当前线程控制块/进程控制块/进程地址空间satp/线程Trap上下文
pub fn current_task() -> Option<Arc<TaskControlBlock>>;
pub fn current_process() -> Arc<ProcessControlBlock>;
pub fn current_user_token() -> usize;
pub fn current_trap_cx() -> &'static mut TrapContext;
/// 当前线程Trap上下文在进程地址空间中的地址
pub fn current_trap_cx_user_va() -> usize;
/// 当前线程内核栈在内核地址空间中的地址
pub fn current_kstack_top() -> usize;
/// 将当前线程的内核态上下文保存指定位置，并切换到调度主循环
pub fn schedule(switched_task_cx_ptr: *mut TaskContext);
```

## 3. 线程的创建

### 1. 创建进程(new() 和 fork())的时候为这个进程创建一个主线程

一、调用 `ProcessControlBlock::new` 创建初始进程 `INITPROC`

1. 创建 `memory_set`地址空间
2. 分配进程的 `pid`
3. 创建 `PCB`
4. 创建主线程
	1. 创建TCB（new的时候会分配好tid、用户栈、TrapContext、内核栈等资源）
	2. 修改TCB的 `TrapContext` 内容使能够正确回到用户空间
5. 将新的TCB加入到进程的tasks中
6. 添加pid到process的映射
7. `add_task()` 将当前线程加入到就绪队列中 

```rust
// os/src/task/process.rs
impl ProcessControlBlock {
    pub fn new(elf_data: &[u8]) -> Arc<Self> {
        // 1.memory_set with elf program headers/trampoline/trap context/user stack
        let (memory_set, ustack_base, entry_point) = MemorySet::from_elf(elf_data);
        // 2.allocate a pid
        let pid_handle = pid_alloc();
        // 3.create PCB
        let process = Arc::new(Self {
            pid: pid_handle,
            inner: unsafe {
                UPSafeCell::new(ProcessControlBlockInner {
                    is_zombie: false,
                    memory_set,
                    parent: None,
                    children: Vec::new(),
                    exit_code: 0,
                    fd_table: vec![
                        // 0 -> stdin
                        Some(Arc::new(Stdin)),
                        // 1 -> stdout
                        Some(Arc::new(Stdout)),
                        // 2 -> stderr
                        Some(Arc::new(Stdout)),
                    ],
                    signals: SignalFlags::empty(),
                    tasks: Vec::new(),
                    task_res_allocator: RecycleAllocator::new(),
                    mutex_list: Vec::new(),
                    semaphore_list: Vec::new(),
                    condvar_list: Vec::new(),
                })
            },
        });
        // 4.1.create a main thread, we should allocate ustack and trap_cx here
        let task = Arc::new(TaskControlBlock::new(
            Arc::clone(&process),
            ustack_base,
            // 这里要True分配user stack 和 TrapContext的空间
            true,
        ));
        // 4.2.prepare trap_cx of main thread
        let task_inner = task.inner_exclusive_access();
        let trap_cx = task_inner.get_trap_cx();
        let ustack_top = task_inner.res.as_ref().unwrap().ustack_top();
        let kstack_top = task.kstack.get_top();
        drop(task_inner);
        *trap_cx = TrapContext::app_init_context(
            entry_point,
            ustack_top,
            KERNEL_SPACE.exclusive_access().token(),
            kstack_top,
            trap_handler as usize,
        );
        // 5.add main thread to the process
        let mut process_inner = process.inner_exclusive_access();
        process_inner.tasks.push(Some(Arc::clone(&task)));
        drop(process_inner);
        // 6.添加pid和process的映射
        insert_into_pid2process(process.getpid(), Arc::clone(&process));
        // 7.add main thread to scheduler
        add_task(task);
        process
    }
}

// os/src/task/mod.rs
lazy_static! {
    pub static ref INITPROC: Arc<ProcessControlBlock> = {
        let inode = open_file("initproc", OpenFlags::RDONLY).unwrap();
        let v = inode.read_all();
        ProcessControlBlock::new(v.as_slice())
    };
}
```

二、`fork()` 创建新的进程

1. 根据父进程创建地址空间 `memory_set`
2. 分配 `pid`
3. 复制文件描述符表 `fd_table`
4. 创建子进程的 PCB
5. 将子进程添加到父进程的 `children` 队列中
6. 创建子进程的主线程（不用再分配TaskUserRes，已存在）
7. 将新的 TCB 添加到进程中
8. 修改 `TrapContexn` 中的内核栈位置
9. 添加 pid 到 process 的映射
10. `add_task()` 将该线程添加到调度队列

```rust
// os/src/task/process.rs
impl ProcessControlBlock {
    /// Only support processes with a single thread.
    pub fn fork(self: &Arc<Self>) -> Arc<Self> {
        let mut parent = self.inner_exclusive_access();
        assert_eq!(parent.thread_count(), 1);
        // 1.clone parent's memory_set completely including trampoline/ustacks/trap_cxs
        let memory_set = MemorySet::from_existed_user(&parent.memory_set);
        // 2.alloc a pid
        let pid = pid_alloc();
        // 3.copy fd table
        let mut new_fd_table: Vec<Option<Arc<dyn File + Send + Sync>>> = Vec::new();
        for fd in parent.fd_table.iter() {
            ...
        }
        // 4.create child process pcb
        let child = ...;
        // 5.add child
        parent.children.push(Arc::clone(&child));
        // 6.create main thread of child process
        let task = Arc::new(TaskControlBlock::new(
            Arc::clone(&child),
            parent
                .get_task(0)
                .inner_exclusive_access()
                .res
                .as_ref()
                .unwrap()
                .ustack_base(),
            // here we do not allocate trap_cx or ustack again
            // but mention that we allocate a new kstack here
            false,
        ));
        // 7.attach task to child process
        let mut child_inner = child.inner_exclusive_access();
        child_inner.tasks.push(Some(Arc::clone(&task)));
        drop(child_inner);
        // 8.modify kstack_top in trap_cx of this thread
        let task_inner = task.inner_exclusive_access();
        let trap_cx = task_inner.get_trap_cx();
        trap_cx.kernel_sp = task.kstack.get_top();
        drop(task_inner);
        // 9.
        insert_into_pid2process(child.getpid(), Arc::clone(&child));
        // 10.add this thread to scheduler
        add_task(task);
        child
    }
}
```

### 2. 直接系统调用创建线程 sys_thread_create()

1. 创建一个新的 TCB（包括user stack 和 TrapContext）
2. 将该线程加入到调度队列
3. 将该线程加入到对应进程 PCB 的 `tasks` 数组中对应的位置
4. 修改线程TCB的 `TrapContext` 使能够正常回到用户空间

```rust
// os/src/syscall/thread.rs

pub fn sys_thread_create(entry: usize, arg: usize) -> isize {
    let task = current_task().unwrap();
    let process = task.process.upgrade().unwrap();
    // 1.create a new thread
    let new_task = Arc::new(TaskControlBlock::new(
        Arc::clone(&process),
        task.inner_exclusive_access().res.as_ref().unwrap().ustack_base,
        true,
    ));
    // 2.add new task to scheduler
    add_task(Arc::clone(&new_task));
    let new_task_inner = new_task.inner_exclusive_access();
    let new_task_res = new_task_inner.res.as_ref().unwrap();
    let new_task_tid = new_task_res.tid;
    let mut process_inner = process.inner_exclusive_access();
    // 3.add new thread to current process
    let tasks = &mut process_inner.tasks;
    while tasks.len() < new_task_tid + 1 {
        tasks.push(None);
    }
    tasks[new_task_tid] = Some(Arc::clone(&new_task));
    let new_task_trap_cx = new_task_inner.get_trap_cx();
    // 4.修改TrapContext
    *new_task_trap_cx = TrapContext::app_init_context(
        entry,
        new_task_res.ustack_top(),
        kernel_token(),
        new_task.kstack.get_top(),
        trap_handler as usize,
    );
    (*new_task_trap_cx).x[10] = arg;
    new_task_tid as isize
}
```

## 4. 线程退出

线程先自己通过 `exit()` 退出释放TaskUserRes资源，再由其他线程 `waitted()` 释放剩下资源

一、线程自己 `sys_exit()` 退出（如果是主线程退出就要把进程也删除）

```rust
// os/src/syscall/process.rs
pub fn sys_exit(exit_code: i32) -> ! {
    exit_current_and_run_next(exit_code);
    panic!("Unreachable in sys_exit!");
}
```

1. 从当前 `Processor` 管理器中取出当前任务（线程）（之后该线程就只有在进程PCB中的tasks中有引用，而其他进程还在TaskManager调度队列中有引用）
2. 记录线程的 `exit_code`
3. 将现场的 `res` 设置为 None，释放 `TaskUserRes` 资源
4. 如果是主线程还要删除进程
	1. 删除 pid 到 process 的映射
	2. 修改进程状态为 `zombie`
	3. 记录进程退出码 exit_code
	4. 将所有子进程的父进程设置为初始进程 `INITPROC`
	5. 将进程中所有线程从就绪队列中除去（因为这里只有一个CPU，当执行这里的时候其他task都处在就绪队列中，所以不会有线程落下，但如果是多CPU就可能出现落下线程情况）（至此，所有线程都只有在进程PCB的tasks中有引用）
	6. 将进程中所有线程的 `TaskUserRes` 拿出来放到一个数组中并 `clear()` 回收（这里要提前回收 `TaskUserRes` 否则可能出现 `memory_set`先回收了物理页帧，然后 `TaskUserRes`再回收一次物理页帧导致回收两次）
	7. 将当前进程子进程清空
	8. 将 `memory_set` 清空
	9. 将所有的 `fd_table` 文件描述符清空
	10. 将进程中所有的 `tasks` 线程清空（至此，所有线程无引用）
5. `schedule()` 调度切换线程（传入临时变量，不需要再保存当前上下文）

```rust
// os/src/task/mod.rs

pub fn exit_current_and_run_next(exit_code: i32) {
	// 1.取出当前task
    let task = take_current_task().unwrap();
    let mut task_inner = task.inner_exclusive_access();
    let process = task.process.upgrade().unwrap();
    let tid = task_inner.res.as_ref().unwrap().tid;
    // 2.record exit code
    task_inner.exit_code = Some(exit_code);
    // 3.释放TaskUserRes资源
    task_inner.res = None;
    // here we do not remove the thread since we are still using the kstack
    // it will be deallocated when sys_waittid is called
    drop(task_inner);
    drop(task);
    // 4.however, if this is the main thread of current process
    // the process should terminate at once
    if tid == 0 {
        let pid = process.getpid();
        ...
        // 4.1
        remove_from_pid2process(pid);
        let mut process_inner = process.inner_exclusive_access();
        // 4.2.mark this process as a zombie process
        process_inner.is_zombie = true;
        // 4.3.record exit code of main process
        process_inner.exit_code = exit_code;

        {
            // 4.4move all child processes under init process
            let mut initproc_inner = INITPROC.inner_exclusive_access();
            for child in process_inner.children.iter() {
                child.inner_exclusive_access().parent = Some(Arc::downgrade(&INITPROC));
                initproc_inner.children.push(child.clone());
            }
        }

        // deallocate user res (including tid/trap_cx/ustack) of all threads
        // it has to be done before we dealloc the whole memory_set
        // otherwise they will be deallocated twice
        let mut recycle_res = Vec::<TaskUserRes>::new();
        for task in process_inner.tasks.iter().filter(|t| t.is_some()) {
            let task = task.as_ref().unwrap();
            // if other tasks are Ready in TaskManager or waiting for a timer to be
            // expired, we should remove them.
            //
            // Mention that we do not need to consider Mutex/Semaphore since they
            // are limited in a single process. Therefore, the blocked tasks are
            // 4.5.removed when the PCB is deallocated.
            remove_inactive_task(Arc::clone(&task));
            let mut task_inner = task.inner_exclusive_access();
            if let Some(res) = task_inner.res.take() {
                recycle_res.push(res);
            }
        }
        // dealloc_tid and dealloc_user_res require access to PCB inner, so we
        // need to collect those user res first, then release process_inner
        // for now to avoid deadlock/double borrow problem.
        drop(process_inner);
        // 4.6.
        recycle_res.clear();

        let mut process_inner = process.inner_exclusive_access();
        // 4.7
        process_inner.children.clear();
        // 4.8.deallocate other data in user space i.e. program code/data section
        process_inner.memory_set.recycle_data_pages();
        // 4.9.drop file descriptors
        process_inner.fd_table.clear();
        // 4.10.remove all tasks
        process_inner.tasks.clear();
    }
    drop(process);
    // 5.we do not have to save task context
    let mut _unused = TaskContext::zero_init();
    schedule(&mut _unused as *mut _);
```

二、其他线程等待线程结束

当线程 `sys_exit()` 的时候，释放了 `TaskUserRes` 资源，且只有进程PCB中的 `tasks` 中有对该线程的引用，所以在 `sys_waittid()` 的时候直接将线程从该 `tasks` 对应的位置除去就可直接回收整个 TCB和其对应的资源

1. 如果等待的线程是自己就返回-1出错
2. 获得对应线程的 `exit_code` ，如果能得到说明对应线程经过了 `sys_exit()` 退出；如果没有该线程就返回-1；如果线程没有退出就返回-2
3. 将对应线程从 PCB 中的 `tasks` 中除去，这时就会回收 PCB 剩下的资源

```rust
// os/src/syscall/thread.rs

/// thread does not exist, return -1
/// thread has not exited yet, return -2
/// otherwise, return thread's exit code
pub fn sys_waittid(tid: usize) -> i32 {
    let task = current_task().unwrap();
    let process = task.process.upgrade().unwrap();
    let task_inner = task.inner_exclusive_access();
    let mut process_inner = process.inner_exclusive_access();
    // 1.a thread cannot wait for itself
    if task_inner.res.as_ref().unwrap().tid == tid {
        return -1;
    }
    let mut exit_code: Option<i32> = None;
    let waited_task = process_inner.tasks[tid].as_ref();
    // 2.
    if let Some(waited_task) = waited_task {
        if let Some(waited_exit_code) = waited_task.inner_exclusive_access().exit_code {
            exit_code = Some(waited_exit_code);
        }
    } else {
        // waited thread does not exist
        return -1;
    }
    if let Some(exit_code) = exit_code {
        // 3.dealloc the exited thread
        process_inner.tasks[tid] = None;
        exit_code
    } else {
        // waited thread has not exited
        -2
    }
}
```
# 互斥锁

> 当线程之间需要**访问共享的资源**的时候，就需要用到互斥锁：进入临界区之前获得锁，离开临界区之后释放锁

## 1. 基于软件的实现

Peterson 算法（适用于两个线程的互斥访问）

FLAG\[i] 表示 i 线程想要进入临界区；TURN = i 表示轮到 i 线程进入临界区

每次线程 i 进入临界区之前都设置 FLAG\[i] = True, TURN = j 表示 i 想要进入，但现在轮到 j 进入；当线程 j 想要进入FLAG\[j] = True 且 已经进入 TURN = j 时线程 i 就等待；当 j 退出时 FLAG\[j] = False，线程 i 就可以进入

```rust
// user/src/bin/adder_peterson_spin.rs

/// FLAG[i]=true 表示线程 i 想要进入或已经进入临界区
static mut FLAG: [bool; 2] = [false; 2];
/// TURN=i 表示轮到线程 i 进入临界区
static mut TURN: usize = 0;

/// id 表示当前的线程 ID ，为 0 或 1
unsafe fn lock(id: usize) {
    FLAG[id] = true;
    let j = 1 - id;
    TURN = j;
    // Tell the compiler not to reorder memory operations
    // across this fence.
    compiler_fence(Ordering::SeqCst);
    // Why do we need to use volatile_read here?
    // Otherwise the compiler will assume that they will never
    // be changed on this thread. Thus, they will be accessed
    // only once!
    while vload!(&FLAG[j]) && vload!(&TURN) == j {}
    // while FLAG[j] && TURN == j {}
}

unsafe fn unlock(id: usize) {
    FLAG[id] = false;
}
```

## 2. 基于硬件机制及特殊指令的锁🔒实现

### 1. 关中断

* 优点：简单
* 缺点：
	1. 使用户态程序能够使用中断这种特权，线程可以恶意永久关闭中断而独占所有CPU资源
	2. 对于多处理器架构，关闭当前CPU中断对其他CPU无影响

```rust
fn lock() {
    disable_interrupt(); //屏蔽中断的机器指令
}

fn unlock() {
    enable_interrupt(); //使能中断的机器指令
}
```

### 2. 原子指令

用全局变量来作为锁的标记，那么对全局变量的修改就要互斥访问，可以通过原子操作来实现

```rust
// user/src/bin/adder_atomic.rs
static OCCUPIED: AtomicBool = AtomicBool::new(false);

fn lock() {
    while OCCUPIED
    .compare_exchange(false, true, Ordering::Relaxed, Ordering::Relaxed)
    .is_err()
    // 只有当前OCCUPIED==false且成功修改为true后才返回正确，跳出循环
    {
        yield_();
    }
}

fn unlock() {
    OCCUPIED.store(false, Ordering::Relaxed);
}

// 如果原子变量的值与current的值相同，就设置原子变量的值为new
pub fn compare_exchange(
    &self,
    current: bool,
    new: bool,
    success: Ordering,
    failure: Ordering,
) -> Result<bool, bool>;
```

CAS指令（Compare And Swap）比较并返回

```rust
// 与上面的类似，如果原子变量和expected相同将new交换到内存中，然后将内存中原来的值交换出来并返回
// 过程伪代码，实际上是原子操作
fn compare_and_swap(ptr: *mut i32, expected: i32, new: i32) -> i32 {
    let original = unsafe { *ptr };
    if original == expected {
        unsafe { *ptr = new; }
    }
    original
}
```

TAS指令（Test And Set）测试并设置

```rust
// 过程伪代码，实际上是原子操作
fn test_and_set(ptr: *mut i32, new: i32) -> i32 {
    let original = unsafe { *ptr };
    unsafe { *ptr = new };
    original
}

// 使用------------------------------------------------
static mut OCCUPIED: i32 = 0;

unsafe fn lock() {
    while (test_and_set(&mut OCCUPIED, 1) == 1) {}
}

unsafe fn unlock() {
    OCCUPIED = 0;
}
```

使用Risc-V下提供的 LR/SC 指令来实现CAS指令

1. LR (Load Reserved)：将内存中 `rs1` 指向的值保存到 `rd` 中
2. SC (Store Conditional)：将内存中 `rs1` 指向的这个值改写成 `rs2` 保存的值，如果在 LR 和 SC 这个过程中 `rs1` 指向的值不变就可正确执行，并且 `rd` 的值设置为0，否则 `rd` 为任意非零值

```rust
# 参数 a0 存放内存中的值的所在地址
# 参数 a1 存放 expected
# 参数 a2 存放 new
# 返回值 a0 略有不同：这里若比较结果相同则返回 0 ，否则返回 1
# 而不是返回 CAS 之前内存中的值
cas:
    lr.w t0, (a0) # LR 将值加载到 t0
    bne t0, a1, fail # 如果值和 a1 中的 expected 不同，跳转到 fail
    sc.w t0, a2, (a0) # SC 尝试将值修改为 a2 中的 new
    bnez t0, cas # 如果 SC 的目标寄存器 t0 不为 0 ，说明 LR/SC 中间值被修改，重试
    li a0, 0 # 成功，返回值为 0
    ret # 返回
fail:
    li a0, 1 # 失败，返回值为 1
    ret # 返回
```

## 3. 操作系统下的让权等待

> 本章中的锁包括 阻塞锁 和 自旋锁：阻塞锁就是基于阻塞的，自旋锁基于 `suspend_current_and_run_next` 调度出去

### 1. 忙等待

当该线程没能获得锁的时候，通过循环不断尝试

* 优点：在条件成立的第一时间就能够响应，延迟低，且不涉及开销大的上下文切换
* 缺点：浪费一些CPU时间在等待上（对于单核环境下且等待不涉及外设的时候，忙等没有意义）

### 2. yield 暂时让权

当该线程没能获得锁的时候，通过 `yield_()` 让出CPU使用权

* 优点：让CPU执行其他不需要等待的线程，提高CPU利用率
* 缺点：
	1. 如果在该线程能获得锁之前调度多次，会导致增加了很多上下文切换的开销，破坏程序时间和空间局部性，要清空缓存；
	2. 如果该线程能获得锁之后很久才调度到，会导致响应延迟过长

```rust
// user/src/bin/adder_peterson_yield.rs
unsafe fn lock(id: usize) {
    FLAG[id] = true;
    let j = 1 - id;
    TURN = j;
    // Tell the compiler not to reorder memory operations
    // across this fence.
    compiler_fence(Ordering::SeqCst);
    while FLAG[j] && TURN == j {
        yield_();
    }
}
```

### 3. 阻塞

不将对应线程加入到等待队列中，而是将该线程加入到与该锁有关的阻塞队列中，当释放锁的时候再从对应的阻塞队列中唤醒，取出加入到等待队列中使得该线程可以调度

* 优点：只有两次上下文切换，阻塞和唤醒的配合可以实现精确高效的等待
* 缺点：会有两次上下文切换，如果事件产生频率高的时候，就会有很多切换，就用忙等代替

#### 1. 阻塞与唤醒

```rust
// os/src/task/mod.rs
pub fn block_current_and_run_next() {
    let task = take_current_task().unwrap();
    let mut task_inner = task.inner_exclusive_access();
    let task_cx_ptr = &mut task_inner.task_cx as *mut TaskContext;
    task_inner.task_status = TaskStatus::Blocked;
    drop(task_inner);
    // add_task(task); 
    // 相比于suspend_current_and_run_next()少了这一句，不需要加入到就绪队列，而是加入到对应锁的阻塞队列
    schedule(task_cx_ptr);
}

// os/src/task/manager.rs
// 将对应的线程唤醒，加入到就绪队列
pub fn wakeup_task(task: Arc<TaskControlBlock>) {
    let mut task_inner = task.inner_exclusive_access();
    task_inner.task_status = TaskStatus::Ready;
    drop(task_inner);
    add_task(task);
}
```

#### 2. 基于阻塞的 sleep 的系统调用

因为 sleep 也是要等到特定时间才唤醒，这个时间可以当作是特定的事件

记录等待唤醒的线程的数据结构，将所有等待特定时间需要唤醒的线程的结构加入到大根堆里面（时间短的先会被查看和唤醒）

```rust
pub struct TimerCondVar {
    pub expire_ms: usize,
    pub task: Arc<TaskControlBlock>,
}

// os/src/timer.rs
use alloc::collections::BinaryHeap;

lazy_static! {
    static ref TIMERS: UPSafeCell<BinaryHeap<TimerCondVar>> =
        unsafe { UPSafeCell::new(BinaryHeap::<TimerCondVar>::new()) };
}
```

`sys_sleep()` 加入到大根堆

```rust
// os/src/syscall/sync.rs
pub fn sys_sleep(ms: usize) -> isize {
    let expire_ms = get_time_ms() + ms;
    let task = current_task().unwrap();
    add_timer(expire_ms, task);
    block_current_and_run_next();
    0
}

// os/src/timer.rs
pub fn add_timer(expire_ms: usize, task: Arc<TaskControlBlock>) {
    let mut timers = TIMERS.exclusive_access();
    timers.push(TimerCondVar { expire_ms, task });
}
```

每一个时钟中断就 `check_timer()` 唤醒睡眠超时的线程

```rust
// os/src/timer.rs
pub fn check_timer() {
    let current_ms = get_time_ms();
    let mut timers = TIMERS.exclusive_access();
    while let Some(timer) = timers.peek() {
        if timer.expire_ms <= current_ms {
            // 调用 wakeup_task 唤醒超时线程
            wakeup_task(Arc::clone(&timer.task));
            timers.pop();
        } else {
            break;
        }
    }
}
```

#### 3. 基于阻塞的锁的实现

系统调用接口

```rust
/// 功能：为当前进程新增一把互斥锁。
/// 参数： blocking 为 true 表示互斥锁基于阻塞机制实现，
/// 否则表示互斥锁基于类似 yield 的方法实现。
/// 返回值：假设该操作必定成功，返回创建的锁的 ID 。
/// syscall ID: 1010
pub fn sys_mutex_create(blocking: bool) -> isize;

/// 功能：当前线程尝试获取所属进程的一把互斥锁。
/// 参数： mutex_id 表示要获取的锁的 ID 。
/// 返回值： 0
/// syscall ID: 1011
pub fn sys_mutex_lock(mutex_id: usize) -> isize;

/// 功能：当前线程释放所属进程的一把互斥锁。
/// 参数： mutex_id 表示要释放的锁的 ID 。
/// 返回值： 0
/// syscall ID: 1012
pub fn sys_mutex_unlock(mutex_id: usize) -> isize;
```

接口的使用

```rust
// user/src/bin/adder_mutex_blocking.rs
unsafe fn f() -> ! {
    let mut t = 2usize;
    for _ in 0..PER_THREAD {
	    // 1.获得锁
        mutex_lock(0);
        // 2.进入临界区
        critical_section(&mut t);
        // 3.释放锁
        mutex_unlock(0);
    }
    exit(t as i32)
}

#[no_mangle]
pub fn main(argc: usize, argv: &[&str]) -> i32 {
    ...
    assert_eq!(mutex_blocking_create(), 0);
    let mut v = Vec::new();
    for _ in 0..thread_count {
        v.push(thread_create(f as usize, 0) as usize);
    }
    ...
}
```

对于一个互斥锁需要实现的 `Mutex` Trait（本章中有 阻塞锁和自旋锁）

```rust
// os/src/sync/mutex.rs
pub trait Mutex: Sync + Send {
    fn lock(&self);
    fn unlock(&self);
}
```

进程 PCB 中包含所有的锁

```rust
// os/src/task/process.rs
pub struct ProcessControlBlockInner {
    ...
    pub fd_table: Vec<Option<Arc<dyn File + Send + Sync>>>,
    // 就像fd_table一样，所有锁的所有权
    pub mutex_list: Vec<Option<Arc<dyn Mutex>>>,
    ...
}
```

阻塞锁结构的定义

每个阻塞锁中都包含一个阻塞队列，里面管理着所有依赖于该锁的阻塞线程

```rust
// os/src/sync/mutex.rs
pub struct MutexBlocking {
    inner: UPSafeCell<MutexBlockingInner>,
}

pub struct MutexBlockingInner {
	// 获取锁的关键
    locked: bool,
    // 阻塞队列
    wait_queue: VecDeque<Arc<TaskControlBlock>>,
}

impl MutexBlocking {
    pub fn new() -> Self {
        Self {
            inner: unsafe {
                UPSafeCell::new(MutexBlockingInner {
                    locked: false,
                    wait_queue: VecDeque::new(),
                })
            },
        }
    }
}
```

创建锁：在PCB中存储一个新的锁

```rust
// os/src/syscall/sync.rs
// 1.创建阻塞锁
pub fn sys_mutex_create(blocking: bool) -> isize {
    let process = current_process();
    let mutex: Option<Arc<dyn Mutex>> = if !blocking {
        Some(Arc::new(MutexSpin::new()))
    } else {
        Some(Arc::new(MutexBlocking::new()))
    };
    let mut process_inner = process.inner_exclusive_access();
    if let Some(id) = process_inner
        .mutex_list
        .iter()
        .enumerate()
        .find(|(_, item)| item.is_none())
        .map(|(id, _)| id)
    {
        process_inner.mutex_list[id] = mutex;
        id as isize
    } else {
        process_inner.mutex_list.push(mutex);
        process_inner.mutex_list.len() as isize - 1
    }
}
```

获得锁：这里的 `lock()` 是直接判断 MutexBlocking结构中locked的值，这对于单个CPU没问题，因为本章代码线程在进入内核的时候不会被中断抢占切换；但对于多CPU的时候就会出问题，所以应该在**修改共享变量 `locked` 的时候用原子操作**（比如上面的CAS和TAS）

```rust
// os/src/syscall/sync.rs

pub fn sys_mutex_lock(mutex_id: usize) -> isize {
    let process = current_process();
    let process_inner = process.inner_exclusive_access();
    let mutex = Arc::clone(process_inner.mutex_list[mutex_id].as_ref().unwrap());
    drop(process_inner);
    drop(process);
    mutex.lock();
    0
}

// os/src/sync/mutex.rs
impl Mutex for MutexBlocking {
    fn lock(&self) {
        let mut mutex_inner = self.inner.exclusive_access();
        // 这里是直接判断和修改，对单CPU没问题
        if mutex_inner.locked {
mutex_inner.wait_queue.push_back(current_task().unwrap());
            drop(mutex_inner);
            block_current_and_run_next();
        } else {
            mutex_inner.locked = true;
        }
    }
}
```

释放锁：由于上面的 `lock()` 的实现中，如果锁被占有就 `block_current_and_run_next()` ，当被重新唤醒的时候，该线程就直接返回了，就可以直接进入临界区，所以如果可以唤醒线程的话就不用修改锁 `locked` 为false，因为唤醒的线程一定会直接返回进入临界区；如果没有阻塞的线程才修改 `locked` 为false

```rust
// os/src/syscall/sync.rs
pub fn sys_mutex_unlock(mutex_id: usize) -> isize {
    let process = current_process();
    let process_inner = process.inner_exclusive_access();
    let mutex = Arc::clone(process_inner.mutex_list[mutex_id].as_ref().unwrap());
    drop(process_inner);
    drop(process);
    mutex.unlock();
    0
}

// os/src/sync/mutex.rs
impl Mutex for MutexBlocking {
    fn unlock(&self) {
        let mut mutex_inner = self.inner.exclusive_access();
        assert!(mutex_inner.locked);
        // 直接就从阻塞队列中取出一个
        if let Some(waking_task) = mutex_inner.wait_queue.pop_front() {
            wakeup_task(waking_task);
        } else { // 如果没有线程阻塞才修改为false
            mutex_inner.locked = false;
        }
    }
}
```

补充自旋锁

```rust
pub struct MutexSpin {
    locked: UPSafeCell<bool>,
}

impl MutexSpin {
    pub fn new() -> Self {
        Self {
            locked: unsafe { UPSafeCell::new(false) },
        }
    }
}

impl Mutex for MutexSpin {
    fn lock(&self) {
        loop {
            let mut locked = self.locked.exclusive_access();
            if *locked {
                drop(locked);
                // 调度切换线程
                suspend_current_and_run_next();
                continue;
            } else {
                *locked = true;
                return;
            }
        }
    }

    fn unlock(&self) {
        let mut locked = self.locked.exclusive_access();
        *locked = false;
    }
}
```
# 信号量

> 为了实现多个线程之间的**合作**和**条件同步**，出现了信号量。

初始状态下，信号量有 N 个可用资源，线程可以在某个时刻占用一个该种资源，并在使用过后归还资源；如果此时没有可用资源了，就需要暂时进入等待。**P操作** 表示线程尝试占用一个资源，**V操作** 表示线程尝试释放一个资源，二者基于阻塞-唤醒机制

* 当初始资源数量 N > 0 时，表示为计数信号量，可以用来描述可用资源数；
* 特别的当 N = 1 时，成为二值信号量，也就是互斥锁；
* 当 N = 0 时，可以当成一种同步原语：例如线程A可以在P等待线程B执行到某一个位置后调用V后才能被唤醒执行

## 1. 实现方法

实现方法有两种：

```rust
// 一、
fn P(S) {
    if S >= 1
        // 如果还有可用资源，更新资源剩余数量 S
        S = S - 1;
        // 使用资源
    else
        // 已经没有可用资源
        // 阻塞当前线程并将其加入阻塞队列
        <block and enqueue the thread>;
        // 阻塞恢复后直接就进入临界区，所以不用减少资源，且V唤醒时也不用增加资源
}

fn V(S) {
    if <some threads are blocked on the queue>
        // 如果已经有线程在阻塞队列中
        // 则唤醒这个线程，不用增加资源
        <unblock a thread>;
    else
        // 否则只需恢复 1 资源可用数量
        S = S + 1;
}

// 二、
fn P(S) {
	// 先减少资源
    S = S - 1;
    // 如果资源 < 0 表示有多少个资源欠缺(就要有多少个线程阻塞)
    if 0 > S then
        // 阻塞当前线程并将其加入阻塞队列
        <block and enqueue the thread>;
}

fn V(S) {
	// 先增加资源(恢复线程)，因为上面的P一进去马上就消耗一个资源
    S = S + 1;
    if <some threads are blocked on the queue>
        // 如果已经有线程在阻塞队列中
        // 则唤醒这个线程
        <unblock a thread>;
}
```

## 2. 系统调用接口

创建、P 操作、V 操作

```rust
/// 功能：为当前进程新增一个信号量。
/// 参数：res_count 表示该信号量的初始资源可用数量，即 N ，为一个非负整数。
/// 返回值：假定该操作必定成功，返回创建的信号量的 ID 。
/// syscall ID : 1020
pub fn sys_semaphore_create(res_count: usize) -> isize;

/// 功能：对当前进程内的指定信号量进行 V 操作。
/// 参数：sem_id 表示要进行 V 操作的信号量的 ID 。
/// 返回值：假定该操作必定成功，返回 0 。
pub fn sys_semaphore_up(sem_id: usize) -> isize;

/// 功能：对当前进程内的指定信号量进行 P 操作。
/// 参数：sem_id 表示要进行 P 操作的信号量的 ID 。
/// 返回值：假定该操作必定成功，返回 0 。
pub fn sys_semaphore_down(sem_id: usize) -> isize;
```

## 3. 信号量的应用

1. 作为同步原语：资源个数为0

```rust
// user/src/bin/sync_sem.rs
// 信号量的编号0(第一个)
const SEM_SYNC: usize = 0;

unsafe fn first() -> ! {
    sleep(10);
    println!("First work and wakeup Second");
	// 执行了V操作之后second()才能继续执行
    semaphore_up(SEM_SYNC);
    exit(0)
}

unsafe fn second() -> ! {
    println!("Second want to continue,but need to wait first");
    // 要等到 first() 执行完V操作才能执行下去
    semaphore_down(SEM_SYNC);
    println!("Second can work now");
    exit(0)
}

#[no_mangle]
pub fn main() -> i32 {
    // create semaphores
    assert_eq!(semaphore_create(0) as usize, SEM_SYNC);
    // create threads
    let threads = vec![
        thread_create(first as usize, 0),
        thread_create(second as usize, 0),
    ];
    // wait for all threads to complete
    for thread in threads.iter() {
        waittid(*thread as usize);
    }
    println!("sync_sem passed!");
    0
}
```

2. 生产者和消费者基于有限缓冲进行协作

```rust
// user/src/bin/mpsc_sem.rs
// 信号量的编号
// 互斥信号量，对缓冲区的指针的访问要互斥
const SEM_MUTEX: usize = 0;
// 空闲资源的信号量
const SEM_EMPTY: usize = 1;
// 已有资源的信号量
const SEM_AVAIL: usize = 2;
// 缓冲区大小
const BUFFER_SIZE: usize = 8;
static mut BUFFER: [usize; BUFFER_SIZE] = [0; BUFFER_SIZE];
static mut FRONT: usize = 0;
static mut TAIL: usize = 0;
const PRODUCER_COUNT: usize = 4;
const NUMBER_PER_PRODUCER: usize = 100;
// 生产者模型
unsafe fn producer(id: *const usize) -> ! {
    let id = *id;
    for _ in 0..NUMBER_PER_PRODUCER {
	    // 互斥信号量只有在访问临界区才要使用，不能放在外面，否则会导致死锁
	    // 1.先获取空闲资源信号量
        semaphore_down(SEM_EMPTY);
        // 2.对缓冲区操作要互斥
        semaphore_down(SEM_MUTEX);
        BUFFER[TAIL] = id;
        TAIL = (TAIL + 1) % BUFFER_SIZE;
        // 3.释放互斥信号量
        semaphore_up(SEM_MUTEX);
        // 4.新增了已有资源，要释放已有资源信号量
        semaphore_up(SEM_AVAIL);
    }
    exit(0)
}

unsafe fn consumer() -> ! {
    for _ in 0..PRODUCER_COUNT * NUMBER_PER_PRODUCER {
	    // 1.已有资源的使用
        semaphore_down(SEM_AVAIL);
        // 2.互斥访问缓冲区
        semaphore_down(SEM_MUTEX);
        print!("{} ", BUFFER[FRONT]);
        FRONT = (FRONT + 1) % BUFFER_SIZE;
        // 3.释放互斥信号量
        semaphore_up(SEM_MUTEX);
        // 4.占用资源，释放空闲资源信号量
        semaphore_up(SEM_EMPTY);
    }
    println!("");
    exit(0)
}

#[no_mangle]
pub fn main() -> i32 {
    // create semaphores
    assert_eq!(semaphore_create(1) as usize, SEM_MUTEX);
    assert_eq!(semaphore_create(BUFFER_SIZE) as usize, SEM_EMPTY);
    assert_eq!(semaphore_create(0) as usize, SEM_AVAIL);
    // create threads
    let ids: Vec<_> = (0..PRODUCER_COUNT).collect();
    let mut threads = Vec::new();
    for i in 0..PRODUCER_COUNT {
        threads.push(thread_create(
            producer as usize,
            &ids.as_slice()[i] as *const _ as usize,
        ));
    }
    threads.push(thread_create(consumer as usize, 0));
    // wait for all threads to complete
    for thread in threads.iter() {
        waittid(*thread as usize);
    }
    println!("mpsc_sem passed!");
    0
}
```

## 4. 内核实现信号量

先在 PCB 内部添加信号量数组，再实现信号量的创建、占用、释放

```rust
// os/src/task/process.rs
pub struct ProcessControlBlockInner {
    ...
    pub mutex_list: Vec<Option<Arc<dyn Mutex>>>,
    pub semaphore_list: Vec<Option<Arc<Semaphore>>>,
    ...
}

// --------------------------信号量结构--------------------
// os/src/sync/semaphore.rs
pub struct Semaphore {
    pub inner: UPSafeCell<SemaphoreInner>,
}

pub struct SemaphoreInner {
	// 多少个可用资源
    pub count: isize,
    // 阻塞队列
    pub wait_queue: VecDeque<Arc<TaskControlBlock>>,
}

impl Semaphore {
    pub fn new(res_count: usize) -> Self {...}
    pub fn up(&self) {...}
    pub fn down(&self) {...}
}

//
// os/src/sync/semaphore.rs
impl Semaphore {
    pub fn new(res_count: usize) -> Self {
        Self {
            inner: unsafe {
                UPSafeCell::new(SemaphoreInner {
                    count: res_count as isize,
                    wait_queue: VecDeque::new(),
                })
            },
        }
    }

	// 创建和释放采用的是信号量的实现方法二
    pub fn up(&self) {
        let mut inner = self.inner.exclusive_access();
        inner.count += 1;
        if inner.count <= 0 {
            if let Some(task) = inner.wait_queue.pop_front() {
                wakeup_task(task);
            }
        }
    }

    pub fn down(&self) {
        let mut inner = self.inner.exclusive_access();
        inner.count -= 1;
        if inner.count < 0 {
            inner.wait_queue.push_back(current_task().unwrap());
            drop(inner);
            block_current_and_run_next();
        }
    }
}
```

# 条件变量

> 条件变量可以看出信号量的简化版 (没有了资源个数)，他需要与锁共同使用，实现类似于**管程**的同步原语。其目的也是为了实现线程之间的**同步合作**

## 1. 管程

管程（Monitor）是一种高级同步原语，由过程 (函数)、共享变量和数据结构组成的一个集合。

进入管程时要获得锁保证互斥访问；当条件不满足时要阻塞等待；当条件满足时就唤醒等待的线程。我们将阻塞的队列称为 **条件变量**，将阻塞操作成为 `wait`，唤醒操作称为 `signal`

因为不能持有锁的情况下陷入阻塞（否则可能导致无法释放锁而死锁），所以在 `wait` 的时候需要：

1. 释放锁
2. 阻塞当前线程
3. 唤醒后需要重新获得锁
4. `wait` 返回，线程成功向下执行

同时在线程唤醒的过程中，如 T1 唤醒 T2， `signal` 唤醒也有不同的语义：

1. Hoare语义：优先级 T2 > T1 > 其他线程。当 T1 发现条件满足后`signal` **唤醒 T2 并将锁转交给 T2**，T2 就能立即继续执行，而 T1 进入等待队列；当 T2 退出管程后将锁交回 等待队列中的 T1，T1继续执行
2. Hansen语义：优先级 T1 > T2 > 其他线程。T1 发现条件满足后先继续执行，在退出管程的时候再 `signal` **唤醒并将锁交给 T2** 继续执行（我们互斥锁和信号量就是这样实现的）
3. Mesa语义：优先级 T1 > T2 = 其他线程。T1 发现满足条件后可以用 `signal` **唤醒 T2，但并不会将锁交给 T2**，而是在 T1 退出管程后释放锁，T2 再与其他线程竞争，谁抢到锁谁继续执行（这里的条件信号就是依据这个实现的）
4. Hoare 和 Hansen 语义都会将锁交给唤醒的线程，保证了 T2 紧跟着 T1 回到管程，**T2 被唤醒后等待条件一定成立**，就没必要重复检查条件是否成立就可以向下执行；而 Mesa 语义下不会将锁交给 T2，要抢到锁后才能继续执行，所以**需要重复确认条件成立**才能继续执行
 
![[Pasted image 20230507231140.png]]

```rust
// 条件等待的方法
// 第一种方法，基于 if/else(Hoare和Hansen语义)
if (!condition) {
    wait();
} else {
    ...
}

// 第二种方法，基于 while(Mesa语义)
while (!condition) {
    wait();
}
```

## 2. 条件变量的系统调用

包括创建、阻塞等待、发送信号唤醒

```rust
/// 功能：为当前进程新增一个条件变量。
/// 返回值：假定该操作必定成功，返回创建的条件变量的 ID 。
/// syscall ID : 1030
pub fn sys_condvar_create() -> isize;

/// 功能：对当前进程的指定条件变量进行 signal 操作，即
/// 唤醒一个在该条件变量上阻塞的线程（如果存在）。
/// 参数：condvar_id 表示要操作的条件变量的 ID 。
/// 返回值：假定该操作必定成功，返回 0 。
/// syscall ID : 1031
pub fn sys_condvar_signal(condvar_id: usize) -> isize;

/// 功能：对当前进程的指定条件变量进行 wait 操作，分为多个阶段：
/// 1. 释放当前线程持有的一把互斥锁；
/// 2. 阻塞当前线程并将其加入指定条件变量的阻塞队列；
/// 3. 直到当前线程被其他线程通过 signal 操作唤醒；
/// 4. 重新获取当前线程之前持有的锁。
/// 参数：mutex_id 表示当前线程持有的互斥锁的 ID ，而
/// condvar_id 表示要操作的条件变量的 ID 。
/// 返回值：假定该操作必定成功，返回 0 。
/// syscall ID : 1032
pub fn sys_condvar_wait(condvar_id: usize, mutex_id: usize) -> isize;
```

## 3. 使用方法

一、解决条件同步问题

```rust
// user/src/bin/condsync_condvar.rs
const CONDVAR_ID: usize = 0;
const MUTEX_ID: usize = 0;

unsafe fn first() -> ! {
    sleep(10);
    println!("First work, Change A --> 1 and wakeup Second");
    // 访问共享变量要互斥锁(进入管程)
    mutex_lock(MUTEX_ID);
    A = 1;
    // 访问管程结束，发送信号唤醒
    condvar_signal(CONDVAR_ID);
    // 释放互斥锁
    mutex_unlock(MUTEX_ID);
    exit(0)
}

unsafe fn second() -> ! {
    println!("Second want to continue,but need to wait A=1");
    // 访问共享变量，需要互斥锁
    mutex_lock(MUTEX_ID);
    while A == 0 {
        println!("Second: A is {}", A);
        // 等待条件信号
        condvar_wait(CONDVAR_ID, MUTEX_ID);
    }
    println!("A is {}, Second can work now", A);
    // 释放互斥锁
    mutex_unlock(MUTEX_ID);
    exit(0)
}

#[no_mangle]
pub fn main() -> i32 {
    // create condvar & mutex
    assert_eq!(condvar_create() as usize, CONDVAR_ID);
    assert_eq!(mutex_blocking_create() as usize, MUTEX_ID);
    ...
}
```

使用条件信号的时候要注意 **唤醒丢失** 问题，如果先执行 `signal` 且无阻塞队列就不会释放，相当于什么没做，可能导致 `wait` 的线程一直阻塞

二、解决同步屏障问题

```rust
// user/src/bin/barrier_condvar.rs
const THREAD_NUM: usize = 3;

struct Barrier {
    mutex_id: usize,
    condvar_id: usize,
    count: UnsafeCell<usize>,
}

impl Barrier {
    pub fn new() -> Self {
        Self {
            mutex_id: mutex_create() as usize,
            condvar_id: condvar_create() as usize,
            count: UnsafeCell::new(0),
        }
    }
    pub fn block(&self) {
	    // 互斥访问共享变量
        mutex_lock(self.mutex_id);
        let count = self.count.get();
        // SAFETY: Here, the accesses of the count is in the
        // critical section protected by the mutex.
        unsafe { *count = *count + 1; }
        if unsafe { *count } == THREAD_NUM {
	        // 满足条件唤醒
            condvar_signal(self.condvar_id);
        } else {
	        // 不满足条件等待
            condvar_wait(self.condvar_id, self.mutex_id);
            // 唤醒另一个等待的
            condvar_signal(self.condvar_id);
        }
        // 释放互斥锁
        mutex_unlock(self.mutex_id);
    }
}

unsafe impl Sync for Barrier {}

lazy_static! {
    static ref BARRIER_AB: Barrier = Barrier::new();
    static ref BARRIER_BC: Barrier = Barrier::new();
}

// 总共三个阶段，所有线程都到达一个阶段才能继续执行
fn thread_fn() {
    for _ in 0..300 { print!("a"); }
    BARRIER_AB.block();
    for _ in 0..300 { print!("b"); }
    BARRIER_BC.block();
    for _ in 0..300 { print!("c"); }
    exit(0)
}
```

## 4. 实现条件变量

```rust
// os/src/task/process.rs
pub struct ProcessControlBlockInner {
    ...
    pub mutex_list: Vec<Option<Arc<dyn Mutex>>>,
    pub semaphore_list: Vec<Option<Arc<Semaphore>>>,
    pub condvar_list: Vec<Option<Arc<Condvar>>>,
}

// os/src/sync/condvar.rs
pub struct Condvar {
    pub inner: UPSafeCell<CondvarInner>,
}

pub struct CondvarInner {
	// 只有阻塞队列
    pub wait_queue: VecDeque<Arc<TaskControlBlock>>,
}

// os/src/sync/condvar.rs
impl Condvar {
    pub fn new() -> Self {
        Self {
            inner: unsafe {
                UPSafeCell::new(CondvarInner {
                    wait_queue: VecDeque::new(),
                })
            },
        }
    }
	// Mesa语义的唤醒
    pub fn signal(&self) {
        let mut inner = self.inner.exclusive_access();
        if let Some(task) = inner.wait_queue.pop_front() {
            wakeup_task(task);
        }
    }
    
	// Mesa语义的条件信号，需要用户while等待
    pub fn wait(&self, mutex: Arc<dyn Mutex>) {
	    // 1.释放锁
        mutex.unlock();
        let mut inner = self.inner.exclusive_access();
        // 2.加入阻塞队列
        inner.wait_queue.push_back(current_task().unwrap());
        drop(inner);
        // 3.阻塞当前和调度
        block_current_and_run_next();
        // 4.获得锁
        mutex.lock();
    }
}
```

内核只是实现了条件变量，用户要根据同步访问情况自行设计线程的同步（如共享变量要先获得锁，对于条件变量就直接使用）
# 死锁

死锁出现的四个必要条件：（减少一个就可防止死锁）

1. 互斥：线程互斥访问资源
2. 持有并等待：线程持有部分资源，同时又在等待其他资源
3. 非抢占：资源不可被抢占
4. 循环等待：线程间存在资源的持有/等待的环

## 1. 死锁预防

给锁进行排序，每个线程都按照排好的顺序依次申请锁和访问资源（打破循环等待）

## 2. 死锁避免

银行家算法：

### 1. 数据结构

* **Available**：可用资源向量，m个资源的一维数组，表示对应的资源还有多少可用
* **Max**：最大需求矩阵，n * m矩阵，n个线程对m个资源最大需求量
* **Allocation**：已分配资源矩阵，n * m矩阵
* **Need**：需求（还需要多少资源）矩阵，n * m矩阵
* Need\[i, j] = Max\[i, j] - Allocation\[i, j]

### 2. 步骤

Request是线程发出的请求矩阵，n * m矩阵，表示每个线程对每个资源现在所请求的资源。

* 如果可分配就试着分配，再[[#^check-23-05-07|检查是否安全]]，安全就实际分配，不安全就不分配（简单描述）

1. 若 Request\[i, j] <= Need\[i, j]就继续执行；否则出错，线程需要资源数已超过宣布的最大值
2. 若 Request\[i, j] <= Available\[i, j]就继续执行；否则出错，线程需要资源数已超过可用资源
3. 线程试着分配资源，再[[#^check-23-05-07|检查是否安全]]，安全就实际分配，不安全就不分配

### 3. 安全性检查 ^check-23-05-07

将所剩下的资源对所有的线程一个一个的试（分配），如果有线程可以完成就增加其释放的资源接着试，直到最后如果所有线程都执行完就是安全的；否则就是不安全的