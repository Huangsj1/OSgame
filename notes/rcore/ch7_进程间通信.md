![[Pasted image 20230504200310.png]]

![[Pasted image 20230504200326.png]]

![[ch7_进程间通信 2023-05-05 16.44.52.excalidraw|600]]

# 文件类型的扩展

## 1. 支持标准输入/输出文件

将原本与标准输入输出直接 `sys_read()` 和 `sys_write()` 改成通过文件形式来 `read()` 和 `write()`。这需要抽象标准输入输出为一个结构并实现`File Trait`

```rust
// os/src/fs/stdio.rs
// 抽象标准输入
pub struct Stdin;
// 抽象标准输出
pub struct Stdout;
// 实现对应的Trait
impl File for Stdin {
    fn read(&self, mut user_buf: UserBuffer) -> usize {
        assert_eq!(user_buf.len(), 1);
        // busy loop
        let mut c: usize;
        loop {
            c = console_getchar();
            if c == 0 {
                suspend_current_and_run_next();
                continue;
            } else {
                break;
            }
        }
        let ch = c as u8;
        unsafe { user_buf.buffers[0].as_mut_ptr().write_volatile(ch); }
        1
    }
    fn write(&self, _user_buf: UserBuffer) -> usize {
        panic!("Cannot write to stdin!");
    }
}

impl File for Stdout {
    fn read(&self, _user_buf: UserBuffer) -> usize{
        panic!("Cannot read from stdout!");
    }
    fn write(&self, user_buf: UserBuffer) -> usize {
        for buffer in user_buf.buffers.iter() {
            print!("{}", core::str::from_utf8(*buffer).unwrap());
        }
        user_buf.len()
    }
}
```

标准输入输出文件是所有进程创建时就有的，所以需要在创建进程的时候补充

```rust
// os/src/task/task.rs
// 1.创建新的进程
impl TaskControlBlock {
    pub fn new(elf_data: &[u8]) -> Self {
        ...
        let task_control_block = Self {
            pid: pid_handle,
            kernel_stack,
            inner: Mutex::new(TaskControlBlockInner {
                trap_cx_ppn,
                base_size: user_sp,
                task_cx_ptr: task_cx_ptr as usize,
                task_status: TaskStatus::Ready,
                memory_set,
                parent: None,
                children: Vec::new(),
                exit_code: 0,
                fd_table: vec![
                    // 0 -> stdin 标准输入
                    Some(Arc::new(Stdin)),
                    // 1 -> stdout 标准输出
                    Some(Arc::new(Stdout)),
                    // 2 -> stderr 错误输出(这里也是标准输出)
                    Some(Arc::new(Stdout)),
                ],
            }),
        };
        ...
    }
}

// 2.fork继承父进程
impl TaskControlBlock {
    pub fn fork(self: &Arc<TaskControlBlock>) -> Arc<TaskControlBlock> {
        ...
        // push a goto_trap_return task_cx on the top of kernel stack
        let task_cx_ptr = kernel_stack.push_on_top(TaskContext::goto_trap_return());
        // copy fd table 复制父进程所有打开的文件
        let mut new_fd_table: Vec<Option<Arc<dyn File + Send + Sync>>> = Vec::new();
        for fd in parent_inner.fd_table.iter() {
            if let Some(file) = fd {
                new_fd_table.push(Some(file.clone()));
            } else {
                new_fd_table.push(None);
            }
        }
        let task_control_block = Arc::new(TaskControlBlock {
            pid: pid_handle,
            kernel_stack,
            inner: Mutex::new(TaskControlBlockInner {
                trap_cx_ppn,
                base_size: parent_inner.base_size,
                task_cx_ptr: task_cx_ptr as usize,
                task_status: TaskStatus::Ready,
                memory_set,
                parent: Some(Arc::downgrade(self)),
                children: Vec::new(),
                exit_code: 0,
                fd_table: new_fd_table,
            }),
        });
        // add child
        ...
    }
}
```

## 2. 支持管道文件

> 管道可以当作文件来使用 (像文件一样读写)，用于进程间(父子进程)通信；当用完的时候要像文件一样 `close()` 关闭。
> 管道本身实际上只是一个共享的环形队列，经过文件抽象的是管道的端口(读写端口)

### 1. 管道的使用

**用户空间**的使用

```rust
// user/src/lib.rs
pub fn pipe(pipe_fd: &mut [usize]) -> isize { sys_pipe(pipe_fd) }

// -------------------------syscall-----------------------
// user/src/syscall.rs
const SYSCALL_PIPE: usize = 59;

pub fn sys_pipe(pipe: &mut [usize]) -> isize {
    syscall(SYSCALL_PIPE, [pipe.as_mut_ptr() as usize, 0, 0])
}
```

父进程创建管道后，文件描述符表中包含两个读写相关的管道的描述符，通过 `fork()` 后子进程也同样有对应的管道的描述符；之后父进程关闭读通道，子进程关闭写通道，二者向文件一样读写来通信

```rust
// user/src/bin/pipetest.rs
#[no_mangle]
pub fn main() -> i32 {
    // create pipe
    let mut pipe_fd = [0usize; 2];
    pipe(&mut pipe_fd);
    // read end
    assert_eq!(pipe_fd[0], 3);
    // write end
    assert_eq!(pipe_fd[1], 4);
    if fork() == 0 {
        // child process, read from parent
        // close write_end
        close(pipe_fd[1]);
        let mut buffer = [0u8; 32];
        let len_read = read(pipe_fd[0], &mut buffer) as usize;
        // close read_end
        close(pipe_fd[0]);
        assert_eq!(core::str::from_utf8(&buffer[..len_read]).unwrap(), STR);
        println!("Read OK, child process exited!");
        0
    } else {
        // parent process, write to child
        // close read end
        close(pipe_fd[0]);
        assert_eq!(write(pipe_fd[1], STR.as_bytes()), STR.len() as isize);
        // close write end
        close(pipe_fd[1]);
        let mut child_exit_code: i32 = 0;
        wait(&mut child_exit_code);
        assert_eq!(child_exit_code, 0);
        println!("pipetest passed!");
        0
    }
}
```

**内核空间**的使用

```rust
/// 功能：为当前进程打开一个管道。
/// 参数：pipe 表示应用地址空间中的一个长度为 2 的 usize 数组的起始地址，内核需要按顺序将管道读端
/// 和写端的文件描述符写入到数组中。
/// 返回值：如果出现了错误则返回 -1，否则返回 0 。可能的错误原因是：传入的地址不合法。
/// syscall ID：59
pub fn sys_pipe(pipe: *mut usize) -> isize;
```

### 2. 管道结构的实现

将管道的一端（读端或者写端）抽象为 `Pipe` 类型（这只是接入管道的接口）

```rust
// os/src/fs/pipe.rs
// 每个端口只能有一种权限(读/写)
pub struct Pipe {
    readable: bool,
    writable: bool,
    buffer: Arc<Mutex<PipeRingBuffer>>,
}
```

实际管道是一个经过包装的**环形字节队列**, 抽象为 `PipeRingBuffer`结构

```rust
// os/src/fs/pipe.rs
const RING_BUFFER_SIZE: usize = 32;

#[derive(Copy, Clone, PartialEq)]
enum RingBufferStatus {
    FULL,
    EMPTY,
    NORMAL,
}

pub struct PipeRingBuffer {
    arr: [u8; RING_BUFFER_SIZE],
    // 指向第一个未使用的字节
    head: usize,
    // 指向最后一个未使用的字节的后一个位置
    tail: usize,
    status: RingBufferStatus,
    // 弱指针,指向管道的写端
    write_end: Option<Weak<Pipe>>,
}
```

### 3. 管道的创建

管道 `PipeRingBuffer` 的创建

```rust
// os/src/fs/pipe.rs

impl PipeRingBuffer {
    pub fn new() -> Self {
        Self {
            arr: [0; RING_BUFFER_SIZE],
            head: 0,
            tail: 0,
            status: RingBufferStatus::EMPTY,
            write_end: None,
        }
    }
}
```

管道端口（读写端）`Pipe` 的创建：根据管道构建对应的端口

```rust
// os/src/fs/pipe.rs

impl Pipe {
    pub fn read_end_with_buffer(buffer: Arc<Mutex<PipeRingBuffer>>) -> Self {
        Self {
            readable: true,
            writable: false,
            buffer,
        }
    }
    pub fn write_end_with_buffer(buffer: Arc<Mutex<PipeRingBuffer>>) -> Self {
        Self {
            readable: false,
            writable: true,
            buffer,
        }
    }
}
```

创建一个管道并返回他的读写端口

```rust
// os/src/fs/pipe.rs
impl PipeRingBuffer {
	// 添加写端口
    pub fn set_write_end(&mut self, write_end: &Arc<Pipe>) {
        self.write_end = Some(Arc::downgrade(write_end));
    }
}

/// Return (read_end, write_end)
pub fn make_pipe() -> (Arc<Pipe>, Arc<Pipe>) {
	// 1.新建管道
    let buffer = Arc::new(Mutex::new(PipeRingBuffer::new()));
    // 2.新建读端
    let read_end = Arc::new(
        Pipe::read_end_with_buffer(buffer.clone())
    );
    // 3.新建写端
    let write_end = Arc::new(
        Pipe::write_end_with_buffer(buffer.clone())
    );
    buffer.lock().set_write_end(&write_end);
    (read_end, write_end)
}
```

创建管道的**系统调用** `sys_pipe`

```rust
// os/src/task/task.rs
impl TaskControlBlockInner {
	// 分配一个文件描述符来描述管道端口
    pub fn alloc_fd(&mut self) -> usize {
        if let Some(fd) = (0..self.fd_table.len())
            .find(|fd| self.fd_table[*fd].is_none()) {
            fd
        } else {
            self.fd_table.push(None);
            self.fd_table.len() - 1
        }
    }
}

// os/src/syscall/fs.rs
pub fn sys_pipe(pipe: *mut usize) -> isize {
    let task = current_task().unwrap();
    let token = current_user_token();
    let mut inner = task.acquire_inner_lock();
    // 1.先创建管道并返回两个端口
    let (pipe_read, pipe_write) = make_pipe();
    // 2.分配读端口的文件描述符并加入到文件描述符表中
    let read_fd = inner.alloc_fd();
    inner.fd_table[read_fd] = Some(pipe_read);
    // 3.分配写端口的文件描述符并加入到文件描述符表中
    let write_fd = inner.alloc_fd();
    inner.fd_table[write_fd] = Some(pipe_write);
    // 4.将读写端口的文件描述符写到对应的用户空间的存储的数组中
    *translated_refmut(token, pipe) = read_fd;
    *translated_refmut(token, unsafe { pipe.add(1) }) = write_fd;
    0
}
```

### 4. 通过管道进行读写（读写类似, 下面只提供读）

首先让管道本身提供读写接口

```rust
// os/src/fs/pipe.rs

impl PipeRingBuffer {
	// 读取一个字节(读取之前要保证有得读)
    pub fn read_byte(&mut self) -> u8 {
        self.status = RingBufferStatus::NORMAL;
        let c = self.arr[self.head];
        self.head = (self.head + 1) % RING_BUFFER_SIZE;
        if self.head == self.tail {
            self.status = RingBufferStatus::EMPTY;
        }
        c
    }
    // 查看可以读多少个字节
    pub fn available_read(&self) -> usize {
        if self.status == RingBufferStatus::EMPTY {
            0
        } else {
            if self.tail > self.head {
                self.tail - self.head
            } else {
                self.tail + RING_BUFFER_SIZE - self.head
            }
        }
    }
    // 判断是否所有的写端都关闭了
    pub fn all_write_ends_closed(&self) -> bool {
        self.write_end.as_ref().unwrap().upgrade().is_none()
    }
}
```

再为管道端口 `Pipe` 实现 `File` Trait 来读写

```rust
impl File for Pipe {
    fn read(&self, buf: UserBuffer) -> usize {
        assert!(self.readable());
        let want_to_read = buf.len();
        let mut buf_iter = buf.into_iter();
        let mut already_read = 0usize;
        loop {
            let mut ring_buffer = self.buffer.exclusive_access();
            let loop_read = ring_buffer.available_read();
            // 1.如果没有得读
            if loop_read == 0 {
	            // 如果没有写端口就返回
                if ring_buffer.all_write_ends_closed() {
                    return already_read;
                }
                // 否则调度等待
                drop(ring_buffer);
                suspend_current_and_run_next();
                continue;
            }
            // 2.将管道队列中的字节读出来
            for _ in 0..loop_read {
	            // 如果还需要读(没读完)
                if let Some(byte_ref) = buf_iter.next() {
                    unsafe {
                        *byte_ref = ring_buffer.read_byte();
                    }
                    already_read += 1;
                    // 如果读满了需要读的数据
                    if already_read == want_to_read {
                        return want_to_read;
                    } 
                } else { // 否则存放读的内容的地方已满
                    return already_read;
                }
            }
        }
    }
}
```

# 实现标准 I/O 重定向

## 1. 为程序执行添加命令行参数

> 将**用户态**的 `String` 类型的命令行参数转成 `usize` 用户虚拟地址类型，然后在内核中将 *用户虚拟地址转化成内核地址* 得到命令行参数字符串的地址，再将 `usize` 类型提取转化成在**内核中**的 `String` 类型，最后将字符串里的各个字符压入到用户栈中让用户栈的指针能在用户态中找到对应的字符串

### 1. **用户态**的使用
```rust
// user/src/syscall.rs
// path为执行文件名的字符串；args为命令行参数字符串数组
pub fn sys_exec(path: &str, args: &[*const u8]) -> isize {
    syscall(SYSCALL_EXEC, [path.as_ptr() as usize, args.as_ptr() as usize, 0])
}

// user/src/lib.rs
pub fn exec(path: &str, args: &[*const u8]) -> isize { sys_exec(path, args) }
```

1. **shell程序**对命令行参数的分割（分割成上述 `exec()` 使用的格式) ：将命令行参数按空格分割开，同时为所有字符串后面添加 '\0'；最后添加一个空字符串指针作为数组结尾

```rust
// user/src/bin/user_shell.rs
let args: Vec<_> = line.as_str().split(' ').collect();
// 1.分割成单独的 String 类型
let mut args_copy: Vec<String> = args
.iter()
.map(|&arg| {
    let mut string = String::new();
    string.push_str(arg);
    string
})
.collect();
// 2.为所有的String后面添加 '\0'
// 因为 `exec()` 传入的是普通的字符串指针(不是String)，所以要在字符串末尾添加'\0'内核才能识别出字符串的结尾
args_copy
.iter_mut()
.for_each(|string| {
    string.push('\0');
});
// ...
// 3.在字符串数组末尾添加一个空指针作为结尾标记
// 因为syscall的接口传入的第二个参数中全是usize类型，所以这里的数组将会当作as_ptr()指针传入，就需要0结尾来判断结束
let mut args_addr: Vec<*const u8> = args_copy
.iter()
.map(|arg| arg.as_ptr())
.collect();
args_addr.push(0 as *const u8);
```

2. 在 `fork()` 出来的子进程中执行

```rust
// user/src/bin/user_shell.rs
// child process
if exec(args_copy[0].as_str(), args_addr.as_slice()) == -1 {
    println!("Error when executing!");
    return -4;
}
```

### 2. 内核提取命令行参数并且压入用户栈

首先在 `sys_exec()` 中需要将用户传来的命令行参数提取出来

```rust
// os/src/syscall/process.rs
pub fn sys_exec(path: *const u8, mut args: *const usize) -> isize {
    let token = current_user_token();
    let path = translated_str(token, path);
    let mut args_vec: Vec<String> = Vec::new();
    // 这里的args为命令行参数数组的起始(用户态虚拟)地址
    loop {
	    // 1.先将args翻译成实际物理地址再得到字符串(用户态)地址
        let arg_str_ptr = *translated_ref(token, args);
        if arg_str_ptr == 0 {
            break;
        }
        // 2.再将字符串用户态地址翻译成实际物理地址构建String
        args_vec.push(translated_str(token, arg_str_ptr as *const u8));
        unsafe { args = args.add(1); }
    }
    if let Some(app_inode) = open_file(path.as_str(), OpenFlags::RDONLY) {
        let all_data = app_inode.read_all();
        let task = current_task().unwrap();
        let argc = args_vec.len();
        // 这里传进来的args_vec中的元素是命令行参数String类型
        task.exec(all_data.as_slice(), args_vec);
        // return argc because cx.x[10] will be covered with it later
        argc as isize
    } else {
        -1
    }
}
```

然后再将字符串放入用户栈中：这里首先要放的是字符串的地址（用户态下的虚拟地址）在用户栈中，让地址指向字符串，这就需要将字符串提取出来放到用户栈后面，这样就可以保存提取后的字符串的地址让用户能正确访问到字符串

```rust
// os/src/task/task.rs

impl TaskControlBlock {
    pub fn exec(&self, elf_data: &[u8], args: Vec<String>) {
        // memory_set with elf program headers/trampoline/trap context/user stack
        let (memory_set, mut user_sp, entry_point) = MemorySet::from_elf(elf_data);
        let trap_cx_ppn = memory_set
            .translate(VirtAddr::from(TRAP_CONTEXT).into())
            .unwrap()
            .ppn();
        // push arguments on user stack
        // 这里的user_sp是用户空间下的用户栈的虚拟地址
        user_sp -= (args.len() + 1) * core::mem::size_of::<usize>();
        let argv_base = user_sp;
        // 将argv从虚拟地址转成实际物理地址
        let mut argv: Vec<_> = (0..=args.len())
            .map(|arg| {
                translated_refmut(
                    memory_set.token(),
                    (argv_base + arg * core::mem::size_of::<usize>()) as *mut usize
                )
            })
            .collect();
        // argv中最后一个字符串指针为0
        *argv[args.len()] = 0;
        for i in 0..args.len() {
            user_sp -= args[i].len() + 1;
            // argv中的每一个字符串的地址都是用户的虚拟地址,这样才能在用户态中正确访问字符串
            *argv[i] = user_sp;
            let mut p = user_sp;
            // 在argv中的字符串的地址处填入对应的字符串(一个一个字符地填)
            for c in args[i].as_bytes() {
                *translated_refmut(memory_set.token(), p as *mut u8) = *c;
                p += 1;
            }
            *translated_refmut(memory_set.token(), p as *mut u8) = 0;
        }
        // make the user_sp aligned to 8B for k210 platform
        user_sp -= user_sp % core::mem::size_of::<usize>();

        // **** hold current PCB lock
        let mut inner = self.acquire_inner_lock();
        // substitute memory_set
        inner.memory_set = memory_set;
        // update trap_cx ppn
        inner.trap_cx_ppn = trap_cx_ppn;
        // initialize trap_cx
        let mut trap_cx = TrapContext::app_init_context(
            entry_point,
            user_sp,
            KERNEL_SPACE.lock().token(),
            self.kernel_stack.get_top(),
            trap_handler as usize,
        );
        // a0为命令行参数个数
        trap_cx.x[10] = args.len();
        // a1为命令行参数argv的地址
        trap_cx.x[11] = argv_base;
        *inner.get_trap_cx() = trap_cx;
        // **** release current PCB lock
    }
}
```

![[Pasted image 20230505095111.png|200]]

### 3. 用户入口处从用户栈提取出命令行参数

根据传入的参数 `argc` 和 `argv` 可以从用户栈中提取所有的命令行参数字符串

```rust
// user/src/lib.rs
#[no_mangle]
#[link_section = ".text.entry"]
pub extern "C" fn _start(argc: usize, argv: usize) -> ! {
    unsafe {
        HEAP.lock()
            .init(HEAP_SPACE.as_ptr() as usize, USER_HEAP_SIZE);
    }
    // 存储所有的命令行参数字符串
    let mut v: Vec<&'static str> = Vec::new();
    for i in 0..argc {
        let str_start = unsafe {
            ((argv + i * core::mem::size_of::<usize>()) as *const usize).read_volatile()
        };
        // 一个字符一个字符地检查直到找到'\0'来得到长度
        let len = (0usize..).find(|i| unsafe {
            ((str_start + *i) as *const u8).read_volatile() == 0
        }).unwrap();
        v.push(
            core::str::from_utf8(unsafe {
                core::slice::from_raw_parts(str_start as *const u8, len)
            }).unwrap()
        );
    }
    exit(main(argc, v.as_slice()));
}
```

## 2. 输入输出重定向

> 默认情况下一个程序的标准输入输出是指键盘和屏幕，经过重定向后可以将其改为对应的文件（例如: Linux中 `>` 可以将输出重定向）

输入输出重定向，本质上就是把原本处在进程描述符表中的默认位置的 `Stdin` 和 `Stdout` 删除，再通过dup把重定位的文件描述符加入到标准输入/输出的位置

1. 添加 `sys_dup()` 系统调用

```rust
// user/src/syscall.rs
/// 功能：将进程中一个已经打开的文件复制一份并分配到一个新的文件描述符中。
/// 参数：fd 表示进程中一个已经打开的文件的文件描述符。
/// 返回值：如果出现了错误则返回 -1，否则能够访问已打开文件的新文件描述符。
/// 可能的错误原因是：传入的 fd 并不对应一个合法的已打开文件。
/// syscall ID：24
pub fn sys_dup(fd: usize) -> isize;

// os/src/syscall/fs.rs
pub fn sys_dup(fd: usize) -> isize {
    let task = current_task().unwrap();
    let mut inner = task.acquire_inner_lock();
    if fd >= inner.fd_table.len() {
        return -1;
    }
    if inner.fd_table[fd].is_none() {
        return -1;
    }
    // 分配一个文件描述符fd(分配的是最小的fd)
    let new_fd = inner.alloc_fd();
    inner.fd_table[new_fd] = Some(Arc::clone(inner.fd_table[fd].as_ref().unwrap()));
    new_fd as isize
}
```

2. 在shell中检查是否有 `>` 或者 `<` ，有的话就需要先保存对应的文件名

```rust
// user/src/bin/user_shell.rs
// redirect input
let mut input = String::new();
if let Some((idx, _)) = args_copy
.iter()
.enumerate()
.find(|(_, arg)| arg.as_str() == "<\0") {
    input = args_copy[idx + 1].clone();
    args_copy.drain(idx..=idx + 1);
}

// redirect output
let mut output = String::new();
if let Some((idx, _)) = args_copy
.iter()
.enumerate()
.find(|(_, arg)| arg.as_str() == ">\0") {
    output = args_copy[idx + 1].clone();
    args_copy.drain(idx..=idx + 1);
}
```

在 `fork()` 之后，如果需要重定位就先要打开对应的文件得到描述符，再关闭标准输入/输出的描述符，最后通过 `dup()` 复制到原本标准输入/输出的描述符中去，这样与其相关的输入/输出都会对应到指定的文件

# 信号

> 信号可以通过一个进程或内核向另一个进程发出 (异步)，也可以是进程再执行过程中出现问题自身触发 (同步)，这些信号都会被暂时添加到对应进程的TCB中，待该进程处理Trap进入内核后返回用户态过程中才会被执行

## 1. 用户态的使用

1. 发送信号：可通过 `kill()` 向对应的进程发送对应的信号

```rust
// user/src/lib.rs
/// 功能：当前进程向另一个进程（可以是自身）发送一个信号。
/// 参数：pid 表示接受信号的进程的进程 ID, signum 表示要发送的信号的编号。
/// 返回值：如果传入参数不正确（比如指定进程或信号类型不存在）则返回 -1 ,否则返回 0 。
/// syscall ID: 129
pub fn kill(pid: usize, signum: i32) -> isize;

// 各种信号的定义
pub const SIGDEF: i32 = 0; // Default signal handling
pub const SIGHUP: i32 = 1;
pub const SIGINT: i32 = 2;
pub const SIGQUIT: i32 = 3;
pub const SIGILL: i32 = 4;
pub const SIGTRAP: i32 = 5;
pub const SIGABRT: i32 = 6;
pub const SIGBUS: i32 = 7;
pub const SIGFPE: i32 = 8;
pub const SIGKILL: i32 = 9;
...
```

`SignalAction` 结构包含信号处理函数 `handler` 和 掩码字段 `mask`（这个掩码是当正在执行这个处理例程的时候屏蔽哪些信号的处理，但信号还是在那，只是不被处理）

```rust
// user/src/lib.rs

/// Action for a signal
#[repr(C, align(16))]
#[derive(Debug, Clone, Copy)]
pub struct SignalAction {
	// 用户空间下的信号处理函数
    pub handler: usize,
    // 该信号处理例程的掩码
    pub mask: SignalFlags,
}

bitflags! {
    pub struct SignalFlags: i32 {
        const SIGDEF = 1; // Default signal handling
        const SIGHUP = 1 << 1;
        const SIGINT = 1 << 2;
        const SIGQUIT = 1 << 3;
        const SIGILL = 1 << 4;
        const SIGTRAP = 1 << 5;
        ...
        const SIGSYS = 1 << 31;
    }
}
```

2. 处理信号：通过系统调用自定义信号处理函数
	1. `sys_sigaction()` 设置信号处理例程
	2. `sys_procmask()` 设置进程的信号屏蔽掩码（这是全局信号屏蔽，虽然信号还在，但是不处理）
	3. `sys_sigreturn()` 从信号处理例程返回 (自定义函数返回的时候一定要调用这个)

```rust
// os/src/syscall/process.rs
// 1.内核中:设置相关信号处理例程
/// 功能：为当前进程设置某种信号的处理函数，同时保存设置之前的处理函数。
/// 参数：signum 表示信号的编号，action 表示要设置成的处理函数的指针
/// old_action 表示用于保存设置之前的处理函数的指针（SignalAction 结构稍后介绍）。
/// 返回值：如果传入参数错误（比如传入的 action 或 old_action 为空指针或者）
/// 信号类型不存在返回 -1 ，否则返回 0 。
/// syscall ID: 134
pub fn sys_sigaction(
    signum: i32,
    action: *const SignalAction,
    old_action: *mut SignalAction,
) -> isize;

//---------------------------user下------------------------
// user/src/lib.rs
// 1.设置信号处理例程
pub fn sigaction(
    signum: i32,
    action: Option<&SignalAction>,
    old_action: Option<&mut SignalAction>,
) -> isize {
    sys_sigaction(
        signum,
        action.map_or(core::ptr::null(), |a| a),
        old_action.map_or(core::ptr::null_mut(), |a| a)
    )
}

// 2.设置信号掩码
/// 功能：设置当前进程的全局信号掩码。
/// 参数：mask 表示当前进程要设置成的全局信号掩码，代表一个信号集合，
/// 在集合中的信号始终被该进程屏蔽。
/// 返回值：如果传入参数错误返回 -1 ，否则返回之前的信号掩码 。
/// syscall ID: 135
pub fn sigprocmask(mask: u32) -> isize;

// 3.信号处理函数的返回函数
/// 功能：进程通知内核信号处理例程退出，可以恢复原先的进程执行。
/// 返回值：如果出错返回 -1，否则返回 0 。
/// syscall ID: 139
pub fn sigreturn() -> isize;
```

使用例子

```rust
// user/src/bin/sig_simple.rs
fn func() {
    println!("user_sig_test passed");
    // 3.信号处理函数返回
    sigreturn();
}

#[no_mangle]
pub fn main() -> i32 {
	// 先定义相关信号结构
    let mut new = SignalAction::default();
    let mut old = SignalAction::default();
    // 设置对应的处理函数
    new.handler = func as usize;

    println!("signal_simple: sigaction");
    // 1.设置信号处理例程
    if sigaction(SIGUSR1, Some(&new), Some(&mut old)) < 0 {
        panic!("Sigaction failed!");
    }
    println!("signal_simple: kill");
    // 2.发送信号给自己
    if kill(getpid() as usize, SIGUSR1) < 0 {
        println!("Kill failed!");
        exit(1);
    }
    println!("signal_simple: Done");
    0
}
```

## 2. 内核的实现

整个过程包括信号[[#^prepare-23-05-05|使用前的准备]]，[[#^generate-23-05-05|信号的产生]] 和 [[#^handle-23-05-05|信号的处理]]

### 1. 设置信号处理例程和信号掩码 ^prepare-23-05-05

先为TCB新增相关的成员

```rust
// os/src/task/task.rs
pub struct TaskControlBlockInner {
    ...
    // 当前待处理的信号
    pub signals: SignalFlags,
    // 全局掩码
    pub signal_mask: SignalFlags,
    // 当前正在被处理的信号
    pub handling_sig: isize,
    // 一个包含着SignalAction的数组
    pub signal_actions: SignalActions,
    // if the task is killed(杀死)
    pub killed: bool,
    // if the task is frozen by a signal(暂停)
    pub frozen: bool,
    // 回到用户的信号处理函数时要保护原来的用户上下文
    pub trap_ctx_backup: Option<TrapContext>,
}

// os/src/task/signal.rs
pub const MAX_SIG: usize = 31;

// os/src/task/action.rs
#[derive(Clone)]
pub struct SignalActions {
	// 每个SignalAction都包含了用户的信号处理函数和该例程的掩码
    pub table: [SignalAction; MAX_SIG + 1],
}
```

全局掩码的设置

```rust
// os/src/process.rs
pub fn sys_sigprocmask(mask: u32) -> isize {
    if let Some(task) = current_task() {
        let mut inner = task.inner_exclusive_access();
        let old_mask = inner.signal_mask;
        if let Some(flag) = SignalFlags::from_bits(mask) {
            inner.signal_mask = flag;
            old_mask.bits() as isize
        } else {
            -1
        }
    } else {
        -1
    }
}
```

设置对应信号的信号处理函数

```rust
// os/src/syscall/process.rs
// 判断该设置函数是否有问题(缺少其中一个signal 或者 为SIGKILL和SIGSTOP信号设置了信号处理函数 都是错误的)
fn check_sigaction_error(signal: SignalFlags, action: usize, old_action: usize) -> bool {
    if action == 0
        || old_action == 0
        || signal == SignalFlags::SIGKILL
        || signal == SignalFlags::SIGSTOP
    {
        true
    } else {
        false
    }
}

pub fn sys_sigaction(
    signum: i32,
    action: *const SignalAction,
    old_action: *mut SignalAction,
) -> isize {
    let token = current_user_token();
    let task = current_task().unwrap();
    let mut inner = task.inner_exclusive_access();
    if signum as usize > MAX_SIG {
        return -1;
    }
    if let Some(flag) = SignalFlags::from_bits(1 << signum) {
        if check_sigaction_error(flag, action as usize, old_action as usize) {
            return -1;
        }
        let prev_action = inner.signal_actions.table[signum as usize];
        // 将之前的信号处理例程返回到用户空间
        *translated_refmut(token, old_action) = prev_action;
        // 设置当前的信号处理例程
        inner.signal_actions.table[signum as usize] = *translated_ref(token, action);
        0
    } else {
        -1
    }
}
```

### 2. 信号的产生 ^generate-23-05-05

信号的产生包括以下几种 (1、2是异步，3是同步)：

1. 一个进程通过 `kill()` 系统调用向自己/其他进程发送信号
2. 内核检测到某些事件向某个进程发送信号，但该事件与接收信号的进程的执行无关（`SIGCHLD`子进程状态改变发送信号到父进程）
3. 进程执行过程中出事, 在 Trap 到内核的时候内核向该进程发送信号

`kill()` 的系统调用

```rust
// os/src/syscall/process.rs
pub fn sys_kill(pid: usize, signum: i32) -> isize {
    if let Some(task) = pid2task(pid) {
        if let Some(flag) = SignalFlags::from_bits(1 << signum) {
            // insert the signal if legal
            let mut task_ref = task.inner_exclusive_access();
            if task_ref.signals.contains(flag) {
                return -1;
            }
            // 插入对应的信号
            task_ref.signals.insert(flag);
            0
        } else {
            -1
        }
    } else {
        -1
    }
}
```

执行出错 Trap 到内核中插入信号（不再是马上exit，而是通过出错的信号在信号处理结束的时候再exit）

```rust
// os/src/trap/mod.rs
#[no_mangle]
pub fn trap_handler() -> ! {
    ...
    match scause.cause() {
        ...
        Trap::Exception(Exception::StoreFault)
        | Trap::Exception(Exception::StorePageFault)
        | Trap::Exception(Exception::InstructionFault)
        | Trap::Exception(Exception::InstructionPageFault)
        | Trap::Exception(Exception::LoadFault)
        | Trap::Exception(Exception::LoadPageFault) => {
            /*
            println!(
                "[kernel] {:?} in application, bad addr = {:#x}, bad instruction = {:#x}, kernel killed it.",
                scause.cause(),
                stval,
                current_trap_cx().sepc,
            );
            */
            // 插入SIGSEGV的信号
            current_add_signal(SignalFlags::SIGSEGV);
        }
        Trap::Exception(Exception::IllegalInstruction) => {
            // 插入SIGILL的信号
            current_add_signal(SignalFlags::SIGILL);
        ...
    }
    ...
}

// os/src/task/mod.rs
pub fn current_add_signal(signal: SignalFlags) {
    let task = current_task().unwrap();
    let mut task_inner = task.inner_exclusive_access();
    task_inner.signals |= signal;
}
```

### 3. 信号的处理 ^handle-23-05-05

在 `trap_handler()` 返回用户之前会调用 `handle_signals()` 处理信号，并在之后 `chesk_signals_error_of_current()` 判断是否有对应信号的出错（如上面的程序执行出错时记录的信号）并直接退出出错进程

```rust
// os/src/trap/mod.rs

#[no_mangle]
pub fn trap_handler() -> ! {
    ...
    // 1.处理信号
    handle_signals();
    // 2.check error signals (if error then exit)
    if let Some((errno, msg)) = check_signals_error_of_current() {
        println!("[kernel] {}", msg);
        exit_current_and_run_next(errno);
    }
    trap_return();
}
```

1. `handle_signals()` 处理信号

```rust
// os/src/task/mod.rs
pub fn handle_signals() {
    loop {
	    // 实际处理信号函数
        check_pending_signals();
        let (frozen, killed) = {
            let task = current_task().unwrap();
            let task_inner = task.inner_exclusive_access();
            (task_inner.frozen, task_inner.killed)
        };
        // 如果该进程未暂停/已终止就退出
        if !frozen || killed {
            break;
        }
        // 如果该进程暂停就切换进程
        suspend_current_and_run_next();
    }
}

// os/src/task/mod.rs
fn check_pending_signals() {
	// 对所有的信号进行处理
    for sig in 0..(MAX_SIG + 1) {
        let task = current_task().unwrap();
        let task_inner = task.inner_exclusive_access();
        let signal = SignalFlags::from_bits(1 << sig).unwrap();
        // 1.有信号且不在全局掩码中
        if task_inner.signals.contains(signal) && (!task_inner.signal_mask.contains(signal)) {
            let mut masked = true;
            let handling_sig = task_inner.handling_sig;
            // 没有正在执行的处理例程,就没有局部掩码
            if handling_sig == -1 {
                masked = false;
            } else {
                let handling_sig = handling_sig as usize;
                if !task_inner.signal_actions.table[handling_sig]
                    .mask
                    .contains(signal)
                {
                    masked = false;
                }
            }
            // 2.不在局部掩码中
            if !masked {
                drop(task_inner);
                drop(task);
                if signal == SignalFlags::SIGKILL
                    || signal == SignalFlags::SIGSTOP
                    || signal == SignalFlags::SIGCONT
                    || signal == SignalFlags::SIGDEF
                {
                    // signal is a kernel signal
                    call_kernel_signal_handler(signal);
                } else {
                    // signal is a user signal
                    call_user_signal_handler(sig, signal);
                    return;
                }
            }
        }
    }
}
```

有的信号只能由内核来处理，如 `SIGKILL`, `SIGSTOP`, `SIGCONT`, `SIGDEF`；其他信号由用户来处理，若用户未定义处理函数就打印并忽略，否则准备跳转

```rust
// os/src/task/mod.rs
fn call_kernel_signal_handler(signal: SignalFlags) {
    let task = current_task().unwrap();
    let mut task_inner = task.inner_exclusive_access();
    match signal {
        SignalFlags::SIGSTOP => {
            task_inner.frozen = true;
            task_inner.signals ^= SignalFlags::SIGSTOP;
        }
        SignalFlags::SIGCONT => {
            if task_inner.signals.contains(SignalFlags::SIGCONT) {
                task_inner.signals ^= SignalFlags::SIGCONT;
                task_inner.frozen = false;
            }
        }
        _ => {
            // println!(
            //     "[K] call_kernel_signal_handler:: current task sigflag {:?}",
            //     task_inner.signals
            // );
            task_inner.killed = true;
        }
    }
}

fn call_user_signal_handler(sig: usize, signal: SignalFlags) {
    let task = current_task().unwrap();
    let mut task_inner = task.inner_exclusive_access();

    let handler = task_inner.signal_actions.table[sig].handler;
    // 如果用户定义了信号处理函数
    if handler != 0 {
        // user handler

        // handle flag
        // 1.设置当前信号处理例程，清除信号
        task_inner.handling_sig = sig as isize;
        task_inner.signals ^= signal;

        // backup trapframe
        // 2.保存用户上下文到TCB中
        let mut trap_ctx = task_inner.get_trap_cx();
        task_inner.trap_ctx_backup = Some(*trap_ctx);

        // modify trapframe
        // 3.修改返回地址到用户处理程序中
        trap_ctx.sepc = handler;

        // put args (a0)
        // 4.修改传入参数a0为信号的类型
        trap_ctx.x[10] = sig;
    } else {
        // default action
        println!("[K] task/call_user_signal_handler: default action: ignore it or kill process");
    }
}
```

2. `chesk_signals_error_of_current()` 检查是否有错误要退出进程

```rust
impl SignalFlags {
    pub fn check_error(&self) -> Option<(i32, &'static str)> {
        if self.contains(Self::SIGINT) {
            Some((-2, "Killed, SIGINT=2"))
        } else if self.contains(Self::SIGILL) {
            Some((-4, "Illegal Instruction, SIGILL=4"))
        } else if self.contains(Self::SIGABRT) {
            Some((-6, "Aborted, SIGABRT=6"))
        } else if self.contains(Self::SIGFPE) {
            Some((-8, "Erroneous Arithmetic Operation, SIGFPE=8"))
        } else if self.contains(Self::SIGKILL) {
            Some((-9, "Killed, SIGKILL=9"))
        } else if self.contains(Self::SIGSEGV) {
            Some((-11, "Segmentation Fault, SIGSEGV=11"))
        } else {
            //println!("[K] signalflags check_error  {:?}", self);
            None
        }
    }
}
```

### 4. 从用户的信号处理函数返回

需要通过系统调用 `sys_sigreturn()` 在内核中恢复原来用户上下文并返回

```rust
// os/src/syscall/process.rs
pub fn sys_sigreturn() -> isize {
    if let Some(task) = current_task() {
        let mut inner = task.inner_exclusive_access();
        inner.handling_sig = -1;
        // 1.restore the trap context
        let trap_ctx = inner.get_trap_cx();
        *trap_ctx = inner.trap_ctx_backup.unwrap();
        // 2.记录系统调用的返回值
        trap_ctx.x[10] as isize
    } else {
        -1
    }
}
```