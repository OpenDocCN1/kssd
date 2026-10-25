
# CS1.5 CD-KEY 算法分析-简单

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-185.htm`](https://www.pediy.com/kssd/pediy09/pediy09-185.htm)

> └文章标题┐:CS1.5 CD-KEY 算法分析  
> └破文作者┐:大菜一号  
> └破解对象┐:CS1.5 CD-KEY 生成器  
> └下载地址┐:不知道  
> └对象大小┐:不知道  
> └加壳方式┐:没有  
> └保护方式┐:软柿子算法  
> └编写语言┐:Delphi  
> └使用工具┐:`DEDE`, `OD`  
> └破解平台┐:D 版 XP  
> └破解声明┐:  
> 用自己的 CD-KEY 生成器感觉就是不一样! :)  

> └破解过程┐:  
> 要分析 `CS1.5` 的 CD-key 算法，既然有现成的 CD-KEY 生成器，何不直接分析它咧！呵呵，毕竟想知道算法罢了！~分析之前，先感谢小虾的 `VC` 指点！  
> 好了  
> 用 `DEDE` 找到窗口仅有的那个按钮，`OD` 断在下面了:

```
00426C38  /.  55           push    ebp                     //断在这
00426C39  |.  8BEC         mov     ebp, esp
00426C3B  |.  33C9         xor     ecx, ecx
00426C3D  |.  51           push    ecx
00426C3E  |.  51           push    ecx
00426C3F  |.  51           push    ecx
00426C40  |.  51           push    ecx
00426C41  |.  51           push    ecx
00426C42  |.  51           push    ecx
00426C43  |.  51           push    ecx
00426C44  |.  51           push    ecx
00426C45  |.  53           push    ebx
00426C46  |.  8945 F8      mov     dword ptr [ebp-8], eax
00426C49  |.  33C0         xor     eax, eax
00426C4B  |.  55           push    ebp
00426C4C  |.  68 666D4200  push    00426D66
00426C51  |.  64:FF30      push    dword ptr fs:[eax]
00426C54  |.  64:8920      mov     dword ptr fs:[eax], esp
00426C57  |.  8D45 F4      lea     eax, dword ptr [ebp-C]
00426C5A  |.  BA 7C6D4200  mov     edx, 00426D7C             ;  ASCII "1111222223333"//填充字符数组,没用的东西
00426C5F  |.  E8 BCC9FDFF  call    00403620
00426C64  |.  C745 FC 03000>mov    dword ptr [ebp-4], 3         //3 传到 ebp-4
00426C6B  |.  E8 7CBAFDFF  call    004026EC                  //关键，跟进
```

跟进后:

```
  004026EC  /$  55           push    ebp
  004026ED  |.  8BEC         mov     ebp, esp
  004026EF  |.  83C4 E8      add     esp, -18
  004026F2  |.  8D45 E8      lea     eax, dword ptr [ebp-18]
  004026F5  |.  50           push    eax                       ; /pSystemTime  //SYSTEMTIME 结构体变量指针
  004026F6  |.  E8 39EAFFFF  call    <jmp.&kernel32.GetSystemTime>    ; \GetSystemTime //填充 SYSTEMTIME 结构
  004026FB  |.  0FB745 F0    movzx   eax, word ptr [ebp-10]      //当前小时传到 eax
  004026FF  |.  6BC0 3C      imul    eax, eax, 3C              //eax<-eax*0x3c
  00402702  |.  66:0345 F2   add     ax, word ptr [ebp-E]       //当前分钟+上面结果
  00402706  |.  6BC0 3C      imul    eax, eax, 3C              //再*0x3c
  00402709  |.  31D2         xor     edx, edx
  0040270B  |.  66:8B55 F4   mov     dx, word ptr [ebp-C]       //当前秒钟传到 dx
  0040270F  |.  01D0         add     eax, edx                   //秒钟+上面结果
  00402711  |.  69C0 E8030000 imul    eax, eax, 3E8             //结果*0x3e8
  00402717  |.  66:8B55 F6   mov     dx, word ptr [ebp-A]       //当前毫秒传到 dx
  0040271B  |.  01D0         add     eax, edx                   //+上面结果
  0040271D  |.  8905 3C804200 mov    dword ptr [42803C], eax    //最终结果传到[42803c]
  00402723  |.  8BE5         mov     esp, ebp
  00402725  |.  5D           pop     ebp
  00402726  |\.  C3           retn                              //回去
```

```
00426C70  |.  BB 01000000  mov     ebx, 1                     //1 传到 ebx,指向字符用的
00426C75  |>  B8 0A000000  /mov    eax, 0A                    //0x0a 传到 eax,后面有用
00426C7A  |.  E8 09BCFDFF  |call  00402888                    //关键，跟进
```

跟进后:

```
  00402888  /$  6915 3C804200>imul    edx, dword ptr [42803C], 8088405 //call1 结果*0x8088405
  00402892  |.  42           inc     edx                       //再+1
  00402893  |.  8915 3C804200 mov    dword ptr [42803C], edx   //传到[42803c]
  00402899  |.  F7E2         mul     edx                       //结果*0x0a
  0040289B  |.  89D0         mov     eax, edx                  //32 位寄存器放不下的 dword 字节外的那些传给 eax
  0040289D  |\   . C3         retn
```

```
00426C7F  |.  83C0 30      |add    eax, 30                   //call2 结果+30
00426C82  |.  50           |push  eax
00426C83  |.  8D45 F4      |lea   eax, dword ptr [ebp-C]
00426C86  |.  E8 49CDFDFF  |call  004039D4                  //结果转换为字符
00426C8B  |.  5A           |pop   edx
00426C8C  |.  885418 FF    |mov   byte ptr [eax+ebx-1], dl  //依次组成字符串，在[eax+ebx-1]里
00426C90  |.  43           |inc   ebx                         //计数器加一，并指向下一个内存格
00426C91  |.  83FB 0D      |cmp   ebx, 0D                     //循环 12 次
00426C94  |.^ 75 DF        \jnz  short 00426C75
```

*循环后得到字符串`"1655983353753"`,(你那边肯定不一样，精确到毫秒的)

```
00426C96  |.  BB 01000000  mov     ebx, 1                      
00426C9B  |>  8B55 FC      /mov    edx, dword ptr [ebp-4]     //3(忘记[ebp-4]里是 3 的，请看 426c64)传到 edx
00426C9E  |.  03D2         |add   edx, edx                    //*2
00426CA0  |.  8B45 F4      |mov   eax, dword ptr [ebp-C]
00426CA3  |.  0FB64418 FF  |movzx eax, byte ptr [eax+ebx-1]   //新字符串字符传到 eax
00426CA8  |.  83E8 30      |sub   eax, 30                     //减去 30
00426CAB  |.  33C2         |xor   eax, edx                    //异或上面的和
00426CAD  |.  0145 FC      |add   dword ptr [ebp-4], eax      //结果加到[ebp-4]里
00426CB0  |.  43           |inc   ebx                         //计数器加一，并指向下一个内存格
00426CB1  |.  83FB 0D      |cmp   ebx, 0D                     //循环 12 次
00426CB4  |.^ 75 E5        \jnz  short 00426C9B
00426CB6  |.  31D2         xor     edx, edx
00426CB8  |.  8B45 FC      mov     eax, dword ptr [ebp-4]     //上面的结果传到 eax
00426CBB  |.  B9 0A000000  mov     ecx, 0A                   //0x0a 传到 ecx
00426CC0  |.  F7F1         div     ecx                        //上面结果除以 0x0a
00426CC2  |.  89D0         mov     eax, edx                   //余数传到 eax
00426CC4  |.  83C0 30      add     eax, 30                    //余数加上 0x30,我这里结果为 0x39
00426CC7  |.  8945 FC      mov     dword ptr [ebp-4], eax
00426CCA  |.  8D45 EC      lea     eax, dword ptr [ebp-14]
00426CCD  |.  50           push    eax
00426CCE  |.  B9 04000000  mov     ecx, 4
00426CD3  |.  BA 01000000  mov     edx, 1
00426CD8  |.  8B45 F4      mov     eax, dword ptr [ebp-C]
00426CDB  |.  E8 28CDFDFF  call    00403A08                   //取出字符串的前四位
00426CE0  |.  FF75 EC      push    dword ptr [ebp-14]         //[ebp-14]的 dword 是"1655"
00426CE3  |.  68 946D4200  push    00426D94
00426CE8  |.  8D45 E8      lea     eax, dword ptr [ebp-18]
00426CEB  |.  50           push    eax
00426CEC  |.  B9 05000000  mov     ecx, 5
00426CF1  |.  BA 05000000  mov     edx, 5
00426CF6  |.  8B45 F4      mov     eax, dword ptr [ebp-C]
00426CF9  |.  E8 0ACDFDFF  call    00403A08                   //取出字符串中间五位"98335"
00426CFE  |.  FF75 E8      push    dword ptr [ebp-18]          
00426D01  |.  68 946D4200  push    00426D94
00426D06  |.  8D45 E4      lea     eax, dword ptr [ebp-1C]
00426D09  |.  50           push    eax
00426D0A  |.  B9 03000000  mov     ecx, 3
00426D0F  |.  BA 0A000000  mov     edx, 0A
00426D14  |.  8B45 F4      mov     eax, dword ptr [ebp-C]
00426D17  |.  E8 ECCCFDFF  call    00403A08                   //再接着取出三位(375)(注意,key 除了两个'-'之外一共有 13 位
```



> 到这里为止取出了 12 位,还有一位

```
00426D1C  |.  FF75 E4       push    dword ptr [ebp-1C]
00426D1F  |.  8D45 E0       lea     eax, dword ptr [ebp-20]
00426D22  |.  8B55 FC       mov     edx, dword ptr [ebp-4]
00426D25  |.  E8 02CAFDFF   call    0040372C
00426D2A  |.  FF75 E0       push    dword ptr [ebp-20]
00426D2D  |.  8D45 F0       lea     eax, dword ptr [ebp-10]
00426D30  |.  BA 06000000   mov     edx, 6
00426D35  |.  E8 8ACBFDFF   call    004038C4          //经过之后得到"1655-98335-3759",
                                                        而不是上面的"1655-98335-3753".
```

* 知道那个第二区的循环在干什么了吧！那个循环的结果`[ebp-14]`最后不是等于 `0x39`,转换成 10 进制对应的 asc 就是 `'9'`
* 而 `0x426d17` 这个 `call` 只取出三位，就是要留一位出来放指点二区循环的结果对应的字符!

```
00426D3A  |.  8B55 F0       mov     edx, dword ptr [ebp-10]
00426D3D  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
00426D40  |.  8B80 E0010000 mov     eax, dword ptr [eax+1E0]
00426D46  |.  E8 C9EEFEFF   call    00415C14
00426D4B  |.  33C0          xor     eax, eax
00426D4D  |.  5A            pop     edx
00426D4E  |.  59            pop     ecx
00426D4F  |.  59            pop     ecx
00426D50  |.  64:8910       mov     dword ptr fs:[eax], edx
00426D53  |.  68 6D6D4200   push    00426D6D
00426D58  |>  8D45 E0       lea     eax, dword ptr [ebp-20]
00426D5B  |.  BA 06000000   mov     edx, 6
00426D60  |.  E8 47C8FDFF   call    004035AC
00426D65  \.  C3            retn
```

, 嘿嘿, 常常玩的 CS, 盗版 cd-key 也不难算嘛, 下面搞个自己的 cd-key 生成器:

```cpp
void CCSCD_keyDlg::OnOK() 
{
  SYSTEMTIME  time;//系统时间结构体
  char key[100],a[100];
  __int64 i,k=0,num=0,m=0,p=3;//看到这里，就想到小虾:)
  ::GetSystemTime(&time);//填充结构体成员
  //SetDlgItemText(IDC_EDIT1,key);
  k=((time.wHour*0x3c+time.wMinute)*0x3c+time.wSecond)*0x3e8+time.wMilliseconds;//call1 的计算
  for(i=0;i<12;i++)//第一区循环
  {
    k=k*0x8088405+1;
    num=(DWORD)k;//取双字，以免数据出错
    num=num*0x0a;
    m=num>>32;//看到这里，也想到小虾:)
     key[i]=m+0x30;
  }

  for(i=0;i<12;i++)//第二区循环
  {
    m=p;
    m*=2;
    k=key[i]-0x30;
    k^=m;
    p+=k;
  }
  p%=0x0a;
  p+=0x30;//得到 cd-key 的最后一位
  sprintf(a,"%c%c%c%c-%c%c%c%c%c-%c%c%c%c",
    key[0],key[1],key[2],key[3],key[4],key[5],key[6],key[7],key[8],key[9],key[10],key[11],
    p);//格式化
  SetDlgItemText(IDC_EDIT1,a);//输出
}
```

----------------------------------------------------------------------------------
└经验总结┐:
 嘿嘿\`这次的生成器本来搞了个很好看的界面，不过移植好像有问题，其它机子无法运行！汗\`~~看来普通就好！！- -b
----------------------------------------------------------------------------------
└版权声明┐ 本文原创于看雪软件安全论坛, 转载请注明作者并保持文章的完整, 谢谢!
                                                              2007 年 6 月 18 日  21:43:41 

看到楼主的源码就有翻译到 Python 的冲动

代码:

```python
import time
t=time.time()
t*=1000
t=long(t)
key=[]
for i in range(12):
    t=t*0x8088405+1
    num=int(t)%(1<<32)
    num*=0x0a
    m=num>>32
    key.append(m+0x30)

p=3
for i in range(12):
    m=p
    m*=2
    k=key[i]-0x30
    k^=m
    p+=k
p%=0x0a
p+=0x30

print '--------------CS1.5 CDKEY GENNERATOR-----------------.'
print '%c%c%c%c-%c%c%c%c%c-%c%c%c%c' %(key[0],key[1],key[2],key[3],key[4],key[5],key[6],key[7],key[8],key[9],key[10],key[11],p)
print '--------------that\'s you want.-----------------------'
```

引用: --------------CS1.5 CDKEY GENNERATOR-----------------.
9576-69520-3230
--------------that's you want.-----------------------
--------------CS1.5 CDKEY GENNERATOR-----------------.
2710-90042-5915
--------------that's you want.-----------------------

4dd466d0e1e0cc123afbe504da2462ca
