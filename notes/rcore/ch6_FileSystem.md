> 可以将文件系统当作一个单独的模块，它实现了如何从磁盘读写内容到内存中，但需要使用的人提供最基础的接口的实现（这里的easy-fs需要提供最底层块设备接口层的实现）

![[Pasted image 20230503212026.png]]

![[ch6_FileSystem 2023-05-03 21.21.26.excalidraw]]

![[ch6_FileSystem 2023-05-04 11.40.22.excalidraw|600]]

# easy-fs文件系统

`easy-fs` crate 自下而上大致可以分成五个不同的层次：

1. **磁盘块设备接口层**：定义了以块大小为单位对磁盘块设备进行读写的 *trait接口* 
2. **块缓存层**：在内存中缓存磁盘块的数据，避免频繁读写磁盘
3. **磁盘数据结构层**：磁盘上的超级块、位图、索引节点、数据块、目录项等核心数据结构和相关处理
4. **磁盘块管理器层**：合并了上述核心数据结构和磁盘布局所形成的磁盘文件系统数据结构，以及基于这些结构的创建/打开文件系统的相关处理和磁盘块的分配和回收处理
5. **索引节点层**：管理索引节点（即文件控制块）数据结构，并实现文件创建/文件打开/文件读写等成员函数来向上支持文件操作相关的系统调用

## 1. 块设备接口层

> 块设备接口层只是提供了trait，需要具体的块设备驱动来实现方法

```rust
// easy-fs/src/block_dev.rs

pub trait BlockDevice : Send + Sync + Any {
    fn read_block(&self, block_id: usize, buf: &mut [u8]);
    fn write_block(&self, block_id: usize, buf: &[u8]);
}
```

## 2. 块缓存层

### 1. 块缓存

* `BlockCache` 结构拥有一块内存缓冲区的**所有权**，同时还包含一个指向块设备接口层的结构

```rust
// easy-fs/src/lib.rs
pub const BLOCK_SZ: usize = 512;

// easy-fs/src/block_cache.rs
pub struct BlockCache {
    cache: [u8; BLOCK_SZ],
    block_id: usize,
    block_device: Arc<dyn BlockDevice>,
    modified: bool,
}

impl BlockCache {
	// 因为可能还有其他系统调用可以直接写回块,所以提供接口
    pub fn sync(&mut self) {
        if self.modified {
            self.modified = false;
            self.block_device.write_block(self.block_id, &self.cache);
        }
    }
}

// 当drop的时候调用sync()将缓冲区内容写回块中
impl Drop for BlockCache {
    fn drop(&mut self) {
        self.sync()
    }
}
```

当创建 `BlockCache` 的时候就触发 `read_block()` 将一个块上的数据写入到缓冲区中

```rust
// easy-fs/src/block_cache.rs

impl BlockCache {
    /// Load a new BlockCache from disk.
    pub fn new(
        block_id: usize,
        block_device: Arc<dyn BlockDevice>
    ) -> Self {
        let mut cache = [0u8; BLOCK_SZ];
        block_device.read_block(block_id, &mut cache);
        Self {
            cache,
            block_id,
            block_device,
            modified: false,
        }
    }
}
```

当块存在于内存缓冲中，CPU可以直接访问缓冲

```rust
// easy-fs/src/block_cache.rs

impl BlockCache {
	// 访问块的偏移地址
    fn addr_of_offset(&self, offset: usize) -> usize {
        &self.cache[offset] as *const _ as usize
    }

	// 返回对应类型的引用
    pub fn get_ref<T>(&self, offset: usize) -> &T where T: Sized {
        let type_size = core::mem::size_of::<T>();
        assert!(offset + type_size <= BLOCK_SZ);
        let addr = self.addr_of_offset(offset);
        unsafe { &*(addr as *const T) }
    }

	// 返回对应类型的可变引用
    pub fn get_mut<T>(&mut self, offset: usize) -> &mut T where T: Sized {
        let type_size = core::mem::size_of::<T>();
        assert!(offset + type_size <= BLOCK_SZ);
        self.modified = true;
        let addr = self.addr_of_offset(offset);
        unsafe { &mut *(addr as *mut T) }
    }
}
```

封装接口 (闭包执行)

```rust
// easy-fs/src/block_cache.rs

impl BlockCache {
    pub fn read<T, V>(&self, offset: usize, f: impl FnOnce(&T) -> V) -> V {
        f(self.get_ref(offset))
    }

    pub fn modify<T, V>(&mut self, offset:usize, f: impl FnOnce(&mut T) -> V) -> V {
        f(self.get_mut(offset))
    }
}
```

### 2. 块缓存管理器

> 通过一个双端队列来保存一定数量的BlockCache，使得内存同时只能驻留有限个磁盘块的缓冲区

```rust
// easy-fs/src/block_cache.rs
const BLOCK_CACHE_SIZE: usize = 16;
```

队列的元素是 `(块编号: usize，块缓存: Arc<Mutex\<BlockCache>>)` 的二元组（块缓存要提供共享引用Arc 和 互斥访问 Mutex）

```rust
// easy-fs/src/block_cache.rs
use alloc::collections::VecDeque;

pub struct BlockCacheManager {
    queue: VecDeque<(usize, Arc<Mutex<BlockCache>>)>,
}

impl BlockCacheManager {
    pub fn new() -> Self {
        Self { queue: VecDeque::new() }
    }
}
```

从块管理器中获取编号为 `block_id` 的块缓冲区的步骤：
1. 判断**是否已经存在缓冲**(遍历缓冲区管理器的队列)，若存在则直接返回；否则需要分配
2. 若**队列未满**则可以直接分配新的 `BlockCache` 并加入队列，否则需要替换
3. 若队列中**有 `BlockCache` 只有一个引用**（即缓冲区管理器的引用）就可以替换，否则说明所有的缓冲区都正在被使用，不可替换，直接panic

```rust
// easy-fs/src/block_cache.rs

impl BlockCacheManager {
    pub fn get_block_cache(
        &mut self,
        block_id: usize,
        block_device: Arc<dyn BlockDevice>,
    ) -> Arc<Mutex<BlockCache>> {
	    // 1.已存在缓冲
        if let Some(pair) = self.queue
            .iter()
            .find(|pair| pair.0 == block_id) {
                Arc::clone(&pair.1)
        } else {
            // 2.队列已满需要替换
            if self.queue.len() == BLOCK_CACHE_SIZE {
                // 3.from front to tail有只有一个引用的BlockCache
                if let Some((idx, _)) = self.queue
                    .iter()
                    .enumerate()
                    .find(|(_, pair)| Arc::strong_count(&pair.1) == 1) {
                    self.queue.drain(idx..=idx);
                } else {
                    panic!("Run out of BlockCache!");
                }
            }
            // load block into mem and push back
            let block_cache = Arc::new(Mutex::new(
                BlockCache::new(block_id, Arc::clone(&block_device))
            ));
            self.queue.push_back((block_id, Arc::clone(&block_cache)));
            block_cache
        }
    }
}
```

全局缓冲区管理器变量和通用接口

```rust
// easy-fs/src/block_cache.rs

lazy_static! {
    pub static ref BLOCK_CACHE_MANAGER: Mutex<BlockCacheManager> = Mutex::new(
        BlockCacheManager::new()
    );
}

// 返回对应的缓冲区
pub fn get_block_cache(
    block_id: usize,
    block_device: Arc<dyn BlockDevice>
) -> Arc<Mutex<BlockCache>> {
    BLOCK_CACHE_MANAGER.lock().get_block_cache(block_id, block_device)
}
```

## 3. 磁盘布局及磁盘上的数据结构

![[Pasted image 20230503151012.png]]

### 1. SuperBlock 超级块

* `SuperBlock` 存放在磁盘的编号0的块的起始处，是一个**磁盘数据结构**

```rust
// easy-fs/src/layout.rs

#[repr(C)]
pub struct SuperBlock {
	// magic 用于验证文件系统是否合法
    magic: u32,
	// 文件系统总块数
    pub total_blocks: u32,
    // 4个连续区域各占多少块
    pub inode_bitmap_blocks: u32,
    pub inode_area_blocks: u32,
    pub data_bitmap_blocks: u32,
    pub data_area_blocks: u32,
}
```

### 2. Bitmap 位图

* `Bitmap` 结构存放在**内存中**，但可以表示索引节点/数据块区域中的磁盘块分配情况

```rust
// easy-fs/src/bitmap.rs

pub struct Bitmap {
	// 起始块编号
    start_block_id: usize,
    // 占的块数目
    blocks: usize,
}

impl Bitmap {
    pub fn new(start_block_id: usize, blocks: usize) -> Self {
        Self {
            start_block_id,
            blocks,
        }
    }
}
```

* `BitmapBlock` 结构存放在磁盘中，是**磁盘数据结构**，负责将位图区域的一个磁盘块解释为长度64的 u64 数组 (64\*64=4096即一个块位数)

```rust
// easy-fs/src/bitmap.rs
type BitmapBlock = [u64; 64];
```

`Bitmap` 通过修改 `BitmapBlock` 来修改位图

#### 1. 分配一个bit

1. 首先将对应的位图 `start_block_id + block_id` 读到内存磁盘缓冲区
2. 然后找到位图块中第一个没有分配的位： `bits64_pos` 为第几个元素，`inner_pos` 为对应元素的第一个为0的位
3. 之后修改位图对应的位并返回该位是**所有位图块中的第几位**

```rust
// easy-fs/src/bitmap.rs

const BLOCK_BITS: usize = BLOCK_SZ * 8;

impl Bitmap {
    pub fn alloc(&self, block_device: &Arc<dyn BlockDevice>) -> Option<usize> {
	    // 1.block_id为第几块位图
        for block_id in 0..self.blocks {
            let pos = get_block_cache(
                block_id + self.start_block_id as usize,
                Arc::clone(block_device),
            )
            .lock()
            .modify(0, |bitmap_block: &mut BitmapBlock| {
	            // 2.bits64_pos为BitmapBlock的第几个元素
	            // inner_pos为BitmapBlock对应元素的第几位
                if let Some((bits64_pos, inner_pos)) = bitmap_block
                    .iter()
                    .enumerate()
                    .find(|(_, bits64)| **bits64 != u64::MAX)
                    .map(|(bits64_pos, bits64)| {
                        (bits64_pos, bits64.trailing_ones() as usize)
                    }) {
                    // 3.modify cache修改位图
                    bitmap_block[bits64_pos] |= 1u64 << inner_pos;
                    // 4.返回位图中的第几位(位图对应的块中第几个块)
                    Some(block_id * BLOCK_BITS + bits64_pos * 64 + inner_pos as usize)
                } else {
                    None
                }
            });
            if pos.is_some() {
                return pos;
            }
        }
        None
    }
}
```

#### 2. 回收一个bit

1. 首先解构对应的bit为对应的块、第几个元素、第几位
2. 然后读取位图对应的块到内存缓冲区
3. 修改对应的位为0

```rust
// easy-fs/src/bitmap.rs

/// Return (block_pos, bits64_pos, inner_pos)
fn decomposition(mut bit: usize) -> (usize, usize, usize) {
    let block_pos = bit / BLOCK_BITS;
    bit = bit % BLOCK_BITS;
    (block_pos, bit / 64, bit % 64)
}

impl Bitmap {
    pub fn dealloc(&self, block_device: &Arc<dyn BlockDevice>, bit: usize) {
        let (block_pos, bits64_pos, inner_pos) = decomposition(bit);
        get_block_cache(
            block_pos + self.start_block_id,
            Arc::clone(block_device)
        ).lock().modify(0, |bitmap_block: &mut BitmapBlock| {
            assert!(bitmap_block[bits64_pos] & (1u64 << inner_pos) > 0);
            bitmap_block[bits64_pos] -= 1u64 << inner_pos;
        });
    }
}
```

### 3. DiskInode 磁盘索引节点

* 磁盘上的索引节点区域的每个块上都保存着索引节点 `DiskInode` , 每个结构占128个字节，所以每个索引块包含4个 `DiskInode`

```rust
// easy-fs/src/layout.rs
// 直接索引最多的块的个数
const INODE_DIRECT_COUNT: usize = 28;

#[repr(C)]
pub struct DiskInode {
	// 占的字节数
    pub size: u32,
    // 直接索引的数组(元素为索引号)
    pub direct: [u32; INODE_DIRECT_COUNT],
    // 一级间接索引,指向一个一级索引块(位于数据块中),块中每个u32用来指向数据块
    pub indirect1: u32,
    // 二级间接索引,指向一个二级索引块(位于数据块中),块中每个u32指向一级索引块(位于数据块中)
    pub indirect2: u32,
    // 文件/目录类型
    type_: DiskInodeType,
}

impl DiskInode {
    /// indirect1 and indirect2 block are allocated only when they are needed.
    // 初始化所有的数据块索引都为0
    pub fn initialize(&mut self, type_: DiskInodeType) {
        self.size = 0;
        self.direct.iter_mut().for_each(|v| *v = 0);
        self.indirect1 = 0;
        self.indirect2 = 0;
        self.type_ = type_;
    }
}

// -------------------------DiskInode类型-------------------
#[derive(PartialEq)]
pub enum DiskInodeType {
    File,
    Directory,
}

impl DiskInode {
    pub fn is_dir(&self) -> bool {
        self.type_ == DiskInodeType::Directory
    }
    pub fn is_file(&self) -> bool {
        self.type_ == DiskInodeType::File
    }
}
```

* 通过 inner_id文件中的第几个块 得到 block_id对应的块的编号(索引)

```rust
// easy-fs/src/layout.rs
// 一级间接索引块中可引用数据块数量
const INODE_INDIRECT1_COUNT: usize = BLOCK_SZ / 4;
// 一级间接索引总共可引用数据块数量(直接引用+一级间接)
const INDIRECT1_BOUND: usize = DIRECT_BOUND + INODE_INDIRECT1_COUNT;
// 间接索引块的数据类型
type IndirectBlock = [u32; BLOCK_SZ / 4];

impl DiskInode {
    pub fn get_block_id(&self, inner_id: u32, block_device: &Arc<dyn BlockDevice>) -> u32 {
        let inner_id = inner_id as usize;
        // 1.直接索引
        if inner_id < INODE_DIRECT_COUNT {
            self.direct[inner_id]
        } 
        // 2.一级间接索引(先读取一级间接索引块到缓冲区，在从缓冲区读取对应的块的索引)
        else if inner_id < INDIRECT1_BOUND {
            get_block_cache(self.indirect1 as usize, Arc::clone(block_device))
                .lock()
                .read(0, |indirect_block: &IndirectBlock| {
                    indirect_block[inner_id - INODE_DIRECT_COUNT]
                })
        } 
        // 3.二级间接索引(分别读取二级和对应一级间接索引块到缓冲区,再从缓冲区读取对应的块的索引)
        else {
            let last = inner_id - INDIRECT1_BOUND;
            let indirect1 = get_block_cache(
                self.indirect2 as usize,
                Arc::clone(block_device)
            )
            .lock()
            .read(0, |indirect2: &IndirectBlock| {
                indirect2[last / INODE_INDIRECT1_COUNT]
            });
            get_block_cache(
                indirect1 as usize,
                Arc::clone(block_device)
            )
            .lock()
            .read(0, |indirect1: &IndirectBlock| {
                indirect1[last % INODE_INDIRECT1_COUNT]
            })
        }
    }
}
```

* 容量扩展的准备：根据文件字节大小计算需要多少个块

```rust
// easy-fs/src/layout.rs

impl DiskInode {
    pub fn data_blocks(&self) -> u32 {
        Self::_data_blocks(self.size)
    }
    // 返回其字节大小需要多少个数据块
    fn _data_blocks(size: u32) -> u32 {
        (size + BLOCK_SZ as u32 - 1) / BLOCK_SZ as u32
    }
    // 返回其字节大小需要多少个块(索引块+数据块)
    pub fn total_blocks(size: u32) -> u32 {
        let data_blocks = Self::_data_blocks(size) as usize;
        let mut total = data_blocks as usize;
        // indirect1
        if data_blocks > INODE_DIRECT_COUNT {
            total += 1;
        }
        // indirect2
        if data_blocks > INDIRECT1_BOUND {
            total += 1;
            // sub indirect1
            total += (data_blocks - INDIRECT1_BOUND + INODE_INDIRECT1_COUNT - 1) / INODE_INDIRECT1_COUNT;
        }
        total as u32
    }
    // 返回需要多少新增的块
    pub fn blocks_num_needed(&self, new_size: u32) -> u32 {
        assert!(new_size >= self.size);
        Self::total_blocks(new_size) - Self::total_blocks(self.size)
    }
}
```

* 容量扩展 (由上层磁盘块管理器负责分配新的块)

```rust
// easy-fs/src/layout.rs

impl DiskInode {
    pub fn increase_size(
        &mut self,
        // 新的字节大小
        new_size: u32,
        // 新增的块的编号(上层磁盘管理器负责)
        new_blocks: Vec<u32>,
        block_device: &Arc<dyn BlockDevice>,
    );
}
```

* 清除回收所有的数据块和索引块

```rust
// easy-fs/src/layout.rs

impl DiskInode {
    /// Clear size to zero and return blocks that should be deallocated.
    /// We will clear the block contents to zero later.
    // 会返回所有的块的索引数组
    pub fn clear_size(&mut self, block_device: &Arc<dyn BlockDevice>) -> Vec<u32>;
}
```

* 通过 DiskInode 读写对应偏移量的数据块中的数据

```rust
// easy-fs/src/layout.rs

type DataBlock = [u8; BLOCK_SZ];

impl DiskInode {
    pub fn read_at(
        &self,
        offset: usize,
        buf: &mut [u8],
        block_device: &Arc<dyn BlockDevice>,
    ) -> usize {
        let mut start = offset;
        let end = (offset + buf.len()).min(self.size as usize);
        // 如果字节偏移量大于文件大小就直接返回0
        if start >= end {
            return 0;
        }
        let mut start_block = start / BLOCK_SZ;
        let mut read_size = 0usize;
        // 读取磁盘块内容到对应数组中,每次最多读取一个磁盘块的内容
        loop {
            // calculate end of current block
            let mut end_current_block = (start / BLOCK_SZ + 1) * BLOCK_SZ;
            end_current_block = end_current_block.min(end);
            // read and update read size
            let block_read_size = end_current_block - start;
            let dst = &mut buf[read_size..read_size + block_read_size];
// 根据字节偏移usize->DiskInode中的第个数据块->磁盘的数据块索引
            get_block_cache(
                self.get_block_id(start_block as u32, block_device) as usize,
                Arc::clone(block_device),
            )
            .lock()
            .read(0, |data_block: &DataBlock| {
                let src = &data_block[start % BLOCK_SZ..start % BLOCK_SZ + block_read_size];
                dst.copy_from_slice(src);
            });
            read_size += block_read_size;
            // move to next block
            if end_current_block == end { break; }
            start_block += 1;
            start = end_current_block;
        }
        read_size
    }
}
```

### 4. 数据块与目录项

> 数据块中存放的内容可以是**字节序列(数据)**、DiskInode的**间接索引**、**目录项DirEntry**；
> **文件**类型的DiskInode用到数据块有: 间接索引和存储数据的字节序列
> **目录**类型的DiskInode用到数据块有: 目录项

1. 字节序列的数据块 (存储着文件的内容)

```rust
// easy-fs/src/layout.rs
type DataBlock = [u8; BLOCK_SZ];
```

2. 目录项：子目录/文件的名字+子目录/文件的DiskInode；每个块中可以存储16个目录项 (一个目录项32字节, 512/32)

```rust
// easy-fs/src/layout.rs
// 名字长度(包括有文件和目录的名字)
const NAME_LENGTH_LIMIT: usize = 27;

// 这里的目录项的内容都是该目录下所包含的文件/目录的内容
#[repr(C)]
pub struct DirEntry {
    name: [u8; NAME_LENGTH_LIMIT + 1],
    inode_number: u32,
}

pub const DIRENT_SZ: usize = 32;

impl DirEntry {
	// 返回空的目录项
    pub fn empty() -> Self {
        Self {
            name: [0u8; NAME_LENGTH_LIMIT + 1],
            inode_number: 0,
        }
    }
    // 生成一个新的目录项
    pub fn new(name: &str, inode_number: u32) -> Self {
        let mut bytes = [0u8; NAME_LENGTH_LIMIT + 1];
        &mut bytes[..name.len()].copy_from_slice(name.as_bytes());
        Self {
            name: bytes,
            inode_number,
        }
    }
}
```

* 修改磁盘中目录项的内容：需要先将内存中的目录项转化为对应的符合 `read_at()` 或者 `write_at()` 的接口形式

```rust
// easy-fs/src/layout.rs

impl DirEntry {
	// 转化为字节形式
    pub fn as_bytes(&self) -> &[u8] {
        unsafe {
            core::slice::from_raw_parts(
                self as *const _ as usize as *const u8,
                DIRENT_SZ,
            )
        }
    }
    // 转化为可变字节形式
    pub fn as_bytes_mut(&mut self) -> &mut [u8] {
        unsafe {
            core::slice::from_raw_parts_mut(
                self as *mut _ as usize as *mut u8,
                DIRENT_SZ,
            )
        }
    }
}
```

* 得到目录项的内容

```rust
// easy-fs/src/layout.rs

impl DirEntry {
	// 返回名字
    pub fn name(&self) -> &str {
        let len = (0usize..).find(|i| self.name[*i] == 0).unwrap();
        core::str::from_utf8(&self.name[..len]).unwrap()
    }
    // 返回DiskInode索引
    pub fn inode_number(&self) -> u32 {
        self.inode_number
    }
}
```

## 4. 磁盘块管理器

> 磁盘块管理器EasyFileSystem实现了磁盘布局并且管理磁盘块的所有部分

* 磁盘管理器结构

```rust
// easy-fs/src/efs.rs

pub struct EasyFileSystem {
    pub block_device: Arc<dyn BlockDevice>,
    pub inode_bitmap: Bitmap,
    pub data_bitmap: Bitmap,
    inode_area_start_block: u32,
    data_area_start_block: u32,
}
```

* 创建并初始化一个 easy-fs 文件系统

```rust
// easy-fs/src/efs.rs

impl EasyFileSystem {
    pub fn create(
        block_device: Arc<dyn BlockDevice>,
        total_blocks: u32,
        inode_bitmap_blocks: u32,
    ) -> Arc<Mutex<Self>> {
        // calculate block size of areas & create bitmaps
        let inode_bitmap = Bitmap::new(1, inode_bitmap_blocks as usize);
        let inode_num = inode_bitmap.maximum();
        let inode_area_blocks =
            ((inode_num * core::mem::size_of::<DiskInode>() + BLOCK_SZ - 1) / BLOCK_SZ) as u32;
        let inode_total_blocks = inode_bitmap_blocks + inode_area_blocks;
        let data_total_blocks = total_blocks - 1 - inode_total_blocks;
        let data_bitmap_blocks = (data_total_blocks + 4096) / 4097;
        let data_area_blocks = data_total_blocks - data_bitmap_blocks;
        let data_bitmap = Bitmap::new(
            (1 + inode_bitmap_blocks + inode_area_blocks) as usize,
            data_bitmap_blocks as usize,
        );
        // 1.根据计算好的各个磁盘块部分的位置及大小创建磁盘管理器
        let mut efs = Self {
            block_device: Arc::clone(&block_device),
            inode_bitmap,
            data_bitmap,
            inode_area_start_block: 1 + inode_bitmap_blocks,
            data_area_start_block: 1 + inode_total_blocks + data_bitmap_blocks,
        };
        // 2.clear all blocks
        for i in 0..total_blocks {
            get_block_cache(
                i as usize,
                Arc::clone(&block_device)
            )
            .lock()
            .modify(0, |data_block: &mut DataBlock| {
                for byte in data_block.iter_mut() { *byte = 0; }
            });
        }
        // 3.initialize SuperBlock
        get_block_cache(0, Arc::clone(&block_device))
        .lock()
        .modify(0, |super_block: &mut SuperBlock| {
            super_block.initialize(
                total_blocks,
                inode_bitmap_blocks,
                inode_area_blocks,
                data_bitmap_blocks,
                data_area_blocks,
            );
        });
        // write back immediately
        // 4.create a inode for root node "/"
        assert_eq!(efs.alloc_inode(), 0);
        let (root_inode_block_id, root_inode_offset) = efs.get_disk_inode_pos(0);
        get_block_cache(
            root_inode_block_id as usize,
            Arc::clone(&block_device)
        )
        .lock()
        .modify(root_inode_offset, |disk_inode: &mut DiskInode| {
            disk_inode.initialize(DiskInodeType::Directory);
        });
        Arc::new(Mutex::new(efs))
    }
}
```

* 从磁盘块0中读出 `SuperBlock` 来构建 easy-fs

```rust
// easy-fs/src/efs.rs

impl EasyFileSystem {
    pub fn open(block_device: Arc<dyn BlockDevice>) -> Arc<Mutex<Self>> {
        // read SuperBlock
        get_block_cache(0, Arc::clone(&block_device))
            .lock()
            .read(0, |super_block: &SuperBlock| {
                assert!(super_block.is_valid(), "Error loading EFS!");
                let inode_total_blocks =
                    super_block.inode_bitmap_blocks + super_block.inode_area_blocks;
                let efs = Self {
                    block_device,
                    inode_bitmap: Bitmap::new(
                        1,
                        super_block.inode_bitmap_blocks as usize
                    ),
                    data_bitmap: Bitmap::new(
                        (1 + inode_total_blocks) as usize,
                        super_block.data_bitmap_blocks as usize,
                    ),
                    inode_area_start_block: 1 + super_block.inode_bitmap_blocks,
                    data_area_start_block: 1 + inode_total_blocks + super_block.data_bitmap_blocks,
                };
                Arc::new(Mutex::new(efs))
            })
    }
}
```

* 获取 DiskInode 的**块索引和块内偏移**：因为*没有实现删除* DiskInode且分配都是按顺序分配的，所以可以直接得到 DiskInode位置
* 获取 data block的**块索引**

```rust
// easy-fs/src/efs.rs

impl EasyFileSystem {
	// 返回(inode的索引, 块内的偏移)
    pub fn get_disk_inode_pos(&self, inode_id: u32) -> (u32, usize) {
        let inode_size = core::mem::size_of::<DiskInode>();
        let inodes_per_block = (BLOCK_SZ / inode_size) as u32;
        let block_id = self.inode_area_start_block + inode_id / inodes_per_block;
        (block_id, (inode_id % inodes_per_block) as usize * inode_size)
    }
	// 返回data block的索引
    pub fn get_data_block_id(&self, data_block_id: u32) -> u32 {
        self.data_area_start_block + data_block_id
    }
}
```

* DiskInode 和 data block的分配和回收

```rust
// easy-fs/src/efs.rs

impl EasyFileSystem {
	// 分配一个DiskInode,返回第几个inode
    pub fn alloc_inode(&mut self) -> u32 {
        self.inode_bitmap.alloc(&self.block_device).unwrap() as u32
    }

	// 分配一个data block,返回对应的块索引
    /// Return a block ID not ID in the data area.
    pub fn alloc_data(&mut self) -> u32 {
        self.data_bitmap.alloc(&self.block_device).unwrap() as u32 + self.data_area_start_block
    }

	// 释放data block
    pub fn dealloc_data(&mut self, block_id: u32) {
        get_block_cache(
            block_id as usize,
            Arc::clone(&self.block_device)
        )
        .lock()
        .modify(0, |data_block: &mut DataBlock| {
            data_block.iter_mut().for_each(|p| { *p = 0; })
        });
        self.data_bitmap.dealloc(
            &self.block_device,
            (block_id - self.data_area_start_block) as usize
        )
    }
}
```

## 5. 索引节点层

> 索引节点 `Inode` 是在内存中使用的结构，而 `DiskInode` 是存储在磁盘中的结构

```rust
// easy-fs/src/vfs.rs

pub struct Inode {
	// 对应的DiskInode的块索引
    block_id: usize,
	// 块内的偏移(字节)
    block_offset: usize,
    fs: Arc<Mutex<EasyFileSystem>>,
    block_device: Arc<dyn BlockDevice>,
}
```

* 获取/修改 `Inode` 对应的磁盘中的 `DiskInode`

```rust
// easy-fs/src/vfs.rs

impl Inode {
	// 读取对应的DiskInode并调用闭包
    fn read_disk_inode<V>(&self, f: impl FnOnce(&DiskInode) -> V) -> V {
        get_block_cache(
            self.block_id,
            Arc::clone(&self.block_device)
        ).lock().read(self.block_offset, f)
    }
	// 修改对应的DiskInode并调用闭包
    fn modify_disk_inode<V>(&self, f: impl FnOnce(&mut DiskInode) -> V) -> V {
        get_block_cache(
            self.block_id,
            Arc::clone(&self.block_device)
        ).lock().modify(self.block_offset, f)
    }
}
```

* 获取根目录的 `Inode`

```rust
// easy-fs/src/efs.rs

impl EasyFileSystem {
    pub fn root_inode(efs: &Arc<Mutex<Self>>) -> Inode {
        let block_device = Arc::clone(&efs.lock().block_device);
        // acquire efs lock temporarily
        let (block_id, block_offset) = efs.lock().get_disk_inode_pos(0);
        // release efs lock
        Inode::new(
            block_id,
            block_offset,
            Arc::clone(efs),
            block_device,
        )
    }
}

impl Inode {
    /// We should not acquire efs lock here.
    pub fn new(
        block_id: u32,
        block_offset: usize,
        fs: Arc<Mutex<EasyFileSystem>>,
        block_device: Arc<dyn BlockDevice>,
    ) -> Self {
        Self {
            block_id: block_id as usize,
            block_offset,
            fs,
            block_device,
        }
    }
```

包括 `find` 在内，所有暴露给文件系统的使用者的文件系统操作（还包括接下来将要介绍的几种），全程均需持有 `EasyFileSystem` 的互斥锁

### 1. 文件索引（只被根目录Inode调用）

由于该 easy-fs 中只有一个目录——根目录，所以只需要在根目录的 `DiskInode` 中的目录项中一个个查找即可

```rust
// easy-fs/src/vfs.rs
impl Inode {
    pub fn find(&self, name: &str) -> Option<Arc<Inode>> {
        let fs = self.fs.lock();
        self.read_disk_inode(|disk_inode| {
            self.find_inode_id(name, disk_inode)
            .map(|inode_id| {
	            // 从DiskInode的索引和偏移构建Inode
                let (block_id, block_offset) = fs.get_disk_inode_pos(inode_id);
                Arc::new(Self::new(
                    block_id,
                    block_offset,
                    self.fs.clone(),
                    self.block_device.clone(),
                ))
            })
        })
    }

	// 根据名字返回第几个DiskInode
    fn find_inode_id(
        &self,
        name: &str,
        disk_inode: &DiskInode,
    ) -> Option<u32> {
        // assert it is a directory
        assert!(disk_inode.is_dir());
        let file_count = (disk_inode.size as usize) / DIRENT_SZ;
        let mut dirent = DirEntry::empty();
        // 遍历所有的目录项
        for i in 0..file_count {
            assert_eq!(
                disk_inode.read_at(
                    DIRENT_SZ * i,
                    dirent.as_bytes_mut(),
                    &self.block_device,
                ),
                DIRENT_SZ,
            );
            if dirent.name() == name {
                return Some(dirent.inode_number() as u32);
            }
        }
        None
    }
}
```

### 2. 文件列举（只被根目录Inode调用）

* 将根目录下的所有目录项的名字以向量Vec\<String>的形式返回

```rust
// easy-fs/src/vfs.rs

impl Inode {
    pub fn ls(&self) -> Vec<String> {
        let _fs = self.fs.lock();
        self.read_disk_inode(|disk_inode| {
            let file_count = (disk_inode.size as usize) / DIRENT_SZ;
            let mut v: Vec<String> = Vec::new();
            for i in 0..file_count {
                let mut dirent = DirEntry::empty();
                assert_eq!(
                    disk_inode.read_at(
                        i * DIRENT_SZ,
                        dirent.as_bytes_mut(),
                        &self.block_device,
                    ),
                    DIRENT_SZ,
                );
                v.push(String::from(dirent.name()));
            }
            v
        })
    }
}
```

### 3. 文件创建（只被根目录Inode调用）

* 在根目录下创建一个新的文件

1. 在根目录下查找是否已经存在需要创建的文件
2. 分配一个新的 `DiskInode` 并初始化
3. 修改根目录 root_inode (包括大小、数据块里面的目录项)
4. 根据 `DiskInode` 构建新的 `Inode`

```rust
// easy-fs/src/vfs.rs

impl Inode {
    pub fn create(&self, name: &str) -> Option<Arc<Inode>> {
        let mut fs = self.fs.lock();
        // 1.判断是否存在
        if self.modify_disk_inode(|root_inode| {
            // assert it is a directory
            assert!(root_inode.is_dir());
            // has the file been created?
            self.find_inode_id(name, root_inode)
        }).is_some() {
            return None;
        }
        // 2.create a new file
        // alloc a inode with an indirect block
        let new_inode_id = fs.alloc_inode();
        // initialize inode
        let (new_inode_block_id, new_inode_block_offset)
            = fs.get_disk_inode_pos(new_inode_id);
        get_block_cache(
            new_inode_block_id as usize,
            Arc::clone(&self.block_device)
        ).lock().modify(new_inode_block_offset, |new_inode: &mut DiskInode| {
            new_inode.initialize(DiskInodeType::File);
        });
        // 3.修改根目录
        self.modify_disk_inode(|root_inode| {
            // append file in the dirent
            let file_count = (root_inode.size as usize) / DIRENT_SZ;
            let new_size = (file_count + 1) * DIRENT_SZ;
            // increase size
            self.increase_size(new_size as u32, root_inode, &mut fs);
            // write dirent
            let dirent = DirEntry::new(name, new_inode_id);
            root_inode.write_at(
                file_count * DIRENT_SZ,
                dirent.as_bytes(),
                &self.block_device,
            );
        });
		// 4.构建Inode
        let (block_id, block_offset) = fs.get_disk_inode_pos(new_inode_id);
        // return inode
        Some(Arc::new(Self::new(
            block_id,
            block_offset,
            self.fs.clone(),
            self.block_device.clone(),
        )))
        // release efs lock automatically by compiler
    }
}
```

### 4. 文件清空

* 将文件已经分配的所有数据块都清零并回收

```rust
// easy-fs/src/vfs.rs

impl Inode {
    pub fn clear(&self) {
        let mut fs = self.fs.lock();
        self.modify_disk_inode(|disk_inode| {
            let size = disk_inode.size;
            let data_blocks_dealloc = disk_inode.clear_size(&self.block_device);
            assert!(data_blocks_dealloc.len() == DiskInode::total_blocks(size) as usize);
            // 遍历所有的数据块的索引来清零并回收
            for data_block in data_blocks_dealloc.into_iter() {
                fs.dealloc_data(data_block);
            }
        });
    }
}
```

### 5. 文件读写

* 根据 `Inode` 对文件内容进行读写

```rust
// easy-fs/src/vfs.rs

impl Inode {
	// 根据偏移量从DiskInode的数据块中读取内容到buf中
    pub fn read_at(&self, offset: usize, buf: &mut [u8]) -> usize {
        let _fs = self.fs.lock();
        self.read_disk_inode(|disk_inode| {
            disk_inode.read_at(offset, buf, &self.block_device)
        })
    }
	// 将buf的内容写入到DiskInode的数据块对应的偏移量中
    pub fn write_at(&self, offset: usize, buf: &[u8]) -> usize {
        let mut fs = self.fs.lock();
        self.modify_disk_inode(|disk_inode| {
	        // 先扩容再写入
            self.increase_size((offset + buf.len()) as u32, disk_inode, &mut fs);
            disk_inode.write_at(offset, buf, &self.block_device)
        })
    }
}

impl Inode {
	// 扩容
    fn increase_size(
        &self,
        new_size: u32,
        disk_inode: &mut DiskInode,
        fs: &mut MutexGuard<EasyFileSystem>,
    ) {
        if new_size < disk_inode.size {
            return;
        }
        let blocks_needed = disk_inode.blocks_num_needed(new_size);
        let mut v: Vec<u32> = Vec::new();
        // 提供新的数据块的索引
        for _ in 0..blocks_needed {
            v.push(fs.alloc_data());
        }
        disk_inode.increase_size(new_size, v, &self.block_device);
    }
}
```

# 内核中接入 easy-fs

内核中需要有对接 easy-fs 的各种结构，自下而上可以分为五层：

1. **块设备驱动层**：针对内核所要运行在的 qemu 或 k210 平台，我们需要将平台上的块设备驱动起来并实现 `easy-fs` 所需的 `BlockDevice` Trait ，这样 `easy-fs` 才能将该块设备用作 easy-fs 镜像的载体。
2. **easy-fs 层**：站在内核的角度，只需知道它接受一个块设备 BlockDevice ，并可以在上面打开文件系统 EasyFileSystem ，进而获取 Inode 核心数据结构，进行各种文件系统操作即可。
3. **内核索引节点层**：在内核中需要将 easy-fs 提供的 Inode 进一步封装成 OSInode ，以表示进程中一个打开的常规文件。由于有很多种不同的打开方式，因此在 OSInode 中要维护一些额外的信息。
4. **文件描述符层**：常规文件对应的 OSInode 是文件的内核内部表示，因此需要为它实现 File Trait 从而能够可以将它放入到进程文件描述符表中并通过 sys_read/write 系统调用进行读写。
5. **系统调用层**：由于引入了常规文件这种文件类型，导致一些系统调用以及相关的内核机制需要进行一定的修改。

## 1. 块设备驱动层

> 内核访问的块设备实例 `BLOCK_DEVICE` 可以是qemu或k210,它们都实现了 `easy-fs` 要求的 `BlockDevice` Trait
>  qemu 上，我们使用 `VirtIOBlock` 访问 VirtIO 块设备；
>  而在 k210 上，我们使用 `SDCardWrapper` 来访问插入 k210 开发板上真实的 microSD 卡

```rust
// os/drivers/block/mod.rs

#[cfg(feature = "board_qemu")]
type BlockDeviceImpl = virtio_blk::VirtIOBlock;

#[cfg(feature = "board_k210")]
type BlockDeviceImpl = sdcard::SDCardWrapper;

lazy_static! {
    pub static ref BLOCK_DEVICE: Arc<dyn BlockDevice> = Arc::new(BlockDeviceImpl::new());
}
```

### 1. Qemu 模拟器

1. 在启动Qemu模拟器的时候通过配置参数**添加一块VirtIO块设备**（VirtIO block device是一种在虚拟机中使用的块存储设备）

```rust
# os/Makefile

FS_IMG := ../user/target/$(TARGET)/$(MODE)/fs.img

run-inner: build
ifeq ($(BOARD),qemu)
    @qemu-system-riscv64 \
        -machine virt \
        -nographic \
        -bios $(BOOTLOADER) \
        -device loader,file=$(KERNEL_BIN),addr=$(KERNEL_ENTRY_PA) \
        // 添加一块虚拟硬盘,为之前用easy-fs-fuse打包的包含应用ELF的镜像x0(其实就是一个普通文件被当作磁盘在各个块写入了相关内容)
        -drive file=$(FS_IMG),if=none,format=raw,id=x0 \
        // 将硬盘x0作为VirtIO总线中的一个块设备接入到虚拟机中。virtio-mmio-bus.0表示VirtIO总线通过MMIO进行控制,该设备在总线编号为0
        -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0
```

2. **内存映射 I/O** (MMIO, Memory-Mapped I/O) 指的是外设的设备寄存器可以通过特定的物理内存地址来访问。Qemu for RISC-V 64 平台的  VirtIO 外设总线的 MMIO 物理地址区间为从 0x10001000 开头的 4KiB

```rust
// os/src/config.rs
// 为了能够在内核中访问 VirtIO 外设总线，我们就必须在内核地址空间中对特定内存区域提前进行映射
#[cfg(feature = "board_qemu")]
pub const MMIO: &[(usize, usize)] = &[
    (0x10001000, 0x1000),
];

// os/src/mm/memory_set.rs
use crate::config::MMIO;

impl MemorySet {
    /// Without kernel stacks.
    pub fn new_kernel() -> Self {
        ...
        println!("mapping memory-mapped registers");
        for pair in MMIO {
            memory_set.push(MapArea::new(
                (*pair).0.into(),
                ((*pair).0 + (*pair).1).into(),
                MapType::Identical,
                MapPermission::R | MapPermission::W,
            ), None);
        }
        memory_set
    }
}
```

3. 将 `virtio-drivers` crate 提供的 VirtIO 块设备抽象 `VirtIOBlk` 包装为我们自己的 `VirtIOBlock`，并在**设置好VirtIO块设备的一组寄存器**后就可以访问了

```rust
// os/src/drivers/block/virtio_blk.rs

use virtio_drivers::{VirtIOBlk, VirtIOHeader};
const VIRTIO0: usize = 0x10001000;

pub struct VirtIOBlock(Mutex<VirtIOBlk<'static>>);

impl VirtIOBlock {
    pub fn new() -> Self {
        Self(Mutex::new(VirtIOBlk::new(
	        // 将对应的MMIO的起始地址作为一组设备寄存器来交互
            unsafe { &mut *(VIRTIO0 as *mut VirtIOHeader) }
        ).unwrap()))
    }
}

// 实现Trait(该crate里面已经实现)
impl BlockDevice for VirtIOBlock {
    fn read_block(&self, block_id: usize, buf: &mut [u8]) {
        self.0.lock().read_block(block_id, buf).expect("Error when reading VirtIOBlk");
    }
    fn write_block(&self, block_id: usize, buf: &[u8]) {
        self.0.lock().write_block(block_id, buf).expect("Error when writing VirtIOBlk");
    }
}
```

4. 在 VirtIO 架构下，需要在公共区域中**放置一种叫做 VirtQueue 的环形队列**，CPU 可以向此环形队列中向 VirtIO 设备提交请求，也可以从队列中取得请求的结果，我们需要实现其涉及到的物理内存的分配和回收

```rust
// https://github.com/rcore-os/virtio-drivers/blob/master/src/hal.rs#L57

extern "C" {
	// 分配&回收连续的物理页帧
    fn virtio_dma_alloc(pages: usize) -> PhysAddr;
    fn virtio_dma_dealloc(paddr: PhysAddr, pages: usize) -> i32;
	// 物理与虚拟地址之间的转化
    fn virtio_phys_to_virt(paddr: PhysAddr) -> VirtAddr;
    fn virtio_virt_to_phys(vaddr: VirtAddr) -> PhysAddr;
}
```

```rust
// os/src/drivers/block/virtio_blk.rs
// 保存所有物理页帧的所有权,防止提前回收
lazy_static! {
    static ref QUEUE_FRAMES: Mutex<Vec<FrameTracker>> = Mutex::new(Vec::new());
}

#[no_mangle]
pub extern "C" fn virtio_dma_alloc(pages: usize) -> PhysAddr {
    let mut ppn_base = PhysPageNum(0);
    for i in 0..pages {
        let frame = frame_alloc().unwrap();
        if i == 0 { ppn_base = frame.ppn; }
        assert_eq!(frame.ppn.0, ppn_base.0 + i);
        QUEUE_FRAMES.lock().push(frame);
    }
    ppn_base.into()
}

#[no_mangle]
pub extern "C" fn virtio_dma_dealloc(pa: PhysAddr, pages: usize) -> i32 {
    let mut ppn_base: PhysPageNum = pa.into();
    for _ in 0..pages {
        frame_dealloc(ppn_base);
        ppn_base.step();
    }
    0
}

#[no_mangle]
pub extern "C" fn virtio_phys_to_virt(paddr: PhysAddr) -> VirtAddr {
    VirtAddr(paddr.0)
}

#[no_mangle]
pub extern "C" fn virtio_virt_to_phys(vaddr: VirtAddr) -> PhysAddr {
PageTable::from_token(kernel_token()).translate_va(vaddr).unwrap()
}
```

在 VirtIO 队列中，VirtIO 驱动程序会创建和管理一个或多个**队列描述符表**（Queue Descriptor Table），每个描述符对应一个**队列条目**（Queue Entry），描述该条目的元数据和数据信息。VirtIO 驱动程序将需要发送给 VirtIO 设备的数据写入队列条目对应的数据缓冲区中，并将描述该数据缓冲区的描述符写入队列条目对应的描述符表中。同时，VirtIO 驱动程序还会向 VirtIO 设备寄存器中的可用环（Available Ring）写入一个或多个可用的队列条目索引，告知 VirtIO 设备可以读取该索引对应的队列条目。

在 VirtIO 设备中，当收到 VirtIO 驱动程序写入的可用环索引后，会从队列描述符表中读取对应的描述符，并读取描述符中的元数据和数据缓冲区地址。然后，VirtIO 设备会使用这些元数据和数据缓冲区地址，对 VirtIO 设备进行读写操作，并将处理结果写入 VirtIO 驱动程序创建的已用环（Used Ring）中的相应队列条目。同时，VirtIO 设备会向 VirtIO 设备寄存器中的可用环写入一个或多个可用的队列条目索引，告知 VirtIO 驱动程序已经处理完这些队列条目。

## 2. 内核索引节点层

进一步封装 `Inode` 为 `OSInode`，增加读写权限和当前的偏移量

```rust
// os/src/fs/inode.rs

pub struct OSInode {
    readable: bool,
    writable: bool,
    inner: Mutex<OSInodeInner>,
}

pub struct OSInodeInner {
    offset: usize,
    inode: Arc<Inode>,
}

impl OSInode {
    pub fn new(
        readable: bool,
        writable: bool,
        inode: Arc<Inode>,
    ) -> Self {
        Self {
            readable,
            writable,
            inner: Mutex::new(OSInodeInner {
                offset: 0,
                inode,
            }),
        }
    }
}
```

## 3. 文件描述符层

每个进程里面都有一个**文件描述符表**，其中每个**文件描述符**(表中下标)都指向一个特定读写属性的 I/O资源

```rust
// os/src/task/task.rs

pub struct TaskControlBlockInner {
    pub trap_cx_ppn: PhysPageNum,
    pub base_size: usize,
    pub task_cx_ptr: usize,
    pub task_status: TaskStatus,
    pub memory_set: MemorySet,
    pub parent: Option<Weak<TaskControlBlock>>,
    pub children: Vec<Arc<TaskControlBlock>>,
    pub exit_code: i32,
    // 文件描述符表
    pub fd_table: Vec<Option<Arc<dyn File + Send + Sync>>>,
}
```

为 `OSInode` 添加 `File Trait` ，之后就可以放入到文件描述符表中

```rust
// os/src/fs/inode.rs

impl File for OSInode {
    fn readable(&self) -> bool { self.readable }
    fn writable(&self) -> bool { self.writable }
    fn read(&self, mut buf: UserBuffer) -> usize {
        let mut inner = self.inner.lock();
        let mut total_read_size = 0usize;
        for slice in buf.buffers.iter_mut() {
            let read_size = inner.inode.read_at(inner.offset, *slice);
            if read_size == 0 {
                break;
            }
            inner.offset += read_size;
            total_read_size += read_size;
        }
        total_read_size
    }
    fn write(&self, buf: UserBuffer) -> usize {
        let mut inner = self.inner.lock();
        let mut total_write_size = 0usize;
        for slice in buf.buffers.iter() {
            let write_size = inner.inode.write_at(inner.offset, *slice);
            assert_eq!(write_size, slice.len());
            inner.offset += write_size;
            total_write_size += write_size;
        }
        total_write_size
    }
}
```
## 4. 系统调用层

### 1. 文件系统的初始化

1. 打开块设备：前面已经打开并访问装载有 easy-fs 文件系统镜像的块设备 `BLOCK_DEVICE`
2. 从块设备 `BLOCK_DEVICE` 中打开文件系统
3. 从文件系统中获取根目录的 `inode`

```rust
// os/src/fs/inode.rs

lazy_static! {
    pub static ref ROOT_INODE: Arc<Inode> = {
	    // 2.打开文件系统
        let efs = EasyFileSystem::open(BLOCK_DEVICE.clone());
        // 3.获取根目录inode
        Arc::new(EasyFileSystem::root_inode(&efs))
    };
}
```

之后就可以直接用 `ROOT_INODE` 进行 easy-fs 相关操作

```rust
// os/src/fs/inode.rs

pub fn list_apps() {
    println!("/**** APPS ****");
    for app in ROOT_INODE.ls() {
        println!("{}", app);
    }
    println!("**************/")
}
```

### 2. 打开和关闭文件

打开文件的标志 `OpenFlags`

```rust
// os/src/fs/inode.rs
bitflags! {
    pub struct OpenFlags: u32 {
        const RDONLY = 0;
        const WRONLY = 1 << 0;
        const RDWR = 1 << 1;
        const CREATE = 1 << 9;
        const TRUNC = 1 << 10;
    }
}

impl OpenFlags {
    /// Do not check validity for simplicity
    /// Return (readable, writable)
    pub fn read_write(&self) -> (bool, bool) {
        if self.is_empty() {
            (true, false)
        } else if self.contains(Self::WRONLY) {
            (false, true)
        } else {
            (true, true)
        }
    }
}
```

`open_file()` 根据文件名在磁盘镜像中打开根目录下的文件, 返回指向 `OsInode` 的指针

```rust
// os/src/fs/inode.rs

pub fn open_file(name: &str, flags: OpenFlags) -> Option<Arc<OSInode>> {
    let (readable, writable) = flags.read_write();
    if flags.contains(OpenFlags::CREATE) {
        if let Some(inode) = ROOT_INODE.find(name) {
            // clear size
            inode.clear();
            Some(Arc::new(OSInode::new(
                readable,
                writable,
                inode,
            )))
        } else {
            // create file
            ROOT_INODE.create(name)
                .map(|inode| {
                    Arc::new(OSInode::new(
                        readable,
                        writable,
                        inode,
                    ))
                })
        }
    } else {
        ROOT_INODE.find(name)
            .map(|inode| {
                if flags.contains(OpenFlags::TRUNC) {
                    inode.clear();
                }
                Arc::new(OSInode::new(
                    readable,
                    writable,
                    inode
                ))
            })
    }
}
```

`sys_open()` 将 `open_file()` 得到的Arc\<OSInode>放入到当前 task 的文件描述符表 fd_table 中

```rust
// os/src/syscall/fs.rs

pub fn sys_open(path: *const u8, flags: u32) -> isize {
    let task = current_task().unwrap();
    let token = current_user_token();
    let path = translated_str(token, path);
    if let Some(inode) = open_file(
        path.as_str(),
        OpenFlags::from_bits(flags).unwrap()
    ) {
        let mut inner = task.acquire_inner_lock();
        let fd = inner.alloc_fd();
        // 将该OSInode加入到task的文件描述符表中，拥有所有权
        inner.fd_table[fd] = Some(inode);
        fd as isize
    } else {
        -1
    }
}
```

`sys_close()` 将OSInode从文件描述符表中移除 (减少引用计数)

```rust
// os/src/syscall/fs.rs
pub fn sys_close(fd: usize) -> isize {
    let task = current_task().unwrap();
    let mut inner = task.inner_exclusive_access();
    if fd >= inner.fd_table.len() {
        return -1;
    }
    if inner.fd_table[fd].is_none() {
        return -1;
    }
    inner.fd_table[fd].take();
    0
}
```

### 3. 加载并执行应用

`sys_exec()` 从文件系统中根据根目录找到对应ELF格式文件加载执行

```rust
// os/src/syscall/process.rs
pub fn sys_exec(path: *const u8) -> isize {
    let token = current_user_token();
    let path = translated_str(token, path);
    // 打开对应文件
    if let Some(app_inode) = open_file(path.as_str(), OpenFlags::RDONLY) {
		// 将数据全部读出
        let all_data = app_inode.read_all();
        let task = current_task().unwrap();
        // 根据数据加载并执行文件
        task.exec(all_data.as_slice());
        0
    } else {
        -1
    }
}

// -----------------读出文件内容--------------------------
// os/src/fs/inode.rs
impl OSInode {
    pub fn read_all(&self) -> Vec<u8> {
        let mut inner = self.inner.lock();
        let mut buffer = [0u8; 512];
        let mut v: Vec<u8> = Vec::new();
        loop {
            let len = inner.inode.read_at(inner.offset, &mut buffer);
            if len == 0 {
                break;
            }
            inner.offset += len;
            v.extend_from_slice(&buffer[..len]);
        }
        v
    }
}
```

初始进程的创建和执行

```rust
// os/src/task/mod.rs

lazy_static! {
    pub static ref INITPROC: Arc<TaskControlBlock> = Arc::new({
        let inode = open_file("initproc", OpenFlags::RDONLY).unwrap();
        let v = inode.read_all();
        TaskControlBlock::new(v.as_slice())
    });
}
```

### 4. 读写文件

通过文件描述符 fd 在描述符表中找到对应的 OSInode，然后read/write

```rust
// os/src/syscall/fs.rs

pub fn sys_write(fd: usize, buf: *const u8, len: usize) -> isize {
    let token = current_user_token();
    let task = current_task().unwrap();
    let inner = task.inner_exclusive_access();
    if fd >= inner.fd_table.len() {
        return -1;
    }
    if let Some(file) = &inner.fd_table[fd] {
        let file = file.clone();
        // release current task TCB manually to avoid multi-borrow
        drop(inner);
        file.write(
            UserBuffer::new(translated_byte_buffer(token, buf, len))
        ) as isize
    } else {
        -1
    }
}

pub fn sys_read(fd: usize, buf: *const u8, len: usize) -> isize {
    let token = current_user_token();
    let task = current_task().unwrap();
    let inner = task.inner_exclusive_access();
    if fd >= inner.fd_table.len() {
        return -1;
    }
    if let Some(file) = &inner.fd_table[fd] {
        let file = file.clone();
        // release current task TCB manually to avoid multi-borrow
        drop(inner);
        file.read(
            UserBuffer::new(translated_byte_buffer(token, buf, len))
        ) as isize
    } else {
        -1
    }
}
```