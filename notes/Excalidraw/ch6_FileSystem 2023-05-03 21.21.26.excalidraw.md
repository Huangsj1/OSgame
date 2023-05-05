---

excalidraw-plugin: parsed
tags: [excalidraw]

---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠==


# Text Elements
磁盘的块 ^ix0UZVWf

Super block ^rXbM9Kbt

Inode bitmap ^c7Tnff8E

Inode block ^C78tDg6f

data bitmap ^ye0FZexi

data block ^N3Lc5XH0

BlockCache结构 ^8aZfoMY5

cache: [u8; BLOCK_SZ] ^2z7bI9s3

block_id: usize ^EAc47WF4

block_device: Arc<dyn BlockDevice> ^BM2FlVCu

modify: bool ^Q5GwHyr6

cache中存储着一个块的内容 ^HHRjQQw9

BlockCacheManager结构 ^RI0wIKyx

queue: VecDeque<(usize, Arc<Mutex<BlockCache>>)> ^T7iD4Gko

最多同时存在16个BlockCache ^I2aSUIVJ

BLOCK_CACHE_MANAGER ^A3Z00Qed

     get_block_cache()
返回对应block_id的BlockCache
     (如果不存在就构建) ^mjG14DpA

sync()函数可以根据是否modify来将cache内容写回磁盘
            drop()的时候会调用sync() ^wQEmJhLJ

trait ^DJEya7QP

BlockDevice ^Yi3tmjrV

BlockDevice::read_block(block_id, buf)
   读取磁盘block_id块的内容到buf中 ^0r7YGDLt

BlockDevice::write_block(block_id, buf)
    将buf内容写道磁盘block_id块中 ^sSA6WbAy

使用该文件系统需要实现这个trait ^mNqdb9bR

SuperBlock结构 ^ggKcUk3Z

// magic 用于验证文件系统是否合法
magic: u32,
// 文件系统总块数
pub total_blocks: u32,
// 4个连续区域各占多少块
pub inode_bitmap_blocks: u32,
pub inode_area_blocks: u32,
pub data_bitmap_blocks: u32,
pub data_area_blocks: u32, ^rDHVuvdi

处在Super block起始位置 ^k9qyI3C2

type BitmapBlock ^Dstr4OZ1

每一个bitmap块都可看成这样的类型 ^0UGVj80g

[u64, 64] ^Fr9pTQuq

DiskInode结构 ^EVtTrhwM

// 占的字节数
pub size: u32,
// 直接索引的数组(元素为索引号)
pub direct: [u32; INODE_DIRECT_COUNT],
// 一级间接索引
pub indirect1: u32,
// 二级间接索引
pub indirect2: u32,
// 文件/目录类型
type_: DiskInodeType, ^wpUnIheX

1.DataBlock字节序列 ^YACGxxAP

u8 ^2tNBEhsL

2.DiskInode间接索引 ^SxsmhZtZ

u32 ^slWisgG9

3.目录项 ^iiCCn66t

DirEntry结构 ^tWOUyUQ0

name: [u8: 28] ^VhtL1H1g

inode_num: u32 ^aG6DElsz

数据块有3种不同的结构 ^zTHFicMg

EasyFileSystem结构 ^C0LmvaAS

pub block_device: Arc<dyn BlockDevice>, ^1ZEs3VyZ

pub inode_bitmap: Bitmap, ^mmzFhevt

pub data_bitmap: Bitmap, ^dYkGTzTi

inode_area_start_block: u32, ^a5jf5qsc

data_area_start_block: u32, ^Dss86BZN

open() 创建 ^KFLiBZhh

alloc_inode() ^6mZeVH09

alloc_data() ^zhYtsvvY

dealloc_data() ^zcVHrb8Z

Inode结构 ^VvruwVtq

block_id: u32, ^2JpMLpdo

block_offset: usize, ^z4IHblwd

fs: Arc<Mutex<EasyFileSystem>>, ^M5wBSjlB

block_device: Arc<dyn BlockDevice>, ^yfUcplRo

             与一个DiskInode相关联
通过block_id和block_offset找到对应的DiskInode ^NIiqirll

%%
# Drawing
```json
{
	"type": "excalidraw",
	"version": 2,
	"source": "https://github.com/zsviczian/obsidian-excalidraw-plugin/releases/tag/1.8.18",
	"elements": [
		{
			"type": "text",
			"version": 424,
			"versionNonce": 1385112787,
			"isDeleted": false,
			"id": "ix0UZVWf",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -663.3758171995379,
			"y": 122.03391220341439,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 184.5999755859375,
			"height": 55.199999999999996,
			"seed": 1298409725,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252029,
			"link": null,
			"locked": false,
			"fontSize": 46.152088899873654,
			"fontFamily": 1,
			"text": "磁盘的块",
			"rawText": "磁盘的块",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "磁盘的块"
		},
		{
			"type": "text",
			"version": 266,
			"versionNonce": 1023444445,
			"isDeleted": false,
			"id": "rXbM9Kbt",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -439.3223404805039,
			"y": 130.92251901930143,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 162.5441131591797,
			"height": 34.8,
			"seed": 1629274099,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252029,
			"link": null,
			"locked": false,
			"fontSize": 29.773526217996434,
			"fontFamily": 1,
			"text": "Super block",
			"rawText": "Super block",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Super block"
		},
		{
			"type": "text",
			"version": 91,
			"versionNonce": 1972966003,
			"isDeleted": false,
			"id": "c7Tnff8E",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -230.6790647847966,
			"y": 129.2514303494797,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 200.73191833496094,
			"height": 38.4,
			"seed": 1946631037,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "l0MNK0fnXAGLDSuk5c5lO",
					"type": "arrow"
				}
			],
			"updated": 1683126252029,
			"link": null,
			"locked": false,
			"fontSize": 32.030372361573875,
			"fontFamily": 1,
			"text": "Inode bitmap",
			"rawText": "Inode bitmap",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Inode bitmap"
		},
		{
			"type": "text",
			"version": 266,
			"versionNonce": 437109309,
			"isDeleted": false,
			"id": "C78tDg6f",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 32.95244174408879,
			"y": 127.78192698859249,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 204.4511260986328,
			"height": 44.4,
			"seed": 1111523891,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "l0MNK0fnXAGLDSuk5c5lO",
					"type": "arrow"
				},
				{
					"id": "tqZxdk74HTbsV07eGSMRD",
					"type": "arrow"
				}
			],
			"updated": 1683126252029,
			"link": null,
			"locked": false,
			"fontSize": 37.20502768596072,
			"fontFamily": 1,
			"text": "Inode block",
			"rawText": "Inode block",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Inode block"
		},
		{
			"type": "text",
			"version": 109,
			"versionNonce": 676613139,
			"isDeleted": false,
			"id": "ye0FZexi",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 326.2319066608581,
			"y": 130.93848426325624,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 202.47279357910156,
			"height": 39.6,
			"seed": 1424523475,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "jGYzGi8J6UtjJFtuEs7v9",
					"type": "arrow"
				}
			],
			"updated": 1683126252029,
			"link": null,
			"locked": false,
			"fontSize": 33.45766202093839,
			"fontFamily": 1,
			"text": "data bitmap",
			"rawText": "data bitmap",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "data bitmap"
		},
		{
			"type": "text",
			"version": 165,
			"versionNonce": 1861513885,
			"isDeleted": false,
			"id": "N3Lc5XH0",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 665.690898506115,
			"y": 127.34682588281402,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 204.2548370361328,
			"height": 45.6,
			"seed": 260770675,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "tqZxdk74HTbsV07eGSMRD",
					"type": "arrow"
				},
				{
					"id": "jGYzGi8J6UtjJFtuEs7v9",
					"type": "arrow"
				}
			],
			"updated": 1683126252029,
			"link": null,
			"locked": false,
			"fontSize": 38.6767169811115,
			"fontFamily": 1,
			"text": "data block",
			"rawText": "data block",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "data block"
		},
		{
			"type": "rectangle",
			"version": 127,
			"versionNonce": 2125943219,
			"isDeleted": false,
			"id": "SY5w45lSVZNMCJuWJCk7A",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -462.8959138114885,
			"y": 102.51384832342632,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 1447.058565177823,
			"height": 94.37334434109027,
			"seed": 1048548531,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "l0MNK0fnXAGLDSuk5c5lO",
					"type": "arrow"
				},
				{
					"id": "VpenBkf1c0LB8YCUIvEjD",
					"type": "arrow"
				}
			],
			"updated": 1683126252029,
			"link": null,
			"locked": false
		},
		{
			"type": "line",
			"version": 71,
			"versionNonce": 1751521021,
			"isDeleted": false,
			"id": "aZX0ExY2Nb7JnZLFMrq5I",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -259.34547229917047,
			"y": 100.66338695233964,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0,
			"height": 94.37334434109027,
			"seed": 601948669,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252029,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": null,
			"points": [
				[
					0,
					0
				],
				[
					0,
					94.37334434109027
				]
			]
		},
		{
			"type": "line",
			"version": 59,
			"versionNonce": 1239982931,
			"isDeleted": false,
			"id": "uz4pxFHK-WNah-QPYi3lG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -10.458819987393952,
			"y": 102.51384832342632,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.9251997548213922,
			"height": 88.82196022783012,
			"seed": 1330724701,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252030,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": null,
			"points": [
				[
					0,
					0
				],
				[
					-0.9251997548213922,
					88.82196022783012
				]
			]
		},
		{
			"type": "line",
			"version": 58,
			"versionNonce": 551825245,
			"isDeleted": false,
			"id": "cm1PCobXveVESepVGl7P8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 297.64253433771324,
			"y": 102.51384832342632,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.9251997548212785,
			"height": 84.19583773083536,
			"seed": 1879406291,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252030,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": null,
			"points": [
				[
					0,
					0
				],
				[
					0.9251997548212785,
					84.19583773083536
				]
			]
		},
		{
			"type": "line",
			"version": 69,
			"versionNonce": 1916715251,
			"isDeleted": false,
			"id": "EGHw15Tgmi4fCbPFeAeFy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 547.454386404311,
			"y": 100.66338695233964,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.9253234777092985,
			"height": 95.29860595735556,
			"seed": 527588445,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252030,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": null,
			"points": [
				[
					0,
					0
				],
				[
					0.9253234777092985,
					95.29860595735556
				]
			]
		},
		{
			"type": "arrow",
			"version": 261,
			"versionNonce": 1723874237,
			"isDeleted": false,
			"id": "l0MNK0fnXAGLDSuk5c5lO",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -116.7945305628391,
			"y": 119.16793880176277,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 265.51918760188687,
			"height": 69.3922704850295,
			"seed": 1324933523,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252030,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "c7Tnff8E",
				"focus": -0.07330742707103387,
				"gap": 10.083491547716946
			},
			"endBinding": {
				"elementId": "C78tDg6f",
				"focus": 0.2868713756342315,
				"gap": 4.913003583212372
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					44.34540929765018,
					-59.21457829044283
				],
				[
					212.73708475932102,
					-65.69128588141217
				],
				[
					265.51918760188687,
					3.7009846036173286
				]
			]
		},
		{
			"type": "arrow",
			"version": 510,
			"versionNonce": 289417875,
			"isDeleted": false,
			"id": "tqZxdk74HTbsV07eGSMRD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 154.9730581497102,
			"y": 120.09320041802805,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 699.2620574532975,
			"height": 92.9112388703392,
			"seed": 2037643453,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252030,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "C78tDg6f",
				"focus": -0.23111012391641605,
				"gap": 7.688726570564427
			},
			"endBinding": {
				"elementId": "N3Lc5XH0",
				"focus": 1.1013078382936612,
				"gap": 12.804947716602214
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					105.90899197552591,
					-60.38872347051954
				],
				[
					482.3680058035537,
					-92.9112388703392
				],
				[
					699.2620574532975,
					-5.551322251816231
				]
			]
		},
		{
			"type": "arrow",
			"version": 296,
			"versionNonce": 1109113885,
			"isDeleted": false,
			"id": "jGYzGi8J6UtjJFtuEs7v9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 425.18119268730646,
			"y": 126.56978428610961,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 328.37466113426603,
			"height": 59.21464015188678,
			"seed": 1873232627,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252030,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ye0FZexi",
				"focus": -0.2716108073201811,
				"gap": 4.368699977146619
			},
			"endBinding": {
				"elementId": "N3Lc5XH0",
				"focus": 0.2641926078889303,
				"gap": 3.552702722612551
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					73.23599831392892,
					-54.588455793448134
				],
				[
					213.87106251651846,
					-59.21464015188678
				],
				[
					328.37466113426603,
					-2.7756611259081296
				]
			]
		},
		{
			"type": "rectangle",
			"version": 75,
			"versionNonce": 1268998195,
			"isDeleted": false,
			"id": "TMdk-HDM79zIGWMVIEZDJ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -459.1949910693151,
			"y": 104.36430969451305,
			"strokeColor": "#000000",
			"backgroundColor": "#7950f2",
			"width": 200.77478038640993,
			"height": 93.44814458626882,
			"seed": 779793021,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "N0IGR2eYHJTaogkgqREpn",
					"type": "arrow"
				}
			],
			"updated": 1683126252030,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 75,
			"versionNonce": 988295293,
			"isDeleted": false,
			"id": "SWz7JnC6YbBFyWYMCkxTp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -258.4202106829052,
			"y": 104.36430969451305,
			"strokeColor": "#000000",
			"backgroundColor": "#fa5252",
			"width": 244.26046795333787,
			"height": 93.44814458626882,
			"seed": 14472125,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683126252030,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 85,
			"versionNonce": 1800331731,
			"isDeleted": false,
			"id": "GEb6GLf4NKCug5IO5ViEv",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -12.309281358480632,
			"y": 102.51384832342632,
			"strokeColor": "#000000",
			"backgroundColor": "#fa5252",
			"width": 307.1760927088418,
			"height": 89.74722184409552,
			"seed": 1810813,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "RgQi4v5rOLYIi0d27TMOY",
					"type": "arrow"
				},
				{
					"id": "W8_Saw0eqoJX_jobdDjiz",
					"type": "arrow"
				},
				{
					"id": "QfZOovxB3nnRP3it8JKxy",
					"type": "arrow"
				}
			],
			"updated": 1683126252030,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 75,
			"versionNonce": 1592203485,
			"isDeleted": false,
			"id": "b9APmgRwq2DK-9RkLAwYd",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 300.4182573250653,
			"y": 103.43904807824771,
			"strokeColor": "#000000",
			"backgroundColor": "#228be6",
			"width": 246.11092932442443,
			"height": 91.5976832151822,
			"seed": 1156598877,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "bDgfq8KVpNXdTRqzoOET8",
					"type": "arrow"
				}
			],
			"updated": 1683126252030,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 72,
			"versionNonce": 962146163,
			"isDeleted": false,
			"id": "OyK9r24bPJpPReEeaXkRT",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 546.5291866494897,
			"y": 103.43904807824771,
			"strokeColor": "#000000",
			"backgroundColor": "#228be6",
			"width": 436.70826496202335,
			"height": 92.52294483144749,
			"seed": 300995005,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "UInlKBgwk8f3iZ-vyNdlT",
					"type": "arrow"
				},
				{
					"id": "g1NpUlR_UguMD6KW2TEtC",
					"type": "arrow"
				},
				{
					"id": "2vXeSPq6NzrJ2AKDuyVUr",
					"type": "arrow"
				},
				{
					"id": "nV9aoRPFpmOAKgCg-2cIY",
					"type": "arrow"
				}
			],
			"updated": 1683126252030,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 1125,
			"versionNonce": 1202383773,
			"isDeleted": false,
			"id": "Iz1x0HhHt7VCspfg_aVFp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 525.366074654147,
			"y": -788.5479817911776,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 419.59776523864025,
			"height": 369.9031663694851,
			"seed": 1518707795,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "jFX5ImuQj2wfh8aiFUdIg",
					"type": "arrow"
				}
			],
			"updated": 1683126603429,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 370,
			"versionNonce": 904228861,
			"isDeleted": false,
			"id": "8aZfoMY5",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 602.1224322023082,
			"y": -838.1212087562068,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 252.0443572998047,
			"height": 40.8,
			"seed": 667449853,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"fontSize": 34.12918374552763,
			"fontFamily": 1,
			"text": "BlockCache结构",
			"rawText": "BlockCache结构",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "BlockCache结构"
		},
		{
			"type": "text",
			"version": 667,
			"versionNonce": 426203731,
			"isDeleted": false,
			"id": "2z7bI9s3",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 546.7234779350957,
			"y": -751.8253993406865,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 374.9596862792969,
			"height": 37.199999999999996,
			"seed": 973917459,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "RgQi4v5rOLYIi0d27TMOY",
					"type": "arrow"
				},
				{
					"id": "VpenBkf1c0LB8YCUIvEjD",
					"type": "arrow"
				}
			],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"fontSize": 31.055883803849497,
			"fontFamily": 1,
			"text": "cache: [u8; BLOCK_SZ]",
			"rawText": "cache: [u8; BLOCK_SZ]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "cache: [u8; BLOCK_SZ]"
		},
		{
			"type": "text",
			"version": 500,
			"versionNonce": 486364349,
			"isDeleted": false,
			"id": "EAc47WF4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 598.4319147226893,
			"y": -622.9269178289035,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 244.11041259765625,
			"height": 40.8,
			"seed": 2116489405,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"fontSize": 34.294654031473044,
			"fontFamily": 1,
			"text": "block_id: usize",
			"rawText": "block_id: usize",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "block_id: usize"
		},
		{
			"type": "text",
			"version": 501,
			"versionNonce": 1002250643,
			"isDeleted": false,
			"id": "BM2FlVCu",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 540.9594248271383,
			"y": -529.7573183912285,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 388.00701904296875,
			"height": 26.4,
			"seed": 1934801021,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"fontSize": 22.667746185448287,
			"fontFamily": 1,
			"text": "block_device: Arc<dyn BlockDevice>",
			"rawText": "block_device: Arc<dyn BlockDevice>",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "block_device: Arc<dyn BlockDevice>"
		},
		{
			"type": "text",
			"version": 478,
			"versionNonce": 1129166109,
			"isDeleted": false,
			"id": "Q5GwHyr6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 644.527014412973,
			"y": -470.8576366086027,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 157.37074279785156,
			"height": 33.6,
			"seed": 1822341907,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"fontSize": 28.354039980554862,
			"fontFamily": 1,
			"text": "modify: bool",
			"rawText": "modify: bool",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "modify: bool"
		},
		{
			"type": "line",
			"version": 398,
			"versionNonce": 694777651,
			"isDeleted": false,
			"id": "TEkD_m7uTnwRoGitje1Y5",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 525.1263779772369,
			"y": -651.8403506966577,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 416.505658732822,
			"height": 1.4901836539027045,
			"seed": 861989149,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": null,
			"points": [
				[
					0,
					0
				],
				[
					416.505658732822,
					1.4901836539027045
				]
			]
		},
		{
			"type": "line",
			"version": 383,
			"versionNonce": 359860605,
			"isDeleted": false,
			"id": "y-7AWzvgLo9_bBedtDdz7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 527.3615787322033,
			"y": -554.233495726434,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 409.79985719888896,
			"height": 2.2353003894833137,
			"seed": 386133619,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": null,
			"points": [
				[
					0,
					0
				],
				[
					409.79985719888896,
					2.2353003894833137
				]
			]
		},
		{
			"type": "line",
			"version": 375,
			"versionNonce": 1983518931,
			"isDeleted": false,
			"id": "I9xcAGyHhLD73CUL7JZTR",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 527.3615787322033,
			"y": -487.17533093532836,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 409.05479028056675,
			"height": 2.9803673078055226,
			"seed": 1055844179,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": null,
			"points": [
				[
					0,
					0
				],
				[
					409.05479028056675,
					2.9803673078055226
				]
			]
		},
		{
			"type": "text",
			"version": 229,
			"versionNonce": 883872221,
			"isDeleted": false,
			"id": "HHRjQQw9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 587.1163845103921,
			"y": -697.8041899552481,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 290.08770751953125,
			"height": 26.4,
			"seed": 1395004147,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"fontSize": 22.8262490080607,
			"fontFamily": 1,
			"text": "cache中存储着一个块的内容",
			"rawText": "cache中存储着一个块的内容",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "cache中存储着一个块的内容"
		},
		{
			"type": "rectangle",
			"version": 435,
			"versionNonce": 1338345075,
			"isDeleted": false,
			"id": "Gpve6dmqbx9V8YgWIjno3",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -559.4573221636435,
			"y": -642.7496189159103,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 735.106204864298,
			"height": 130.57247741488445,
			"seed": 2140966717,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 298,
			"versionNonce": 504532541,
			"isDeleted": false,
			"id": "RI0wIKyx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -388.701598597287,
			"y": -728.8499914025903,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 390.5169677734375,
			"height": 40.8,
			"seed": 635869171,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"fontSize": 34.16290801693591,
			"fontFamily": 1,
			"text": "BlockCacheManager结构",
			"rawText": "BlockCacheManager结构",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "BlockCacheManager结构"
		},
		{
			"type": "text",
			"version": 431,
			"versionNonce": 1825718291,
			"isDeleted": false,
			"id": "T7iD4Gko",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -540.8185911915599,
			"y": -603.0206814506402,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 702.1364135742188,
			"height": 32.4,
			"seed": 2013320243,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "jFX5ImuQj2wfh8aiFUdIg",
					"type": "arrow"
				}
			],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"fontSize": 27.772161151592016,
			"fontFamily": 1,
			"text": "queue: VecDeque<(usize, Arc<Mutex<BlockCache>>)>",
			"rawText": "queue: VecDeque<(usize, Arc<Mutex<BlockCache>>)>",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "queue: VecDeque<(usize, Arc<Mutex<BlockCache>>)>"
		},
		{
			"type": "text",
			"version": 379,
			"versionNonce": 1733739955,
			"isDeleted": false,
			"id": "I2aSUIVJ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -354.2937530363913,
			"y": -557.4746767009625,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 303.1942138671875,
			"height": 26.4,
			"seed": 1067703837,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"fontSize": 22.805413413175444,
			"fontFamily": 1,
			"text": "最多同时存在16个BlockCache",
			"rawText": "最多同时存在16个BlockCache",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "最多同时存在16个BlockCache"
		},
		{
			"type": "text",
			"version": 338,
			"versionNonce": 1923605245,
			"isDeleted": false,
			"id": "A3Z00Qed",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -400.7484318478523,
			"y": -681.8950525994143,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 420.0635986328125,
			"height": 38.4,
			"seed": 1368512883,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"fontSize": 32.33342168593334,
			"fontFamily": 1,
			"text": "BLOCK_CACHE_MANAGER",
			"rawText": "BLOCK_CACHE_MANAGER",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "BLOCK_CACHE_MANAGER"
		},
		{
			"type": "arrow",
			"version": 459,
			"versionNonce": 1025777299,
			"isDeleted": false,
			"id": "jFX5ImuQj2wfh8aiFUdIg",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 162.36066542083347,
			"y": -590.166761506107,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 358.65034719503365,
			"height": 0.00002834468972423565,
			"seed": 1733873235,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126603806,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "T7iD4Gko",
				"focus": -0.2065495155885256,
				"gap": 1.0428430381746239
			},
			"endBinding": {
				"elementId": "Iz1x0HhHt7VCspfg_aVFp",
				"focus": -0.07261187460375323,
				"gap": 4.355062038279925
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					358.65034719503365,
					0.00002834468972423565
				]
			]
		},
		{
			"type": "text",
			"version": 274,
			"versionNonce": 420248413,
			"isDeleted": false,
			"id": "mjG14DpA",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 195.3547912392154,
			"y": -582.5025274845891,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 302.3326416015625,
			"height": 75.6,
			"seed": 51312029,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"fontSize": 21.125368981026195,
			"fontFamily": 1,
			"text": "     get_block_cache()\n返回对应block_id的BlockCache\n     (如果不存在就构建)",
			"rawText": "     get_block_cache()\n返回对应block_id的BlockCache\n     (如果不存在就构建)",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "     get_block_cache()\n返回对应block_id的BlockCache\n     (如果不存在就构建)"
		},
		{
			"type": "text",
			"version": 306,
			"versionNonce": 680604915,
			"isDeleted": false,
			"id": "wQEmJhLJ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 472.42946166952663,
			"y": -413.2606937940775,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 521.8897705078125,
			"height": 50.4,
			"seed": 64734045,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"fontSize": 21.41312450403035,
			"fontFamily": 1,
			"text": "sync()函数可以根据是否modify来将cache内容写回磁盘\n            drop()的时候会调用sync()",
			"rawText": "sync()函数可以根据是否modify来将cache内容写回磁盘\n            drop()的时候会调用sync()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "sync()函数可以根据是否modify来将cache内容写回磁盘\n            drop()的时候会调用sync()"
		},
		{
			"type": "rectangle",
			"version": 195,
			"versionNonce": 1223497885,
			"isDeleted": false,
			"id": "FXAxMFvU5Eljfbwmv49aH",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 13.67832723054633,
			"y": -119.57845763618582,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 408.67498298239707,
			"height": 80.54795500628006,
			"seed": 667353597,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "wzrCNFaXCjPYbXf9tjTFq",
					"type": "arrow"
				}
			],
			"updated": 1683126252030,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 230,
			"versionNonce": 1518603187,
			"isDeleted": false,
			"id": "DJEya7QP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 154.89293176480103,
			"y": -108.20815543891888,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 110.80377197265625,
			"height": 54,
			"seed": 1147391411,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252030,
			"link": null,
			"locked": false,
			"fontSize": 45.30448560390263,
			"fontFamily": 1,
			"text": "trait",
			"rawText": "trait",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "trait"
		},
		{
			"type": "text",
			"version": 264,
			"versionNonce": 140115197,
			"isDeleted": false,
			"id": "Yi3tmjrV",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 113.23133997834296,
			"y": -165.06896348346868,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 185.861328125,
			"height": 38.4,
			"seed": 1193520435,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252030,
			"link": null,
			"locked": false,
			"fontSize": 32.89264945326145,
			"fontFamily": 1,
			"text": "BlockDevice",
			"rawText": "BlockDevice",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "BlockDevice"
		},
		{
			"type": "arrow",
			"version": 530,
			"versionNonce": 386951261,
			"isDeleted": false,
			"id": "RgQi4v5rOLYIi0d27TMOY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 537.294404974738,
			"y": -706.7488934155579,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 547.6965796446946,
			"height": 796.5661309022504,
			"seed": 1122739251,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "2z7bI9s3",
				"focus": 0.7306331796428858,
				"gap": 12.28603933252441
			},
			"endBinding": {
				"elementId": "GEb6GLf4NKCug5IO5ViEv",
				"focus": -1.0204723997123415,
				"gap": 12.696610836733782
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-288.8945528391671,
					193.1410972656032
				],
				[
					-547.6965796446946,
					796.5661309022504
				]
			]
		},
		{
			"type": "arrow",
			"version": 409,
			"versionNonce": 1859033075,
			"isDeleted": false,
			"id": "VpenBkf1c0LB8YCUIvEjD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 382.78582208342056,
			"y": 80.4905874154025,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 162.52485936547447,
			"height": 789.7972724326365,
			"seed": 1387647581,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126603429,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "SY5w45lSVZNMCJuWJCk7A",
				"focus": 0.15944616190389124,
				"gap": 22.023260908023815
			},
			"endBinding": {
				"elementId": "2z7bI9s3",
				"focus": 0.9442245089217038,
				"gap": 5.503155091937629
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					28.40563278016765,
					-321.08312621235393
				],
				[
					162.52485936547447,
					-789.7972724326365
				]
			]
		},
		{
			"type": "text",
			"version": 367,
			"versionNonce": 967113459,
			"isDeleted": false,
			"id": "0r7YGDLt",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 6.280230635014343,
			"x": -266.42129676404943,
			"y": -256.9848858000709,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 487.15850830078125,
			"height": 62.4,
			"seed": 784412093,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 26.359083888711158,
			"fontFamily": 1,
			"text": "BlockDevice::read_block(block_id, buf)\n   读取磁盘block_id块的内容到buf中",
			"rawText": "BlockDevice::read_block(block_id, buf)\n   读取磁盘block_id块的内容到buf中",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "BlockDevice::read_block(block_id, buf)\n   读取磁盘block_id块的内容到buf中"
		},
		{
			"type": "text",
			"version": 600,
			"versionNonce": 1534239165,
			"isDeleted": false,
			"id": "sSA6WbAy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 6.280230635014343,
			"x": 279.2563687772151,
			"y": -239.66376557720778,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 491.29547119140625,
			"height": 62.4,
			"seed": 1063181661,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 26.359083888711158,
			"fontFamily": 1,
			"text": "BlockDevice::write_block(block_id, buf)\n    将buf内容写道磁盘block_id块中",
			"rawText": "BlockDevice::write_block(block_id, buf)\n    将buf内容写道磁盘block_id块中",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "BlockDevice::write_block(block_id, buf)\n    将buf内容写道磁盘block_id块中"
		},
		{
			"type": "text",
			"version": 140,
			"versionNonce": 1137648787,
			"isDeleted": false,
			"id": "mNqdb9bR",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 34.88871520797886,
			"y": -34.467007591315905,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 363.4443359375,
			"height": 27.599999999999998,
			"seed": 2084558611,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 23.53278763950101,
			"fontFamily": 1,
			"text": "使用该文件系统需要实现这个trait",
			"rawText": "使用该文件系统需要实现这个trait",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "使用该文件系统需要实现这个trait"
		},
		{
			"type": "rectangle",
			"version": 237,
			"versionNonce": 2140234269,
			"isDeleted": false,
			"id": "mdW2oynskzPPbvGRMKpaD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -629.7853600320856,
			"y": 371.9626166030336,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 347.906463069253,
			"height": 239.30580692649136,
			"seed": 2057100531,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "N0IGR2eYHJTaogkgqREpn",
					"type": "arrow"
				}
			],
			"updated": 1683126252031,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 180,
			"versionNonce": 105060915,
			"isDeleted": false,
			"id": "ggKcUk3Z",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -574.0180988696358,
			"y": 329.1714075691161,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 247.02932739257812,
			"height": 40.8,
			"seed": 1672453373,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 34.41598407854844,
			"fontFamily": 1,
			"text": "SuperBlock结构",
			"rawText": "SuperBlock结构",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "SuperBlock结构"
		},
		{
			"type": "text",
			"version": 312,
			"versionNonce": 1186535037,
			"isDeleted": false,
			"id": "rDHVuvdi",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -616.0484976181764,
			"y": 391.25258158374584,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 326.14410400390625,
			"height": 205.20000000000002,
			"seed": 329403677,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "N0IGR2eYHJTaogkgqREpn",
					"type": "arrow"
				},
				{
					"id": "Ev10UtjomSx29W-oLPpqQ",
					"type": "arrow"
				}
			],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 19.744950733584268,
			"fontFamily": 1,
			"text": "// magic 用于验证文件系统是否合法\nmagic: u32,\n// 文件系统总块数\npub total_blocks: u32,\n// 4个连续区域各占多少块\npub inode_bitmap_blocks: u32,\npub inode_area_blocks: u32,\npub data_bitmap_blocks: u32,\npub data_area_blocks: u32,",
			"rawText": "// magic 用于验证文件系统是否合法\nmagic: u32,\n// 文件系统总块数\npub total_blocks: u32,\n// 4个连续区域各占多少块\npub inode_bitmap_blocks: u32,\npub inode_area_blocks: u32,\npub data_bitmap_blocks: u32,\npub data_area_blocks: u32,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "// magic 用于验证文件系统是否合法\nmagic: u32,\n// 文件系统总块数\npub total_blocks: u32,\n// 4个连续区域各占多少块\npub inode_bitmap_blocks: u32,\npub inode_area_blocks: u32,\npub data_bitmap_blocks: u32,\npub data_area_blocks: u32,"
		},
		{
			"type": "arrow",
			"version": 288,
			"versionNonce": 1843878867,
			"isDeleted": false,
			"id": "N0IGR2eYHJTaogkgqREpn",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -321.76321999759796,
			"y": 379.8428505006003,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 19.55179243289672,
			"height": 178.75854528133675,
			"seed": 1036107901,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "rDHVuvdi",
				"focus": 0.6798938396796474,
				"gap": 11.409731083145516
			},
			"endBinding": {
				"elementId": "TMdk-HDM79zIGWMVIEZDJ",
				"focus": -0.5883010461730346,
				"gap": 3.271850938481691
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					19.55179243289672,
					-178.75854528133675
				]
			]
		},
		{
			"type": "text",
			"version": 256,
			"versionNonce": 1269513949,
			"isDeleted": false,
			"id": "k9qyI3C2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -619.5114053115244,
			"y": 623.3810501852533,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 320.87994384765625,
			"height": 33.6,
			"seed": 316087005,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 28.00890979253873,
			"fontFamily": 1,
			"text": "处在Super block起始位置",
			"rawText": "处在Super block起始位置",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "处在Super block起始位置"
		},
		{
			"type": "rectangle",
			"version": 312,
			"versionNonce": 114452851,
			"isDeleted": false,
			"id": "2Vsm4XygE_02S5AQfO2YY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 178.51733972431845,
			"y": 321.8971253849171,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 278.6981582900921,
			"height": 70.41833899349085,
			"seed": 260066621,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "4KCWlpO4F835QqvTQaHP2",
					"type": "arrow"
				},
				{
					"id": "bDgfq8KVpNXdTRqzoOET8",
					"type": "arrow"
				},
				{
					"id": "5-EQ0lLuwYK6dY4QXddm0",
					"type": "arrow"
				},
				{
					"id": "Yp4a-BGSf3sqUknvzjaK3",
					"type": "arrow"
				}
			],
			"updated": 1683126252031,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 231,
			"versionNonce": 1425627965,
			"isDeleted": false,
			"id": "Dstr4OZ1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 205.71852188314358,
			"y": 286.9970074819329,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 229.1996612548828,
			"height": 32.4,
			"seed": 119582419,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 27.251730763041127,
			"fontFamily": 1,
			"text": "type BitmapBlock",
			"rawText": "type BitmapBlock",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "type BitmapBlock"
		},
		{
			"type": "text",
			"version": 305,
			"versionNonce": 373998355,
			"isDeleted": false,
			"id": "0UGVj80g",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 162.60169657113852,
			"y": 401.5132721692961,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 310.1187438964844,
			"height": 22.8,
			"seed": 1594405181,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 19.28318344695485,
			"fontFamily": 1,
			"text": "每一个bitmap块都可看成这样的类型",
			"rawText": "每一个bitmap块都可看成这样的类型",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "每一个bitmap块都可看成这样的类型"
		},
		{
			"type": "text",
			"version": 247,
			"versionNonce": 1527049117,
			"isDeleted": false,
			"id": "Fr9pTQuq",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 230.04463894185008,
			"y": 340.6622368320375,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 175.97479248046875,
			"height": 43.199999999999996,
			"seed": 666863901,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 36.09859545284222,
			"fontFamily": 1,
			"text": "[u64, 64]",
			"rawText": "[u64, 64]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[u64, 64]"
		},
		{
			"type": "arrow",
			"version": 221,
			"versionNonce": 1959192755,
			"isDeleted": false,
			"id": "4KCWlpO4F835QqvTQaHP2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 173.50898873289418,
			"y": 369.3409643585289,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 362.612601790231,
			"height": 178.26970001209276,
			"seed": 1990129373,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "2Vsm4XygE_02S5AQfO2YY",
				"gap": 4.959016475621905,
				"focus": -0.8045359478471226
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-362.612601790231,
					-178.26970001209276
				]
			]
		},
		{
			"type": "arrow",
			"version": 429,
			"versionNonce": 1308585981,
			"isDeleted": false,
			"id": "bDgfq8KVpNXdTRqzoOET8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 418.6539923866642,
			"y": 318.81354711961075,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 46.78163585146808,
			"height": 113.98378319884694,
			"seed": 1456529213,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "2Vsm4XygE_02S5AQfO2YY",
				"gap": 2.9753966227791717,
				"focus": 0.7625143011659349
			},
			"endBinding": {
				"elementId": "b9APmgRwq2DK-9RkLAwYd",
				"gap": 9.793032627333895,
				"focus": 0.5246128238177629
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-46.78163585146808,
					-113.98378319884694
				]
			]
		},
		{
			"type": "rectangle",
			"version": 427,
			"versionNonce": 669866579,
			"isDeleted": false,
			"id": "pIQe6X82dv88IbysqHM5_",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -204.44942639541523,
			"y": 514.9698897799558,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 469.00111487481263,
			"height": 285.4369988162633,
			"seed": 853479741,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "W8_Saw0eqoJX_jobdDjiz",
					"type": "arrow"
				}
			],
			"updated": 1683126252031,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 298,
			"versionNonce": 468346973,
			"isDeleted": false,
			"id": "EVtTrhwM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -96.90735037438549,
			"y": 473.89018260000887,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 221.93267822265625,
			"height": 39.6,
			"seed": 1833353875,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 33.07412118315554,
			"fontFamily": 1,
			"text": "DiskInode结构",
			"rawText": "DiskInode结构",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "DiskInode结构"
		},
		{
			"type": "text",
			"version": 318,
			"versionNonce": 1228972413,
			"isDeleted": false,
			"id": "wpUnIheX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -193.08482275759843,
			"y": 542.5263869102052,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 445.7596740722656,
			"height": 240,
			"seed": 32811325,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "DOvwM9nIa7W9IODDjJ6sD",
					"type": "arrow"
				},
				{
					"id": "LLoiyO6NV2zglum2vrUgy",
					"type": "arrow"
				}
			],
			"updated": 1683126285209,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "// 占的字节数\npub size: u32,\n// 直接索引的数组(元素为索引号)\npub direct: [u32; INODE_DIRECT_COUNT],\n// 一级间接索引\npub indirect1: u32,\n// 二级间接索引\npub indirect2: u32,\n// 文件/目录类型\ntype_: DiskInodeType,",
			"rawText": "// 占的字节数\npub size: u32,\n// 直接索引的数组(元素为索引号)\npub direct: [u32; INODE_DIRECT_COUNT],\n// 一级间接索引\npub indirect1: u32,\n// 二级间接索引\npub indirect2: u32,\n// 文件/目录类型\ntype_: DiskInodeType,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "// 占的字节数\npub size: u32,\n// 直接索引的数组(元素为索引号)\npub direct: [u32; INODE_DIRECT_COUNT],\n// 一级间接索引\npub indirect1: u32,\n// 二级间接索引\npub indirect2: u32,\n// 文件/目录类型\ntype_: DiskInodeType,"
		},
		{
			"type": "arrow",
			"version": 93,
			"versionNonce": 1989629117,
			"isDeleted": false,
			"id": "W8_Saw0eqoJX_jobdDjiz",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -149.55812094273278,
			"y": 509.30763634018945,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 231.61725352565418,
			"height": 316.1912766772376,
			"seed": 1229756499,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "pIQe6X82dv88IbysqHM5_",
				"focus": -0.8503338361151594,
				"gap": 5.662253439766403
			},
			"endBinding": {
				"elementId": "GEb6GLf4NKCug5IO5ViEv",
				"focus": 0.13795087657458754,
				"gap": 1
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					231.61725352565418,
					-316.1912766772376
				]
			]
		},
		{
			"type": "rectangle",
			"version": 179,
			"versionNonce": 1623400851,
			"isDeleted": false,
			"id": "rozrbqeSS8kWNx2lmJhE9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 597.3437299198315,
			"y": 294.028462404876,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 258.5273189438353,
			"height": 131.6663163452729,
			"seed": 727342131,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "UInlKBgwk8f3iZ-vyNdlT",
					"type": "arrow"
				},
				{
					"id": "GLQDSkz-vkoFEZHYa-S6B",
					"type": "arrow"
				},
				{
					"id": "uwA_oNhWjQYT_BxeJi1wu",
					"type": "arrow"
				}
			],
			"updated": 1683126252031,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 187,
			"versionNonce": 1504163101,
			"isDeleted": false,
			"id": "YACGxxAP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 566.8150278075004,
			"y": 257.4164542218274,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 326.4691162109375,
			"height": 39.6,
			"seed": 1586634333,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 33.46679527047255,
			"fontFamily": 1,
			"text": "1.DataBlock字节序列",
			"rawText": "1.DataBlock字节序列",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "1.DataBlock字节序列"
		},
		{
			"type": "text",
			"version": 82,
			"versionNonce": 1034909491,
			"isDeleted": false,
			"id": "2tNBEhsL",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 691.0434282655337,
			"y": 297.9898164988721,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 50.68064880371094,
			"height": 45.6,
			"seed": 142006195,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 38.02454722557139,
			"fontFamily": 1,
			"text": "u8",
			"rawText": "u8",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "u8"
		},
		{
			"type": "line",
			"version": 89,
			"versionNonce": 1351420285,
			"isDeleted": false,
			"id": "NRiaD49spkNvhrvpt2FIk",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 600.1747923820894,
			"y": 339.19868150598165,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 248.91656293516348,
			"height": 0,
			"seed": 1934153885,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": null,
			"points": [
				[
					0,
					0
				],
				[
					248.91656293516348,
					0
				]
			]
		},
		{
			"type": "freedraw",
			"version": 24,
			"versionNonce": 297801939,
			"isDeleted": false,
			"id": "XinSh3C8AV5Eb7tUsUikX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 659.761016980952,
			"y": 374.75814459841536,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 1195984115,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.0001,
					0.0001
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 24,
			"versionNonce": 2089341405,
			"isDeleted": false,
			"id": "RxI19xf8c0LiUwCrxV4mC",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 659.761016980952,
			"y": 374.75814459841536,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 580122653,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.0001,
					0.0001
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 24,
			"versionNonce": 176916083,
			"isDeleted": false,
			"id": "qsrGz554CmgCtpJAcDLan",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 710.6976511326857,
			"y": 373.7971075521235,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 12051923,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.0001,
					0.0001
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 24,
			"versionNonce": 1521258045,
			"isDeleted": false,
			"id": "uZX46LKtSqoy1r3QsvAsF",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 773.1669868704239,
			"y": 375.7192459023328,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 767275325,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.0001,
					0.0001
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "rectangle",
			"version": 219,
			"versionNonce": 1591230483,
			"isDeleted": false,
			"id": "HtxZ5iIBCFq3i1fYlq6Kx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 600.78967360006,
			"y": 526.7029787709399,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 258.5273189438353,
			"height": 131.6663163452729,
			"seed": 248012755,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "g1NpUlR_UguMD6KW2TEtC",
					"type": "arrow"
				}
			],
			"updated": 1683126252031,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 213,
			"versionNonce": 1239006877,
			"isDeleted": false,
			"id": "SxsmhZtZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 574.1786631707423,
			"y": 489.0673518000832,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 324.46148681640625,
			"height": 39.6,
			"seed": 1749842653,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 33.46679527047255,
			"fontFamily": 1,
			"text": "2.DiskInode间接索引",
			"rawText": "2.DiskInode间接索引",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "2.DiskInode间接索引"
		},
		{
			"type": "text",
			"version": 131,
			"versionNonce": 681362867,
			"isDeleted": false,
			"id": "slWisgG9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 694.4893719457623,
			"y": 530.664332864936,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 74.55720520019531,
			"height": 45.6,
			"seed": 1827859827,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 38.02454722557139,
			"fontFamily": 1,
			"text": "u32",
			"rawText": "u32",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "u32"
		},
		{
			"type": "line",
			"version": 131,
			"versionNonce": 1354883837,
			"isDeleted": false,
			"id": "bQYKy95fcPk4e0_cCXJFG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 603.620736062318,
			"y": 571.8731978720456,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 248.91656293516348,
			"height": 0,
			"seed": 1108547389,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": null,
			"points": [
				[
					0,
					0
				],
				[
					248.91656293516348,
					0
				]
			]
		},
		{
			"type": "freedraw",
			"version": 66,
			"versionNonce": 2130679635,
			"isDeleted": false,
			"id": "aHx-Gbq8ZvnpUUAFItb6M",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 663.2069606611806,
			"y": 607.4326609644792,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 1840957203,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.0001,
					0.0001
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 66,
			"versionNonce": 602307421,
			"isDeleted": false,
			"id": "oGARG7dd53iYHIzYFH2zc",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 663.2069606611806,
			"y": 607.4326609644792,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 1971791773,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.0001,
					0.0001
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 66,
			"versionNonce": 1347892467,
			"isDeleted": false,
			"id": "sxMX0INnhaGP5p_39By64",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 714.1435948129142,
			"y": 606.4716239181874,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 1737911475,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.0001,
					0.0001
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 66,
			"versionNonce": 1805191101,
			"isDeleted": false,
			"id": "QkHGaTP70y7-Hwy9aw9BK",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 776.6129305506524,
			"y": 608.3937622683967,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 1423584253,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.0001,
					0.0001
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "rectangle",
			"version": 388,
			"versionNonce": 2083135123,
			"isDeleted": false,
			"id": "eUJe_XhjRde9GPXnwuGCm",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 518.7696382701997,
			"y": 767.8580559003756,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 443.0522150181746,
			"height": 360.40045873205133,
			"seed": 1903623005,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "2vXeSPq6NzrJ2AKDuyVUr",
					"type": "arrow"
				}
			],
			"updated": 1683126252031,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 305,
			"versionNonce": 1241188381,
			"isDeleted": false,
			"id": "iiCCn66t",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 651.975659879885,
			"y": 721.5729027400154,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 148.3915557861328,
			"height": 44.4,
			"seed": 1108260083,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 37.52786727480456,
			"fontFamily": 1,
			"text": "3.目录项",
			"rawText": "3.目录项",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "3.目录项"
		},
		{
			"type": "line",
			"version": 385,
			"versionNonce": 758043699,
			"isDeleted": false,
			"id": "hrc6qRsriLFbKnmNGXsi0",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 525.444848917625,
			"y": 1039.8403432956763,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 417.1034436767875,
			"height": 1.9220740925836708,
			"seed": 1257348755,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": null,
			"points": [
				[
					0,
					0
				],
				[
					417.1034436767875,
					1.9220740925836708
				]
			]
		},
		{
			"type": "freedraw",
			"version": 229,
			"versionNonce": 553516157,
			"isDeleted": false,
			"id": "c9NZ7DlxOVivzN_rBiUvC",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 660.9554419633166,
			"y": 1084.0493968352391,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 1298892829,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.0001,
					0.0001
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 229,
			"versionNonce": 1934349779,
			"isDeleted": false,
			"id": "yo1sL1woZn4qDajP1szm6",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 660.9554419633166,
			"y": 1084.0493968352391,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 923161651,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.0001,
					0.0001
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 229,
			"versionNonce": 409167069,
			"isDeleted": false,
			"id": "gi-rOf69FCROH7F_cRCFc",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 711.8920761150503,
			"y": 1083.0883597889472,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 1650359421,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.0001,
					0.0001
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 229,
			"versionNonce": 496477043,
			"isDeleted": false,
			"id": "oXwYw8vf2bhQ8yqx8ZIvr",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 774.3614118527885,
			"y": 1085.0104981391562,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 1446204883,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0.0001,
					0.0001
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "rectangle",
			"version": 290,
			"versionNonce": 1673314621,
			"isDeleted": false,
			"id": "Etpaf-QGP2oraXdhkfvqY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 518.678887910799,
			"y": 827.6546363170087,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 442.0911779718825,
			"height": 208.55172183839355,
			"seed": 1944527773,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 97,
			"versionNonce": 1415101715,
			"isDeleted": false,
			"id": "tWOUyUQ0",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 616.599238461464,
			"y": 782.7894481639338,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 227.3007049560547,
			"height": 44.4,
			"seed": 1786237021,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 37.67956810791147,
			"fontFamily": 1,
			"text": "DirEntry结构",
			"rawText": "DirEntry结构",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "DirEntry结构"
		},
		{
			"type": "text",
			"version": 134,
			"versionNonce": 1215009181,
			"isDeleted": false,
			"id": "VhtL1H1g",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 574.39997067323,
			"y": 852.5533095432247,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 317.2581787109375,
			"height": 49.199999999999996,
			"seed": 270151347,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 41.753355087968465,
			"fontFamily": 1,
			"text": "name: [u8: 28]",
			"rawText": "name: [u8: 28]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "name: [u8: 28]"
		},
		{
			"type": "text",
			"version": 146,
			"versionNonce": 1929749171,
			"isDeleted": false,
			"id": "aG6DElsz",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 577.0995620338088,
			"y": 961.1269774832956,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 313.44940185546875,
			"height": 49.199999999999996,
			"seed": 267084317,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 41.50410900424323,
			"fontFamily": 1,
			"text": "inode_num: u32",
			"rawText": "inode_num: u32",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "inode_num: u32"
		},
		{
			"type": "line",
			"version": 100,
			"versionNonce": 1671517693,
			"isDeleted": false,
			"id": "pHxWE4XJQtoCFUd0E5bP8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 521.4842016575094,
			"y": 926.6446649663492,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 434.4026245710454,
			"height": 3.8442767004185043,
			"seed": 732677491,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": null,
			"points": [
				[
					0,
					0
				],
				[
					434.4026245710454,
					3.8442767004185043
				]
			]
		},
		{
			"type": "arrow",
			"version": 191,
			"versionNonce": 574936147,
			"isDeleted": false,
			"id": "UInlKBgwk8f3iZ-vyNdlT",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 858.8977103446798,
			"y": 366.4929393921186,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 121.37247100116247,
			"height": 164.8834391667056,
			"seed": 1572731165,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "rozrbqeSS8kWNx2lmJhE9",
				"focus": 0.06057042795546266,
				"gap": 3.0266614810129795
			},
			"endBinding": {
				"elementId": "OyK9r24bPJpPReEeaXkRT",
				"focus": -0.565252077402488,
				"gap": 7.937558271799048
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					121.37247100116247,
					2.290050956081245
				],
				[
					59.54132485811192,
					-162.59338821062437
				]
			]
		},
		{
			"type": "arrow",
			"version": 166,
			"versionNonce": 811568733,
			"isDeleted": false,
			"id": "g1NpUlR_UguMD6KW2TEtC",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 866.9129269694877,
			"y": 596.6427159715679,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 182.05878305879105,
			"height": 380.1478462531033,
			"seed": 1411239731,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "HtxZ5iIBCFq3i1fYlq6Kx",
				"focus": 0.27524442876820415,
				"gap": 7.595934425592304
			},
			"endBinding": {
				"elementId": "OyK9r24bPJpPReEeaXkRT",
				"focus": -0.6035167976052941,
				"gap": 20.5328768087694
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					182.05878305879105,
					-25.19048395984612
				],
				[
					61.8312992571457,
					-380.1478462531033
				]
			]
		},
		{
			"type": "arrow",
			"version": 381,
			"versionNonce": 837603827,
			"isDeleted": false,
			"id": "2vXeSPq6NzrJ2AKDuyVUr",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 963.0949140108041,
			"y": 775.9321244010913,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 175.18855363350008,
			"height": 564.4966802679755,
			"seed": 1781318653,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "eUJe_XhjRde9GPXnwuGCm",
				"focus": -0.07821576396302694,
				"gap": 1.2730607224298183
			},
			"endBinding": {
				"elementId": "OyK9r24bPJpPReEeaXkRT",
				"focus": -0.5705326367067669,
				"gap": 15.473451223420568
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					133.96778953813282,
					-88.16684697355583
				],
				[
					-41.22076409536726,
					-564.4966802679755
				]
			]
		},
		{
			"type": "text",
			"version": 229,
			"versionNonce": 1299692221,
			"isDeleted": false,
			"id": "zTHFicMg",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 1.208264286577056,
			"x": 893.4682505409219,
			"y": 442.69250448961486,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 322.67291259765625,
			"height": 36,
			"seed": 974371827,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 30.21731820784044,
			"fontFamily": 1,
			"text": "数据块有3种不同的结构",
			"rawText": "数据块有3种不同的结构",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "数据块有3种不同的结构"
		},
		{
			"type": "rectangle",
			"version": 551,
			"versionNonce": 63397661,
			"isDeleted": false,
			"id": "sqJdtCjCTrEUcJ-n9urcd",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -981.7226021428671,
			"y": 1444.6812807348745,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 996.5529747883091,
			"height": 405.6745483843809,
			"seed": 949328019,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "GLQDSkz-vkoFEZHYa-S6B",
					"type": "arrow"
				},
				{
					"id": "uwA_oNhWjQYT_BxeJi1wu",
					"type": "arrow"
				}
			],
			"updated": 1683126405400,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 406,
			"versionNonce": 2114845811,
			"isDeleted": false,
			"id": "C0LmvaAS",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -797.9721942611193,
			"y": 1354.3578295455204,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 653.8680419921875,
			"height": 84,
			"seed": 206801299,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "Ev10UtjomSx29W-oLPpqQ",
					"type": "arrow"
				},
				{
					"id": "DOvwM9nIa7W9IODDjJ6sD",
					"type": "arrow"
				}
			],
			"updated": 1683126405400,
			"link": null,
			"locked": false,
			"fontSize": 70.15586572800073,
			"fontFamily": 1,
			"text": "EasyFileSystem结构",
			"rawText": "EasyFileSystem结构",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "EasyFileSystem结构"
		},
		{
			"type": "text",
			"version": 405,
			"versionNonce": 534203645,
			"isDeleted": false,
			"id": "1ZEs3VyZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -915.3759666458311,
			"y": 1467.723118819208,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 853.0328979492188,
			"height": 51.6,
			"seed": 1676017853,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "wzrCNFaXCjPYbXf9tjTFq",
					"type": "arrow"
				}
			],
			"updated": 1683126405401,
			"link": null,
			"locked": false,
			"fontSize": 43.8673010423929,
			"fontFamily": 1,
			"text": "pub block_device: Arc<dyn BlockDevice>,",
			"rawText": "pub block_device: Arc<dyn BlockDevice>,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "pub block_device: Arc<dyn BlockDevice>,"
		},
		{
			"type": "text",
			"version": 405,
			"versionNonce": 12494173,
			"isDeleted": false,
			"id": "mmzFhevt",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -915.3759666458311,
			"y": 1542.297530591276,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 554.3463134765625,
			"height": 51.6,
			"seed": 906311059,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "5-EQ0lLuwYK6dY4QXddm0",
					"type": "arrow"
				}
			],
			"updated": 1683126405401,
			"link": null,
			"locked": false,
			"fontSize": 43.8673010423929,
			"fontFamily": 1,
			"text": "pub inode_bitmap: Bitmap,",
			"rawText": "pub inode_bitmap: Bitmap,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "pub inode_bitmap: Bitmap,"
		},
		{
			"type": "text",
			"version": 405,
			"versionNonce": 266072509,
			"isDeleted": false,
			"id": "dYkGTzTi",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -915.3759666458311,
			"y": 1616.8719423633438,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 559.2586669921875,
			"height": 51.6,
			"seed": 4251933,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "Yp4a-BGSf3sqUknvzjaK3",
					"type": "arrow"
				}
			],
			"updated": 1683126405401,
			"link": null,
			"locked": false,
			"fontSize": 43.8673010423929,
			"fontFamily": 1,
			"text": "pub data_bitmap: Bitmap,",
			"rawText": "pub data_bitmap: Bitmap,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "pub data_bitmap: Bitmap,"
		},
		{
			"type": "text",
			"version": 405,
			"versionNonce": 2052108829,
			"isDeleted": false,
			"id": "a5jf5qsc",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -915.3759666458311,
			"y": 1691.4463541354116,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 666.1893310546875,
			"height": 51.6,
			"seed": 1234111283,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "QfZOovxB3nnRP3it8JKxy",
					"type": "arrow"
				}
			],
			"updated": 1683126405401,
			"link": null,
			"locked": false,
			"fontSize": 43.8673010423929,
			"fontFamily": 1,
			"text": "inode_area_start_block: u32,",
			"rawText": "inode_area_start_block: u32,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "inode_area_start_block: u32,"
		},
		{
			"type": "text",
			"version": 405,
			"versionNonce": 1292789373,
			"isDeleted": false,
			"id": "Dss86BZN",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -915.3759666458311,
			"y": 1766.0207659074795,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 671.1015625,
			"height": 51.6,
			"seed": 589346173,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "nV9aoRPFpmOAKgCg-2cIY",
					"type": "arrow"
				}
			],
			"updated": 1683126405401,
			"link": null,
			"locked": false,
			"fontSize": 43.8673010423929,
			"fontFamily": 1,
			"text": "data_area_start_block: u32,",
			"rawText": "data_area_start_block: u32,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "data_area_start_block: u32,"
		},
		{
			"type": "arrow",
			"version": 708,
			"versionNonce": 835174141,
			"isDeleted": false,
			"id": "wzrCNFaXCjPYbXf9tjTFq",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -899.1831460298329,
			"y": 1462.003650663387,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 931.6063954021183,
			"height": 1549.0677555187503,
			"seed": 979469683,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126487779,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "1ZEs3VyZ",
				"focus": -0.9587786159640207,
				"gap": 5.719468155821005
			},
			"endBinding": {
				"elementId": "FXAxMFvU5Eljfbwmv49aH",
				"focus": 0.512996114840833,
				"gap": 3.32434651732342
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-22.069268659062573,
					-893.9715796882226
				],
				[
					188.54363702406476,
					-1458.620489049368
				],
				[
					909.5371267430558,
					-1549.0677555187503
				]
			]
		},
		{
			"type": "arrow",
			"version": 831,
			"versionNonce": 305656915,
			"isDeleted": false,
			"id": "5-EQ0lLuwYK6dY4QXddm0",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -353.1075641033784,
			"y": 1573.9457045470938,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 810.5992608226902,
			"height": 1192.1417660167729,
			"seed": 1322281971,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126449094,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "mmzFhevt",
				"focus": 0.8587165607274235,
				"gap": 7.922089065890191
			},
			"endBinding": {
				"elementId": "2Vsm4XygE_02S5AQfO2YY",
				"focus": -0.9821867903833769,
				"gap": 1
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					529.03936604657,
					-183.2309840921116
				],
				[
					810.5992608226902,
					-1192.1417660167729
				]
			]
		},
		{
			"type": "arrow",
			"version": 671,
			"versionNonce": 268723155,
			"isDeleted": false,
			"id": "Yp4a-BGSf3sqUknvzjaK3",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -346.951775175876,
			"y": 1649.4239760289781,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 805.735533740614,
			"height": 1265.0358274169357,
			"seed": 247881779,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126459326,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "dYkGTzTi",
				"focus": 0.8687827007590642,
				"gap": 9.165524477767633
			},
			"endBinding": {
				"elementId": "2Vsm4XygE_02S5AQfO2YY",
				"focus": -0.9955478128066245,
				"gap": 1.568260550327409
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					497.2601115069359,
					-169.83758833643196
				],
				[
					805.735533740614,
					-1265.0358274169357
				]
			]
		},
		{
			"type": "arrow",
			"version": 860,
			"versionNonce": 632019091,
			"isDeleted": false,
			"id": "QfZOovxB3nnRP3it8JKxy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -920.2327430482718,
			"y": 1716.6868627875679,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 964.6220873611363,
			"height": 1522.167746623348,
			"seed": 1740785405,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126549736,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "a5jf5qsc",
				"focus": -1.0058626569936902,
				"gap": 4.856776402440687
			},
			"endBinding": {
				"elementId": "GEb6GLf4NKCug5IO5ViEv",
				"focus": -0.0863152635182686,
				"gap": 2.2580459966980584
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-40.42586900415199,
					-369.05510568534555
				],
				[
					193.6905105002487,
					-1314.2684422946309
				],
				[
					690.3255947201027,
					-1454.978327694024
				],
				[
					924.1962183569843,
					-1522.167746623348
				]
			]
		},
		{
			"type": "arrow",
			"version": 540,
			"versionNonce": 163818355,
			"isDeleted": false,
			"id": "nV9aoRPFpmOAKgCg-2cIY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -241.94025932503584,
			"y": 1807.4608410645649,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 795.3098940737736,
			"height": 1603.0223500225418,
			"seed": 277643773,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126466976,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Dss86BZN",
				"focus": 0.9488840734103557,
				"gap": 2.334144820795302
			},
			"endBinding": {
				"elementId": "OyK9r24bPJpPReEeaXkRT",
				"focus": 0.8679947743080277,
				"gap": 8.47649813232789
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					473.7383965445995,
					-214.9417956179352
				],
				[
					795.3098940737736,
					-1603.0223500225418
				]
			]
		},
		{
			"type": "arrow",
			"version": 825,
			"versionNonce": 1721497107,
			"isDeleted": false,
			"id": "Ev10UtjomSx29W-oLPpqQ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -516.1681020523904,
			"y": 605.3032152180219,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 156.61324693983488,
			"height": 744.826141779198,
			"seed": 1030257587,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126405401,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "rDHVuvdi",
				"focus": 0.21504218771868747,
				"gap": 8.85063363427605
			},
			"endBinding": {
				"elementId": "C0LmvaAS",
				"focus": -0.6297957606040644,
				"gap": 4.2284725483005445
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-156.61324693983488,
					744.826141779198
				]
			]
		},
		{
			"type": "text",
			"version": 88,
			"versionNonce": 878027101,
			"isDeleted": false,
			"id": "KFLiBZhh",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -592.3548435677644,
			"y": 807.1276835181513,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 192.5792999267578,
			"height": 43.199999999999996,
			"seed": 1023219037,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126252031,
			"link": null,
			"locked": false,
			"fontSize": 36.24537867272334,
			"fontFamily": 1,
			"text": "open() 创建",
			"rawText": "open() 创建",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "open() 创建"
		},
		{
			"type": "arrow",
			"version": 651,
			"versionNonce": 712620979,
			"isDeleted": false,
			"id": "DOvwM9nIa7W9IODDjJ6sD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -339.2539747907715,
			"y": 1348.2786075051645,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 173.95607627143661,
			"height": 546.4551121564654,
			"seed": 1796262557,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126405401,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "C0LmvaAS",
				"focus": 0.34231618009006703,
				"gap": 6.079222040355887
			},
			"endBinding": {
				"elementId": "wpUnIheX",
				"focus": 0.57740783059501,
				"gap": 19.297108438493865
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					173.95607627143661,
					-546.4551121564654
				]
			]
		},
		{
			"type": "text",
			"version": 156,
			"versionNonce": 926729804,
			"isDeleted": false,
			"id": "6mZeVH09",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -198.5000262802643,
			"y": 911.6039271337352,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 173.70848083496094,
			"height": 33.6,
			"seed": 142724403,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683162673240,
			"link": null,
			"locked": false,
			"fontSize": 28.09202059135043,
			"fontFamily": 1,
			"text": "alloc_inode()",
			"rawText": "alloc_inode()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "alloc_inode()"
		},
		{
			"type": "arrow",
			"version": 663,
			"versionNonce": 757560189,
			"isDeleted": false,
			"id": "GLQDSkz-vkoFEZHYa-S6B",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -70.9852373370775,
			"y": 1436.6930459679975,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 653.4844212161545,
			"height": 1037.4915352558564,
			"seed": 234106653,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126405400,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "sqJdtCjCTrEUcJ-n9urcd",
				"focus": 0.4467516523201132,
				"gap": 7.988234766877099
			},
			"endBinding": {
				"elementId": "rozrbqeSS8kWNx2lmJhE9",
				"focus": 0.6989353127657226,
				"gap": 14.844546040754494
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					653.4844212161545,
					-1037.4915352558564
				]
			]
		},
		{
			"type": "arrow",
			"version": 761,
			"versionNonce": 1850080221,
			"isDeleted": false,
			"id": "uwA_oNhWjQYT_BxeJi1wu",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 592.3963458472872,
			"y": 427.06685479816593,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 635.6630399977131,
			"height": 1010.7050951871047,
			"seed": 1014115923,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126405400,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "rozrbqeSS8kWNx2lmJhE9",
				"focus": 0.5383264065776834,
				"gap": 5.134121331134111
			},
			"endBinding": {
				"elementId": "sqJdtCjCTrEUcJ-n9urcd",
				"focus": 0.4925539845659426,
				"gap": 6.90933074960401
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-635.6630399977131,
					1010.7050951871047
				]
			]
		},
		{
			"type": "text",
			"version": 380,
			"versionNonce": 1437441341,
			"isDeleted": false,
			"id": "zhYtsvvY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -135.79168485544102,
			"y": 1258.3800663030834,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 179.8136749267578,
			"height": 33.6,
			"seed": 2001901747,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126501180,
			"link": null,
			"locked": false,
			"fontSize": 28.568451746562182,
			"fontFamily": 1,
			"text": "alloc_data()",
			"rawText": "alloc_data()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "alloc_data()"
		},
		{
			"type": "text",
			"version": 449,
			"versionNonce": 1644708211,
			"isDeleted": false,
			"id": "zcVHrb8Z",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -35.35218365677781,
			"y": 1369.5950210134051,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 188.19058227539062,
			"height": 30,
			"seed": 544765363,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126507483,
			"link": null,
			"locked": false,
			"fontSize": 25.394640317266436,
			"fontFamily": 1,
			"text": "dealloc_data()",
			"rawText": "dealloc_data()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "dealloc_data()"
		},
		{
			"type": "rectangle",
			"version": 144,
			"versionNonce": 1703661757,
			"isDeleted": false,
			"id": "MCO9ktFL_ywphyQRn0yoi",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 379.4699823914366,
			"y": 1570.2836515162949,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 583.4173599188007,
			"height": 252.54795381387248,
			"seed": 227211187,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "LLoiyO6NV2zglum2vrUgy",
					"type": "arrow"
				}
			],
			"updated": 1683126422841,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 172,
			"versionNonce": 2046795645,
			"isDeleted": false,
			"id": "VvruwVtq",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 538.0579964830499,
			"y": 1509.3766519751339,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 242.24661254882812,
			"height": 61.199999999999996,
			"seed": 1042760147,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126419270,
			"link": null,
			"locked": false,
			"fontSize": 51.749602493970144,
			"fontFamily": 1,
			"text": "Inode结构",
			"rawText": "Inode结构",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Inode结构"
		},
		{
			"type": "text",
			"version": 228,
			"versionNonce": 1592386259,
			"isDeleted": false,
			"id": "2JpMLpdo",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 396.02719622722134,
			"y": 1599.8499826569082,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 214.62997436523438,
			"height": 37.199999999999996,
			"seed": 862793309,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126419270,
			"link": null,
			"locked": false,
			"fontSize": 31.16347606459735,
			"fontFamily": 1,
			"text": "block_id: u32,",
			"rawText": "block_id: u32,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "block_id: u32,"
		},
		{
			"type": "text",
			"version": 135,
			"versionNonce": 1194895325,
			"isDeleted": false,
			"id": "z4IHblwd",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 396.02719622722134,
			"y": 1652.8278919667237,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 304.40191650390625,
			"height": 37.199999999999996,
			"seed": 1745433075,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126419270,
			"link": null,
			"locked": false,
			"fontSize": 31.16347606459735,
			"fontFamily": 1,
			"text": "block_offset: usize,",
			"rawText": "block_offset: usize,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "block_offset: usize,"
		},
		{
			"type": "text",
			"version": 135,
			"versionNonce": 1676415091,
			"isDeleted": false,
			"id": "M5wBSjlB",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 396.02719622722134,
			"y": 1705.8058012765391,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 502.4547119140625,
			"height": 37.199999999999996,
			"seed": 1574001341,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126419270,
			"link": null,
			"locked": false,
			"fontSize": 31.16347606459735,
			"fontFamily": 1,
			"text": "fs: Arc<Mutex<EasyFileSystem>>,",
			"rawText": "fs: Arc<Mutex<EasyFileSystem>>,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "fs: Arc<Mutex<EasyFileSystem>>,"
		},
		{
			"type": "text",
			"version": 135,
			"versionNonce": 754033725,
			"isDeleted": false,
			"id": "yfUcplRo",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 396.02719622722134,
			"y": 1758.7837105863546,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 541.560546875,
			"height": 37.199999999999996,
			"seed": 647265171,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126419270,
			"link": null,
			"locked": false,
			"fontSize": 31.16347606459735,
			"fontFamily": 1,
			"text": "block_device: Arc<dyn BlockDevice>,",
			"rawText": "block_device: Arc<dyn BlockDevice>,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "block_device: Arc<dyn BlockDevice>,"
		},
		{
			"type": "arrow",
			"version": 94,
			"versionNonce": 109591347,
			"isDeleted": false,
			"id": "LLoiyO6NV2zglum2vrUgy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 457.5813653843579,
			"y": 1555.2402565627847,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 219.0063897797702,
			"height": 751.4650674801738,
			"seed": 95894525,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683126423317,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "MCO9ktFL_ywphyQRn0yoi",
				"focus": -0.5248301173798147,
				"gap": 15.043394953510187
			},
			"endBinding": {
				"elementId": "wpUnIheX",
				"focus": -0.6500402103349927,
				"gap": 21.24880217240559
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-219.0063897797702,
					-751.4650674801738
				]
			]
		},
		{
			"type": "text",
			"version": 406,
			"versionNonce": 261589523,
			"isDeleted": false,
			"id": "NIiqirll",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 329.1713461686611,
			"y": 1443.160551441357,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 642.4205322265625,
			"height": 64.8,
			"seed": 1029318205,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683126419270,
			"link": null,
			"locked": false,
			"fontSize": 27.984576501359797,
			"fontFamily": 1,
			"text": "             与一个DiskInode相关联\n通过block_id和block_offset找到对应的DiskInode",
			"rawText": "             与一个DiskInode相关联\n通过block_id和block_offset找到对应的DiskInode",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "             与一个DiskInode相关联\n通过block_id和block_offset找到对应的DiskInode"
		},
		{
			"type": "rectangle",
			"version": 133,
			"versionNonce": 1596395475,
			"isDeleted": false,
			"id": "ZWWd6FudP2H1bq-ToTiKo",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1087.3831255042523,
			"y": 246.5880552116986,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 2287.960779560622,
			"height": 920.1580017950355,
			"seed": 1023777555,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683126554375,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 89,
			"versionNonce": 1091821501,
			"isDeleted": false,
			"id": "nc5jyAc6EEcvVk3ZRfCd1",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1085.8839121322048,
			"y": 1227.9701461534055,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 2290.119894165138,
			"height": 735.8288579841658,
			"seed": 503029107,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683126569906,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 200,
			"versionNonce": 273569875,
			"isDeleted": false,
			"id": "T-s7D7aq5YFMV35Y8e7JC",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1084.2005064657828,
			"y": -290.3266259052559,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 2276.347608356956,
			"height": 302.98844614323207,
			"seed": 1515905053,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683126596365,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 52,
			"versionNonce": 201792989,
			"isDeleted": false,
			"id": "Xno2G_v58_j7UjHEgiZl9",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -729.7746278312595,
			"y": -941.4062972761537,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 1808.2852878568744,
			"height": 607.5723187185827,
			"seed": 1055446067,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683126609952,
			"link": null,
			"locked": false
		}
	],
	"appState": {
		"theme": "light",
		"viewBackgroundColor": "#ffffff",
		"currentItemStrokeColor": "#000000",
		"currentItemBackgroundColor": "transparent",
		"currentItemFillStyle": "hachure",
		"currentItemStrokeWidth": 4,
		"currentItemStrokeStyle": "dashed",
		"currentItemRoughness": 1,
		"currentItemOpacity": 100,
		"currentItemFontFamily": 1,
		"currentItemFontSize": 20,
		"currentItemTextAlign": "left",
		"currentItemStartArrowhead": null,
		"currentItemEndArrowhead": "arrow",
		"scrollX": 1758.479890998588,
		"scrollY": 56.259575973819324,
		"zoom": {
			"value": 0.3655104114115241
		},
		"currentItemRoundness": "round",
		"gridSize": null,
		"colorPalette": {},
		"currentStrokeOptions": null,
		"previousGridSize": null
	},
	"files": {}
}
```
%%