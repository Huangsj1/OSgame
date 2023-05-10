![[Pasted image 20230508101128.png]]

> 当**外设**产生中断请求时，**中断控制器芯片**会检测到该中断信号并产生一个中断请求发送给**PLIC**，之后PLIC会根据优先级等信息将中断请求分配到一个/多个**CPU**中进行处理。CPU处理完中断请求后会修改 `Claim` / `Complete` 寄存器来通知PLIC中断请求已经处理完成

为了使整个中断过程可行，需要完成以下三个步骤：

1. **配置PLIC的各个寄存器**，包括优先级、中断向量号、中断使能、中断屏蔽等信息，以便管理和处理中断请求
2. **配置中断控制芯片**，将外设的中断信号引脚连接到中断控制器芯片上，并将中断控制器芯片的输出连接到PLIC的输入上（QEMU中是由QEMU根据设备树的信息为设备模拟实现的）
3. **配置外设**，使其能产生中断请求（QEMU会将中断请求传送到中断控制器再传送到PLIC中）

![[ch9_IO设备 2023-05-08 19.59.32.excalidraw|600]]

# I/O设备

## 1. I/O传输方式

### 1. Programmed I/O ( PIO )

CPU通过发出I/O指令的方式来进行数据传输，通过**不断循环读取相关寄存器**直到CPU可以继续执行I/O操作后才可做其他事。PIO包括MMIO（Memory-mapped PIO）和PMIO（Port-Mapped PIO）：MMIO通过将I/O设备物理地址映射到内存空间，CPU通过 *普通访存* 指令传送到设备在主存位置；PMIO的I/O设备有自己的独立地址空间，CPU需要通过 *特殊I/O指令访问端口*，如x86中的 `IN` 和 `OUT` 指令来访问

简化的抽象设备接口：状态、命令、数据

```rust
while STATUS == BUSY {};   // 等待设备执行完毕
DATA =  data;              // 把数据传给设备
COMMAND = command;         // 发命令给设备
while STATUS == BUSY {};   // 等待设备执行完毕
```

### 2. Interrupt based I/O

CPU可通过PIO方式来通知外设，发完通知就可继续执行与I/O设备无关的其他事情；当中断控制器检查到I/O设备准备好传输数据后就**发送中断信号给CPU**，打断当前执行，处理I/O传输

简化的抽象设备接口：状态、命令、数据、中断

```rust
DATA =  data;          // 把数据传给设备
COMMAND = command;     // 发命令给设备
do_otherwork();        // 做其它事情
...                    // I/O设备完成I/O操作，并产生中断
...                    // CPU执行被打断以响应中断
trap_handler();        // 执行中断处理例程中的相关I/O中断处理
					   // CPU读取准备好的数据
restore_do_otherwork();// 恢复CPU之前被打断的执行
...                    // 可继续进行I/O操作
```

### 3. Direct Memory Access ( DMA )

当CPU需要从内存中读取/写入设备数据时，提前向DMA控制器发出准备请求，然后DMA控制器会在后续阶段直接将数据写道目标位置，允许**设备直接将数据传输到内存中**，不需要通过CPU来直接处理，最后再向CPU发送中断通知执行完成

简化的抽象设备接口：状态、命令、数据、中断

```rust
DATA =  data block;    // 把数据传给设备
COMMAND = command;     // 发命令给设备
do_otherwork();        // 做其它事情
...                    // I/O设备完成I/O所有操作，并产生中断
...                    // CPU执行被打断以响应中断
trap_handler();        // 执行中断处理例程中的相关I/O中断处理
restore_do_otherwork();// 恢复CPU之前被打断的执行
...                    // 可继续进行I/O操作
```

## 2. I/O执行模型

1. blocking IO
2. nonblocking IO
3. IO multiplexing
4. signal driven IO
5. asynchronous IO

当用户进程发出 `read` I/O系统调用时，主要经历了两个阶段：

1. 等待数据准备好
2. 把数据从内核拷贝到用户进程中

上述五种模型在这两个阶段有不同处理方式：阻塞和非阻塞区别在于内核数据还没准备好（第一阶段），用户进程是否会阻塞；同步与异步区别在于当数据从内核copy到用户空间时（第二阶段），用户进程是否会阻塞/参与（前4个都是同步，最后一个异步）

* 阻塞：用户发出IO系统调用后，进程会 *等待* 该IO操作完成
* 非阻塞：用户发出IO系统调用后，如果数据没准备好，进程会 *立即返回*；如果数据准备好了，用户进程会通过系统调用完成数据拷贝
* 同步：第二阶段导致进程阻塞，直到IO操作完成
* 异步：第二阶段不会导致进程阻塞

### 1. 阻塞IO（blocking IO）

阻塞+同步

![[Pasted image 20230508113942.png]]

### 2. 非阻塞IO（non-blocking IO）

非阻塞（轮询）+同步

![[Pasted image 20230508114029.png]]

### 3. 多路复用IO（IO multiplexing）

阻塞（轮询）+同步

![[Pasted image 20230508114231.png]]

### 4. 信号驱动IO（signal driven IO）

非阻塞（信号驱动下一步）+同步

![[Pasted image 20230508114304.png]]

### 5. 异步IO（asynchronous IO）

非阻塞（第一阶段直接返回做其他事）+异步

![[Pasted image 20230508114503.png]]
# 外设信息

## 1. 外设设备信息提取

在Risc-V中，设备信息由 bootloader，即OpenSBI / RustSBI固件完成的，它会探测包括物理内存在内的各种外设，并将结果以 **设备树二进制对象（DTB，Device Tree Blob）** 的格式保存在物理内存某个地方。然后 bootloader 会启动操作系统，并将防止 DTB 的物理地址放置在 `a1` 寄存器，同时将 **HART ID（HART，Hardware Thread，硬件线程，执行CPU的核）** 放在 `a0` 寄存器上，之后跳转到操作系统入口地址执行

下面是一个裸机环境下的测试用例（打印设备树信息）

```rust
//virtio_drivers/examples/riscv/src/main.rs
// a0寄存器值为hart_id，a1寄存器值为设备树信息地址
#[no_mangle]
extern "C" fn main(_hartid: usize, device_tree_paddr: usize) {
   ...
   init_dt(device_tree_paddr);
   ...
}

fn init_dt(dtb: usize) {
   info!("device tree @ {:#x}", dtb);
   // Safe because the pointer is a valid pointer to unaliased memory.
   let fdt = unsafe { Fdt::from_ptr(dtb as *const u8).unwrap() };
   walk_dt(fdt);
}
// 遍历所有设备
fn walk_dt(fdt: Fdt) {
   for node in fdt.all_nodes() {
      if let Some(compatible) = node.compatible() {
            if compatible.all().any(|s| s == "virtio,mmio") {
               virtio_probe(node);
            }
      }
   }
}
// 解析每个设备
fn virtio_probe(node: FdtNode) {
   //分析 reg 信息
   if let Some(reg) = node.reg().and_then(|mut reg| reg.next()) {
      let paddr = reg.starting_address as usize;
      let size = reg.size.unwrap();
      let vaddr = paddr;
      info!("walk dt addr={:#x}, size={:#x}", paddr, size);
      info!(
            "Device tree node {}: {:?}",
            node.name,
            node.compatible().map(Compatible::first),
      );
      let header = NonNull::new(vaddr as *mut VirtIOHeader).unwrap();
      //判断virtio设备类型
      match unsafe { MmioTransport::new(header) } {
            Err(e) => warn!("Error creating VirtIO MMIO transport: {}", e),
            Ok(transport) => {
               info!(
                  "Detected virtio MMIO device with vendor id {:#X}, device type {:?}, version {:?}",
                  transport.vendor_id(),
                  transport.device_type(),
                  transport.version(),
               );
               virtio_device(transport);
            }
      }
   }
}
// 对不同的virtio设备进行进一步的初始化工作
fn virtio_device(transport: impl Transport) {
   match transport.device_type() {
      DeviceType::Block => virtio_blk(transport),
      DeviceType::GPU => virtio_gpu(transport),
      DeviceType::Input => virtio_input(transport),
      DeviceType::Network => virtio_net(transport),
      t => warn!("Unrecognized virtio device: {:?}", t),
   }
}

fn virtio_gpu<T: Transport>(transport: T) {
   let mut gpu = VirtIOGpu::<HalImpl, T>::new(transport).expect("failed to create gpu driver");
   // 获得显示设备的长宽信息
   let (width, height) = gpu.resolution().expect("failed to get resolution");
   let width = width as usize;
   let height = height as usize;
   info!("GPU resolution is {}x{}", width, height);
   // 设置显示缓冲区
   let fb = gpu.setup_framebuffer().expect("failed to get fb");
   // 设置显示设备中的每个显示点的红、绿、蓝分量值，形成丰富色彩的图形
   for y in 0..height {
      for x in 0..width {
            let idx = (y * width + x) * 4;
            fb[idx] = x as u8;
            fb[idx + 1] = y as u8;
            fb[idx + 2] = (x + y) as u8;
      }
   }
   gpu.flush().expect("failed to flush");
   info!("virtio-gpu test finished");
}
```

## 2. 平台中断控制器（Platfrom Level Interrupt Controller, PLIC）

如果要让操作系统处理外设中断，需要对平台中断控制器初始化，他的一端汇聚了各种外设的中断信号，另一端连接到CPU外部中断引脚上。当一个外部设备发出中断请求，PLIC会将其发送到CPU，CPU再执行对应的中断处理程序来响应中断

与PLIC相关的寄存器

```
寄存器         地址      功能描述
Priority      0x0c00_0000    设置特定中断源的优先级
Pending       0x0c00_1000    包含已触发（正在处理）的中断列表
Enable        0x0c00_2000    启用/禁用某些中断源
Threshold     0x0c20_0000    设置中断能够触发的阈值
Claim         0x0c20_0004    按优先级顺序返回下一个中断
Complete      0x0c20_0004    写操作表示完成对特定中断的处理
```

不同设备的中断号也不同，根据中断号来区别不同中断

```rust
enum {
     UART0_IRQ = 10,
     RTC_IRQ = 11,
     VIRTIO_IRQ = 1, /* 1 to 8 */
     VIRTIO_COUNT = 8,
     PCIE_IRQ = 0x20, /* 32 to 35 */
     VIRTIO_NDEV = 0x35 /* Arbitrary maximum number of interrupts */
};
```

操作系统要响应外设的中断，需要两方面的工作：

1. 完成中断初始化，并将 `sie` 寄存器中的 `seie` 位设置为1，使CPU能接收通过PLIC传来的外部设备的中断信号
2. 通过MMIO方式对PLIC的寄存器进行初始化设置，才能使外设产生的中断传到PLIC
	1. 设置外设中断的优先级
	2. 设置外设中断的阈值，优先级小于阈值的中断会被屏蔽（不同特权级阈值可以不同）
	3. 激活外设中断，将 `Enable` 寄存器的外设中断编号索引的位置1

```rust
// os/src/boards/qemu.rs
pub fn device_init() {
   use riscv::register::sie;
   let mut plic = unsafe { PLIC::new(VIRT_PLIC) };
   let hart_id: usize = 0;
   let supervisor = IntrTargetPriority::Supervisor;
   let machine = IntrTargetPriority::Machine;
   // 2.2.设置PLIC中外设中断的阈值
   plic.set_threshold(hart_id, supervisor, 0);
   plic.set_threshold(hart_id, machine, 1);
   // 使能PLIC在CPU处于S-Mode下传递键盘/鼠标/块设备/串口外设中断
   // irq nums: 5 keyboard, 6 mouse, 8 block, 10 uart
   for intr_src_id in [5usize, 6, 8, 10] {
	  // 2.3.激活外设中断
      plic.enable(hart_id, supervisor, intr_src_id);
      // 2.1.设置优先级
      plic.set_priority(intr_src_id, 1);
   }
   // 1.设置S-Mode CPU使能中断
   unsafe {
      sie::set_sext();
   }
}

// 允许PLIC接收到对应CPU的对应中断号的中断信号
pub fn enable(
	&mut self,
	hart_id: usize,
	target_priority: IntrTargetPriority,
	intr_source_id: usize,
) {
	let (reg_ptr, shift) = self.enable_ptr(hart_id, target_priority, intr_source_id);
	// 在对应地址(寄存器)中写入值
	unsafe {
		reg_ptr.write_volatile(reg_ptr.read_volatile() | 1 << shift);
	}
}
```

当外设产生中断后，会将中断请求发送到CPU的中断控制单元（ICU），当CPU执行指令的时候会周期性检查中断控制单元的状态来了解是否有中断请求。CPU要通过读取PLIC的 `Claim` 寄存器才能知道是哪个设备传来的中断。操作系统完成中断处理后，还需要通知PLIC中断处理完毕，CPU需要向PLIC的 `Complete` 寄存器写入对应中断号为索引的位来通知

```rust
// os/src/boards/qemu.rs
pub fn irq_handler() {
   let mut plic = unsafe { PLIC::new(VIRT_PLIC) };
   // 1.读PLIC的 ``Claim`` 寄存器获得外设中断号
   let intr_src_id = plic.claim(0, IntrTargetPriority::Supervisor);
   match intr_src_id {
      5 => KEYBOARD_DEVICE.handle_irq(),
      6 => MOUSE_DEVICE.handle_irq(),
      8 => BLOCK_DEVICE.handle_irq(),
      10 => UART.handle_irq(),
      _ => panic!("unsupported IRQ {}", intr_src_id),
   }
   // 2.通知PLIC中断已处理完毕
   plic.complete(0, IntrTargetPriority::Supervisor, intr_src_id);
}
```

# 串口驱动程序

> 串口（Universal Asynchronous Receiver-Transmitter，UART）是一种用于传输、接收系列数据的外部设备，数据是逐位顺序发送的

每个UART使用8个I/O字节来访问其寄存器，下表显示UART每个寄存器的地址和基本含义：其中base是串口设备起始地址，有的寄存器的某些位表示这8个寄存器用的是哪一组（QEMU模拟的计算机中有8个串口设备，其在MMIO的起始地址为0x1000_0000）

![[Pasted image 20230508173226.png|400]]

通过查看 `dtc` （Device Tree Compiler）工具生成的 `riscv64-virt.dts`文件可以得到串口设备相关的MMIO模式的寄存器信息和中断信息

```rust
...// 字符会通过串口设备打印出来
chosen {
  bootargs = [00];
  stdout-path = "/uart@10000000";  
};

uart@10000000 {
  interrupts = <0x0a>;         // 中断向量号
  interrupt-parent = <0x02>;
  clock-frequency = <0x384000>;
  reg = <0x00 0x10000000 0x00 0x100>; //MMIO起始地址和范围
  compatible = "ns16550a";
};
```

以下情况串口会产生中断：（本章只有读取输入即1.的中断）

1. 有新的输入数据进入串口的接收缓存
2. 串口完成缓存数据的发送
3. 串口发送出错

## 1. 串口设备相关结构

串口设备相关的寄存器

```rust
// os/src/drivers/ns16550a.rs
#[repr(C)]
#[allow(dead_code)]
struct ReadWithoutDLAB {
    /// receiver buffer register
    pub rbr: ReadOnly<u8>,
    /// interrupt enable register
    pub ier: Volatile<IER>,
    /// interrupt identification register
    pub iir: ReadOnly<u8>,
    /// line control register
    pub lcr: Volatile<u8>,
    /// model control register
    pub mcr: Volatile<MCR>,
    /// line status register
    pub lsr: ReadOnly<LSR>,
    /// ignore MSR
    _padding1: ReadOnly<u8>,
    /// ignore SCR
    _padding2: ReadOnly<u8>,
}

#[repr(C)]
#[allow(dead_code)]
struct WriteWithoutDLAB {
    /// transmitter holding register
    pub thr: WriteOnly<u8>,
    /// interrupt enable register
    pub ier: Volatile<IER>,
    /// ignore FCR
    _padding0: ReadOnly<u8>,
    /// line control register
    pub lcr: Volatile<u8>,
    /// modem control register
    pub mcr: Volatile<MCR>,
    /// line status register
    pub lsr: ReadOnly<LSR>,
    /// ignore other registers
    _padding1: ReadOnly<u16>,
}

// --------------------寄存器读写------------------------
impl NS16550aRaw{
	// 读对应的寄存器
	pub fn read(&mut self) -> Option<u8> {
        let read_end = self.read_end();
        let lsr = read_end.lsr.read();
        if lsr.contains(LSR::DATA_AVAILABLE) {
            Some(read_end.rbr.read())
        } else {
            None
        }
    }
	// 写对应的寄存器
    pub fn write(&mut self, ch: u8) {
        let write_end = self.write_end();
        loop {
            if write_end.lsr.read().contains(LSR::THR_EMPTY) {
                write_end.thr.write(ch);
                break;
            }
        }
    }
}
```

UART的结构

1. `NS16550aRaw` 负责串口寄存器的读写
2. `NS16550aInner` 包含 `NS16550aRaw` 和 一个读缓冲区队列
3. `NS16550a<BASE_ADDR>` 为包含了基地址的UART，且拥有条件信号，负责读写的包装

```rust
// os/src/drivers/ns16550a.rs
// 根据基地址可以得到对应的寄存器，负责寄存器的读写
pub struct NS16550aRaw {
    base_addr: usize,
}

struct NS16550aInner {
    ns16550a: NS16550aRaw,
    read_buffer: VecDeque<u8>,
}

pub struct NS16550a<const BASE_ADDR: usize> {
    inner: UPIntrFreeCell<NS16550aInner>,
    condvar: Condvar,
}

// --------------------------需要为UART实现的Trait----------
// os/src/boards/qemu.rs
pub type CharDeviceImpl = crate::drivers::chardev::NS16550a<VIRT_UART>;
// os/src/drivers/mod.rs
pub trait CharDevice {
    fn init(&self);
    fn read(&self) -> u8;
    fn write(&self, ch: u8);
    fn handle_irq(&self);
}
```

串口设备初始化

```rust
// os/src/drivers/chardev/mod.rs
...
lazy_static! {
   pub static ref UART: Arc<CharDeviceImpl> = Arc::new(CharDeviceImpl::new());
}

// os/src/drivers/chardev/ns16550a.rs
impl<const BASE_ADDR: usize> NS16550a<BASE_ADDR> {
   pub fn new() -> Self {
      let mut inner = NS16550aInner {
            ns16550a: NS16550aRaw::new(BASE_ADDR),
            read_buffer: VecDeque::new(),
      };
      inner.ns16550a.init();
      Self {
            inner: unsafe { UPIntrFreeCell::new(inner) },
            condvar: Condvar::new(),
      }
   }
}

impl<const BASE_ADDR: usize> CharDevice for NS16550a<BASE_ADDR> {
    fn init(&self) {
        let mut inner = self.inner.exclusive_access();
        // 主要还是NS16550aRaw的初始化
        inner.ns16550a.init();
        drop(inner);
    }
}

impl NS16550aRaw {
	// 初始化各种对应寄存器的值
   pub fn init(&mut self) {
      let read_end = self.read_end();
      let mut mcr = MCR::empty();
      mcr |= MCR::DATA_TERMINAL_READY;
      mcr |= MCR::REQUEST_TO_SEND;
      mcr |= MCR::AUX_OUTPUT2;
      read_end.mcr.write(mcr);
      let ier = IER::RX_AVAILABLE;
      read_end.ier.write(ier);
   }
   
    fn read_end(&mut self) -> &mut ReadWithoutDLAB {
        unsafe { &mut *(self.base_addr as *mut ReadWithoutDLAB) }
    }

    fn write_end(&mut self) -> &mut WriteWithoutDLAB {
        unsafe { &mut *(self.base_addr as *mut WriteWithoutDLAB) }
    }
}
```

串口设备的输入（读取）

```rust
//os/src/fs/stdio.rs
impl File for Stdin {
   ...
   fn read(&self, mut user_buf: UserBuffer) -> usize {
      assert_eq!(user_buf.len(), 1);
      //println!("before UART.read() in Stdin::read()");
      let ch = UART.read();
      unsafe {
            user_buf.buffers[0].as_mut_ptr().write_volatile(ch);
      }
      1
   }
   
// os/src/drivers/chardev/ns16550a.rs
impl<const BASE_ADDR: usize> CharDevice for NS16550a<BASE_ADDR> {
   fn read(&self) -> u8 {
      loop {
            let mut inner = self.inner.exclusive_access();
            // 串口设备会通过中断将输入写到缓冲区中，只需从中读取
            if let Some(ch) = inner.read_buffer.pop_front() {
               return ch;
            } else {
            // 如果输入缓冲区中没有的话就阻塞该线程，并在后面调度
               let task_cx_ptr = self.condvar.wait_no_sched();
               drop(inner);
               schedule(task_cx_ptr);
            }
      }
   }
}

impl Condvar {
	// 这个只是阻塞但并未切换，调用者后面还要schedule()调度
	pub fn wait_no_sched(&self) -> *mut TaskContext {
		self.inner.exclusive_session(|inner| {
		inner.wait_queue.push_back(current_task().unwrap());
		});
		// 只是阻塞并未调度切换
		block_current_task()
	}
}
```

串口设备的输出（写）

```rust
// os/src/drivers/chardev/ns16550a.rs
impl<const BASE_ADDR: usize> CharDevice for NS16550a<BASE_ADDR> {
   fn write(&self, ch: u8) {
      let mut inner = self.inner.exclusive_access();
	// 通过串口设备的寄存器来写
      inner.ns16550a.write(ch);
   }
}

impl NS16550aRaw {
   pub fn write(&mut self, ch: u8) {
      let write_end = self.write_end();
      loop {
		      // 寄存器显示可写
            if write_end.lsr.read().contains(LSR::THR_EMPTY) {
               write_end.thr.write(ch);
               break;
            }
      }
   }
}
```

串口设备产生输入中断

```rust
// os/src/trap/mod.rs
#[no_mangle]
pub fn trap_handler() -> ! {
    set_kernel_trap_entry();
    let scause = scause::read();
    let stval = stval::read();
    // println!("into {:?}", scause.cause());
    match scause.cause() {
		...
		// 内核模式下的中断
        Trap::Interrupt(Interrupt::SupervisorExternal) => {
            crate::board::irq_handler();
        }
    ...
}

// os/src/boards/qemu.rs
pub fn irq_handler() {
   let mut plic = unsafe { PLIC::new(VIRT_PLIC) };
   // 根据PLIC的Claim寄存器得到对应的中断号
   let intr_src_id = plic.claim(0, IntrTargetPriority::Supervisor);
   match intr_src_id {
      5 => KEYBOARD_DEVICE.handle_irq(),
      6 => MOUSE_DEVICE.handle_irq(),
      8 => BLOCK_DEVICE.handle_irq(),
      // UART 的中断处理
      10 => UART.handle_irq(),
   }
   plic.complete(0, IntrTargetPriority::Supervisor, intr_src_id);
}

// os/src/drivers/chardev/ns16550a.rs
impl<const BASE_ADDR: usize> CharDevice for NS16550a<BASE_ADDR> {
   fn handle_irq(&self) {
      let mut count = 0;
      self.inner.exclusive_session(|inner| {
	      // 从寄存器读出值写入到输入缓冲区供操作系统从缓冲区读取
            while let Some(ch) = inner.ns16550a.read() {
               count += 1;
               inner.read_buffer.push_back(ch);
            }
      });
      if count > 0 {
	      // 如果缓冲区有内容就signal唤醒等待的线程
            self.condvar.signal();
      }
   }
}
```
# Virtio设备驱动程序

![[Pasted image 20230508221612.png]]

![[Pasted image 20230509103308.png]]

左上图中传统的虚拟机模拟外设方案中，guest os（rcore）要使用硬件资源的时候，需要通过请求I/O的指令到Hypervisor（QEMU）中，然后Hypervisor模拟出这些指令行为再传递给硬件来执行

右上图中虚拟机模拟外设的virtio方案中，guest使用硬件资源可以直接通过设备驱动前端的接口（Front-end drivers），再经过中间层的驱动程序与虚拟设备之间的交互接口后，到Hypervisor的virtio中（Back-end drivers），这样Hypervisor只需要少量寄存器访问和中断机制就能使用底层硬件资源，实现高效的I/O虚拟化过程

![[Pasted image 20230509104157.png|400]]

![[Pasted image 20230509104228.png]]

## 1. virto设备基本组成要素

1. 设备状态域：设备程序对virtio设备初始化过程中，不同阶段对应着设备状态域的不同状态
2. 特征位：表示virtio设备具有的特征和功能
3. [[#^notification-23-5-9|通知]]：驱动程序和virtio设备之间如何通信
4. 设备配置空间：初始化阶段需要设置的设备参数，含设备特征
5. 一个或多个[[#^virtqueue-23-5-9|虚拟队列]]：传输数据用的

## 2. virtio设备交互机制

### 1. Notification 通知 ^notification-23-5-9

驱动程序通过**PIO/MMIO方式访问特定寄存器**，QEMU进行拦截再通知模拟的设备；设备驱动程序通过**中断**机制，在QEMU中进行中断注入，让CPU响应并执行中断例程

### 2. virtqueue 虚拟队列 ^virtqueue-23-5-9

virtqueue是一种数据结构，用于设备和驱动程序中执行各种数据传输操作，包含一下三部分

1. **描述符表 Descriptor Table**：每个描述符描述了一个内存buffer的 address和length、读写标志Flags以及下一个有关的描述符索引Next
2. **可用环 Available Ring**：环形队列，记录了设备驱动程序发出的I/O请求索引（指向描述符表中的描述符），virtio设备需要进行读取并完成相关I/O操作
3. **已用环 Used Ring**：环形队列，记录了virtio设备完成的I/O的索引，需要设备驱动程序读取并对完成的I/O结果进一步处理

![[Pasted image 20230509105455.png]]

### 3. 基于virtqueue的交互过程

#### 1. 初始化过程（驱动程序执行）

1. 设备驱动程序申请virtqueue（描述符表、可用环、已用环）的内存空间
2. 把virtqueue的描述符表、可用环、已用环三部分物理地址写入到virtio设备的对应控制寄存器中（MMIO内存地址）（至此，设备驱动程序就与virtio设备共享了整个virtqueue内存空间）

#### 2. I/O请求过程（驱动程序执行） ^request-23-5-9

1. 驱动程序发出I/O请求, 首先将I/O命令/数据等**放到一个/多个buffer**中
2. 在描述符表中**分配新的描述符**只想这些buffer
3. 将描述符的**索引**写入可用环中，更新可用环的idx指针
4. 驱动程序通过 **kick** 机制（写virtio设备中特定的通知控制寄存器）来通知virtio设备有新的请求

#### 3. I/O完成过程（virtio设备执行） ^finish-23-5-9

1. virtio设备通过 kick 机制直到有新的I/O请求，通过**访问可用环**的idx指针解析出I/O请求
2. 完成I/O请求并把I/O操作结果放到I/O请求对应的**buffer**中
3. 将描述符索引（首描述符）写入到**已用环**中，更新已用环idx指针
4. virtio设备通过中断机制通知设备驱动程序I/O操作完成

#### 4. I/O后处理过程（驱动程序执行） ^back-23-5-9

1. 设备驱动程序读取已用环idx来读取描述符表中的描述符，获得I/O操作完成的信息

## 3. virtio驱动程序

这部分使各种virtio驱动程序的共性，包括初始化设备、驱动程序和设备的交互

### 1. virtio设备的描述（基于MMIO）

操作系统采用Device Tree的方式探测各种基于MMIO方式的virtio设备，从而知道与设备相关的寄存器和所用的中断。当我们找到了与设备相关的内存区域，也就可以获得各种寄存器信息（VirtIOHeader结构会被创建到MMIO对应的位置，这样对该结构的读写就直接是对对应寄存器的读写）

```rust
//virtio-drivers/src/header.rs
pub struct VirtIOHeader {
   magic: ReadOnly<u32>,  //魔数 Magic value
   ...
   //设备初始化相关的特征/状态/配置空间对应的寄存器
   device_features: ReadOnly<u32>, //设备支持的功能
   device_features_sel: WriteOnly<u32>,//设备选择的功能
   driver_features: WriteOnly<u32>, //驱动程序理解的设备功能
   driver_features_sel: WriteOnly<u32>, //驱动程序选择的设备功能
   config_generation: ReadOnly<u32>, //配置空间
   status: Volatile<DeviceStatus>, //设备状态

   //virtqueue虚拟队列对应的寄存器
   queue_sel: WriteOnly<u32>, //虚拟队列索引号
   queue_num_max: ReadOnly<u32>,//虚拟队列最大容量值
   queue_num: WriteOnly<u32>, //虚拟队列当前容量值
   queue_notify: WriteOnly<u32>, //虚拟队列通知
   queue_desc_low: WriteOnly<u32>, //设备描述符表的低32位地址
   queue_desc_high: WriteOnly<u32>,//设备描述符表的高32位地址
   queue_avail_low: WriteOnly<u32>,//可用环的低32位地址
   queue_avail_high: WriteOnly<u32>,//可用环的高32位地址
   queue_used_low: WriteOnly<u32>,//已用环的低32位地址
   queue_used_high: WriteOnly<u32>,//已用环的高32位地址

   //中断相关的寄存器
   interrupt_status: ReadOnly<u32>, //中断状态
   interrupt_ack: WriteOnly<u32>, //中断确认
}
```

### 2. 设备的初始化

驱动程序进行设备初始化的常规步骤（都是向设备的对应内存的寄存器位置写入值）

1.  重启设备状态，设置设备状态域为0
2.  设置设备状态域为 `ACKNOWLEDGE` ，表明当前已经识别到了设备
3.  设置设备状态域为 `DRIVER` ，表明驱动程序知道如何驱动当前设备
4.  进行设备特定的安装和配置，包括协商特征位，建立virtqueue，访问设备配置空间等, 设置设备状态域为 `FEATURES_OK`
5.  设置设备状态域为 `DRIVER_OK` 或者 `FAILED` （如果中途出现错误）

我们的各种virtio设备驱动程序的共同给初始化过程为：

1.  确定协商特征位，调用 VirtIOHeader 的 begin_init 方法进行virtio设备初始化的第1-4步骤；
2.  读取配置空间，确定设备的配置情况；
3.  建立虚拟队列1~n个virtqueue；
4.  调用 VirtIOHeader finish_init 方法进行virtio设备初始化的第5步骤。

例如对于virtio_blk设备初始化过程 ^blkinit-23-5-10

```rust
// virtio_drivers/src/blk.rs
//virtio_blk驱动初始化
impl<H: Hal> VirtIOBlk<'_, H> {
   /// Create a new VirtIO-Blk driver.
   pub fn new(header: &'static mut VirtIOHeader) -> Result<Self> {
	   // 1.调用header.begin_init方法
      header.begin_init(|features| {
            ...
            (features & supported_features).bits()
      });
      // 2.读取virtio_blk设备的配置空间
      let config = unsafe { &mut *(header.config_space() ...) };
      // 3.建立1个虚拟队列
      let queue = VirtQueue::new(header, 0, 16)?;
      //4.结束设备初始化
      header.finish_init();
      ...
   }
// virtio_drivers/src/header.rs
// virtio设备初始化的第1~4步骤
impl VirtIOHeader {
   pub fn begin_init(&mut self, negotiate_features: impl FnOnce(u64) -> u64) {
      self.status.write(DeviceStatus::ACKNOWLEDGE);
      self.status.write(DeviceStatus::DRIVER);
      let features = self.read_device_features();
      self.write_driver_features(negotiate_features(features));
      self.status.write(DeviceStatus::FEATURES_OK);
      self.guest_page_size.write(PAGE_SIZE as u32);
   }

   // virtio设备初始化的第5步骤
   pub fn finish_init(&mut self) {
      self.status.write(DeviceStatus::DRIVER_OK);
   }
```

### 3. 驱动程序与设备的交互

用户进程发出I/O操作的处理大致分为以下三步：

1.  用户进程发出I/O请求，经过层层下传给到驱动程序, [[#^request-23-5-9|驱动程序将执行请求发送]]；
2.  设备收到通知后，[[#^finish-23-5-9|处理请求并在处理完后通知驱动处处理程序]]；
3.  驱动程序[[#^back-23-5-9|解析已用环获得I/O响应的结果]]，在进一步处理后，最终返回给用户进程。

通知virtio设备：

```rust
// virtio_drivers/src/header.rs
pub struct VirtIOHeader {
// Queue notifier 用户虚拟队列通知的寄存器
queue_notify: WriteOnly<u32>,
...
}

impl VirtIOHeader {
   // Notify device.
   pub fn notify(&mut self, queue: u32) {
      self.queue_notify.write(queue);
   }
```

virtio设备处理完I/O请求后会通过中断来使CPU到 `trap_handler()` 并执行对应的后处理，最后会通知virtio设备收到中断

```rust
// virtio_drivers/src/blk.rs
impl<H: Hal> VirtIOBlk<'_, H> {
	// acknowledge interrupt告知收到了中断
   pub fn ack_interrupt(&mut self) -> bool {
      self.header.ack_interrupt()
   }

// virtio_drivers/src/header.rs
pub struct VirtIOHeader {
   // 中断状态寄存器 Interrupt status
   interrupt_status: ReadOnly<u32>,
   // 中断响应寄存器 Interrupt acknowledge
   interrupt_ack: WriteOnly<u32>,
}

impl VirtIOHeader {
   /// Acknowledge interrupt and return true if success.
    pub fn ack_interrupt(&mut self) -> bool {
        let interrupt = self.interrupt_status.read();
        if interrupt != 0 {
            self.interrupt_ack.write(interrupt);
            true
        } else {
            false
        }
    }
```

# virtio-blk驱动程序

## 1. 相关数据结构（驱动程序的，还没有对接内核）

```rust
// virtio-drivers/src/blk.rs
pub struct VirtIOBlk<'a, H: Hal> {
	// virtio设备共有的属性(设备信息)
   header: &'static mut VirtIOHeader,
   // virtqueue里面各种结构
   queue: VirtQueue<'a, H>,
   capacity: usize,
}

#[repr(C)]
pub struct VirtQueue<'a, H: Hal> {
   dma: DMA<H>, // DMA guard
   desc: &'a mut [Descriptor], // 描述符表
   avail: &'a mut AvailRing, // 可用环 Available ring
   used: &'a mut UsedRing, // 已用环 Used ring
   queue_idx: u32, //虚拟队列索引值
   queue_size: u16, // 虚拟队列长度(多少个描述符/可用环长度/已用环长度)
   num_used: u16, // 已经使用的队列项目数
   free_head: u16, // 空闲队列项目头的索引值
   avail_idx: u16, //可用环的索引值
   last_used_idx: u16, //上次已用环的索引值
}

//用于抽象出具体操作系统相关的操作,主要与内存分配和虚实地址转换相关
pub trait Hal {
   /// Allocates the given number of contiguous physical pages of DMA memory for virtio use.
   fn dma_alloc(pages: usize) -> PhysAddr;
   /// Deallocates the given contiguous physical DMA memory pages.
   fn dma_dealloc(paddr: PhysAddr, pages: usize) -> i32;
   /// Converts a physical address used for virtio to a virtual address which the program can
   /// access.
   fn phys_to_virt(paddr: PhysAddr) -> VirtAddr;
   /// Converts a virtual address which the program can access to the corresponding physical
   /// address to use for virtio.
   fn virt_to_phys(vaddr: VirtAddr) -> PhysAddr;
}

#[repr(C, align(16))]
#[derive(Debug)]
struct Descriptor {
    addr: Volatile<u64>,
    len: Volatile<u32>,
    flags: Volatile<DescFlags>,
    next: Volatile<u16>,
}

/// The driver uses the available ring to offer buffers to the device:
/// each ring entry refers to the head of a descriptor chain.
/// It is only written by the driver and read by the device.
#[repr(C)]
#[derive(Debug)]
struct AvailRing {
    flags: Volatile<u16>,
    /// A driver MUST NOT decrement the idx.
    idx: Volatile<u16>,
    ring: [Volatile<u16>; 32], // actual size: queue_size
    used_event: Volatile<u16>, // unused
}

/// The used ring is where the device returns buffers once it is done with them:
/// it is only written to by the device, and read by the driver.
#[repr(C)]
#[derive(Debug)]
struct UsedRing {
    flags: Volatile<u16>,
    idx: Volatile<u16>,
    ring: [UsedElem; 32],       // actual size: queue_size
    avail_event: Volatile<u16>, // unused
}
```

## 2. virtio-blk设备的初始化

1. [[#^blkinit-23-5-10|设备初始化]]
2. 创建虚拟队列 `VirtQueue` 数据结构实例，分配虚拟队列的内存空间，并进行初始化

```rust
impl<H: Hal> VirtQueue<'_, H> {
    /// Create a new VirtQueue.
    // idx为队列的索引值(第几个队列)
    // size为队列长度(描述符表大小/可用环长度/已用环长度)
    pub fn new(header: &mut VirtIOHeader, idx: usize, size: u16) -> Result<Self> {
        if header.queue_used(idx as u32) {
            return Err(Error::AlreadyUsed);
        }
        if !size.is_power_of_two() || header.max_queue_size() < size as u32 {
            return Err(Error::InvalidParam);
        }
        // 1.根据大小计算出内存字节大小
        let layout = VirtQueueLayout::new(size);
        // 2.Allocate contiguous pages.
        let dma = DMA::new(layout.size / PAGE_SIZE)?;
		// 3.将虚拟队列相关信息(地址内存等)写入到virt-blk折别的MMIo寄存器中
        header.queue_set(idx as u32, size as u32, PAGE_SIZE as u32, dma.pfn());
		// 4.初始化队列各个成员变量
        let desc =
            unsafe { slice::from_raw_parts_mut(dma.vaddr() as *mut Descriptor, size as usize) };
        let avail = unsafe { &mut *((dma.vaddr() + layout.avail_offset) as *mut AvailRing) };
        let used = unsafe { &mut *((dma.vaddr() + layout.used_offset) as *mut UsedRing) };

        // Link descriptors together.
        for i in 0..(size - 1) {
            desc[i as usize].next.write(i + 1);
        }

        Ok(VirtQueue {
            dma,
            desc,
            avail,
            used,
            queue_size: size,
            queue_idx: idx as u32,
            num_used: 0,
            free_head: 0,
            avail_idx: 0,
            last_used_idx: 0,
        })
    }
}
```

## 3. 内核对接virtio-blk的初始化

1. 包装VirtIOBlk成VirtIOBlock并初始化
2. 定义一个实现了Hal Trait的VirtIOHal结构，为其实现Hal的四个操作，使得VirtIOBlk内部的VirtIQueue能使用DMA::动态内存分配

结构的包装

```rust
// 全局变量BLOCK_DEVICE
// os/src/drivers/block/mod.rs
lazy_static! {
   pub static ref BLOCK_DEVICE: Arc<dyn BlockDevice> = Arc::new(BlockDeviceImpl::new());
}

// os/src/boards/qemu.rs
pub type BlockDeviceImpl = crate::drivers::block::VirtIOBlock;

// os/src/drivers/block/virtio_blk.rs
pub struct VirtIOBlock {
   virtio_blk: UPIntrFreeCell<VirtIOBlk<'static, VirtioHal>>,
   condvars: BTreeMap<u16, Condvar>,
}

// -------------------将会为VirtIOBlock实现的Trait-----------
// 实现了这些Trait内核就可以使用BLOCK_DEVICE来读写和处理中断
// os/easy-fs/src/block_dev.rs
pub trait BlockDevice: Send + Sync + Any {
   fn read_block(&self, block_id: usize, buf: &mut [u8]);
   fn write_block(&self, block_id: usize, buf: &[u8]);
   fn handle_irq(&self);
}
```

初始化构建

```rust
impl VirtIOBlock {
   pub fn new() -> Self {
      let virtio_blk = unsafe {
            UPIntrFreeCell::new(
	            // 1.新建VirtioHal类型的VirtIOBlk
               VirtIOBlk::<VirtioHal>::new(&mut *(VIRTIO0 as *mut VirtIOHeader)).unwrap(),
            )
      };
      // 2.新建一个条件变量数组
      let mut condvars = BTreeMap::new();
      let channels = virtio_blk.exclusive_access().virt_queue_size();
      // 根据队列大小为每一个条件变量对应一个虚拟队列条目的编号
      for i in 0..channels {
            let condvar = Condvar::new();
            condvars.insert(i, condvar);
      }
      Self {
            virtio_blk,
            condvars,
      }
   }
}
```

实现了Hal Trait的结构VirtioHal，在创建VirtIOBlk结构的时候要用到来动态分配内存

```rust
pub struct VirtioHal;

impl Hal for VirtioHal {
	// 分配物理页帧
    fn dma_alloc(pages: usize) -> usize {
        let trakcers = frame_alloc_more(pages);
        let ppn_base = trakcers.as_ref().unwrap().last().unwrap().ppn;
        QUEUE_FRAMES
            .exclusive_access()
            .append(&mut trakcers.unwrap());
        let pa: PhysAddr = ppn_base.into();
        pa.0
    }

	// 释放物理页帧
    fn dma_dealloc(pa: usize, pages: usize) -> i32 {
        let pa = PhysAddr::from(pa);
        let mut ppn_base: PhysPageNum = pa.into();
        for _ in 0..pages {
            frame_dealloc(ppn_base);
            ppn_base.step();
        }
        0
    }

    fn phys_to_virt(addr: usize) -> usize {
        addr
    }

    fn virt_to_phys(vaddr: usize) -> usize {
        PageTable::from_token(kernel_token())
            .translate_va(VirtAddr::from(vaddr))
            .unwrap()
            .0
    }
}
```

## 4. I/O请求的整个过程

I/O请求的处理包括两个部分，首先是从内核中的处理，之后就转到了virtio-blk驱动程序的处理

### 1. 内核的处理

1. `sys_write()` 系统调用 -> 
2. `OSInode::write()` 文件系统内的write ->
3. `write_at()` Inode&DiskInode的write，修改内存中的BlockCache ->
4. `BlockCache::sync()` BlockCache自动/手动将内存缓冲区的内容写到磁盘中 ->
5. `VirtIOBlock::write_block()`跳转到了内核和virtio-blk驱动程序的接口（ `virtIOBlk::write_block()` 中）

```rust
// os/src/syscall/fs.rs
// 1.系统调用
pub fn sys_write(fd: usize, buf: *const u8, len: usize) -> isize {
    ...
    file.write(UserBuffer::new(translated_byte_buffer(token, buf, len))) as isize
    ...
}

// 2.文件系统内的write(上面会跳转到实现了File Trait的OSInode)
impl File for OSInode {
	fn write(&self, buf: UserBuffer) -> usize {
        ...
        inner.inode.write_at(inner.offset, *slice);
        ...
    }
}

// 3.Inode和DiskInode的write_at()
impl Inode{
	pub fn write_at(&self, offset: usize, buf: &[u8]) -> usize {
		...
        disk_inode.write_at(offset, buf, &self.block_device)
        });
        ...
    }
}
impl DiskInode{
	pub fn write_at(
        &mut self,
        offset: usize,
        buf: &[u8],
        block_device: &Arc<dyn BlockDevice>,
    ) -> usize {
        ...
		get_block_cache()
		.lock()
		.modify(0, |data_block: &mut DataBlock| {
			let src = &buf[write_size..write_size + block_write_size];
			let dst = &mut data_block[start % BLOCK_SZ..start % BLOCK_SZ + block_write_size];
			dst.copy_from_slice(src);
		});
		...
    }
}

// 4.上面会修改缓存BlockCache内容，缓存会自动调用sync()写回磁盘
impl BlockCache{
	pub fn sync(&mut self) {
        if self.modified {
            self.modified = false;
            self.block_device.write_block(self.block_id, &self.cache);
        }
    }
}

// 5.之后就到了VirtIOBlock的write_block()
impl VirtIOBlock{
	fn write_block(&self, block_id: usize, buf: &[u8]) {
		// 获取轮询/中断的配置方式
        let nb = *DEV_NON_BLOCKING_ACCESS.exclusive_access();
        // 一、如果是中断方式
        if nb {
	        // 1.获取第三个缓冲区的结构
	        // 将来中断唤醒线程时会确认I/O已完成
            let mut resp = BlkResp::default();
            let task_cx_ptr = self.virtio_blk.exclusive_session(|blk| {
	            // 2.基于中断方式的写块请求
                let token = unsafe { blk.write_block_nb(block_id, buf, &mut resp).unwrap() };
                // 3.阻塞，将当前线程加入到对应信号量的阻塞
                // 上面的写磁盘是基于virtIOBlk的驱动程序中的，它是个独立的模块，所以上面就没有阻塞，需要到下面在操作系统部分阻塞
                self.condvars.get(&token).unwrap().wait_no_sched()
            });
            // 4.调度切换
            schedule(task_cx_ptr);
            assert_eq!(
                resp.status(),
                RespStatus::Ok,
                "Error when writing VirtIOBlk"
            );
		} else {   // 二、如果是轮询
            self.virtio_blk
                .exclusive_access()
                .write_block(block_id, buf)
                .expect("Error when writing VirtIOBlk");
        }
    }
    // read_block()与write_block()类似
    fn read_block(&self, block_id: usize, buf: &mut [u8]) {
        let nb = *DEV_NON_BLOCKING_ACCESS.exclusive_access();
        if nb {
            let mut resp = BlkResp::default();
            let task_cx_ptr = self.virtio_blk.exclusive_session(|blk| {
                let token = unsafe { blk.read_block_nb(block_id, buf, &mut resp).unwrap() };
                self.condvars.get(&token).unwrap().wait_no_sched()
            });
            schedule(task_cx_ptr);
            assert_eq!(
                resp.status(),
                RespStatus::Ok,
                "Error when reading VirtIOBlk"
            );
        } else {
            self.virtio_blk
                .exclusive_access()
                .read_block(block_id, buf)
                .expect("Error when reading VirtIOBlk");
        }
    }
}
```

### 2. virtio-blk驱动程序的处理

1. `VirtIOBlk::write_block()` 轮询方式 / `write_block_nb()` 阻塞方式->
	* 一个完整的virtio-blk的I/O请求包含三部分：表示I/O写/读请求信息的结构 `BlkReq`，需要传输的数据块 `buf`，表示设备响应信息的结构 `BlkResp`，三部分分别用三个描述符表示
2. `VirtQueue::add()` 往队列中增加请求
	* 从描述符表申请3个空闲描述符，每个指向一个内存块，分别描述上面三部分信息，并将描述符连成一个描述符链表
3. 2完成后1中的函数继续调用 `VirtIOHeader::notify()` 写寄存器通知virtio-blk设备
4. （设备处理）virtio-blk设备接收到通知后，会比较 `VirtQueue::avail_idx`（可用环上一次的下标） 和 `VirtQueue::AvailRing::idx`（可用环最新的下标） 是否相同来判断是否有新的请求，如果有， `avail_idx`+1并从索引表中读取I/O请求来执行
5. （设备处理）virtio-blk设备完成I/O操作后，将已完成的I/O米哦啊舒服放入 `UsedRing` 对应的ring中，并更新idx（若是中断响应的处理，还会产生中断来通知操作系统响应；轮询就不用通知）
6. 驱动程序可以用轮询/中断来查看设备是否有响应（轮询的话通过`VirtQueue::can_pop()`来比较`last_used_idx`和`VirtQueue::UsedRing::used来判断`），如果有相应的话：
	1. 轮询会调用 `VirtQueue::pop_used()` 回收三个描述符
	2. 中断的话：`trap_handler()` -> `irq_handler()` -> `BLOCK_DEVICE::irq_handler()` -> `VirtIOBlock::handle_irq()`-> `VirtIOBlk::pop_used()` -> `VirtQueue::pop_used()` 回收描述符

```rust
// 1.write_block()轮询方式
//virtio-drivers/src/blk.rs
 pub fn write_block(&mut self, block_id: usize, buf: &[u8]) -> Result {
     assert_eq!(buf.len(), BLK_SIZE);
     // (1).I/O写/读请求信息的结构 BlkReq,写入第一个缓冲区
     let req = BlkReq {
         type_: ReqType::Out,
         reserved: 0,
         sector: block_id as u64,
     };
     // (3).表示设备响应信息的结构 BlkResp,IO结束后检查
     let mut resp = BlkResp::default();
     // 2.往队列中添加请求
     self.queue.add(&[req.as_buf(), buf], &[resp.as_buf_mut()])?;
     // 3.通知设备
     self.header.notify(0);
     // 6.轮询检查设备是否响应结束
     while !self.queue.can_pop() {
         spin_loop();
     }
     self.queue.pop_used()?;
     match resp.status {
         RespStatus::Ok => Ok(()),
         _ => Err(Error::IoError),
     }
 }

// 1.write_block_nb()中断方式
pub unsafe fn write_block_nb(
     &mut self,
     block_id: usize,
     // (2)数据块,写入第二个缓冲区
     buf: &[u8],
     // (1).I/O写/读请求信息的结构 BlkReq,写入第一个缓冲区
     // 上层 VirtIOBlock传入
     resp: &mut BlkResp,
 ) -> Result<u16> {
     assert_eq!(buf.len(), BLK_SIZE);
     // (3).表示设备响应信息的结构 BlkResp,IO结束后检查
     let req = BlkReq {
         type_: ReqType::Out,
         reserved: 0,
         sector: block_id as u64,
     };
     // 2.往队列中添加请求
     let token = self.queue.add(&[req.as_buf(), buf], &[resp.as_buf_mut()])?;
     // 3.通知设备
     self.header.notify(0);
     Ok(token)
}
// 中断通知，执行6.回收描述符-------------------------------
// os/src/trap/mode.rs
//在用户态接收到外设中断
pub fn trap_handler() -> ! {
   ...
   crate::board::irq_handler();
}
//在内核态接收到外设中断
pub fn trap_from_kernel(_trap_cx: &TrapContext) {
   ...
   crate::board::irq_handler();
}
// os/src/boards/qemu.rs
pub fn irq_handler() {
   let mut plic = unsafe { PLIC::new(VIRT_PLIC) };
   // 获得外设中断号
   let intr_src_id = plic.claim(0, IntrTargetPriority::Supervisor);
   match intr_src_id {
      ...
      //处理virtio_blk设备产生的中断
      8 => BLOCK_DEVICE.handle_irq(),
   }
   // 完成中断响应
   plic.complete(0, IntrTargetPriority::Supervisor, intr_src_id);
}
// VirtIOBlock处理中断
fn handle_irq(&self) {
	self.virtio_blk.exclusive_session(|blk| {
		while let Ok(token) = blk.pop_used() {
			self.condvars.get(&token).unwrap().signal();
		}
	});
}

// -------------------------VirtQueue::add()---------------
/// Add buffers to the virtqueue, return a token.
///
/// Ref: linux virtio_ring.c virtqueue_add
pub fn add(&mut self, inputs: &[&[u8]], outputs: &[&mut [u8]]) -> Result<u16> {
	if inputs.is_empty() && outputs.is_empty() {
		return Err(Error::InvalidParam);
	}
	if inputs.len() + outputs.len() + self.num_used as usize > self.queue_size as usize {
		return Err(Error::BufferTooSmall);
	}

	// 1.allocate descriptors from free list
	let head = self.free_head;
	let mut last = self.free_head;
	for input in inputs.iter() {
		let desc = &mut self.desc[self.free_head as usize];
		desc.set_buf::<H>(input);
		desc.flags.write(DescFlags::NEXT);
		last = self.free_head;
		self.free_head = desc.next.read();
	}
	for output in outputs.iter() {
		let desc = &mut self.desc[self.free_head as usize];
		desc.set_buf::<H>(output);
		desc.flags.write(DescFlags::NEXT | DescFlags::WRITE);
		last = self.free_head;
		self.free_head = desc.next.read();
	}
	// 2.set last_elem.next = NULL
	{
		let desc = &mut self.desc[last as usize];
		let mut flags = desc.flags.read();
		flags.remove(DescFlags::NEXT);
		desc.flags.write(flags);
	}
	self.num_used += (inputs.len() + outputs.len()) as u16;

	let avail_slot = self.avail_idx & (self.queue_size - 1);
	self.avail.ring[avail_slot as usize].write(head);

	// 3.write barrier
	fence(Ordering::SeqCst);

	// 4.increase head of avail ring
	self.avail_idx = self.avail_idx.wrapping_add(1);
	self.avail.idx.write(self.avail_idx);
	Ok(head)
}
```