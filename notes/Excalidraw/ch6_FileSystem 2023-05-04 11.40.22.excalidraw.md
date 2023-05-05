---

excalidraw-plugin: parsed
tags: [excalidraw]

---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠==


# Text Elements
物理内存 ^XaJDI8nw

0x10001000 ^JPav19nD

0x10002000 ^qQ4fBZd1

VirtIO块设备的寄存器组映射 ^CES72MRb

0x80200000 ^wJDrwRx8

ekernel ^x667wjz7

0x80800000 ^1wF7T6L0

物理页帧 ^4ox7dln4

VirtQueue环形队列
   数据缓冲区 ^7RcrkWoa

VirtIO驱动程序 ^yeh6DllW

VirtIOBlock ^uFkVcE3N

read_block()和write_block()
都会转化成对应的与VirtIO块设备通信的函数 ^cVDfUhEa

队列描述符表 ^6wIEkNSv

队列条目Queue Entry
包含元数据和数据信息 ^VdYRtqwK

VirtIO块设备 ^nlE312od

一个条目对应一个数据缓冲内容 ^O01sVdiu

队列条目的索引 ^TtjOguLF

TaskControlBlockInner ^YSS10AvX

fd_table: 
Vec<Option<Arc<dyn File+Send+Sync>>>
文件描述符表 ^4255elP4

OSInode结构 ^PVSMyiLW

readable: bool, ^X2HFx0dr

writable: bool, ^cva3tyOt

inner: Mutex<OSInodeInner> ^gHxdvzqZ

offset: usize, ^wYroDP2Q

inode: Arc<Inode>, ^LEo5FHf4

Inode结构 ^ehLvmATN

block_id: u32, ^69WR2WKT

block_offset: usize, ^y4tDR2Xo

fs: Arc<Mutex<EasyFileSystem>>, ^9wZbW37h

block_device: Arc<dyn BlockDevice>, ^okcNCoXm

DiskInode结构 ^D8NBPS6C

// 占的字节数
pub size: u32,
// 直接索引的数组(元素为索引号)
pub direct: [u32; INODE_DIRECT_COUNT],
// 一级间接索引
pub indirect1: u32,
// 二级间接索引
pub indirect2: u32,
// 文件/目录类型
type_: DiskInodeType, ^RaHfPjAb

      实现了File Trait
包括 read(() 和 write() 函数 ^U2x7OFkV

sys_read() ^c29sWFjn

sys_write() ^A5KM6zHU

sys_open() ^cSzfojak

open_file() ^2uusjIxL

read() ^tpL4S2ey

write() ^QFcnYxRF

read_at() ^0PoMsGza

write_at() ^jUB39Aif

read_at() ^lQoZMngT

write_at() ^9sS50WCs

BlockCache结构 ^AzWDz4BJ

find_inode_id() ^HLUjwsQY

find() ^UNV71lym

ROOT_INODE ^BpjsdFC6

cache: [u8; BLOCK_SZ], ^qPPxcnL2

block_id: usize, ^cTJfVXhl

block_device: Arc<dyn BlockDevice>, ^UsWq6aWq

modified: bool, ^qh7DNPTm

write_block()
将BlockCache中的内容写回到磁盘块中
drop一个BlockCache的时候会调用 ^LcKrKzKh

read_block()
将一个磁盘块的内容读取到BlockCache中
new一个BlockCache的时候会调用 ^KWEsFJ92

read() ^NKjLL8FW

write() ^u7RtB9zX

%%
# Drawing
```json
{
	"type": "excalidraw",
	"version": 2,
	"source": "https://github.com/zsviczian/obsidian-excalidraw-plugin/releases/tag/1.8.18",
	"elements": [
		{
			"type": "rectangle",
			"version": 217,
			"versionNonce": 784665972,
			"isDeleted": false,
			"id": "bz7TlbCBg6aMjTJX09tCY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -356.7092407984955,
			"y": -171.60114196422063,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 235,
			"height": 466,
			"seed": 2089981044,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683192034522,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 207,
			"versionNonce": 1442034636,
			"isDeleted": false,
			"id": "XaJDI8nw",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -290.7854134163819,
			"y": -204.89323622631673,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 97.239990234375,
			"height": 28.799999999999997,
			"seed": 939954548,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034522,
			"link": null,
			"locked": false,
			"fontSize": 24.31662547119896,
			"fontFamily": 1,
			"text": "物理内存",
			"rawText": "物理内存",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "物理内存"
		},
		{
			"type": "line",
			"version": 204,
			"versionNonce": 2066431732,
			"isDeleted": false,
			"id": "0gjRk_gXXpkvTHXO06i-J",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -357.4132491995838,
			"y": 216.78438041705476,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 234.74322439549658,
			"height": 1.1098616044532827,
			"seed": 89029108,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034522,
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
					234.74322439549658,
					-1.1098616044532827
				]
			]
		},
		{
			"type": "line",
			"version": 223,
			"versionNonce": 258169420,
			"isDeleted": false,
			"id": "FItAJVrkajyoAXDzEddp5",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -357.9681800018104,
			"y": 145.1960806762654,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 232.5233527696865,
			"height": 0.5549308022266414,
			"seed": 171564532,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034522,
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
					232.5233527696865,
					0.5549308022266414
				]
			]
		},
		{
			"type": "line",
			"version": 200,
			"versionNonce": 248463476,
			"isDeleted": false,
			"id": "uZOQoEaDJyQ29SS9qsnPl",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -358.5231850124888,
			"y": 8.678725029858043,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 233.0783577803649,
			"height": 1.1098987086791605,
			"seed": 1718661580,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034522,
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
					233.0783577803649,
					1.1098987086791605
				]
			]
		},
		{
			"type": "text",
			"version": 153,
			"versionNonce": 2031759564,
			"isDeleted": false,
			"id": "JPav19nD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -488.46225624750696,
			"y": 206.2114717293984,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 118.39994812011719,
			"height": 24,
			"seed": 1645613940,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034522,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "0x10001000",
			"rawText": "0x10001000",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "0x10001000"
		},
		{
			"type": "text",
			"version": 176,
			"versionNonce": 1635193332,
			"isDeleted": false,
			"id": "qQ4fBZd1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -488.1823800175458,
			"y": 135.92292895705577,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 127.21994018554688,
			"height": 24,
			"seed": 1788589556,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034522,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "0x10002000",
			"rawText": "0x10002000",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "0x10002000"
		},
		{
			"type": "text",
			"version": 173,
			"versionNonce": 1655846732,
			"isDeleted": false,
			"id": "CES72MRb",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -347.9002555935597,
			"y": 172.69983055618286,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 216.3532257080078,
			"height": 19.2,
			"seed": 1266109940,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "PAqZ9mFc_d0I1531A8Y26",
					"type": "arrow"
				},
				{
					"id": "J1zIok3hxhzq_b1LF__qM",
					"type": "arrow"
				},
				{
					"id": "tR0CATOZ4O_UaIvnn_lue",
					"type": "arrow"
				},
				{
					"id": "AF81uIsTruOk-a9cTJaZl",
					"type": "arrow"
				}
			],
			"updated": 1683192034522,
			"link": null,
			"locked": false,
			"fontSize": 16.648529801497038,
			"fontFamily": 1,
			"text": "VirtIO块设备的寄存器组映射",
			"rawText": "VirtIO块设备的寄存器组映射",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "VirtIO块设备的寄存器组映射"
		},
		{
			"type": "text",
			"version": 94,
			"versionNonce": 36221812,
			"isDeleted": false,
			"id": "wJDrwRx8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -501.8967467255852,
			"y": 1.194811014543916,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 137.0999298095703,
			"height": 24,
			"seed": 233699404,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034522,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "0x80200000",
			"rawText": "0x80200000",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "0x80200000"
		},
		{
			"type": "line",
			"version": 84,
			"versionNonce": 411397580,
			"isDeleted": false,
			"id": "6gba6j3bJqD9LUG6v5Tac",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -358.52177184993775,
			"y": -42.36417680142188,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 232.40465568607885,
			"height": 1.3204830047671976,
			"seed": 1994352244,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034522,
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
					232.40465568607885,
					1.3204830047671976
				]
			]
		},
		{
			"type": "text",
			"version": 80,
			"versionNonce": 791291124,
			"isDeleted": false,
			"id": "x667wjz7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -431.84541688107186,
			"y": -54.1508770550862,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 65.75993347167969,
			"height": 24,
			"seed": 439722740,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034522,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "ekernel",
			"rawText": "ekernel",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "ekernel"
		},
		{
			"type": "line",
			"version": 91,
			"versionNonce": 1269683276,
			"isDeleted": false,
			"id": "UDNW6Jf_SdPU8EoUpT22Q",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -356.5410252707279,
			"y": -132.81710812356033,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 233.0648751164034,
			"height": 1.9807465792098355,
			"seed": 588100044,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034522,
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
					233.0648751164034,
					1.9807465792098355
				]
			]
		},
		{
			"type": "text",
			"version": 90,
			"versionNonce": 1845585524,
			"isDeleted": false,
			"id": "1wF7T6L0",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -503.84738701624997,
			"y": -146.12766333408496,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 138.15992736816406,
			"height": 24,
			"seed": 1258516980,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034522,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "0x80800000",
			"rawText": "0x80800000",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "0x80800000"
		},
		{
			"type": "text",
			"version": 257,
			"versionNonce": 2047175372,
			"isDeleted": false,
			"id": "4ox7dln4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -287.950159429481,
			"y": -124.17792763464399,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 87.91998291015625,
			"height": 25.2,
			"seed": 1073229004,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034522,
			"link": null,
			"locked": false,
			"fontSize": 21.98070243509173,
			"fontFamily": 1,
			"text": "物理页帧",
			"rawText": "物理页帧",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "物理页帧"
		},
		{
			"type": "text",
			"version": 172,
			"versionNonce": 594835444,
			"isDeleted": false,
			"id": "7RcrkWoa",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -314.07756203368444,
			"y": -86.41245802242372,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 151.48159790039062,
			"height": 40.8,
			"seed": 827168756,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "McQV67psTBXkGz95P3fev",
					"type": "arrow"
				},
				{
					"id": "tR0CATOZ4O_UaIvnn_lue",
					"type": "arrow"
				}
			],
			"updated": 1683192034522,
			"link": null,
			"locked": false,
			"fontSize": 17.485821402630307,
			"fontFamily": 1,
			"text": "VirtQueue环形队列\n   数据缓冲区",
			"rawText": "VirtQueue环形队列\n   数据缓冲区",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "VirtQueue环形队列\n   数据缓冲区"
		},
		{
			"type": "rectangle",
			"version": 274,
			"versionNonce": 1689581900,
			"isDeleted": false,
			"id": "hkWEpocr9rslYasCaBNvI",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1251.7959075896727,
			"y": -14.634033701311125,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 331.4407044671449,
			"height": 132.04807975612744,
			"seed": 1034791756,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "dpuRi0IGWrMg20LOP84oU",
					"type": "arrow"
				},
				{
					"id": "AF81uIsTruOk-a9cTJaZl",
					"type": "arrow"
				},
				{
					"id": "EEGpgZPMPbHiteZcuFEDx",
					"type": "arrow"
				}
			],
			"updated": 1683198994245,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 423,
			"versionNonce": 730421620,
			"isDeleted": false,
			"id": "yeh6DllW",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1189.0728103588185,
			"y": -5.2988329022899165,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 205.50863647460938,
			"height": 34.8,
			"seed": 1947188684,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 29.353402890384974,
			"fontFamily": 1,
			"text": "VirtIO驱动程序",
			"rawText": "VirtIO驱动程序",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "VirtIO驱动程序"
		},
		{
			"type": "text",
			"version": 255,
			"versionNonce": 795486156,
			"isDeleted": false,
			"id": "uFkVcE3N",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1168.1537191117072,
			"y": -51.767756839382116,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 156.36366271972656,
			"height": 33.6,
			"seed": 370036852,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 28.253037173177432,
			"fontFamily": 1,
			"text": "VirtIOBlock",
			"rawText": "VirtIOBlock",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "VirtIOBlock"
		},
		{
			"type": "text",
			"version": 343,
			"versionNonce": 1998063604,
			"isDeleted": false,
			"id": "cVDfUhEa",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1237.7988230544352,
			"y": 54.68578396214423,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 306.43048095703125,
			"height": 36,
			"seed": 1991467380,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192841820,
			"link": null,
			"locked": false,
			"fontSize": 15.32329855480751,
			"fontFamily": 1,
			"text": "read_block()和write_block()\n都会转化成对应的与VirtIO块设备通信的函数",
			"rawText": "read_block()和write_block()\n都会转化成对应的与VirtIO块设备通信的函数",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "read_block()和write_block()\n都会转化成对应的与VirtIO块设备通信的函数"
		},
		{
			"type": "rectangle",
			"version": 205,
			"versionNonce": 1508593228,
			"isDeleted": false,
			"id": "Yj0bBmX15lhAG-rKFqPlY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -817.0685172094769,
			"y": -86.36107426895387,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 227.12272366701006,
			"height": 330.1202656064959,
			"seed": 1317810124,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "dpuRi0IGWrMg20LOP84oU",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 134,
			"versionNonce": 133205108,
			"isDeleted": false,
			"id": "6wIEkNSv",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -788.0618586462278,
			"y": -119.50182949239894,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 156.2999267578125,
			"height": 31.2,
			"seed": 1900117196,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 26.052213771849566,
			"fontFamily": 1,
			"text": "队列描述符表",
			"rawText": "队列描述符表",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "队列描述符表"
		},
		{
			"type": "text",
			"version": 135,
			"versionNonce": 377726156,
			"isDeleted": false,
			"id": "VdYRtqwK",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -806.2908190632564,
			"y": -71.13095622690793,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 200.79989624023438,
			"height": 48,
			"seed": 1292704756,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "McQV67psTBXkGz95P3fev",
					"type": "arrow"
				},
				{
					"id": "tR0CATOZ4O_UaIvnn_lue",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "队列条目Queue Entry\n包含元数据和数据信息",
			"rawText": "队列条目Queue Entry\n包含元数据和数据信息",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "队列条目Queue Entry\n包含元数据和数据信息"
		},
		{
			"type": "line",
			"version": 107,
			"versionNonce": 1083104756,
			"isDeleted": false,
			"id": "W-OwgN0m_Y4EaPxMk29X7",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -812.446892908969,
			"y": -15.05508029976238,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 215.2384207682236,
			"height": 1.3204830047671976,
			"seed": 1791060684,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034523,
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
					215.2384207682236,
					1.3204830047671976
				]
			]
		},
		{
			"type": "freedraw",
			"version": 101,
			"versionNonce": 2060266316,
			"isDeleted": false,
			"id": "BSioN_tL7ZL4DHpNlE1Ms",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -757.6469144273082,
			"y": 105.10869655757858,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 112115276,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
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
			"version": 101,
			"versionNonce": 1147600756,
			"isDeleted": false,
			"id": "nIfNSgID-PqswREOW0TJb",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -701.5264970849983,
			"y": 105.10869655757858,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 1946709492,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
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
			"version": 101,
			"versionNonce": 1495392716,
			"isDeleted": false,
			"id": "fGeBfnDhCo8fkufedSyNY",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -646.7265186033375,
			"y": 103.12799412248688,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 2036027852,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
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
			"type": "arrow",
			"version": 273,
			"versionNonce": 733933812,
			"isDeleted": false,
			"id": "dpuRi0IGWrMg20LOP84oU",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -919.4773973340807,
			"y": 32.698481618198855,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 101.01675121615779,
			"height": 34.33251397982855,
			"seed": 843026548,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "hkWEpocr9rslYasCaBNvI",
				"focus": 0.31001991271043244,
				"gap": 1
			},
			"endBinding": {
				"elementId": "Yj0bBmX15lhAG-rKFqPlY",
				"focus": 0.5862934246440482,
				"gap": 1.3921289084459545
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
					101.01675121615779,
					-34.33251397982855
				]
			]
		},
		{
			"type": "arrow",
			"version": 219,
			"versionNonce": 681297996,
			"isDeleted": false,
			"id": "McQV67psTBXkGz95P3fev",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -601.4848011494646,
			"y": -47.19062980992027,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 285.8841070188543,
			"height": 21.1276839321568,
			"seed": 432796876,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "VdYRtqwK",
				"focus": 0.24367537774410455,
				"gap": 4.006121673557459
			},
			"endBinding": {
				"elementId": "7RcrkWoa",
				"focus": 0.30833344849253,
				"gap": 1.523132096925849
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
					285.8841070188543,
					-21.1276839321568
				]
			]
		},
		{
			"type": "rectangle",
			"version": 161,
			"versionNonce": 370979444,
			"isDeleted": false,
			"id": "D68XL-bK3XkFtEdCQGbLp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 57.50045279316657,
			"y": -78.50649915089798,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 238.8190673449352,
			"height": 270.6099465066308,
			"seed": 1206767988,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "PAqZ9mFc_d0I1531A8Y26",
					"type": "arrow"
				},
				{
					"id": "J1zIok3hxhzq_b1LF__qM",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 102,
			"versionNonce": 1830084300,
			"isDeleted": false,
			"id": "nlE312od",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 87.91262259237612,
			"y": -115.5600001857718,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 170.45672607421875,
			"height": 33.6,
			"seed": 1820059596,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 28.40005749714659,
			"fontFamily": 1,
			"text": "VirtIO块设备",
			"rawText": "VirtIO块设备",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "VirtIO块设备"
		},
		{
			"type": "arrow",
			"version": 167,
			"versionNonce": 1111901172,
			"isDeleted": false,
			"id": "PAqZ9mFc_d0I1531A8Y26",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 52.96243875481787,
			"y": 67.26622410270352,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 183.76670584615204,
			"height": 103.90182693820293,
			"seed": 1695851340,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "D68XL-bK3XkFtEdCQGbLp",
				"focus": 0.2939183247464043,
				"gap": 4.538014038348706
			},
			"endBinding": {
				"elementId": "CES72MRb",
				"focus": 0.7129604121667171,
				"gap": 1.531779515276412
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
					-183.76670584615204,
					103.90182693820293
				]
			]
		},
		{
			"type": "arrow",
			"version": 198,
			"versionNonce": 522659148,
			"isDeleted": false,
			"id": "J1zIok3hxhzq_b1LF__qM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -129.25343939074742,
			"y": 186.67575777517106,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 182.9912401529857,
			"height": 102.35105108048913,
			"seed": 1751049204,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "CES72MRb",
				"focus": 0.9437816906319847,
				"gap": 2.293590494804448
			},
			"endBinding": {
				"elementId": "D68XL-bK3XkFtEdCQGbLp",
				"focus": 0.20469101759962138,
				"gap": 3.7626520309282796
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
					182.9912401529857,
					-102.35105108048913
				]
			]
		},
		{
			"type": "arrow",
			"version": 270,
			"versionNonce": 641836404,
			"isDeleted": false,
			"id": "tR0CATOZ4O_UaIvnn_lue",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -277.60627874392753,
			"y": 164.96494761005147,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 319.45930826030474,
			"height": 184.54201601069963,
			"seed": 851421300,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "CES72MRb",
				"focus": -0.06309822482238713,
				"gap": 7.734882946131393
			},
			"endBinding": {
				"elementId": "VdYRtqwK",
				"focus": -0.43063475724649586,
				"gap": 8.42533581878979
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
					-319.45930826030474,
					-184.54201601069963
				]
			]
		},
		{
			"type": "text",
			"version": 222,
			"versionNonce": 338467788,
			"isDeleted": false,
			"id": "O01sVdiu",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 6.2007177335956145,
			"x": -580.9353165616797,
			"y": -78.36797939807813,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 209.43988037109375,
			"height": 16.8,
			"seed": 821277300,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 14.968310678037657,
			"fontFamily": 1,
			"text": "一个条目对应一个数据缓冲内容",
			"rawText": "一个条目对应一个数据缓冲内容",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "一个条目对应一个数据缓冲内容"
		},
		{
			"type": "text",
			"version": 98,
			"versionNonce": 123000564,
			"isDeleted": false,
			"id": "TtjOguLF",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -517.9270267459192,
			"y": 61.81188728545993,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 131.45999145507812,
			"height": 21.599999999999998,
			"seed": 2013464948,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 18.78154436384054,
			"fontFamily": 1,
			"text": "队列条目的索引",
			"rawText": "队列条目的索引",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "队列条目的索引"
		},
		{
			"type": "arrow",
			"version": 526,
			"versionNonce": 1715421772,
			"isDeleted": false,
			"id": "AF81uIsTruOk-a9cTJaZl",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -916.6058738320414,
			"y": 112.05530389200476,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 571.4600481601165,
			"height": 182.99129199585866,
			"seed": 479266036,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "hkWEpocr9rslYasCaBNvI",
				"focus": -0.6876312597182992,
				"gap": 3.7493292904864006
			},
			"endBinding": {
				"elementId": "CES72MRb",
				"focus": 0.7072171607748237,
				"gap": 10.100317554600963
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
					95.37263748508678,
					182.21582630269245
				],
				[
					469.8843072441747,
					182.99129199585866
				],
				[
					571.4600481601165,
					89.94484421877905
				]
			]
		},
		{
			"type": "rectangle",
			"version": 349,
			"versionNonce": 302372980,
			"isDeleted": false,
			"id": "KFxzqC9aXaz7jAbxNtzSi",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1288.2685672929333,
			"y": -611.6602795929175,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 411.33975414996416,
			"height": 201.19878737517143,
			"seed": 65774964,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 252,
			"versionNonce": 1547245772,
			"isDeleted": false,
			"id": "YSS10AvX",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1274.833885091234,
			"y": -654.0005806794064,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 376.4969482421875,
			"height": 40.8,
			"seed": 629775092,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 34.33030002718245,
			"fontFamily": 1,
			"text": "TaskControlBlockInner",
			"rawText": "TaskControlBlockInner",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "TaskControlBlockInner"
		},
		{
			"type": "freedraw",
			"version": 149,
			"versionNonce": 1914350068,
			"isDeleted": false,
			"id": "jwj68g-9HhpMOA2ypVdf-",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1153.3908192401323,
			"y": -556.4199835407816,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 1886397260,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
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
			"version": 139,
			"versionNonce": 1370758988,
			"isDeleted": false,
			"id": "3ZSRgQbKJ8ROu2HNjDoWg",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1098.2474792556577,
			"y": -558.655503479056,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 764083188,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
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
			"version": 149,
			"versionNonce": 1788431220,
			"isDeleted": false,
			"id": "lU8IHTbz1wTIozzIjSIWB",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1041.6138258611745,
			"y": -558.655503479056,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0.0001,
			"height": 0.0001,
			"seed": 722887628,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
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
			"type": "line",
			"version": 284,
			"versionNonce": 520771020,
			"isDeleted": false,
			"id": "m5usl74fAuJjxGs5iUqlK",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1279.32633807005,
			"y": -515.4350693607432,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 374.8258300232126,
			"height": 0,
			"seed": 2037733876,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034523,
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
					374.8258300232126,
					0
				]
			]
		},
		{
			"type": "text",
			"version": 260,
			"versionNonce": 1841806580,
			"isDeleted": false,
			"id": "4255elP4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1265.3696964763162,
			"y": -502.6987987440335,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 370.8597412109375,
			"height": 72,
			"seed": 1784471412,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "2UJjJ8dgDownXQbaKf4Y1",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "fd_table: \nVec<Option<Arc<dyn File+Send+Sync>>>\n文件描述符表",
			"rawText": "fd_table: \nVec<Option<Arc<dyn File+Send+Sync>>>\n文件描述符表",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "fd_table: \nVec<Option<Arc<dyn File+Send+Sync>>>\n文件描述符表"
		},
		{
			"type": "rectangle",
			"version": 214,
			"versionNonce": 22395980,
			"isDeleted": false,
			"id": "yxyM7NJvQNk4BjxofnHuZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -723.4506961941336,
			"y": -665.7723777025469,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 318.19197733568194,
			"height": 308.72723308666446,
			"seed": 1314278260,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "2UJjJ8dgDownXQbaKf4Y1",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 199,
			"versionNonce": 892075636,
			"isDeleted": false,
			"id": "PVSMyiLW",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -671.3662169599437,
			"y": -709.7565790843255,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 217.9596405029297,
			"height": 43.199999999999996,
			"seed": 1463423564,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "1WIropTvipZR5wABktTud",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 36.23524578492415,
			"fontFamily": 1,
			"text": "OSInode结构",
			"rawText": "OSInode结构",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "OSInode结构"
		},
		{
			"type": "text",
			"version": 274,
			"versionNonce": 40915660,
			"isDeleted": false,
			"id": "X2HFx0dr",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -661.2647329839095,
			"y": -637.2792930172973,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 197.8448486328125,
			"height": 32.4,
			"seed": 2095542260,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 27.965725932433656,
			"fontFamily": 1,
			"text": "readable: bool,",
			"rawText": "readable: bool,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "readable: bool,"
		},
		{
			"type": "text",
			"version": 274,
			"versionNonce": 1502351348,
			"isDeleted": false,
			"id": "cva3tyOt",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -661.2647329839095,
			"y": -589.73755893216,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 186.94046020507812,
			"height": 32.4,
			"seed": 998160716,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 27.965725932433656,
			"fontFamily": 1,
			"text": "writable: bool,",
			"rawText": "writable: bool,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "writable: bool,"
		},
		{
			"type": "text",
			"version": 245,
			"versionNonce": 1626609996,
			"isDeleted": false,
			"id": "gHxdvzqZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -701.0640814792168,
			"y": -516.1509953314255,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 278.92138671875,
			"height": 24,
			"seed": 1272795508,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 20.751345515877944,
			"fontFamily": 1,
			"text": "inner: Mutex<OSInodeInner>",
			"rawText": "inner: Mutex<OSInodeInner>",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "inner: Mutex<OSInodeInner>"
		},
		{
			"type": "text",
			"version": 253,
			"versionNonce": 1793978740,
			"isDeleted": false,
			"id": "wYroDP2Q",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -672.5162955762613,
			"y": -467.5676150543106,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 197.61041259765625,
			"height": 34.8,
			"seed": 415356492,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 29.811551625196657,
			"fontFamily": 1,
			"text": "offset: usize,",
			"rawText": "offset: usize,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "offset: usize,"
		},
		{
			"type": "text",
			"version": 281,
			"versionNonce": 2017136588,
			"isDeleted": false,
			"id": "LEo5FHf4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -697.2584224884646,
			"y": -416.26936001539684,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 259.7940673828125,
			"height": 34.8,
			"seed": 765887604,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "IRVZAWezjDlP_fL4iA4N0",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 29.81155162519666,
			"fontFamily": 1,
			"text": "inode: Arc<Inode>,",
			"rawText": "inode: Arc<Inode>,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "inode: Arc<Inode>,"
		},
		{
			"type": "line",
			"version": 191,
			"versionNonce": 144155380,
			"isDeleted": false,
			"id": "RcrwCc38CbTYxUvHR1kwX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -722.7055394891295,
			"y": -525.678433068193,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 313.1291329397027,
			"height": 2.2821931973674054,
			"seed": 1819571532,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034523,
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
					313.1291329397027,
					2.2821931973674054
				]
			]
		},
		{
			"type": "rectangle",
			"version": 309,
			"versionNonce": 677328460,
			"isDeleted": false,
			"id": "BJ8-MCE1TREPRq7qAZkfJ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -721.9602831376014,
			"y": -524.0451521566399,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 312.9757807541289,
			"height": 166.39773340764108,
			"seed": 738280180,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 181,
			"versionNonce": 2031035508,
			"isDeleted": false,
			"id": "q-iD0ImB5z5uVGmzmo3sB",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -716.3247884235277,
			"y": -521.576054459722,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 299.9988161115929,
			"height": 38.35038356351271,
			"seed": 1001526516,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false
		},
		{
			"type": "arrow",
			"version": 364,
			"versionNonce": 1044024524,
			"isDeleted": false,
			"id": "2UJjJ8dgDownXQbaKf4Y1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -893.0164280011453,
			"y": -468.81914566912883,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 168.8724936021082,
			"height": 176.1903130745818,
			"seed": 311511156,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "4255elP4",
				"focus": 0.8406638408580384,
				"gap": 1.4935272642334212
			},
			"endBinding": {
				"elementId": "yxyM7NJvQNk4BjxofnHuZ",
				"focus": 0.93744528106433,
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
					168.8724936021082,
					-176.1903130745818
				]
			]
		},
		{
			"type": "rectangle",
			"version": 345,
			"versionNonce": 1109030388,
			"isDeleted": false,
			"id": "8WDKdlyg2rGL4phAsnRi2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -246.55459851147089,
			"y": -622.8815409168324,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 541.9493323012198,
			"height": 234.59739861446124,
			"seed": 1625605580,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "IRVZAWezjDlP_fL4iA4N0",
					"type": "arrow"
				},
				{
					"id": "iIAfzzCXjehEj5ZmIx-AZ",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 372,
			"versionNonce": 1533481804,
			"isDeleted": false,
			"id": "ehLvmATN",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -99.23867297938142,
			"y": -679.4594043838999,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 225.06369018554688,
			"height": 57.599999999999994,
			"seed": 554454260,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "imYSMYwcJIKExJjtsg4Bq",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 48.071354137223516,
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
			"version": 427,
			"versionNonce": 1086297972,
			"isDeleted": false,
			"id": "69WR2WKT",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -231.17423515663182,
			"y": -595.4167198109225,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 199.33863830566406,
			"height": 33.6,
			"seed": 2120363084,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 28.94844446820056,
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
			"version": 334,
			"versionNonce": 1581173196,
			"isDeleted": false,
			"id": "y4tDR2Xo",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -231.17423515663182,
			"y": -546.2043642149815,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 282.7147521972656,
			"height": 33.6,
			"seed": 828280436,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 28.948444468200556,
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
			"version": 334,
			"versionNonce": 1095010548,
			"isDeleted": false,
			"id": "9wZbW37h",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -231.17423515663182,
			"y": -496.9920086190406,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 466.6572570800781,
			"height": 33.6,
			"seed": 74367692,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 28.948444468200556,
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
			"version": 334,
			"versionNonce": 1239134284,
			"isDeleted": false,
			"id": "okcNCoXm",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -231.17423515663182,
			"y": -447.7796530230997,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 502.97705078125,
			"height": 33.6,
			"seed": 1357684724,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 28.948444468200556,
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
			"version": 444,
			"versionNonce": 388975220,
			"isDeleted": false,
			"id": "IRVZAWezjDlP_fL4iA4N0",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -433.97491737799817,
			"y": -401.1354255736204,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 184.57999331738006,
			"height": 182.61220557194304,
			"seed": 1650762188,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "LEo5FHf4",
				"focus": 0.8888795681976789,
				"gap": 3.4894377276539217
			},
			"endBinding": {
				"elementId": "8WDKdlyg2rGL4phAsnRi2",
				"focus": 0.9057464389546455,
				"gap": 2.8403255491472237
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
					184.57999331738006,
					-182.61220557194304
				]
			]
		},
		{
			"type": "rectangle",
			"version": 593,
			"versionNonce": 773310156,
			"isDeleted": false,
			"id": "IzLZc_8BbhB3TL-X0K2Bt",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 456.6592883604808,
			"y": -653.5575698799414,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 469.00111487481263,
			"height": 285.4369988162633,
			"seed": 844604492,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 465,
			"versionNonce": 893770740,
			"isDeleted": false,
			"id": "D8NBPS6C",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 564.2013643815105,
			"y": -694.6372770598883,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 221.93267822265625,
			"height": 39.6,
			"seed": 2073859700,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "qp7fxn1jH3wb0ze0QXVzC",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
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
			"version": 485,
			"versionNonce": 260902220,
			"isDeleted": false,
			"id": "RaHfPjAb",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 468.0238919982976,
			"y": -626.001072749692,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 445.7596740722656,
			"height": 240,
			"seed": 183843532,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "iIAfzzCXjehEj5ZmIx-AZ",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
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
			"version": 347,
			"versionNonce": 1166949748,
			"isDeleted": false,
			"id": "iIAfzzCXjehEj5ZmIx-AZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 297.74747473323964,
			"y": -562.8186964476382,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 160.6483868899561,
			"height": 64.49383903373155,
			"seed": 1933600076,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "8WDKdlyg2rGL4phAsnRi2",
				"focus": 0.2321882154231337,
				"gap": 2.352740943490744
			},
			"endBinding": {
				"elementId": "RaHfPjAb",
				"focus": 1.0247126024321662,
				"gap": 9.628030375101844
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
					160.6483868899561,
					-64.49383903373155
				]
			]
		},
		{
			"type": "text",
			"version": 127,
			"versionNonce": 1335198668,
			"isDeleted": false,
			"id": "U2x7OFkV",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -721.1192218238145,
			"y": -347.5186864947451,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 309.241455078125,
			"height": 52.8,
			"seed": 862789324,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 22.904596793774765,
			"fontFamily": 1,
			"text": "      实现了File Trait\n包括 read(() 和 write() 函数",
			"rawText": "      实现了File Trait\n包括 read(() 和 write() 函数",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "      实现了File Trait\n包括 read(() 和 write() 函数"
		},
		{
			"type": "text",
			"version": 145,
			"versionNonce": 1095659252,
			"isDeleted": false,
			"id": "c29sWFjn",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1272.3772936126186,
			"y": -1029.958132202282,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 192.27804565429688,
			"height": 42,
			"seed": 1056408780,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "gCXNCVA89JjwZTsItEnlg",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "sys_read()",
			"rawText": "sys_read()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "sys_read()"
		},
		{
			"type": "text",
			"version": 226,
			"versionNonce": 340470348,
			"isDeleted": false,
			"id": "A5KM6zHU",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1270.4916999857605,
			"y": -945.4919592960312,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 197.9237518310547,
			"height": 42,
			"seed": 1090850036,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "v8e8uPOrF2DUtvFBUjlUZ",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "sys_write()",
			"rawText": "sys_write()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "sys_write()"
		},
		{
			"type": "text",
			"version": 211,
			"versionNonce": 151810164,
			"isDeleted": false,
			"id": "cSzfojak",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1267.1928135212086,
			"y": -1117.2275258012496,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 186.81211853027344,
			"height": 42,
			"seed": 2086060532,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "Vtj6ijyh5OUyOepvOwI0a",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "sys_open()",
			"rawText": "sys_open()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "sys_open()"
		},
		{
			"type": "text",
			"version": 324,
			"versionNonce": 814219468,
			"isDeleted": false,
			"id": "2uusjIxL",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1001.2200875515953,
			"y": -1117.4208954081573,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 185.37368774414062,
			"height": 42,
			"seed": 564020556,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "Vtj6ijyh5OUyOepvOwI0a",
					"type": "arrow"
				},
				{
					"id": "nygXGPyQRtVAEE7S0S_mf",
					"type": "arrow"
				}
			],
			"updated": 1683192034523,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "open_file()",
			"rawText": "open_file()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "open_file()"
		},
		{
			"type": "text",
			"version": 227,
			"versionNonce": 2089697780,
			"isDeleted": false,
			"id": "tpL4S2ey",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -643.9143273151963,
			"y": -1028.2868799011171,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 106.65730285644531,
			"height": 42,
			"seed": 1691918796,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "gCXNCVA89JjwZTsItEnlg",
					"type": "arrow"
				},
				{
					"id": "AY32JmkXz9ovQm8Aax_BV",
					"type": "arrow"
				}
			],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "read()",
			"rawText": "read()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "read()"
		},
		{
			"type": "text",
			"version": 193,
			"versionNonce": 199624524,
			"isDeleted": false,
			"id": "QFcnYxRF",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -640.1305909224814,
			"y": -943.2641685844949,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 112.30300903320312,
			"height": 42,
			"seed": 296054644,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "v8e8uPOrF2DUtvFBUjlUZ",
					"type": "arrow"
				},
				{
					"id": "Z6ytAibtLMgXJaiBD8f4n",
					"type": "arrow"
				}
			],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "write()",
			"rawText": "write()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "write()"
		},
		{
			"type": "text",
			"version": 276,
			"versionNonce": 316269428,
			"isDeleted": false,
			"id": "0PoMsGza",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -144.169376281003,
			"y": -1031.4613162604765,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 180.66297912597656,
			"height": 42,
			"seed": 330526028,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "AY32JmkXz9ovQm8Aax_BV",
					"type": "arrow"
				},
				{
					"id": "5v1VioWzuj5OLdI_SqFQB",
					"type": "arrow"
				}
			],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "read_at()",
			"rawText": "read_at()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "read_at()"
		},
		{
			"type": "text",
			"version": 197,
			"versionNonce": 180305356,
			"isDeleted": false,
			"id": "jUB39Aif",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -141.4913880367535,
			"y": -950.425275295348,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 186.30868530273438,
			"height": 42,
			"seed": 1612440436,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "Z6ytAibtLMgXJaiBD8f4n",
					"type": "arrow"
				},
				{
					"id": "23jv0ry5fi4WfJwBR9eKI",
					"type": "arrow"
				}
			],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "write_at()",
			"rawText": "write_at()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "write_at()"
		},
		{
			"type": "text",
			"version": 281,
			"versionNonce": 509091316,
			"isDeleted": false,
			"id": "lQoZMngT",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 549.1651262362292,
			"y": -1027.6709979251443,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 180.66297912597656,
			"height": 42,
			"seed": 2109213940,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "5v1VioWzuj5OLdI_SqFQB",
					"type": "arrow"
				},
				{
					"id": "13Ioj1vPkwA0ORC3qz-Ck",
					"type": "arrow"
				},
				{
					"id": "K33mxVODod_4DFDlYvvKj",
					"type": "arrow"
				}
			],
			"updated": 1683192563086,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "read_at()",
			"rawText": "read_at()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "read_at()"
		},
		{
			"type": "text",
			"version": 201,
			"versionNonce": 1601132788,
			"isDeleted": false,
			"id": "9sS50WCs",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 551.8431144804787,
			"y": -946.6349569600159,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 186.30868530273438,
			"height": 42,
			"seed": 1666346060,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "23jv0ry5fi4WfJwBR9eKI",
					"type": "arrow"
				},
				{
					"id": "humv1afur4kb82GF85ytk",
					"type": "arrow"
				}
			],
			"updated": 1683192591600,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "write_at()",
			"rawText": "write_at()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "write_at()"
		},
		{
			"type": "rectangle",
			"version": 399,
			"versionNonce": 2054527820,
			"isDeleted": false,
			"id": "DD3WmFxBugO4jvTqZrokT",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1023.3327358310892,
			"y": -623.5717517589113,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 383.23471704732924,
			"height": 204.98567015859624,
			"seed": 2072523380,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "EEGpgZPMPbHiteZcuFEDx",
					"type": "arrow"
				},
				{
					"id": "m4gU-hJ1aTBLyPx2Mtz6n",
					"type": "arrow"
				}
			],
			"updated": 1683192379223,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 351,
			"versionNonce": 1374981324,
			"isDeleted": false,
			"id": "AzWDz4BJ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1079.0269903266746,
			"y": -667.1706922049742,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 245.98699951171875,
			"height": 39.6,
			"seed": 1414663244,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "aFpzy_P2UVpPXMDjM-wZb",
					"type": "arrow"
				}
			],
			"updated": 1683192599210,
			"link": null,
			"locked": false,
			"fontSize": 33.30283568552756,
			"fontFamily": 1,
			"text": "BlockCache结构",
			"rawText": "BlockCache结构",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "BlockCache结构"
		},
		{
			"type": "arrow",
			"version": 264,
			"versionNonce": 35817844,
			"isDeleted": false,
			"id": "gCXNCVA89JjwZTsItEnlg",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1073.0705064444446,
			"y": -1009.7867207731913,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 421.23344923249056,
			"height": 0.8380606201722003,
			"seed": 1188465228,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "c29sWFjn",
				"focus": -0.04878635423505938,
				"gap": 7.028741513877094
			},
			"endBinding": {
				"elementId": "tpL4S2ey",
				"focus": 0.0729608143401867,
				"gap": 7.922729896757801
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
					421.23344923249056,
					0.8380606201722003
				]
			]
		},
		{
			"type": "arrow",
			"version": 245,
			"versionNonce": 238811084,
			"isDeleted": false,
			"id": "v8e8uPOrF2DUtvFBUjlUZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1071.4564309157577,
			"y": -928.2909196741302,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 425.26847618262343,
			"height": 1.6722103397829642,
			"seed": 234909132,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "A5KM6zHU",
				"focus": -0.19600898533723243,
				"gap": 1.1115172389481813
			},
			"endBinding": {
				"elementId": "QFcnYxRF",
				"focus": 0.19367452969756355,
				"gap": 6.057363810652873
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
					425.26847618262343,
					1.6722103397829642
				]
			]
		},
		{
			"type": "arrow",
			"version": 239,
			"versionNonce": 2105096948,
			"isDeleted": false,
			"id": "AY32JmkXz9ovQm8Aax_BV",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -533.5742169692276,
			"y": -1010.569321814771,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 377.2067194297181,
			"height": 0.7912529056555968,
			"seed": 441693388,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "tpL4S2ey",
				"focus": -0.16114316270603568,
				"gap": 3.682807489523384
			},
			"endBinding": {
				"elementId": "0PoMsGza",
				"focus": -0.04239461256116361,
				"gap": 12.19812125850649
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
					377.2067194297181,
					0.7912529056555968
				]
			]
		},
		{
			"type": "arrow",
			"version": 268,
			"versionNonce": 503520844,
			"isDeleted": false,
			"id": "Z6ytAibtLMgXJaiBD8f4n",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -523.8900875402751,
			"y": -922.6422809517413,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 379.6277787655538,
			"height": 3.950886704879508,
			"seed": 1768215412,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "QFcnYxRF",
				"focus": 0.011455028899094033,
				"gap": 3.9374943490031455
			},
			"endBinding": {
				"elementId": "jUB39Aif",
				"focus": -0.08346979051304891,
				"gap": 2.770920737967799
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
					379.6277787655538,
					-3.950886704879508
				]
			]
		},
		{
			"type": "arrow",
			"version": 238,
			"versionNonce": 234732660,
			"isDeleted": false,
			"id": "5v1VioWzuj5OLdI_SqFQB",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 44.63071544050615,
			"y": -1015.7705088730015,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 501.5797678042545,
			"height": 8.10705618386271,
			"seed": 1703169908,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "0PoMsGza",
				"focus": -0.3072455576652422,
				"gap": 8.137112595532585
			},
			"endBinding": {
				"elementId": "lQoZMngT",
				"focus": -0.02294445369719067,
				"gap": 2.9546429914685177
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
					501.5797678042545,
					8.10705618386271
				]
			]
		},
		{
			"type": "arrow",
			"version": 244,
			"versionNonce": 656504012,
			"isDeleted": false,
			"id": "23jv0ry5fi4WfJwBR9eKI",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 49.47283411217734,
			"y": -931.0340357708567,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 495.9306653254346,
			"height": 6.4880595637791885,
			"seed": 551236212,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "jUB39Aif",
				"focus": -0.12999713589822692,
				"gap": 4.655536846196469
			},
			"endBinding": {
				"elementId": "9sS50WCs",
				"focus": -0.10765382410234099,
				"gap": 6.4396150428667625
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
					495.9306653254346,
					6.4880595637791885
				]
			]
		},
		{
			"type": "arrow",
			"version": 236,
			"versionNonce": 1836453364,
			"isDeleted": false,
			"id": "Vtj6ijyh5OUyOepvOwI0a",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1075.645815599898,
			"y": -1098.7031814976833,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 73.42572804830274,
			"height": 0.5894827937263472,
			"seed": 532770636,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "cSzfojak",
				"focus": -0.07759818825236038,
				"gap": 4.734879391037111
			},
			"endBinding": {
				"elementId": "2uusjIxL",
				"focus": 0.16666180546858347,
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
					73.42572804830274,
					-0.5894827937263472
				]
			]
		},
		{
			"type": "arrow",
			"version": 419,
			"versionNonce": 1462827852,
			"isDeleted": false,
			"id": "nygXGPyQRtVAEE7S0S_mf",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -809.0815053256764,
			"y": -1099.5682666243124,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 590.0268058139898,
			"height": 5.0614696838265445,
			"seed": 1639312756,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "2uusjIxL",
				"focus": -0.10526388625107001,
				"gap": 6.764894481778242
			},
			"endBinding": {
				"elementId": "UNV71lym",
				"focus": 0.44310394696022865,
				"gap": 8.936692934125858
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
					590.0268058139898,
					-5.0614696838265445
				]
			]
		},
		{
			"type": "text",
			"version": 448,
			"versionNonce": 753414004,
			"isDeleted": false,
			"id": "HLUjwsQY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -18.108488515421072,
			"y": -1119.7741673437479,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 262.14825439453125,
			"height": 42,
			"seed": 327360844,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "ArAukZSUCGDnDVix1HmNM",
					"type": "arrow"
				},
				{
					"id": "13Ioj1vPkwA0ORC3qz-Ck",
					"type": "arrow"
				}
			],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "find_inode_id()",
			"rawText": "find_inode_id()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "find_inode_id()"
		},
		{
			"type": "text",
			"version": 323,
			"versionNonce": 411808204,
			"isDeleted": false,
			"id": "UNV71lym",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -210.11800657756078,
			"y": -1116.6154380881699,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 89.68417358398438,
			"height": 42,
			"seed": 1439148788,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "nygXGPyQRtVAEE7S0S_mf",
					"type": "arrow"
				},
				{
					"id": "ArAukZSUCGDnDVix1HmNM",
					"type": "arrow"
				}
			],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "find()",
			"rawText": "find()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "find()"
		},
		{
			"type": "arrow",
			"version": 245,
			"versionNonce": 1285169396,
			"isDeleted": false,
			"id": "ArAukZSUCGDnDVix1HmNM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -113.50868303227685,
			"y": -1101.2840886881463,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 92.699643424236,
			"height": 1.1710568588850947,
			"seed": 1672724300,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "UNV71lym",
				"focus": -0.23252216650297672,
				"gap": 6.925149961299553
			},
			"endBinding": {
				"elementId": "HLUjwsQY",
				"focus": 0.23706600387718468,
				"gap": 2.7005510926197758
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
					92.699643424236,
					-1.1710568588850947
				]
			]
		},
		{
			"type": "arrow",
			"version": 243,
			"versionNonce": 2051774540,
			"isDeleted": false,
			"id": "13Ioj1vPkwA0ORC3qz-Ck",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 247.41164355327146,
			"y": -1100.594484720436,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 297.89714528334196,
			"height": 80.07972651715886,
			"seed": 1290052084,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "HLUjwsQY",
				"focus": -0.6750547952407691,
				"gap": 3.371877674161283
			},
			"endBinding": {
				"elementId": "lQoZMngT",
				"focus": -0.25341957011807253,
				"gap": 3.856337399615768
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
					297.89714528334196,
					80.07972651715886
				]
			]
		},
		{
			"type": "text",
			"version": 101,
			"versionNonce": 1410458228,
			"isDeleted": false,
			"id": "BpjsdFC6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -249.2508876264351,
			"y": -1082.636444295367,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 161.82823181152344,
			"height": 26.4,
			"seed": 1546646476,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"fontSize": 22.702315131789707,
			"fontFamily": 1,
			"text": "ROOT_INODE",
			"rawText": "ROOT_INODE",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "ROOT_INODE"
		},
		{
			"type": "rectangle",
			"version": 133,
			"versionNonce": 407547596,
			"isDeleted": false,
			"id": "Wd2GK0rWfvBVsJcqRRvdj",
			"fillStyle": "hachure",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1330.5718888709216,
			"y": -226.85505140467887,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 1687.400983314245,
			"height": 576.0826129754845,
			"seed": 1744792140,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 314,
			"versionNonce": 1779790836,
			"isDeleted": false,
			"id": "s4z2mrPsmN1v8WrhygNC8",
			"fillStyle": "hachure",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1327.661429238352,
			"y": -740.0587842141961,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 2754.9364387533833,
			"height": 455.30091117184315,
			"seed": 346848204,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 144,
			"versionNonce": 750092748,
			"isDeleted": false,
			"id": "3j4fUJn7-KurJvnvImBqg",
			"fillStyle": "hachure",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1332.066329397707,
			"y": -1198.0360558348289,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 2752.6480187751695,
			"height": 386.349272581135,
			"seed": 1300239948,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683192058780,
			"link": null,
			"locked": false
		},
		{
			"type": "arrow",
			"version": 48,
			"versionNonce": 1823135092,
			"isDeleted": false,
			"id": "1WIropTvipZR5wABktTud",
			"fillStyle": "hachure",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -570.6847307609596,
			"y": -722.1036359085183,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 1.2951998236193276,
			"height": 134.53883579799106,
			"seed": 339747660,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "PVSMyiLW",
				"focus": -0.07899365978584746,
				"gap": 12.347056824192805
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
					1.2951998236193276,
					-134.53883579799106
				]
			]
		},
		{
			"type": "arrow",
			"version": 69,
			"versionNonce": 674130892,
			"isDeleted": false,
			"id": "imYSMYwcJIKExJjtsg4Bq",
			"fillStyle": "hachure",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -30.795780937340396,
			"y": -684.5010968460183,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 1.3040597098213311,
			"height": 177.3577880859375,
			"seed": 440530676,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ehLvmATN",
				"focus": -0.3888480042288606,
				"gap": 5.041692462118306
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
					-1.3040597098213311,
					-177.3577880859375
				]
			]
		},
		{
			"type": "arrow",
			"version": 48,
			"versionNonce": 152569588,
			"isDeleted": false,
			"id": "qp7fxn1jH3wb0ze0QXVzC",
			"fillStyle": "hachure",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 668.2025449555169,
			"y": -697.5421299096344,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 1.3042340959823377,
			"height": 157.79628208705356,
			"seed": 1275383628,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192034524,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "D8NBPS6C",
				"focus": -0.06098710094694672,
				"gap": 2.904852849746078
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
					-1.3042340959823377,
					-157.79628208705356
				]
			]
		},
		{
			"type": "text",
			"version": 134,
			"versionNonce": 561562996,
			"isDeleted": false,
			"id": "qPPxcnL2",
			"fillStyle": "hachure",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1039.1079188177675,
			"y": -583.7622237784369,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 253.81295776367188,
			"height": 24,
			"seed": 434186444,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192142953,
			"link": null,
			"locked": false,
			"fontSize": 20.581762873316716,
			"fontFamily": 1,
			"text": "cache: [u8; BLOCK_SZ],",
			"rawText": "cache: [u8; BLOCK_SZ],",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "cache: [u8; BLOCK_SZ],"
		},
		{
			"type": "text",
			"version": 134,
			"versionNonce": 1668774860,
			"isDeleted": false,
			"id": "cTJfVXhl",
			"fillStyle": "hachure",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1039.1079188177675,
			"y": -548.7732268937986,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 151.79794311523438,
			"height": 24,
			"seed": 1320917492,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192142953,
			"link": null,
			"locked": false,
			"fontSize": 20.581762873316716,
			"fontFamily": 1,
			"text": "block_id: usize,",
			"rawText": "block_id: usize,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "block_id: usize,"
		},
		{
			"type": "text",
			"version": 134,
			"versionNonce": 1319630580,
			"isDeleted": false,
			"id": "UsWq6aWq",
			"fillStyle": "hachure",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1039.1079188177675,
			"y": -513.78423000916,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 357.6800842285156,
			"height": 24,
			"seed": 1071263564,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192142953,
			"link": null,
			"locked": false,
			"fontSize": 20.581762873316716,
			"fontFamily": 1,
			"text": "block_device: Arc<dyn BlockDevice>,",
			"rawText": "block_device: Arc<dyn BlockDevice>,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "block_device: Arc<dyn BlockDevice>,"
		},
		{
			"type": "text",
			"version": 134,
			"versionNonce": 1227760204,
			"isDeleted": false,
			"id": "qh7DNPTm",
			"fillStyle": "hachure",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1039.1079188177675,
			"y": -478.7952331245217,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 137.3507843017578,
			"height": 24,
			"seed": 1704737652,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192142953,
			"link": null,
			"locked": false,
			"fontSize": 20.581762873316716,
			"fontFamily": 1,
			"text": "modified: bool,",
			"rawText": "modified: bool,",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "modified: bool,"
		},
		{
			"type": "arrow",
			"version": 592,
			"versionNonce": 2043151692,
			"isDeleted": false,
			"id": "EEGpgZPMPbHiteZcuFEDx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1166.4808016261634,
			"y": 90.27310693612401,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 2251,
			"height": 801,
			"seed": 1045166284,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "KWEsFJ92"
				}
			],
			"updated": 1683198990366,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "DD3WmFxBugO4jvTqZrokT",
				"focus": 0.15937546966484836,
				"gap": 8.232200009288306
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					-10.83974898126462
				],
				[
					449.22670490274186,
					300.37301147284904
				],
				[
					1654.9314730784242,
					257.5573947809903
				],
				[
					2251,
					-500.62698852715084
				]
			]
		},
		{
			"type": "text",
			"version": 93,
			"versionNonce": 817720692,
			"isDeleted": false,
			"id": "KWEsFJ92",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -266.0363504384891,
			"y": 438.18732790369506,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 367.73992919921875,
			"height": 72,
			"seed": 356917364,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192825590,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "read_block()\n将一个磁盘块的内容读取到BlockCache中\nnew一个BlockCache的时候会调用",
			"rawText": "read_block()\n将一个磁盘块的内容读取到BlockCache中\nnew一个BlockCache的时候会调用",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "EEGpgZPMPbHiteZcuFEDx",
			"originalText": "read_block()\n将一个磁盘块的内容读取到BlockCache中\nnew一个BlockCache的时候会调用"
		},
		{
			"type": "arrow",
			"version": 446,
			"versionNonce": 1072508916,
			"isDeleted": false,
			"id": "m4gU-hJ1aTBLyPx2Mtz6n",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1148.0215607008824,
			"y": -410.3348081055253,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 2373,
			"height": 908,
			"seed": 629727308,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"type": "text",
					"id": "LcKrKzKh"
				}
			],
			"updated": 1683198994245,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "DD3WmFxBugO4jvTqZrokT",
				"focus": -0.023178457250666324,
				"gap": 8.251273494789814
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0.1402272044165329,
					0
				],
				[
					-559.0083197035523,
					850.6855799314117
				],
				[
					-1919.5952509772585,
					908
				],
				[
					-2372.859772795583,
					521.1269460206839
				]
			]
		},
		{
			"type": "text",
			"version": 125,
			"versionNonce": 1968145524,
			"isDeleted": false,
			"id": "LcKrKzKh",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -274.78455996476146,
			"y": 531.4981461137688,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 347.73992919921875,
			"height": 72,
			"seed": 1366855668,
			"groupIds": [],
			"roundness": null,
			"boundElements": [],
			"updated": 1683192836957,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "write_block()\n将BlockCache中的内容写回到磁盘块中\ndrop一个BlockCache的时候会调用",
			"rawText": "write_block()\n将BlockCache中的内容写回到磁盘块中\ndrop一个BlockCache的时候会调用",
			"textAlign": "center",
			"verticalAlign": "middle",
			"containerId": "m4gU-hJ1aTBLyPx2Mtz6n",
			"originalText": "write_block()\n将BlockCache中的内容写回到磁盘块中\ndrop一个BlockCache的时候会调用"
		},
		{
			"type": "arrow",
			"version": 129,
			"versionNonce": 1565673972,
			"isDeleted": false,
			"id": "K33mxVODod_4DFDlYvvKj",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 735.6790435828464,
			"y": -1014.8076641637981,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 402.07160750658386,
			"height": 3.745283758896676,
			"seed": 1813624180,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192576094,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "lQoZMngT",
				"focus": -0.4136334344460986,
				"gap": 5.850938220640614
			},
			"endBinding": {
				"elementId": "NKjLL8FW",
				"focus": -0.13547437378561775,
				"gap": 6.2105221254951175
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
					402.07160750658386,
					3.745283758896676
				]
			]
		},
		{
			"type": "text",
			"version": 338,
			"versionNonce": 1941010548,
			"isDeleted": false,
			"id": "NKjLL8FW",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1143.9611732149253,
			"y": -1034.4200343392868,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 106.65730285644531,
			"height": 42,
			"seed": 1786224332,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "K33mxVODod_4DFDlYvvKj",
					"type": "arrow"
				}
			],
			"updated": 1683192576093,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "read()",
			"rawText": "read()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "read()"
		},
		{
			"type": "text",
			"version": 363,
			"versionNonce": 1199577716,
			"isDeleted": false,
			"id": "u7RtB9zX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1141.9288841840962,
			"y": -949.013962169964,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 112.30300903320312,
			"height": 42,
			"seed": 1306323828,
			"groupIds": [],
			"roundness": null,
			"boundElements": [
				{
					"id": "humv1afur4kb82GF85ytk",
					"type": "arrow"
				}
			],
			"updated": 1683192591600,
			"link": null,
			"locked": false,
			"fontSize": 35.96343840011534,
			"fontFamily": 1,
			"text": "write()",
			"rawText": "write()",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "write()"
		},
		{
			"type": "arrow",
			"version": 54,
			"versionNonce": 344318028,
			"isDeleted": false,
			"id": "humv1afur4kb82GF85ytk",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 738.48084885085,
			"y": -931.0623959907442,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 395.0670943365749,
			"height": 2.801898936272778,
			"seed": 270950604,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192591600,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "9sS50WCs",
				"focus": -0.28117518317231865,
				"gap": 1
			},
			"endBinding": {
				"elementId": "u7RtB9zX",
				"focus": -0.00986729799138339,
				"gap": 8.38094099667137
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
					395.0670943365749,
					2.801898936272778
				]
			]
		},
		{
			"type": "arrow",
			"version": 55,
			"versionNonce": 984949876,
			"isDeleted": false,
			"id": "aFpzy_P2UVpPXMDjM-wZb",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1196.5906224194273,
			"y": -670.4861695904543,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 0,
			"height": 196.13264453428565,
			"seed": 1123855348,
			"groupIds": [],
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1683192599210,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "AzWDz4BJ",
				"focus": -0.04414759864452151,
				"gap": 3.3154773854801647
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
					0,
					-196.13264453428565
				]
			]
		},
		{
			"type": "rectangle",
			"version": 171,
			"versionNonce": 1415156,
			"isDeleted": false,
			"id": "RBNnHeMbg7s0pdEoyYAXp",
			"fillStyle": "hachure",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1342.052202748895,
			"y": -1210.8351560293677,
			"strokeColor": "#000000",
			"backgroundColor": "transparent",
			"width": 2781.5276249854955,
			"height": 940.9628991280049,
			"seed": 1532015180,
			"groupIds": [],
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1683192920732,
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
		"currentItemStrokeWidth": 1,
		"currentItemStrokeStyle": "solid",
		"currentItemRoughness": 1,
		"currentItemOpacity": 100,
		"currentItemFontFamily": 1,
		"currentItemFontSize": 20,
		"currentItemTextAlign": "left",
		"currentItemStartArrowhead": null,
		"currentItemEndArrowhead": "arrow",
		"scrollX": 1477.7797407025757,
		"scrollY": 1199.4805341151653,
		"zoom": {
			"value": 0.35000000000000003
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