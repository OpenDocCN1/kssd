
``` 
    WORD    NumberOfRelocations;
    WORD    NumberOfLinenumbers;
    DWORD   Characteristics;
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

## 关键字段

### 1. `.text`

1.1 `BYTE Name[IMAGE_SIZEOF_SHORT_NAME];` ==> `.text` ==> `2E 74 65 78 74 00 00 00`（区段名称）

1.2 `DWORD VirtualSize;` ==> `22h` ==> `22 00 00 00`（有效的OpCode数目）

1.3 `DWORD VirtualAddress;` ==> `1000h` ==> `00 10 00 00`（区段rva：`200h`映射到内存中为`1000h`）

1.4 `DWORD SizeOfRawData;` ==> `200h` ==> `00 02 00 00`（在文件中的对齐后大小）

1.5 `DWORD PointerToRawData;` ==> `200h` ==> `00 02 00 00`（在文件中相对与0的偏移）

1.6 `DWORD Characteristics;` ==> `E00000E0` ==> `E0 00 00 E0`（区段属性可以自定义）

### 2. `.rdata`

1.1 `BYTE Name[IMAGE_SIZEOF_SHORT_NAME];` ==> `.rdata` ==> `2E 72 64 61 74 61 00 00`（区段名称）

1.2 `DWORD VirtualSize;` ==> `7Eh` ==> `7E 00 00 00`（有效的字节数目）

1.3 `DWORD VirtualAddress;` ==> `2000h` ==> `00 20 00 00`（区段rva：`400h`映射到内存中为`2000h`）

1.4 `DWORD SizeOfRawData;` ==> `200h` ==> `00 02 00 00`（在文件中的对齐后大小）

1.5 `DWORD PointerToRawData;` ==> `400h` ==> `00 04 00 00`（在文件中相对与0的偏移）

1.6 `DWORD Characteristics;` ==> `E00000E0` ==> `E0 00 00 E0`（区段属性可以自定义）

---

``` 
2E 74 65 78 74 00 00 00
22 00 00 00
00 10 00 00
00 02 00 00
00 02 00 00
00 00 00 00
00 00 00 00
00 00
00 00
E0 00 00 E0
2E 72 64 61 74 61 00 00
7E 00 00 00
00 20 00 00
00 02 00 00
00 04 00 00
00 00 00 00
00 00 00 00
00 00
00 00
E0 00 00 E0
```

---

tip:200对齐其余填0

## 2. 组成身体

### 2.1 区段1 `.text`

伪代码如下：

``` 
push 0x0;
push 0x0;
push 0x0;
push 0x0;
call MessageBoxA;
push 0x0;
call ExitProcess;
```

vs程序IAT调用为 `FF 15 XXXXXXXX`（IAT地址）（有2个dll）

`IAT1:FOA:400h ==> RVA: 2000h ==>VA:400000h + 2000h==>402000h`

`IAT2:FOA:408h ==> RVA: 2008h ==>VA:400000h + 2008h==>402008h`

---

``` 
6A 00
6A 00
6A 00
6A 00
FF 15 00 20 40 00
6A 00
FF 15 08 20 40 00
```

tip:200对齐其余填0

### 2.2 区段2 `.rdata`

![](https://bbs.kanxue.com/upload/attach/201804/798024_CD8XEJ3MP4NE9EU.png)

#### 1. IAT

``` 
typedef struct _IMAGE_THUNK_DATA {
    union {
        DWORD   ForwarderString;
        DWORD   Function;
        DWORD   Ordinal;
        DWORD   AddressOfData;
    } u1;
} IMAGE_THUNK_DATA;
```

**关键字段**

1.`DWORD AddressOfData1;` ==> `MessageBoxA` ==> `5C 20 00 00 00 00 00 00`（以0结尾）

2.`DWORD AddressOfData2;` ==> `ExitProcess` ==> `75 20 00 00 00 00 00 00`（以0结尾）

tip:加载到内存后IAT里面的内容更新为API的为VA,不在和INT的内容相同。

#### 2. Import Table

``` 
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD   Characteristics;            // 0 for terminating null import descriptor
        DWORD   OriginalFirstThunk;         // RVA to original unbound IAT (PIMAGE_THUNK_DATA)
    } DUMMYUNIONNAME;
    DWORD   TimeDateStamp;                  // 0 if not bound,
    DWORD   ForwarderChain;                 // -1 if no forwarders
    DWORD   Name;
    DWORD   FirstThunk;                     // RVA to IAT (if bound this IAT has actual addresses)
} IMAGE_IMPORT_DESCRIPTOR;
```

#### 3. INT

``` 
typedef struct _IMAGE_THUNK_DATA {
    union {
        DWORD   ForwarderString;
        DWORD   Function;
        DWORD   Ordinal;
        DWORD   AddressOfData;
    } u1;
} IMAGE_THUNK_DATA;
```

**关键字段**

1.`DWORD AddressOfData1;` ==> `MessageBoxA` ==> `5C 20 00 00 00 00 00 00`（以0结尾）

2.`DWORD AddressOfData2;` ==> `ExitProcess` ==> `75 20 00 00 00 00 00 00`（以0结尾）

tip:75、5C为字节偏移。

#### 4. HitName

``` 
typedef struct _IMAGE_IMPORT_BY_NAME {
    WORD    Hint;
    CHAR   Name[1];
} IMAGE_IMPORT_BY_NAME, *PIMAGE_IMPORT_BY_NAME;
```

**关键字段**

1.`MessageBoxA`

1.1 `WORD Hint;` ==> API名称在DLL中的编号==> `00 00`（名称导入）

1.2 `CHAR Name[1];` ==> MessageBoxA名称 ==> `4D 65 73 73 61 67 65 42 6F 78 41`

==> `USER32.dll` ==> `00 55 53 45 52 33 32 2E 64 6C 6C 00`

2.`ExitProcess`

2.1 `WORD Hint;` ==> API名称在DLL中的编号==> `00 00`（名称导入）

2.2 `CHAR Name[1];` ==> ExitProcess名称 ==> `45 78 69 74 50 72 6F 63 65 73 73`

==> `KERNEL32.dll` ==> `00 4B 45 52 4E 45 4C 33 32 2E 64 6C 6C 00`

tip:DLL函数结尾处，会加以DLL名称结尾

---

``` 
5C 20 00 00 00 00 00 00
75 20 00 00 00 00 00 00
4C 20 00 00
00 00 00 00
00 00 00 00
6A 20 00 00
00 20 00 00
54 20 00 00
00 00 00 00
00 00 00 00
83 20 00 00
08 20 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
5C 20 00 00 00 00 00 00
75 20 00 00 00 00 00 00
00 00
4D 65 73 73 61 67 65 42 6F 78 41
00 55 53 45 52 33 32 2E 64 6C 6C 00
00 00
45 78 69 74 50 72 6F 63 65 73 73
00 4B 45 52 4E 45 4C 33 32 2E 64 6C 6C 00
```

tip:200对齐其余填0

## 结语

以上将16进制码复制到010Editor中保存为.exe格式即可运行,运行环境为win10,并未考虑兼容性的问题，发帖的本意为0到1质变，1往后的变化希望大家自行发挥，例如：添加新的区段，在弹窗中添加文字，将代码复杂化等等。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2018-11-20 16:25 被Weaving编辑 ，原因：
