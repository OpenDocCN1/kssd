
# 用 Dede 亲密接触 Dede 的 Nag 兼导出其 License 函数 DDSHA1【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-422.htm`](https://www.pediy.com/kssd/pediy08/pediy8-422.htm)

众所周知:`Dede`是反编译`Broland Delphi`程序的利器!`Dede`也是`Broland Delphi`编译的.故用`Dede`反自身!

声明目的无有,据说`Dede`源代码公开,但从来也没有观摩过直到分析这个`Nag`更没有观摩过!

前几天`xp`系统重装了,之后分析`Delphi`程序的时候用了下`Dede`(在看雪主页下的)。当用`Dede`分析程序的时候`Dede`突然跳出一个`Nag`来警告是否同意其`License Statement`并且延迟很长时间.

,感觉很吃惊`Dede`难道也需要注册使用么?!

经过一系列的逆向分析推出:

若是同意点击`"I Agree"`,下一回启动`Dede`的时候就不会再出现`Nag`了.如果是重装了系统或者是改变了文件路径或者是文件名此时候`Dede`的`Nag`再次提示你`License Statement`.

原来是`Dede`根据版本号和其所在的路径全名再和`Window XP ProcductID`号,经过`SHA1`算法产生一个唯一系统的安装目录和软件版本号的散列数值,然后保存在当前`ini`文件中,所以当你重新装了系统或者是改变文件名

`Dede`发现与原来的`SHA1`产生的散列值不一样了,所以就跳出`Nag License Statement`!

若是不同意点击`"I Disagree"`,下一会一定会提示`Nag`出来的.

这一点很类似装再系统或装软件的时候,提问是否同意其安装`License`声明,通常当然是同意了.

不过`Dede`用了个`Nag`提示是否同意其`License`声明并且还延迟了很长时间呀!有些真的烦人浪费时间了!

这个就有必要分析破出它了!

下边文章是按照顺序方式写出来的,分析的时候是逆向的!

`Nag`窗体类是`TLSForm`,所在单元`LSUint`.

## Dede 用了两小技巧产生 Nag

1. 动态产生`TTimer`组件,并且注册自定义`Timer`事件.注意这个`TTimer`没有序列化资源,用`Dede`自我分析不出这个事件地址的.偶根据的`VCL`类库分析的.
2. `Nag`窗体也是动态产生的,而产生`Nag`窗体的这个事件正是上面的动态`TTimer`组件事件.这也是用`Dede`具体分析不出来的,而只能分析出`Nag`窗体控件和`TLSForm`类.

`Nag`窗体是由一个动态`TTimer`组件事件产生的,而这个动态`TTimer`组件是由`TDeDeMainForm@FormCreate`事件根据计算出来`SHA1`散列值和`ini`文件中的`SHA1`比较结果而产生的.如果相同就不产生,否则就提示`Nag`了!

## 大体思路与 TDeDeMainForm@FormCreate 分析

大体思路说出来了,看`TDeDeMainForm@FormCreate`事件程序:

`String\_\[5AC834\]`全局变量保存`ini`文件中`[Common]`节下的字段变量名`Version`的数值,这个数值正是`Dede`计算的`SHA1`散列值!

```assembly
005948C4  >/.  55            push    ebp  -------------------------------------<-TDeDeMainForm@FormCreate
005948C5  |.  8BEC          mov     ebp, esp
005948C7  |.  B9 43000000   mov     ecx, 43
005948CC  |>  6A 00         /push    0
005948CE  |.  6A 00         |push    0
005948D0  |.  49            |dec     ecx
005948D1  |.^ 75 F9         \jnz     short 005948CC
005948D3  |.  8955 E8       mov     [ebp-18], edx
005948D6  |.  8945 FC       mov     [ebp-4], eax
005948D9  |.  33C0          xor     eax, eax
005948DB  |.  55            push    ebp
005948DC  |.  68 934E5900   push    <->System.@HandleFinally;>
005948E1  |.  64:FF30       push    dword ptr fs:[eax]
005948E4  |.  64:8920       mov     fs:[eax], esp
005948E7  |.  8D95 14FEFFFF lea     edx, [ebp-1EC]
005948ED  |.  A1 D0CA5A00   mov     eax, [5ACAD0]
005948F2  |.  8B00          mov     eax, [eax]
005948F4  >|.  E8 93D5EDFF   call    00471E8C                     ;  ->Forms.TApplication.GetExeName(TApplication):AnsiString;
005948F9  |.  8B85 14FEFFFF mov     eax, [ebp-1EC]
005948FF  |.  8D8D 18FEFFFF lea     ecx, [ebp-1E8]
00594905  |.  BA A84E5900   mov     edx, 00594EA8                  ;  ASCII ".ini"
0059490A  >|.  E8 2D5BE7FF   call    0040A43C                     ;  ->Unit_004086A8.Proc_0040A43C
0059490F  |.  8B95 18FEFFFF mov     edx, [ebp-1E8]
00594915  |.  A1 B4C25A00   mov     eax, [5AC2B4]
0059491A  >|.  E8 6D03E7FF   call    00404C8C                     ;  ->System.@LStrAsg(void;void;void;void);
0059491F  |.  8B15 B4C25A00 mov     edx, [5AC2B4]                  ;  复件_DeD.005AE21C
00594925  |.  8B12          mov     edx, [edx]
00594927  |.  8B45 FC       mov     eax, [ebp-4]
0059492A  >|.  8B80 F0040000 mov     eax, [eax+4F0]                ;  *TDeDeMainForm.FP:TFormPlacement
00594930  >|.  E8 976DF3FF   call    004CB6CC                     ;  ->:TIniLink._PROC_004CB6CC()
00594935  |.  8D95 10FEFFFF lea     edx, [ebp-1F0]
0059493B  |.  A1 D0CA5A00   mov     eax, [5ACAD0]
00594940  |.  8B00          mov     eax, [eax]
00594942  >|.  E8 45D5EDFF   call    00471E8C                     ;  ->Forms.TApplication.GetExeName(TApplication):AnsiString;
00594947  |.  8B95 10FEFFFF mov     edx, [ebp-1F0]
0059494D  |.  A1 ACCA5A00   mov     eax, [5ACAAC]
00594952  >|.  E8 3503E7FF   call    00404C8C                     ;  ->System.@LStrAsg(void;void;void;void);
00594957  |.  B2 01         mov     dl, 1
00594959  |.  A1 24964100   mov     eax, [419624]
0059495E  >|.  E8 95F4E6FF   call    00403DF8                     ;  ->System.TObject.Create(TObject;Boolean);
00594963  |.  8B55 FC       mov     edx, [ebp-4]
00594966  >|.  8982 70050000 mov     [edx+570], eax                ;  *TDeDeMainForm.OFFS_0570:TStringList
0059496C  |.  8B45 FC       mov     eax, [ebp-4]
0059496F  |.  05 70050000   add     eax, 570
00594974  |.  B2 01         mov     dl, 1
00594976  >|.  E8 7DDFF5FF   call    004F28F8                     ;  ->Unit_004F1D48.Proc_004F28F8
0059497B  |.  8D95 04FEFFFF lea     edx, [ebp-1FC]
00594981  |.  A1 D0CA5A00   mov     eax, [5ACAD0]
00594986  |.  8B00          mov     eax, [eax]
00594988  >|.  E8 FFD4EDFF   call    00471E8C                     ;  ->Forms.TApplication.GetExeName(TApplication):AnsiString;
0059498D  |.  8B85 04FEFFFF mov     eax, [ebp-1FC]
00594993  |.  8D95 08FEFFFF lea     edx, [ebp-1F8]
00594999  >|.  E8 665BE7FF   call    0040A504                     ;  ->Unit_004086A8.Proc_0040A504
0059499E  |.  FFB5 08FEFFFF push    dword ptr [ebp-1F8]
005949A4  |.  68 B84E5900   push    00594EB8                     ;  ASCII "\\LANGRES\\"
005949A9  |.  A1 B4C85A00   mov     eax, [5AC8B4]
005949AE  |.  FF30          push    dword ptr [eax]
005949B0  |.  8D85 0CFEFFFF lea     eax, [ebp-1F4]
005949B6  |.  BA 03000000   mov     edx, 3
005949BB  >|.  E8 F005E7FF   call    00404FB0                     ;  ->System.@LStrCatN;
005949C0  |.  8B85 0CFEFFFF mov     eax, [ebp-1F4]
005949C6  >|.  E8 3D8EEFFF   call    0048D808                     ;  ->Unit_0048ADC4.Proc_0048D808
005949CB  |.  8B45 FC       mov     eax, [ebp-4]
005949CE  >|.  E8 F5A10000   call    <<-TDeDeMainForm@Proc_0059EBC8>  ;  ->:TDeDeMainForm.Proc_0059EBC8()
005949D3  |.  8B45 FC       mov     eax, [ebp-4]
005949D6  >|.  E8 E1C40000   call    <<-TDeDeMainForm@Proc_005A0EBC>  ;  ->:TDeDeMainForm.Proc_005A0EBC()
005949DB  |.  B2 01         mov     dl, 1
005949DD  |.  A1 24964100   mov     eax, [419624]
005949E2  >|.  E8 11F4E6FF   call    00403DF8                     ;  ->System.TObject.Create(TObject;Boolean);
005949E7  |.  8B55 FC       mov     edx, [ebp-4]
005949EA  >|.  8982 7C050000 mov     [edx+57C], eax                ;  *TDeDeMainForm.OFFS_057C:TStringList
005949F0  |.  8D95 FCFDFFFF lea     edx, [ebp-204]
005949F6  |.  A1 D0CA5A00   mov     eax, [5ACAD0]
005949FB  |.  8B00          mov     eax, [eax]
005949FD  >|.  E8 8AD4EDFF   call    00471E8C                     ;  ->Forms.TApplication.GetExeName(TApplication):AnsiString;
00594A02  |.  8B85 FCFDFFFF mov     eax, [ebp-204]
00594A08  |.  8D95 00FEFFFF lea     edx, [ebp-200]
00594A0E  >|.  E8 F15AE7FF   call    0040A504                     ;  ->Unit_004086A8.Proc_0040A504
00594A13  |.  8B95 00FEFFFF mov     edx, [ebp-200]
00594A19  |.  8D45 F8       lea     eax, [ebp-8]
00594A1C  |.  B9 CC4E5900   mov     ecx, 00594ECC                  ;  ASCII "\\classes.lst"
00594A21  >|.  E8 1605E7FF   call    00404F3C                     ;  ->System.@LStrCat3;
00594A26  |.  8B45 F8       mov     eax, [ebp-8]
00594A29  >|.  E8 6256E7FF   call    0040A090                     ;  ->Unit_004086A8.Proc_0040A090
00594A2E  |.  84C0          test    al, al
00594A30  |.  74 11         je      short 00594A43
00594A32  |.  8B45 FC       mov     eax, [ebp-4]
00594A35  >|.  8B80 7C050000 mov     eax, [eax+57C]                ;  *TDeDeMainForm.OFFS_057C:TStringList
00594A3B  |.  8B55 F8       mov     edx, [ebp-8]
00594A3E  |.  8B08          mov     ecx, [eax]
00594A40  >|.  FF51 68       call    [ecx+68]                     ;  ->TStringList.LoadFromFile(string)
```
