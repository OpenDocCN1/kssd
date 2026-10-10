
```
.text:4EE52FCB   mov   ecx, \[ebx+40h\]
.text:4EE52FCE   mov   esi, \[ebp+pINSSBuffer3\]
.text:4EE52FD1   mov   eax, ecx
.text:4EE52FD3   shr   ecx, 2
.text:4EE52FD6   add   esi, 48h
.text:4EE52FD9   rep movsd
.text:4EE52FDB   mov   ecx, eax
.text:4EE52FDD   and   ecx, 3
.text:4EE52FE0   rep movsb

```

后面就是一些内存释放和返回。这里就不分析了。

### INSSBuffer3 的 GetProperty 方法分析

下面分析下涉及到的 `INSSBuffer3` 的 `GetProperty` 方法，因为通过其获取的 GUID `DShowNewMediaType` 内容和如何返回 Buffer 大小，会影响问题形成的 buffer 大小信息。见 2 楼

下面是 `INSSBuffer3` `GetProperty` 的实现

代码:

```
.text:7D814231
.text:7D814231 loc\_7D814231:
.text:7D814231 mov     edx, \[ebx\]
.text:7D814233 push    ebx                                     ;
.text:7D814234 call    dword ptr \[edx+3Ch\] ;         // 这里会调用函数 sub\_7D8143C0
.text:7D814237 mov     esi, eax                              // 传入 INSSBuffer3\*, GUID, Buf,
.text:7D814239 test    esi, esi                                 // 和 BufSize4 个参数
.text:7D81423B jge     short loc\_7D814284

```

下面的函数主要是先判断是否为一个 GUID 属性，如果不是则进行查找。

代码:

```
.text:7D8143C0 sub\_7D8143C0 proc near
.text:7D8143C0
.text:7D8143C0 p\_arg2= dword ptr -18h
.text:7D8143C0 p\_GUID= byte ptr -14h
.text:7D8143C0 SecCookie= dword ptr -4
.text:7D8143C0 arg\_0= dword ptr  8
.text:7D8143C0 arg\_1= byte ptr  0Ch
.text:7D8143C0 arg\_2= dword ptr  1Ch
.text:7D8143C0 arg\_3= dword ptr  20h
.text:7D8143C0
.text:7D8143C0 mov     edi, edi
.text:7D8143C2 push    ebp
.text:7D8143C3 mov     ebp, esp
.text:7D8143C5 sub     esp, 18h
.text:7D8143C8 mov     ecx, \[ebp+arg\_2\] ; ecx = arg2
.text:7D8143CB mov     eax, dword\_7D933080
.text:7D8143D0 mov     edx, \[ebp+arg\_3\] ; edx = arg\_3
.text:7D8143D3 push    ebx
.text:7D8143D4 push    esi
.text:7D8143D5 push    edi
.text:7D8143D6 lea     esi, \[ebp+arg\_1\]
.text:7D8143D9 lea     edi, \[ebp+p\_GUID\]
.text:7D8143DC movsd
.text:7D8143DD movsd
.text:7D8143DE movsd
.text:7D8143DF movsd
.text:7D8143E0 mov     \[ebp+p\_arg2\], ecx ; var\_18 = ecx = arg\_2
.text:7D8143E3 push    4
.text:7D8143E5 pop     ecx             ; ecx = 4
.text:7D8143E6 lea     edi, \[ebp+p\_GUID\]
.text:7D8143E9 mov     esi, offset dword\_7D87F750    ; 这里是另一个 GUID 值
.text:7D8143EE xor     ebx, ebx        ; ebx = 0
.text:7D8143F0 repe cmpsd
.text:7D8143F2 mov     \[ebp+SecCookie\], eax
.text:7D8143F5 mov     eax, \[ebp+arg\_0\] ; eax = arg0
.text:7D8143F8 jnz     short loc\_7D81442D      ; 如果不是调用函数查找

```

下面函数传入 4 个参数，其中第一个参数应该是指向链表结构，offset 相对于 `INSSBuffer3` 为 `0x3c`

剩下的 3 个参数为 `GUID`, `Buf`, `BufSize`

代码:

```
.text:7D81442D
.text:7D81442D loc\_7D81442D:           ; arg3
.text:7D81442D push    edx
.text:7D81442E push    \[ebp+p\_arg2\]    ; var\_18 = arg2
.text:7D814431 lea     esi, \[ebp+p\_GUID\]
.text:7D814434 sub     esp, 10h
.text:7D814437 mov     edi, esp
.text:7D814439 movsd
.text:7D81443A movsd
.text:7D81443B movsd
.text:7D81443C lea     ecx, \[eax+3Ch\]  ; ecx = arg0+0x3c
.text:7D81443F movsd
.text:7D814440 call    sub\_7D815170

```

下面是 `sub_7D815170`, 主要是查找通过遍历链表结构成员查找 GUID 属性，如果找到 GUID 则获取其后面的 AMMediaType 数据信息

代码:

```
.text:7D8151A5
.text:7D8151A5 loc\_7D8151A5:
.text:7D8151A5 push    \[ebp+var\_18\]
.text:7D8151A8 mov     ecx, ebx        ; ecx = ebx = arg0
.text:7D8151AA call    sub\_7D814F74                     ;   遍历查找 GUID
.text:7D8151AF push    4
.text:7D8151B1 pop     ecx             ; ecx = 4
.text:7D8151B2 lea     edi, \[ebp+dwBufForID\]
.text:7D8151B5 mov     esi, eax
.text:7D8151B7 xor     edx, edx        ; edx = 0
.text:7D8151B9 repe cmpsd
.text:7D8151BB jz      short loc\_7D8151E6

```

代码:

```
.text:7D8151EA push    \[ebp+p\_arg3\]
.text:7D8151ED mov     ecx, ebx        ; ecx = ebx = arg0
.text:7D8151EF push    \[ebp+p\_arg2\]
.text:7D8151F2 push    0
.text:7D8151F4 push    \[ebp+var\_18\]
.text:7D8151F7 call    sub\_7D8150DB    ; 如果找到 GUID 属性，获取其数据内容
.text:7D8151FC jmp     short loc\_7D8151D7
.text:7D8151FC sub\_7D815170 endp

```

下面就不详细写了，其中一个是当输入的参数指向 NULL 的时候，会返回 `BufSize` 为 GUID 后面的第一个 `QWORD` `Size` 里的值，然后返回 0。

另外 2 个是根据 `IndexVector` 来在链表结构中查找对应的 GUID 属性内容，下面是我简单的推测的结构。可能不准或不对。有些是中间的推导过程记录的，后面可能变了，但没改过去。

#### sub_7D814E60

```
+-------------------------------------+    <--- arg0  
|           DWORD             |  
+-------------------------------------+    <--- arg + 0x04: 标记该元素单元的地址范围  
|node offset addr refr link head|            
+-------------------------------------+    <--- arg0 + 0x08  
|            | bit0 |  
+-------------------------------------+    <--- arg0 + 0x0c  
| ID Address ( if +0x8 bit0=1 )   |  
+-------+---------+--------+--------+    <--- arg0 + 0x10  
|      | b7-b4  | b7-b0 | b7-b0 |                    
+-------+---------+--------+----------+    <--- arg0 + 0x14  
|      .........         |  
|                 |  
| 如果从 arg0+0x10, 0x11, 0x12 中 |  
|的 bit Flag 检索值为 1，则：         |  
| arg0+0x14+arg1\*4 指向的 DW 存 |<--- 如果从 Flag 的 bit 中检索值为 1，则 ID 的地址从这个范  
|有 ID 的 Address.           |  
|  Bit Flag 为 1 的优先级大于 offset      |  
|bit0 为 1 的优先级.           |          
|      .........         |  
+----------------------------------------+    <--- arg0 + 0x64  
|  next node start address       |  
+----------------------------------------+    <--- arg0 + 0x68  
|prev node end offset value addr |      
+----------------------------------------+  
```

其中:

`arg0+0x10` 的 `bit7` 到 `bit0`: 分别对应地址 `arg0+0x14` 到 `arg0+0x33`  `(8` 个 `DWORD)`

`arg0+0x11` 的 `bit7` 到 `bit0`: 分别对应地址 `arg0+0x34` 到 `arg0+0x53`  `(8` 个 `DWORD)`

`arg0+0x12` 的 `bit7` 到 `bit4`: 分别对应地址 `arg0+0x54` 到 `arg0+0x63`  `(4` 个 `DWORD)`

#### sub_7D814DBE

```
// 函数: sub\_7D814DBE(BaseAddr, BitOffset);  
//  
// 功能: 根据参数 arg1 指定的 offset 和 bit Mask，来返回对应的 Bit 值是 1 还是 0  
//  
//    +-------------------------------+    <--- arg0  
//    |        DWORD        |  
//    +-------------------------------+    <---- arg + 0x04  
//    |        DWORD        |        
//    +-------------------------------+    <--- arg0 + 0x08  
//    |        DWORD        |  
//    +-------------------------------+    <--- arg0 + 0x0c  
//    |        DWORD        |  
//    +-------+-------+-------+-------+    <--- arg0 + 0x10  
//    |    | Flags | Flags  | Flags  |                      |  
//    +-------+-------+-------+-------+  
//  
//      
//    real pointer = arg0 + 0x16 + offset(Byte Offset, Bit Offset)  
//  
//  
//    DWORD arg1 format: (用于指定 offset 的, Byte offset 和 bit offset)  
//  
//    +------------------------------------+-----------------------------------+      
//    | byte offset of checked byte number | bit offset of checked byte number |  
//    +------------------------------------+-----------------------------------+  
//    bit31      -----        bit4      ------        bit0  
//  
//    == Bit31 - Bit4 : (byte offset of checked number)  
//        
//      指定要被测试的 Bit 所在 Byte 的地址偏移量  
//          
//        
//    == Bit3 - Bit0 : (bit offset of checked number)  
//        
//      指明要被测试的 Bit 的 Byte 偏移基础上的 bit 偏移量  
//  
//      0 - check byte bit-7 value  
//      1 - check byte bit-6 value  
//      2 - check byte bit-5 value    
//      3 - check byte bit-4 value  
//      4 - check byte bit-3 value  
//      5 - check byte bit-2 value  
//      6 - check byte bit-1 value  
//      7 - check byte bit-0 value  
//  
//    所以被测试的 Bit 的偏移过程:  
//      通过 Byte Offset 确定其所在 byte 的地址, 然后同过 bit offset 定位到具体的是哪个 bit  
//  
//  
// 返回: 如果对应的 bit 为 1, 则返回 1; 为 0， 则返回 0;  
//     也就是返回对应 bit 的值.  
//  

代码:

```
BYTE BitMask\[8\] = {0x80, 0x40, 0x20, 0x10, 0x08, 0x04, 0x02, 0x01};

// ChkBitValue
sub\_7D814DBE(
  arg0,    // Base Addr
  arg1)    // BitOffset
{
  BYTE BitMaskIndex;
  BYTE ByteNum;
  BYTE BitFlag;

  ByteNum = (\*( arg0 + 0x10 + (arg1 >> 3) )) & 0x7;

  BitMaskIndex = arg1 & 0x7;

  BitFlag = ByteNum & BitMask\[BitMaskIndex\];

  return( (BitFlag == 0) ? 0 : 1 );  
}
```
