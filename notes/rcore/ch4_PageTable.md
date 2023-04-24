>基于**页表分页机制**的地址空间(虚拟地址 -> 物理地址)

![[Pasted image 20230424170920.png]]

![[ch4_PageTable 2023-04-21 23.36.41.excalidraw|600]]

# 一、rust的动态内存分配

* 对内核的堆空间(物理空间)经过LockedHeap结构处理后变成HEAP_ALLOCATOR的全局变量来管理**内核的堆空间**

```rust
// os/src/mm/heap_allocator.rs
use buddy_system_allocator::LockedHeap;
use crate::config::KERNEL_HEAP_SIZE;

#[global_allocator]
static HEAP_ALLOCATOR: LockedHeap = LockedHeap::empty();

static mut HEAP_SPACE: [u8; KERNEL_HEAP_SIZE] = [0; KERNEL_HEAP_SIZE];

pub fn init_heap() {
    unsafe {
        HEAP_ALLOCATOR
            .lock()
            .init(HEAP_SPACE.as_ptr() as usize, KERNEL_HEAP_SIZE);
    }
}
```

# 二、物理地址与虚拟地址

VirtualAddress：39位

* 线性页表：(2^39 / 2^12) \*8 = 1GB
* 三级页表：S / 512 (B)
	* 一个三级页表可管理4KB \* 512 = **2MB**区域，一个二级页表可管理4KB \* 512 \* 512 = **1GB**区域；对于连续T字节需要分配4KB \* ( ceil( T / 2MB ) + ceil( T / 1GB )) ≈ T / 512 (B) 

PhysicalAddress：56位

* PPN：(Physical Page Number) 物理页号。由于每一个页目录Page Directory占4KB（一页），而且物理内存按页来分配，所以**satp/页表中的PPN**中的物理页号 *直接就可以在物理内存中找到对应的物理地址* （因为每个页表都占4KB一页，所以低12位地址都是0）；而实际需要转换的物理地址就是最后一级页表的PPN+Offset

![[Pasted image 20230410160938.png]]

```rust
// 各种基本结构的定义
// os/src/mm/address.rs

#[derive(Copy, Clone, Ord, PartialOrd, Eq, PartialEq)]
pub struct PhysAddr(pub usize);

#[derive(Copy, Clone, Ord, PartialOrd, Eq, PartialEq)]
pub struct VirtAddr(pub usize);

#[derive(Copy, Clone, Ord, PartialOrd, Eq, PartialEq)]
pub struct PhysPageNum(pub usize);

#[derive(Copy, Clone, Ord, PartialOrd, Eq, PartialEq)]
pub struct VirtPageNum(pub usize);

// 页表项的flags
bitflags! {
    pub struct PTEFlags: u8 {
        const V = 1 << 0;
        const R = 1 << 1;
        const W = 1 << 2;
        const X = 1 << 3;
        const U = 1 << 4;
        const G = 1 << 5;
        const A = 1 << 6;
        const D = 1 << 7;
    }
}

// 页表项
#[derive(Copy, Clone)]
#[repr(C)]
pub struct PageTableEntry {
    pub bits: usize,
}
```

## TLB 快表

> 快表中存储的是页表的缓存，即一个虚拟页号对应的物理页号

在我们切换任务的时候， `satp` 也必须被同时切换。

* 但如果修改了 `satp` 寄存器，说明内核切换到了一个与先前映射方式完全不同的页表。此时**快表里面存储的映射已经失效**了，这种情况下内核要在修改 satp 的指令后面马上使用 `sfence.vma` 指令刷新清空整个 TLB。
* 同样，我们手动修改一个页表项之后，也修改了映射，但 TLB 并不会自动刷新清空，我们也需要使用 `sfence.vma` 指令刷新整个 TLB。（注：可以在 sfence.vma 指令后面加上一个虚拟地址，这样 sfence.vma 只会刷新TLB中关于这个虚拟地址的单个映射项。）


# 三、物理页帧与页表

## 1. 物理页帧管理与分配（frame_allocator.rs）

```rust
// os/src/mm/frame_allocator.rs
// 物理页帧分配器结构定义
pub struct StackFrameAllocator {
    current: usize,  //空闲内存的起始物理页号
    end: usize,      //空闲内存的结束物理页号
    recycled: Vec<usize>,  //将回收后的物理页号放到里面
}

// 分配物理页帧的函数
// os/src/mm/frame_allocator.rs
// alloc():先从recycled中查找是否有得分配（分配最近使用过的），如果没有再从空闲current中分配
fn alloc(&mut self) -> Option<PhysPageNum>
// dealloc():回收前先判断是否是正确的可回收的物理页帧，是的话直接push到recycled中
fn dealloc(&mut self, ppn: PhysPageNum)

//----------------------------------------------------
// 全局变量(物理页帧分配器)
type FrameAllocatorImpl = StackFrameAllocator;

lazy_static! {
    pub static ref FRAME_ALLOCATOR: UPSafeCell<FrameAllocatorImpl> = unsafe {
        UPSafeCell::new(FrameAllocatorImpl::new())
    };
}
//从内核内存结束ekernel到0x80800000全都是可以分配的内存
pub fn init_frame_allocator() {
    extern "C" {
        fn ekernel();
    }
    FRAME_ALLOCATOR
        .exclusive_access()
        .init(PhysAddr::from(ekernel as usize).ceil(), PhysAddr::from(MEMORY_END).floor());
}
```


* 分配物理页帧：每个物理页可通过接口frame_alloc()分配，而**不用手动回收**，因为每个物理页被包装成FrameTracker { pub ppn: PhyPageNum, }，将物理页绑定FrameTracker中，到new创建的时候清零页面内容，FrameTracker生命周期到的时候会用drop主动调用frame_dealloc()（Drop需要impl到FrameTracker）

```rust
// os/src/mm/frame_allocator.rs
// 提供分配页表的接口
pub fn frame_alloc() -> Option<FrameTracker> {
    FRAME_ALLOCATOR
        .exclusive_access()
        .alloc()
        .map(|ppn| FrameTracker::new(ppn))
}

fn frame_dealloc(ppn: PhysPageNum) {
    FRAME_ALLOCATOR
        .exclusive_access()
        .dealloc(ppn);
}

//-----------------------------------------------------
// frame_alloc()分配的时候会发配这个结构包装物理页号
// os/src/mm/frame_allocator.rs
pub struct FrameTracker {
    pub ppn: PhysPageNum,
}

impl FrameTracker {
	// 0初始化
    pub fn new(ppn: PhysPageNum) -> Self {
        // page cleaning
        let bytes_array = ppn.get_bytes_array();
        for i in bytes_array {
            *i = 0;
        }
        Self { ppn }
    }
}

// 通过FrameTracker包装后的PhysPageNum在FrameTracker不用时会被drop释放掉
impl Drop for FrameTracker {
    fn drop(&mut self) {
        frame_dealloc(self.ppn);
    }
}
```

## 2. 页表的分配与管理

PageTable 这个结构管理一整套页表，root_ppn是根页表，所有的页表的FrameTracker都被放入到Vec中保存（所有权），（但这样怎么换页？）

* **PageTable管理了所有页表**（不包括映射后的物理页面的）物理页面（页帧）的所有权

```rust
// os/src/mm/page_table.rs
// 页表: 管理所有的页面(有所有的Frametracker的所有权)
pub struct PageTable {
    root_ppn: PhysPageNum,       // 根页表
    frames: Vec<FrameTracker>,
}

impl PageTable {
    pub fn new() -> Self {
        let frame = frame_alloc().unwrap();
        PageTable {
            root_ppn: frame.ppn,
            frames: vec![frame],
        }
    }
}
```

## 3. 将地址转化为可变引用类型

```rust
// 在内核中应如何访问一个特定的物理页帧：
// os/src/mm/address.rs
impl PhysPageNum {
	// 将地址->裸指针以及指定类型的大小来得到可变的slice来使用
    pub fn get_pte_array(&self) -> &'static mut [PageTableEntry] {
        let pa: PhysAddr = self.clone().into();
        unsafe {
            core::slice::from_raw_parts_mut(pa.0 as *mut PageTableEntry, 512)
        }
    }
    // 将地址->裸指针+大小 得到可变的slice
    pub fn get_bytes_array(&self) -> &'static mut [u8] {
        let pa: PhysAddr = self.clone().into();
        unsafe {
            core::slice::from_raw_parts_mut(pa.0 as *mut u8, 4096)
        }
    }
    // 将地址->裸指针->可变引用(Option<&mut T>)->unwrap得到内部值
    pub fn get_mut<T>(&self) -> &'static mut T {
        let pa: PhysAddr = self.clone().into();
        unsafe {
            (pa.0 as *mut T).as_mut().unwrap()
        }
    }
}
```

## 4. 根据虚拟地址构建页表

* 对于传进来的VirtualPageNum, 需要**先构建页表**的映射后才能使用

```rust
// os/src/mm/address.rs
impl VirtPageNum {
	// 对虚拟页号从低到高通过逆序存储到数组中(每次9位)
	pub fn indexes(&self) -> [usize; 3];
}

//----------------查找(创建)页表-------------------------
// os/src/mm/page_table.rs
impl PageTable {
	// 根据虚拟页号得到对应的indexes数组,通过PageTable.root_ppn得到根页表的物理页号后，根据indexes不断查找页表项得到下一个页表的ppn，若页表项不存在就构建(但第三层就不再构建,即无对应的物理页帧):
	// frame = frame_alloc().unwrap() -> 
	// *pte = PageTableEntry::new(frame.ppn, PTEFlags::V) ->
	// self.frames.push(frame);
    fn find_pte_create(&mut self, vpn: VirtPageNum) -> Option<&mut PageTableEntry> 

	// 同上，不过若页表项不存在就直接返回
    fn find_pte(&self, vpn: VirtPageNum) -> Option<&mut PageTableEntry> 
```

* 构建完页表后就可以**map映射物理页帧**了

```rust
// os/src/mm/page_table.rs
impl PageTable {
	// map映射虚拟页号到物理页号(根据上面的find_pte_create找到最后的页表项来映射)
    pub fn map(&mut self, vpn: VirtPageNum, ppn: PhysPageNum, flags: PTEFlags) {
        let pte = self.find_pte_create(vpn).unwrap();
        assert!(!pte.is_valid(), "vpn {:?} is mapped before mapping", vpn);
        *pte = PageTableEntry::new(ppn, flags | PTEFlags::V);
    }

	// upmap取消映射(将页表项的内容清空)
    pub fn unmap(&mut self, vpn: VirtPageNum) {
        let pte = self.find_pte(vpn).unwrap();
        assert!(pte.is_valid(), "vpn {:?} is invalid before unmapping", vpn);
        *pte = PageTableEntry::empty();
    }
}
```

## 5. 模拟MMU根据页表查找页表项

* PageTable管理了他所有页表的所有权(在self.frames中(存储着FramTracker)); 
* 当遇到需要查一个特定页表（非当前正处在的地址空间的页表时），便可先通过 `PageTable::from_token()` 新建一个*临时的*页表(其内部的root_ppn为对应管理资源的PageTable.root_ppn)，再调用它的 `translate`() 方法查页表。

```rust
// os/src/mm/page_table.rs
impl PageTable {
    /// Temporarily used to get arguments from user space.
    pub fn from_token(satp: usize) -> Self {
        Self {
            root_ppn: PhysPageNum::from(satp & ((1usize << 44) - 1)),
            frames: Vec::new(),
        }
    }
    //调用find_pte得到页表项
    pub fn translate(&self, vpn: VirtPageNum) -> Option<PageTableEntry> {
        self.find_pte(vpn)
            .map(|pte| {pte.clone()})
    }
}
```




# 四、地址空间的抽象

## 1. 逻辑段（程序内连续的的一个段）

* **PageTable**中的Vec\<FrameTrack>中的FrameTrack中保存的只是作为 *页表的物理页帧* 的所有权；*实际存储内容的物理页帧* 的所有权被保存在**MapArea**中
* 二者共同保存了**页表**和**实际用到的内存**的所有权

```rust
// 一个逻辑段（连续空间）
pub struct MapArea {
	vpn_range: VPNRange,   //虚拟地址范围
	// 虚拟页号到物理页号的映射（物理页面的所有权）
    data_frames: BTreeMap<VirtPageNum, FrameTracker>,  
    map_type: MapType,     //映射类型（直接/间接映射）
	map_perm: MapPermission,  //访问方式（U/X/W/R）
}

impl MapArea {
	// 新建一个MapArea只是填充了虚拟地址范围、映射类型、映射功能,但并没有获得物理页表以及得到虚拟页号到物理页号的映射
	pub fn new(start_va: VirtualAddr,
				end_va: VirtualAddr,
				map_type: MapType,
				map_perm: MapPermission) -> Self
// -----------------------map---------------------------
	pub fn map(&mut self, page_table: &mut PageTable) {
        for vpn in self.vpn_range {
            self.map_one(page_table, vpn);
        }
    }
    // 1.保存物理页面到self.data_frames中(直接映射不用)
    // 2.将虚拟页号的映射关系写到指定的页表中
    pub fn map_one(&mut self, page_table: &mut PageTable, vpn: VirtPageNum) {
        let ppn: PhysPageNum;
        match self.map_type {
	        // ①若是直接映射:不用保存物理页面所有权
            MapType::Identical => {
                ppn = PhysPageNum(vpn.0);
            }
            // ②若是间接映射:获得物理页帧并保存所有权
            MapType::Framed => {
                let frame = frame_alloc().unwrap();
                ppn = frame.ppn;
                self.data_frames.insert(vpn, frame);
            }
        }
        let pte_flags = PTEFlags::from_bits(self.map_perm.bits).unwrap();
        page_table.map(vpn, ppn, pte_flags);
    }
    
//-----------------------unmap---------------------------
    pub fn unmap(&mut self, page_table: &mut PageTable) {
        for vpn in self.vpn_range {
            self.unmap_one(page_table, vpn);
        }
    }
    // 1.从self.data_frames中删除物理页面(直接映射不用)
    // 2.删除虚拟页号在页表中的映射
    pub fn unmap_one(&mut self, page_table: &mut PageTable, vpn: VirtPageNum) {
        match self.map_type {
            MapType::Framed => {
                self.data_frames.remove(&vpn);
            }
            _ => {}
        }
        page_table.unmap(vpn);
    }

//---------------------copy_data--------------------------
    // 将data根据page_table的映射写入到对应的物理内存中
    // 不过这里为什么不直接根据data_frames映射得到物理页号呢？？
    pub fn copy_data(&mut self, page_table: &mut PageTable, data: &[u8])
}
```

* 逻辑段的映射：对所有的vpn_range虚拟页号映射到对应物理页号写入page_table中（map()调用了多个map_one()）
	1. **直接映射**：直接调用页表page_table.map(vpn, ppn, pte_flags)
	2. **间接映射**：先分配映射后的物理页面frame_alloc()并放入到self.data_frames中，再调用页表映射page_table.map()
	* 所以对于内核的直接映射，会分配页表的物理页帧, 但不会分配实际用到的的物理页帧
	* 而对于间接映射，既会分配页表的物理页帧, 又要分配实际的物理页面并放入到self.data_frames中


## 2. 地址空间（所有段）

* PageTable中有着所有**页表**的物理页帧
* Vec\<MapArea>有着所有**程序内容**的物理页帧
* 二者共同构成了所有的物理页帧, 当一个地址空间 `MemorySet` 生命周期结束后，这些物理页帧都会被回收

```rust
// os/src/mm/memory_set.rs
pub struct MemorySet {
	page_table: PageTable,  //页表
    areas: Vec<MapArea>,    //所有段
}

impl MemorySet {
	// 将逻辑段放入内存空间中
	// ①先对MapArea中的所有虚拟地址->物理地址进行页表的映射(写入映射关系到self.page_table中); 
	// ②再将一个逻辑段放入到地址空间中(areas.push(map_area)),同时可以选择地向那段空间中写入数据(map_area.copy_date())
	fn push(&mut self, mut map_area: MapArea, data: Option<&[u8]>) {
	    map_area.map(&mut self.page_table);
	    if let Some(data) = data {
	        map_area.copy_data(&mut self.page_table, data);
	    }
	    self.areas.push(map_area);
	}

	// 为当前内存空间插入一段虚拟地址
	pub fn insert_framed_area(&mut self, start_va: VirAddr, end_va: VirtAddr, permission: MapPermission) {
        self.push(MapArea::new(
            start_va,
            end_va,
            MapType::Framed,
            permission,
        ), None);
    }
}
```

* MemorySet管理了一整个内存地址空间，当需要分配新的逻辑段时: `self.insert_framed_area()` 创建新的map_area: MapArea **->** `self.push()` 先对map_area中虚拟地址进行映射 `map_area.map` (保存实际物理页帧到map_area中，保存页表物理页帧到page_table中) **->**  再拷贝数据 `map_area.copy_data()`，**->** 最后再保存map_area到self.areas中 `self.areas.push(map_area)`

## 3. 内核地址空间

![[Pasted image 20230422223629.png|300]]  ![[Pasted image 20230422223615.png|300]]

* `new_kernel()`可生成完整的内核空间(包括页表映射和实际物理页帧)

```rust
impl MemorySet {    
    pub fn new_kernel() -> Self
    // 1.let mut memory_set = Self::new_bare()创建新的地址空间
    // 2.memory_set.map_trampoline()映射跳板
    // 3.将所有的5个逻辑段放入地址空间中并直接映射
}
```

在 `new_kernel()` 中，我们从低地址到高地址依次创建 5 个逻辑段（.text、.rodata、.data、.bss、AvailablePhysicalFrames）并通过 `push` 方法将它们插入到内核地址空间中

## 4. 应用地址空间

![[Pasted image 20230422215311.png|600]]

* `from_elf()`可以生成完整的用户空间(包括页表映射和实际物理页帧)

```rust
impl MemorySet {  
	pub fn from_elf(elf_data: &[u8]) -> (Self, usize, usize)
	// 1.let mut memory_set = Self::new_bare()创建新的地址空间
	// 2.映射trampoline
	// 3.取出elf文件中的每一个段push到每一个新的MapArea中并写入data同时将map_area放入到地址空间中
	// 4.映射并生成user stack with U flags
	// 5.映射并生成TrapContext(之前是切换的时候放到内核栈中的)
}
```

在 `os/src/build.rs` 中，我们不再将丢弃了所有符号的应用二进制镜像（.bin）链接进内核，因为在应用二进制镜像中，内存布局中各个逻辑段的位置和访问限制等信息都被裁剪掉了。我们直接使用 *保存了逻辑段信息* 的 **ELF 格式的应用可执行文件**。

它们和之前一样仍是基于 `build.rs` 生成的 `link_app.S` 给出的符号来确定其位置，并实际放在**内核的数据段**中。 `loader` 模块中原有的内核和用户栈则分别作为逻辑段放在内核和用户地址空间中，我们无需再去专门为其定义一种类型。

# 五、开启分页后需要的操作

## 1. 初始化 mm.init()：

1. `heap_allocator::init_heap()`：初始化**内核堆空间**
2. `frame_allocator::init_frame_allocator()`：初始化**物理页帧分配器**
3. `KERNEL_SPACE.exclusive_access().activate()`：**全局内核空间**的初始化
	1. `let satp = self.page_table.token()`：按照satp CSR 格式要求 根据内核页表构造一个无符号 64 位无符号整数，使得其分页模式为 SV39 ，
	2. `satp::write(satp)`：开启分页(内核页表的直接映射)
	3. `asm!("sfence.vma" :::: "volatile")`: 刷新TLB

## 2. Trap上下文的保存

* 原本应用trap到内核中时，可用sscratch得到内核栈栈顶并保存用户上下文
* 但现在开启了分页后除了要得到sscratch中的**内核栈**之前，还要获取**内核的page_table**，但RISC-V却只提供一个 `sscratch` 寄存器可用来进行周转，所以就需要将Trap的上下文保存在 *应用地址空间* 的一个虚拟页面，而不是切换到内核地址空间去保存

```rust
// os/src/trap/context.rs

#[repr(C)]
pub struct TrapContext {
    pub x: [usize; 32],
    pub sstatus: Sstatus,
    pub sepc: usize,
    // 多了下面三个，它们在应用初始化的时候由内核写入应用地址空间中的 TrapContext 的相应位置，此后就不再被修改。
	pub kernel_satp: usize,     // 内核页表起始物理地址
    pub kernel_sp: usize,       // 内核栈栈顶
    pub trap_handler: usize,    // 内核处理函数
}
```

## 3. 切换地址空间（trampoline）

* 由于TrapContext保存在了用户空间中，所以内核的**内核栈sp**和**内核页表page_table**都可以提前保存在**TrapContext**中，**sscratch寄存器保存的是用户空间中的TrapContext的地址**，在进入__alltraps中交换sscratch和sp后，sp指向TrapContext, sscratch指向用户栈
* 在__alltraps()最后需要借助寄存器 `jr` 而不能直接 `call trap_handler`，是因为：这条跳转指令在被执行的时候，它的**虚拟地址**被操作系统内核设置在地址空间中的 *最高页面* 之内，所以加上这个偏移量并不能正确的得到 `trap_handler` 的入口地址，需要认为设置跳转到对应的地址t1

```rust
// os/src/trap/trap.S
	// 将这段代码放到.text.trampoline中
	.section .text.trampoline
// 跳转到内核空间
__alltraps:
	// sscratch中保存的是TrapContext的地址
	csrrw sp, sscratch, sp
	// 接下来保存上下文到TrapContext中
	...
	// 多了下面的保存TrapContext多处的内容
	// load kernel_satp into t0
    ld t0, 34*8(sp)
    // load trap_handler into t1
    ld t1, 36*8(sp)
    // move to kernel_sp
    ld sp, 35*8(sp)
    // switch to kernel space
    csrw satp, t0
    // 刷新TLB
    sfence.vma
    // jump to trap_handler，不再是原来的ret(t1中为trap_handler)
    jr t1

// 恢复用户空间
__restore:
    // a0: *TrapContext in user space(所有用户空间都相同的); 
    // a1: user space token(用户page_table)
    // sscratch: user stack(之前在__alltraps中切换后保存的)
    // switch to user space
    csrw satp, a1
    sfence.vma
    csrw sscratch, a0   // 这里只是将a0写入sscratch，不是交换
    mv sp, a0           // 此时sp为TrapContext
    // 接下来恢复TrapContext上下文到寄存器中
    ...
    // back to user stack
    ld sp, 2*8(sp)
    sret
```

* 因为在 `__alltraps` 中会切换内核页表，所以无论是内核还是应用的地址空间，**跳板的虚拟页均位于同样位置(内核就需要在这里间接映射)**，且它们也将**会映射到同一个实际存放这段汇编代码的物理页帧**。也就是说，在执行 `__alltraps` 或 `__restore` 函数进行地址空间切换的时候，应用的用户态虚拟地址空间和操作系统内核的内核态虚拟地址空间对切换地址空间的指令**所在页的映射方式均是相同**的，这就说明了这段切换地址空间的指令控制流仍是可以**连续执行**的

```rust
// os/src/config.rs
// TRAMPOLINE的位置都是在地址空间的最顶部(内核和用户都一样)
pub const TRAMPOLINE: usize = usize::MAX - PAGE_SIZE + 1;

// os/src/mm/memory_set.rs

impl MemorySet {
    /// Mention that trampoline is not collected by areas.
    // 内核和用户空间都要调用这个映射trampoline的函数(相同映射方式)
    fn map_trampoline(&mut self) {
        self.page_table.map(
            VirtAddr::from(TRAMPOLINE).into(),
            PhysAddr::from(strampoline as usize).into(),
            PTEFlags::R | PTEFlags::X,
        );
    }
```

## 4.Trap的改进

* `trap_handler()` 的改变
	1. 先设置在**内核中的trap**会跳转到 `panic()` 中(设置stvec)
	2. 获取用户空间中的**TrapContext上下文**
	3. 根据scause()选择相应的处理方式

```rust
// os/src/trap/mod.rs

fn set_kernel_trap_entry() {
    unsafe {
        stvec::write(trap_from_kernel as usize, TrapMode::Direct);
    }
}

#[no_mangle]
pub fn trap_from_kernel() -> ! {
    panic!("a trap from kernel!");
}

#[no_mangle]
pub fn trap_handler() -> ! {
	// 1.设置内核中trap直接跳转去执行panic()
    set_kernel_trap_entry();
    // 2.获得用户空间的TrapContext上下文
    let cx = current_trap_cx();
    let scause = scause::read();
    let stval = stval::read();
    // 3.选择对应的方式处理
    match scause.cause() {
        ...
    }
    trap_return();
}
```

* 当trap_handler执行完后不是原来的直接返回到`__restore()` 中，而是调用`trap_return()` 指向相关的操作后再跳转到`__restore()` 中执行恢复TrapContext上下文
	1. 设置**stvec寄存器为TRAMPOLINE地址**(这是虚拟地址，在分页模式下内核只能通过该虚拟地址得到__alltraps和__restore代码)
		1. 在内核中各种段不是直接映射的吗，应该也可以直接映射到__alltraps和__restore中吧？？
	2. 获取用户空间TrapContext上下文
	3. `fence.i` 清空指令缓存 i-cache（i-cache 中可能还保存着某些物理页帧的错误快照）
	4. 跳转到`__restore()` 执行恢复上下文代码(这里的地址也是根据TRAMPOLINE得到的虚拟地址)

```rust
// os/src/trap/mod.rs

fn set_user_trap_entry() {
    unsafe {
        stvec::write(TRAMPOLINE as usize, TrapMode::Direct);
    }
}

#[no_mangle]
pub fn trap_return() -> ! {
	// 1.设置stvec为用户trap时跳转到的trampoline地址
    set_user_trap_entry();
    let trap_cx_ptr = TRAP_CONTEXT;
    // 2.获取用户TrapContext上下文
    let user_satp = current_user_token();
    extern "C" {
        fn __alltraps();
        fn __restore();
    }
    let restore_va = __restore as usize - __alltraps as usize + TRAMPOLINE;
    unsafe {
        asm!(
	        // 3.刷新指令缓存
            "fence.i",
            // 4.跳转到__restore()恢复上下文
            "jr {restore_va}",
            restore_va = in(reg) restore_va,
            in("a0") trap_cx_ptr,
            in("a1") user_satp,
            options(noreturn)
        );
    }
    panic!("Unreachable in back_to_user!");
}
```

## 5. 内核栈

* 内核栈不仅是每一个用户空间跳转到**内核空间所使用的栈**，而且还保存着**TaskContext**供内核在第一次切换到用户空间或者`__switch()` 切换其他应用时保存着的任务上下文(ra: trap_return as usize)，可以ret**返回到trap_return()** 并执行返回到用户空间代码

```rust
// os/src/config.rs

// 每个app都有对应的内核栈在内核空间中(参考内核空间的图片)
/// Return (bottom, top) of a kernel stack in kernel space.
pub fn kernel_stack_position(app_id: usize) -> (usize, usize) {
    let top = TRAMPOLINE - app_id * (KERNEL_STACK_SIZE + PAGE_SIZE);
    let bottom = top - KERNEL_STACK_SIZE;
    (bottom, top)
}

// os/src/task/context.rs

impl TaskContext {
	// 内核中创建TCB时会用到
    pub fn goto_trap_return() -> Self {
        Self {
            ra: trap_return as usize,
            s: [0; 12],
        }
    }
}
```

## 6. TCB的拓展

```rust
// os/src/task/task.rs
// TCB结构的拓展
pub struct TaskControlBlock {
    pub task_cx: TaskContext,
    pub task_status: TaskStatus,
    // 多了下面三个内容
    // 1.用户的地址空间
    pub memory_set: MemorySet,
    // 2.TrapContext的物理页号
    pub trap_cx_ppn: PhysPageNum,
    // 3.应用数据的大小，在应用地址空间中从0x0开始到用户栈结束一共包含多少字节（后面还要包括堆动态分配的大小）
    pub base_size: usize,
}


```

### 1. 内核中创建一个**用户空间**并创建对他的**管理的内容**

1. `from_elf()` 创建一个完整的用户空间
2. 创建并映射**内核栈**
3. 创建**TCB**
4. 填充TCB指向的用户空间中的**TrapContext中的内容**(使得能返回用户空间的入口和用户可以跳转到内核空间)

```rust
// os/src/task/task.rs
impl TaskControlBlock {
	// 新建一个TCB，也即初始化一个用户空间和对他的管理的内容
    pub fn new(elf_data: &[u8], app_id: usize) -> Self {
        // 1.from_elf()创建一个完整的用户空间(包括program/trampoline/TrapContext/user stack)
        let (memory_set, user_sp, entry_point) = MemorySet::from_elf(elf_data);
        let trap_cx_ppn = memory_set
            .translate(VirtAddr::from(TRAP_CONTEXT).into())
            .unwrap()
            .ppn();
        let task_status = TaskStatus::Ready;
        // 2.map a kernel-stack in kernel space
        let (kernel_stack_bottom, kernel_stack_top) = kernel_stack_position(app_id);
        KERNEL_SPACE
            .exclusive_access()
            .insert_framed_area(
                kernel_stack_bottom.into(),
                kernel_stack_top.into(),
                MapPermission::R | MapPermission::W,
            );
        // 3.创建TCB
        let task_control_block = Self {
            task_status,
            task_cx: TaskContext::goto_trap_return(kernel_stack_top),
            memory_set,
            trap_cx_ppn,
            base_size: user_sp,
        };
        // 4.prepare TrapContext in user space
        let trap_cx = task_control_block.get_trap_cx();
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

### 2. 内核中创建任务管理器TaskManager管理所有用户空间

* 在初始化**全局变量TASK_MANAGER**的时候就会建立所有的 *用户地址空间* 以及 *相关的管理内容*

```rust
// os/src/task/mod.rs
// TaskManager里面的管理所有TCB
struct TaskManagerInner {
    tasks: Vec<TaskControlBlock>,
    current_task: usize,
}

lazy_static! {
	// 全局TaskManager变量
    pub static ref TASK_MANAGER: TaskManager = {
        println!("init TASK_MANAGER");
        let num_app = get_num_app();
        println!("num_app = {}", num_app);
        let mut tasks: Vec<TaskControlBlock> = Vec::new();
        // 创建所有用户空间user space以及管理内容TCB
        for i in 0..num_app {
            tasks.push(TaskControlBlock::new(
                get_app_data(i),
                i,
            ));
        }
        TaskManager {
            num_app,
            inner: RefCell::new(TaskManagerInner {
                tasks,
                current_task: 0,
            }),
        }
    };
}
```

### 3. 提供关于当前应用地址空间有关的一些信息

```rust
// os/src/task/mod.rs
impl TaskManager {
	// 当前正在执行的应用的地址空间的token(页表？)
    fn get_current_token(&self) -> usize {
        let inner = self.inner.borrow();
        let current = inner.current_task;
        inner.tasks[current].get_user_token()
    }

	// 可在内核地址空间中修改该应用地址空间中的TrapContext的可变引用
    fn get_current_trap_cx(&self) -> &mut TrapContext {
        let inner = self.inner.borrow();
        let current = inner.current_task;
        inner.tasks[current].get_trap_cx()
    }
}
// 上面的包装
pub fn current_user_token() -> usize {
    TASK_MANAGER.get_current_token()
}

pub fn current_trap_cx() -> &'static mut TrapContext {
    TASK_MANAGER.get_current_trap_cx()
}
```

# 六、页面置换

> 操作系统利用更大且更慢的存储设备，透明地给应用程序提供远超物理内存空间的虚拟地址空间，这就需要用到页面置换机制

* 使用场景：
	1. **Demand Paging**：开始时应用程序的页面都不加载到内存，当程序执行时会出现异常，这时候再加载对应的内存
	2. **Write-On-Use**：当fork一个子进程的时候，先不复制所有内存，而是通过共享内存和修改页表为只读，当需要写的时候就会出现异常，这时再复制
	3. **Lazy Allocation**：延迟分配，当应用申请拓展内存的时候，只是填充对应的页表项（有效位设置为0），当访存的时候会出现异常，这时再分配


## 1. 交换区

* 在存储设备中分配一个**特定的分区**来作为存放物理内存的交换区：磁盘扇区大小为N(512B)，一个页面大小为k\*N(4kB)，可以通过在 *页表项中存储存储设备的扇区地址* 来实现查找原本对应的页面

## 2.页表项

* 当需要将任务的页面换出的时候，需要将对应的页表项的**有效位v置零**，同时需要在页表项中**存储着对应存储设备的扇区地址**，这样当再次访问对应虚拟页号的地址的时候MMU会出错产生“Page Fault”，将控制权转给内核的异常处理程序来处理(然后在对应的处理程序中再换页进来即可)

## 3. 内存访问异常处理

* 当访问出现"Page Fault"的时候先判断该虚拟地址是否时该任务的**合法地址**(根据TCB中查看), 然后再根据页表项**取出对应扇区交换区中的内容**写到空闲的物理内存页中，再**更新页表项**内容；最后返回任务**重新执行出错的指令**

## 4. 页面置换策略

* **Clock置换策略**（近似LRU策略）: 每当一个页面被引用的时候，**处理器硬件会把对应该页的页表的引用位设置为1**（但处理器硬件不会置零，操作系统可以定期置零），当需要置换页表的时候，操作系统再通过查询该应用所有页表的引用位(可以通过循环列表保存所有的有效页)，如果引用位是0就置换
* 除此之外，硬件修改一个页面的时候还会将对应的页表项的脏位dirty bit置1，所以操作系统可以通过**引用位**和**脏位**一起判断是否要置换该页表
	* 置换优先级：未使用且干净 -> 使用且干净/未使用且脏 -> 使用且脏