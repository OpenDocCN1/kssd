

# 用 Dede 亲密接触 Dede 的 Nag 兼导出其 License 函数 DDSHA1【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-422.htm`](https://www.pediy.com/kssd/pediy08/pediy8-422.htm)

众所周知:Dede 是反编译 Broland Delphi 程序的利器!Dede 也是 Broland Delphi 编译的.故用 Dede 反自身!  
声明目的无有,据说 Dede 源代码公开,但从来也没有观摩过直到分析这个 Nag 更没有观摩过!  
  
前几天 xp 系统重装了,之后分析 Delphi 程序的时候用了下 Dede(在看雪主页下的)。当用 Dede 分析程序的时候 Dede 突然跳出一个 Nag 来警告是否同意其 License Statement 并且延迟很长时间.  
  
,感觉很吃惊 Dede 难道也需要注册使用么?!  
  
经过一系列的逆向分析推出:  
  
若是同意点击"I Agree",下一回启动 Dede 的时候就不会再出现 Nag 了.如果是重装了系统或者是改变了文件路径或者是文件名此时候 Dede 的 Nag 再次提示你 License Statement.  
原来是 Dede 根据版本号和其所在的路径全名再和 Window XP ProcductID 号,经过 SHA1 算法产生一个唯一系统的安装目录和软件版本号的散列数值,然后保存在当前 ini 文件中,所以当你重新装了系统或者是改变文件名  
Dede 发现与原来的 SHA1 产生的散列值不一样了,所以就跳出 Nag License Statement!  
若是不同意点击"I Disagree",下一会一定会提示 Nag 出来的.  
  
这一点很类似装再系统或装软件的时候,提问是否同意其安装 License 声明,通常当然是同意了.  
不过 Dede 用了个 Nag 提示是否同意其 License 声明并且还延迟了很长时间呀!有些真的烦人浪费时间了!  
这个就有必要分析破出它了!  
  
下边文章是按照顺序方式写出来的,分析的时候是逆向的!  
Nag 窗体类是 TLSForm,所在单元 LSUint.  
  
Dede 用了两小技巧产生 Nag  
1 动态产生 TTimer 组件,并且注册自定义 Timer 事件.注意这个 TTimer 没有序列化资源,用 Dede 自我分析不出这个事件地址的.偶根据的 VCL 类库分析的.  
2 Nag 窗体也是动态产生的,而产生 Nag 窗体的这个事件正是上面的动态 TTimer 组件事件.这也是用 Dede 具体分析不出来的,而只能分析出 Nag 窗体控件和 TLSForm 类.  
  
  
Nag 窗体是由一个动态 TTimer 组件事件产生的,而这个动态 TTimer 组件是由 TDeDeMainForm@FormCreate 事件根据计算出来 SHA1 散列值和 ini 文件中的 SHA1 比较结果而产生的.如果相同就不产生,否则就提示 Nag 了!  
  
大体思路说出来了,看 TDeDeMainForm@FormCreate 事件程序:  
  
String\_\[5AC834\] 全局变量保存 ini 文件中\[Common\]节下的字段变量名 Version 的数值,这个数值正是 Dede 计算的 SHA1 散列值!  
  
005948C4 >/.  55            push    ebp  -------------------------------------<-TDeDeMainForm@FormCreate  
005948C5  |.  8BEC          mov     ebp, esp  
005948C7  |.  B9 43000000   mov     ecx, 43  
005948CC  |>  6A 00         /push    0  
005948CE  |.  6A 00         |push    0  
005948D0  |.  49            |dec     ecx  
005948D1  |.^ 75 F9         \\jnz     short 005948CC  
005948D3  |.  8955 E8       mov     \[ebp-18\], edx  
005948D6  |.  8945 FC       mov     \[ebp-4\], eax  
005948D9  |.  33C0          xor     eax, eax  
005948DB  |.  55            push    ebp  
005948DC  |.  68 934E5900   push    <->System.@HandleFinally;>  
005948E1  |.  64:FF30       push    dword ptr fs:\[eax\]  
005948E4  |.  64:8920       mov     fs:\[eax\], esp  
005948E7  |.  8D95 14FEFFFF lea     edx, \[ebp-1EC\]  
005948ED  |.  A1 D0CA5A00   mov     eax, \[5ACAD0\]  
005948F2  |.  8B00          mov     eax, \[eax\]  
005948F4 >|.  E8 93D5EDFF   call    00471E8C                         ;  ->Forms.TApplication.GetExeName(TApplication):AnsiString;  
005948F9  |.  8B85 14FEFFFF mov     eax, \[ebp-1EC\]  
005948FF  |.  8D8D 18FEFFFF lea     ecx, \[ebp-1E8\]  
00594905  |.  BA A84E5900   mov     edx, 00594EA8                    ;  ASCII ".ini"  
0059490A >|.  E8 2D5BE7FF   call    0040A43C                         ;  ->Unit\_004086A8.Proc\_0040A43C  
0059490F  |.  8B95 18FEFFFF mov     edx, \[ebp-1E8\]  
00594915  |.  A1 B4C25A00   mov     eax, \[5AC2B4\]  
0059491A >|.  E8 6D03E7FF   call    00404C8C                         ;  ->System.@LStrAsg(void;void;void;void);  
0059491F  |.  8B15 B4C25A00 mov     edx, \[5AC2B4\]                    ;  复件\_DeD.005AE21C  
00594925  |.  8B12          mov     edx, \[edx\]  
00594927  |.  8B45 FC       mov     eax, \[ebp-4\]  
0059492A >|.  8B80 F0040000 mov     eax, \[eax+4F0\]                   ;  \*TDeDeMainForm.FP:TFormPlacement  
00594930 >|.  E8 976DF3FF   call    004CB6CC                         ;  ->:TIniLink.\_PROC\_004CB6CC()  
00594935  |.  8D95 10FEFFFF lea     edx, \[ebp-1F0\]  
0059493B  |.  A1 D0CA5A00   mov     eax, \[5ACAD0\]  
00594940  |.  8B00          mov     eax, \[eax\]  
00594942 >|.  E8 45D5EDFF   call    00471E8C                         ;  ->Forms.TApplication.GetExeName(TApplication):AnsiString;  
00594947  |.  8B95 10FEFFFF mov     edx, \[ebp-1F0\]  
0059494D  |.  A1 ACCA5A00   mov     eax, \[5ACAAC\]  
00594952 >|.  E8 3503E7FF   call    00404C8C                         ;  ->System.@LStrAsg(void;void;void;void);  
00594957  |.  B2 01         mov     dl, 1  
00594959  |.  A1 24964100   mov     eax, \[419624\]  
0059495E >|.  E8 95F4E6FF   call    00403DF8                         ;  ->System.TObject.Create(TObject;Boolean);  
00594963  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594966 >|.  8982 70050000 mov     \[edx+570\], eax                   ;  \*TDeDeMainForm.OFFS\_0570:TStringList  
0059496C  |.  8B45 FC       mov     eax, \[ebp-4\]  
0059496F  |.  05 70050000   add     eax, 570  
00594974  |.  B2 01         mov     dl, 1  
00594976 >|.  E8 7DDFF5FF   call    004F28F8                         ;  ->Unit\_004F1D48.Proc\_004F28F8  
0059497B  |.  8D95 04FEFFFF lea     edx, \[ebp-1FC\]  
00594981  |.  A1 D0CA5A00   mov     eax, \[5ACAD0\]  
00594986  |.  8B00          mov     eax, \[eax\]  
00594988 >|.  E8 FFD4EDFF   call    00471E8C                         ;  ->Forms.TApplication.GetExeName(TApplication):AnsiString;  
0059498D  |.  8B85 04FEFFFF mov     eax, \[ebp-1FC\]  
00594993  |.  8D95 08FEFFFF lea     edx, \[ebp-1F8\]  
00594999 >|.  E8 665BE7FF   call    0040A504                         ;  ->Unit\_004086A8.Proc\_0040A504  
0059499E  |.  FFB5 08FEFFFF push    dword ptr \[ebp-1F8\]  
005949A4  |.  68 B84E5900   push    00594EB8                         ;  ASCII "\\LANGRES\\"  
005949A9  |.  A1 B4C85A00   mov     eax, \[5AC8B4\]  
005949AE  |.  FF30          push    dword ptr \[eax\]  
005949B0  |.  8D85 0CFEFFFF lea     eax, \[ebp-1F4\]  
005949B6  |.  BA 03000000   mov     edx, 3  
005949BB >|.  E8 F005E7FF   call    00404FB0                         ;  ->System.@LStrCatN;  
005949C0  |.  8B85 0CFEFFFF mov     eax, \[ebp-1F4\]  
005949C6 >|.  E8 3D8EEFFF   call    0048D808                         ;  ->Unit\_0048ADC4.Proc\_0048D808  
005949CB  |.  8B45 FC       mov     eax, \[ebp-4\]  
005949CE >|.  E8 F5A10000   call    <<-TDeDeMainForm@Proc\_0059EBC8>  ;  ->:TDeDeMainForm.Proc\_0059EBC8()  
005949D3  |.  8B45 FC       mov     eax, \[ebp-4\]  
005949D6 >|.  E8 E1C40000   call    <<-TDeDeMainForm@Proc\_005A0EBC>  ;  ->:TDeDeMainForm.Proc\_005A0EBC()  
005949DB  |.  B2 01         mov     dl, 1  
005949DD  |.  A1 24964100   mov     eax, \[419624\]  
005949E2 >|.  E8 11F4E6FF   call    00403DF8                         ;  ->System.TObject.Create(TObject;Boolean);  
005949E7  |.  8B55 FC       mov     edx, \[ebp-4\]  
005949EA >|.  8982 7C050000 mov     \[edx+57C\], eax                   ;  \*TDeDeMainForm.OFFS\_057C:TStringList  
005949F0  |.  8D95 FCFDFFFF lea     edx, \[ebp-204\]  
005949F6  |.  A1 D0CA5A00   mov     eax, \[5ACAD0\]  
005949FB  |.  8B00          mov     eax, \[eax\]  
005949FD >|.  E8 8AD4EDFF   call    00471E8C                         ;  ->Forms.TApplication.GetExeName(TApplication):AnsiString;  
00594A02  |.  8B85 FCFDFFFF mov     eax, \[ebp-204\]  
00594A08  |.  8D95 00FEFFFF lea     edx, \[ebp-200\]  
00594A0E >|.  E8 F15AE7FF   call    0040A504                         ;  ->Unit\_004086A8.Proc\_0040A504  
00594A13  |.  8B95 00FEFFFF mov     edx, \[ebp-200\]  
00594A19  |.  8D45 F8       lea     eax, \[ebp-8\]  
00594A1C  |.  B9 CC4E5900   mov     ecx, 00594ECC                    ;  ASCII "\\classes.lst"  
00594A21 >|.  E8 1605E7FF   call    00404F3C                         ;  ->System.@LStrCat3;  
00594A26  |.  8B45 F8       mov     eax, \[ebp-8\]  
00594A29 >|.  E8 6256E7FF   call    0040A090                         ;  ->Unit\_004086A8.Proc\_0040A090  
00594A2E  |.  84C0          test    al, al  
00594A30  |.  74 11         je      short 00594A43  
00594A32  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594A35 >|.  8B80 7C050000 mov     eax, \[eax+57C\]                   ;  \*TDeDeMainForm.OFFS\_057C:TStringList  
00594A3B  |.  8B55 F8       mov     edx, \[ebp-8\]  
00594A3E  |.  8B08          mov     ecx, \[eax\]  
00594A40 >|.  FF51 68       call    \[ecx+68\]                         ;  ->TStringList.LoadFromFile(string)  
00594A43  |>  8B45 FC       mov     eax, \[ebp-4\]  
00594A46  |.  05 78050000   add     eax, 578  
00594A4B >|.  E8 E801E7FF   call    00404C38                         ;  ->System.@LStrClr(void;void);  
00594A50 >|.  E8 EB2BE7FF   call    <jmp.&kernel32.GetVersion>       ;  ->kernel32.GetVersion()  
00594A55  |.  8945 F4       mov     \[ebp-C\], eax  
00594A58  |.  8B45 F4       mov     eax, \[ebp-C\]  
00594A5B  |.  25 00000080   and     eax, 80000000  
00594A60  |.  C1E8 20       shr     eax, 20  
00594A63  |.  8845 F3       mov     \[ebp-D\], al  
00594A66  |.  8A45 F4       mov     al, \[ebp-C\]  
00594A69  |.  24 0F         and     al, 0F  
00594A6B  |.  8845 F2       mov     \[ebp-E\], al  
00594A6E  |.  807D F3 00    cmp     byte ptr \[ebp-D\], 0  
00594A72  |.  74 06         je      short 00594A7A  
00594A74  |.  807D F2 04    cmp     byte ptr \[ebp-E\], 4  
00594A78  |.  77 0C         ja      short 00594A86  
00594A7A  |>  807D F3 00    cmp     byte ptr \[ebp-D\], 0  
00594A7E  |.  75 16         jnz     short 00594A96  
00594A80  |.  807D F2 03    cmp     byte ptr \[ebp-E\], 3  
00594A84  |.  76 10         jbe     short 00594A96  
00594A86  |>  8B45 FC       mov     eax, \[ebp-4\]  
00594A89 >|.  8B80 98030000 mov     eax, \[eax+398\]                   ;  \*TDeDeMainForm.RxGradientCaption1:TRxGradientCaption  
00594A8F  |.  33D2          xor     edx, edx  
00594A91 >|.  E8 6A3AF5FF   call    004E8500                         ;  ->:TRxCaption.\_PROC\_004E8500()  
00594A96  |>  B2 01         mov     dl, 1  
00594A98  |.  A1 24964100   mov     eax, \[419624\]  
00594A9D >|.  E8 56F3E6FF   call    00403DF8                         ;  ->System.TObject.Create(TObject;Boolean);  
00594AA2  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594AA5 >|.  8982 37050000 mov     \[edx+537\], eax                   ;  \*TDeDeMainForm.OFFS\_0537:TStringList  
00594AAB  |.  B2 01         mov     dl, 1  
00594AAD  |.  A1 24964100   mov     eax, \[419624\]  
00594AB2 >|.  E8 41F3E6FF   call    00403DF8                         ;  ->System.TObject.Create(TObject;Boolean);  
00594AB7  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594ABA >|.  8982 1F050000 mov     \[edx+51F\], eax                   ;  \*TDeDeMainForm.OFFS\_051F:TStringList  
00594AC0  |.  8D95 F4FDFFFF lea     edx, \[ebp-20C\]  
00594AC6  |.  A1 D0CA5A00   mov     eax, \[5ACAD0\]  
00594ACB  |.  8B00          mov     eax, \[eax\]  
00594ACD >|.  E8 BAD3EDFF   call    00471E8C                         ;  ->Forms.TApplication.GetExeName(TApplication):AnsiString;  
00594AD2  |.  8B85 F4FDFFFF mov     eax, \[ebp-20C\]  
00594AD8  |.  8D95 F8FDFFFF lea     edx, \[ebp-208\]  
00594ADE >|.  E8 215AE7FF   call    0040A504                         ;  ->Unit\_004086A8.Proc\_0040A504  
00594AE3  |.  8B95 F8FDFFFF mov     edx, \[ebp-208\]  
00594AE9  |.  8D45 F8       lea     eax, \[ebp-8\]  
00594AEC  |.  B9 E44E5900   mov     ecx, 00594EE4                    ;  ASCII "\\su.lst"  
00594AF1 >|.  E8 4604E7FF   call    00404F3C                         ;  ->System.@LStrCat3;  
00594AF6  |.  8B45 F8       mov     eax, \[ebp-8\]  
00594AF9 >|.  E8 9255E7FF   call    0040A090                         ;  ->Unit\_004086A8.Proc\_0040A090  
00594AFE  |.  84C0          test    al, al  
00594B00  |.  74 11         je      short 00594B13  
00594B02  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594B05 >|.  8B80 1F050000 mov     eax, \[eax+51F\]                   ;  \*TDeDeMainForm.OFFS\_051F:TStringList  
00594B0B  |.  8B55 F8       mov     edx, \[ebp-8\]  
00594B0E  |.  8B08          mov     ecx, \[eax\]  
00594B10 >|.  FF51 68       call    \[ecx+68\]                         ;  ->TStringList.LoadFromFile(string)  
00594B13  |>  C605 F0F55A00>mov     byte ptr \[5AF5F0\], 0  
00594B1A  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594B1D >|.  C680 1E050000>mov     byte ptr \[eax+51E\], 1            ;  \*TDeDeMainForm.OFFS\_051E:Byte  
00594B24  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594B27 >|.  C680 57050000>mov     byte ptr \[eax+557\], 0            ;  \*TDeDeMainForm.OFFS\_0557:Byte  
00594B2E  |.  B2 01         mov     dl, 1  
00594B30  |.  A1 8CDB5800   mov     eax, \[58DB8C\]  
00594B35 >|.  E8 1ACBFFFF   call    <<-TDeDeMainForm@Proc\_00591654>  ;  ->:TDeDeMainForm.Proc\_00591654()  
00594B3A  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594B3D >|.  8982 74050000 mov     \[edx+574\], eax                   ;  \*TDeDeMainForm.OFFS\_0574:TStringList  
00594B43 >|.  E8 E8E0E6FF   call    00402C30                         ;  ->System.ParamCount:Integer;  
00594B48  |.  85C0          test    eax, eax  
00594B4A  |.  74 6D         je      short 00594BB9  
00594B4C  |.  8D95 F0FDFFFF lea     edx, \[ebp-210\]  
00594B52  |.  B8 01000000   mov     eax, 1  
00594B57 >|.  E8 34E1E6FF   call    00402C90                         ;  ->System.ParamStr(Integer):String;  
00594B5C  |.  8B85 F0FDFFFF mov     eax, \[ebp-210\]  
00594B62  |.  BA F44E5900   mov     edx, 00594EF4                    ;  ASCII "more"  
00594B67 >|.  E8 C804E7FF   call    00405034                         ;  ->System.@LStrCmp;  
00594B6C  |.  75 4B         jnz     short 00594BB9  
00594B6E  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594B71 >|.  8B80 18040000 mov     eax, \[eax+418\]                   ;  \*TDeDeMainForm.ClassesLV:TListView  
00594B77  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594B7A >|.  8990 2C010000 mov     \[eax+12C\], edx                   ;  \*TListView.OFFS\_012C  
00594B80 >|.  C780 28010000>mov     dword ptr \[eax+128\], <<-TDeDeMai>;  \*TListView.OnDblClick:TNotifyEvent  
00594B8A  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594B8D >|.  8B80 28040000 mov     eax, \[eax+428\]                   ;  \*TDeDeMainForm.DAP:TMenuItem  
00594B93  |.  B2 01         mov     dl, 1  
00594B95 >|.  E8 9EE8ECFF   call    00463438                         ;  ->Menus.TMenuItem.SetVisible(TMenuItem;Boolean);  
00594B9A  |.  A1 A4C55A00   mov     eax, \[5AC5A4\]  
00594B9F  |.  C600 01       mov     byte ptr \[eax\], 1  
00594BA2  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594BA5 >|.  8B90 C4040000 mov     edx, \[eax+4C4\]                   ;  \*TDeDeMainForm.publh:TPopupMenu  
00594BAB  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594BAE >|.  8B80 C0040000 mov     eax, \[eax+4C0\]                   ;  \*TDeDeMainForm.UnitDataLV:TListView  
00594BB4 >|.  E8 7BC8EBFF   call    00451434                         ;  ->Controls.TControl.SetPopupMenu(TControl;TPopupMenu);  
00594BB9  |>  8B45 FC       mov     eax, \[ebp-4\]  
00594BBC >|.  E8 474D0000   call    <<-TDeDeMainForm@Proc\_00599908>  ;  ->:TDeDeMainForm.Proc\_00599908()  
00594BC1  |.  B2 01         mov     dl, 1  
00594BC3  |.  A1 24964100   mov     eax, \[419624\]  
00594BC8 >|.  E8 2BF2E6FF   call    00403DF8                         ;  ->System.TObject.Create(TObject;Boolean);  
00594BCD  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594BD0 >|.  8982 33050000 mov     \[edx+533\], eax                   ;  \*TDeDeMainForm.OFFS\_0533:TStringList  
00594BD6  |.  B2 01         mov     dl, 1  
00594BD8  |.  A1 24964100   mov     eax, \[419624\]  
00594BDD >|.  E8 16F2E6FF   call    00403DF8                         ;  ->System.TObject.Create(TObject;Boolean);  
00594BE2  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594BE5 >|.  8982 3B050000 mov     \[edx+53B\], eax                   ;  \*TDeDeMainForm.OFFS\_053B:TStringList  
00594BEB  |.  B2 01         mov     dl, 1  
00594BED  |.  A1 24964100   mov     eax, \[419624\]  
00594BF2 >|.  E8 01F2E6FF   call    00403DF8                         ;  ->System.TObject.Create(TObject;Boolean);  
00594BF7  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594BFA >|.  8982 3F050000 mov     \[edx+53F\], eax                   ;  \*TDeDeMainForm.OFFS\_053F:TStringList  
00594C00  |.  B2 01         mov     dl, 1  
00594C02  |.  A1 24964100   mov     eax, \[419624\]  
00594C07 >|.  E8 ECF1E6FF   call    00403DF8                         ;  ->System.TObject.Create(TObject;Boolean);  
00594C0C  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594C0F >|.  8982 47050000 mov     \[edx+547\], eax                   ;  \*TDeDeMainForm.OFFS\_0547:TStringList  
00594C15  |.  B2 01         mov     dl, 1  
00594C17  |.  A1 24964100   mov     eax, \[419624\]  
00594C1C >|.  E8 D7F1E6FF   call    00403DF8                         ;  ->System.TObject.Create(TObject;Boolean);  
00594C21  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594C24 >|.  8982 4B050000 mov     \[edx+54B\], eax                   ;  \*TDeDeMainForm.OFFS\_054B:TStringList  
00594C2A  |.  B2 01         mov     dl, 1  
00594C2C  |.  A1 24964100   mov     eax, \[419624\]  
00594C31 >|.  E8 C2F1E6FF   call    00403DF8                         ;  ->System.TObject.Create(TObject;Boolean);  
00594C36  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594C39 >|.  8982 4F050000 mov     \[edx+54F\], eax                   ;  \*TDeDeMainForm.OFFS\_054F:TStringList  
00594C3F  |.  B2 01         mov     dl, 1  
00594C41  |.  A1 9CB75800   mov     eax, \[58B79C\]  
00594C46 >|.  E8 DD6FFFFF   call    0058BC28                         ;  ->Unit\_0058B6B0.Proc\_0058BC28  
00594C4B  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594C4E >|.  8982 23050000 mov     \[edx+523\], eax                   ;  \*TDeDeMainForm.OFFS\_0523  
00594C54  |.  8B4D FC       mov     ecx, \[ebp-4\]  
00594C57  |.  B2 01         mov     dl, 1  
00594C59  |.  A1 B0174F00   mov     eax, \[4F17B0\]  
00594C5E >|.  E8 A9CCF5FF   call    004F190C                         ;  ->FileDrop.Proc\_004F190C  
00594C63  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594C66 >|.  8982 27050000 mov     \[edx+527\], eax                   ;  \*TDeDeMainForm.OFFS\_0527  
00594C6C  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594C6F >|.  8B80 27050000 mov     eax, \[eax+527\]                   ;  \*TDeDeMainForm.OFFS\_0527  
00594C75  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594C78  |.  8950 49       mov     \[eax+49\], edx  
00594C7B  |.  C740 45 D0385>mov     dword ptr \[eax+45\], 005A38D0  
00594C82  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594C85 >|.  8B80 27050000 mov     eax, \[eax+527\]                   ;  \*TDeDeMainForm.OFFS\_0527  
00594C8B  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594C8E >|.  E8 EDCDF5FF   call    004F1A80                         ;  ->:TFileDrop.\_PROC\_004F1A80()  
00594C93  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594C96 >|.  8B80 27050000 mov     eax, \[eax+527\]                   ;  \*TDeDeMainForm.OFFS\_0527  
00594C9C  |.  B2 01         mov     dl, 1  
00594C9E >|.  E8 4DCEF5FF   call    004F1AF0                         ;  ->:TFileDrop.\_PROC\_004F1AF0()  
00594CA3  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594CA6  |.  33D2          xor     edx, edx  
00594CA8 >|.  8990 53050000 mov     \[eax+553\], edx                   ;  \*TDeDeMainForm.OFFS\_0553:TPEHeader  
00594CAE  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594CB1 >|.  E8 F6F8FFFF   call    <<-TDeDeMainForm@Proc\_005945AC>  ;  ->:TDeDeMainForm.Proc\_005945AC()  
00594CB6  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594CB9 >|.  C680 58050000>mov     byte ptr \[eax+558\], 0            ;  \*TDeDeMainForm.OFFS\_0558:Byte  
00594CC0  |.  8D95 ECFDFFFF lea     edx, \[ebp-214\]  
00594CC6  |.  A1 D0CA5A00   mov     eax, \[5ACAD0\]  
00594CCB  |.  8B00          mov     eax, \[eax\]  
00594CCD >|.  E8 BAD1EDFF   call    00471E8C                         ;  ->Forms.TApplication.GetExeName(TApplication):AnsiString;  
00594CD2  |.  8B85 ECFDFFFF mov     eax, \[ebp-214\]  
00594CD8  |.  8D4D F8       lea     ecx, \[ebp-8\]  
00594CDB  |.  BA 044F5900   mov     edx, 00594F04                    ;  ASCII ".fls"  
00594CE0 >|.  E8 5757E7FF   call    0040A43C                         ;  ->Unit\_004086A8.Proc\_0040A43C  
00594CE5  |.  8B45 F8       mov     eax, \[ebp-8\]  
00594CE8 >|.  E8 A353E7FF   call    0040A090                         ;  ->Unit\_004086A8.Proc\_0040A090  
00594CED  |.  84C0          test    al, al  
00594CEF  |.  75 2E         jnz     short 00594D1F  
00594CF1  |.  8B55 F8       mov     edx, \[ebp-8\]  
00594CF4  |.  8D85 1CFEFFFF lea     eax, \[ebp-1E4\]  
00594CFA >|.  E8 45E3E6FF   call    00403044                         ;  ->System.@Assign(TTextRec;TTextRec;String):Integer;  
00594CFF  |.  8D85 1CFEFFFF lea     eax, \[ebp-1E4\]  
00594D05 >|.  E8 D6E0E6FF   call    00402DE0                         ;  ->System.@RewritText(TTextRec;TTextRec):Integer;  
00594D0A >|.  E8 29DCE6FF   call    00402938                         ;  ->System.Proc\_00402938  
00594D0F  |.  8D85 1CFEFFFF lea     eax, \[ebp-1E4\]  
00594D15 >|.  E8 A6E4E6FF   call    004031C0                         ;  ->System.@Close(TTextRec;TTextRec):Integer;  
00594D1A >|.  E8 19DCE6FF   call    00402938                         ;  ->System.Proc\_00402938  
00594D1F  |>  8B45 FC       mov     eax, \[ebp-4\]  
00594D22 >|.  8B80 94030000 mov     eax, \[eax+394\]                   ;  \*TDeDeMainForm.RecentFileEdit:TComboBox  
00594D28 >|.  8B80 3C020000 mov     eax, \[eax+23C\]                   ;  \*TComboBox.Items:TStrings  
00594D2E  |.  8B55 F8       mov     edx, \[ebp-8\]  
00594D31  |.  8B08          mov     ecx, \[eax\]  
00594D33 >|.  FF51 68       call    \[ecx+68\]                         ;  ->TStrings.LoadFromFile(string)  
00594D36  |.  B2 01         mov     dl, 1  
00594D38  |.  A1 24964100   mov     eax, \[419624\]  
00594D3D >|.  E8 B6F0E6FF   call    00403DF8                         ;  ->System.TObject.Create(TObject;Boolean);  
00594D42  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594D45 >|.  8982 6C050000 mov     \[edx+56C\], eax                   ;  \*TDeDeMainForm.OFFS\_056C:TStringList  
00594D4B  |.  B2 01         mov     dl, 1  
00594D4D  |.  A1 488E4100   mov     eax, \[418E48\]  
00594D52 >|.  E8 A1F0E6FF   call    00403DF8                         ;  ->System.TObject.Create(TObject;Boolean);  
00594D57  |.  8B55 FC       mov     edx, \[ebp-4\]  
00594D5A >|.  8982 68050000 mov     \[edx+568\], eax                   ;  \*TDeDeMainForm.OFFS\_0568:TList  
00594D60  |.  A1 50CF5A00   mov     eax, \[5ACF50\]  
00594D65  |.  8B00          mov     eax, \[eax\]  
00594D67  |.  66:BA F5FF    mov     dx, 0FFF5  
00594D6B >|.  E8 70ABEDFF   call    0046F8E0                         ;  ->Forms.TScreen.SetCursor(TScreen;TCursor);  
00594D70  |.  33C0          xor     eax, eax  
00594D72  |.  55            push    ebp  
00594D73  |.  68 A24D5900   push    <->System.@HandleFinally;>  
00594D78  |.  64:FF30       push    dword ptr fs:\[eax\]  
00594D7B  |.  64:8920       mov     fs:\[eax\], esp  
00594D7E  |.  8B45 FC       mov     eax, \[ebp-4\]  
00594D81 >|.  E8 62530000   call    <<-TDeDeMainForm@Proc\_0059A0E8>  ;  ->:TDeDeMainForm.Proc\_0059A0E8()  
00594D86  |.  33C0          xor     eax, eax  
00594D88  |.  5A            pop     edx  
00594D89  |.  59            pop     ecx  
00594D8A  |.  59            pop     ecx  
00594D8B  |.  64:8910       mov     fs:\[eax\], edx  
00594D8E  |.  68 A94D5900   push    00594DA9  
00594D93  |>  A1 50CF5A00   mov     eax, \[5ACF50\]  
00594D98  |.  8B00          mov     eax, \[eax\]  
00594D9A  |.  33D2          xor     edx, edx  
00594D9C >|.  E8 3FABEDFF   call    0046F8E0                         ;  ->Forms.TScreen.SetCursor(TScreen;TCursor);  
00594DA1  \\.  C3            retn  
00594DA2 > .^ E9 15F8E6FF   jmp     004045BC                         ;  ->System.@HandleFinally;  
00594DA7   .^ EB EA         jmp     short 00594D93  
00594DA9   .  8D85 E8FDFFFF lea     eax, \[ebp-218\]  
00594DAF > .  E8 38D30000   call    <<-TDeDeMainForm@Proc\_005A20EC>------->关键过程 TDeDeMainForm.Proc\_005A20EC(),该方法是根据 Dede 版本号,文件全名,Xp ProduceID 号计算出一个 SHA1 散列值!  
00594DB4   .  8B95 E8FDFFFF mov     edx, \[ebp-218\]  
00594DBA   .  8B45 FC       mov     eax, \[ebp-4\]  
00594DBD   .  05 2F050000   add     eax, 52F  
00594DC2 > .  E8 C5FEE6FF   call    00404C8C    -- ->System.@LStrAsg(void;void;void;void);  
00594DC7   .  8B45 FC       mov     eax, \[ebp-4\]  
00594DCA > .  8B80 2F050000 mov     eax, \[eax+52F\] -------->每一次启动计算出的 Dede,SHA1 散列值  
00594DD0   .  8B15 34C85A00 mov     edx, \[5AC834\] ------>ini 文件中的 SHA1 Version 变量数值  
00594DD6   .  8B12          mov     edx, \[edx\]  
00594DD8 > .  E8 5702E7FF   call    00405034  ------>关键比较 System.@LStrCmp;  
00594DDD   .  0F95C0        setne   al------>确定 Boolean 变量 al 的状态  
00594DE0   .  8B55 FC       mov     edx, \[ebp-4\]  
00594DE3 > .  8882 80050000 mov     \[edx+580\], al                    ;  \*TDeDeMainForm.OFFS\_0580:Byte  
00594DE9   .  8B45 FC       mov     eax, \[ebp-4\]  
00594DEC > .  80B8 80050000>cmp     byte ptr \[eax+580\], 0            ;  \*TDeDeMainForm.OFFS\_0580:Byte  
00594DF3   .  74 3D         je      short 00594E32--------------->关键跳,跳就不产生 Nag,否相反!  
00594DF5   .  8B0D ECF55A00 mov     ecx, \[5AF5EC\]  
00594DFB   .  B2 01         mov     dl, 1  
00594DFD   .  A1 2C204400   mov     eax, \[44202C\]----->TTimer 类,下面将产生 TTimer 对象  
00594E02 > .  E8 6DFCEAFF   call    00444A74  ------------->ExtCtrls.TTimer.Create(TTimer;boolean;TComponent);  
00594E07   .  8945 EC       mov     \[ebp-14\], eax             
00594E0A   .  BA B80B0000   mov     edx, 0BB8------------>设置 0BB8 毫秒  
00594E0F   .  8B45 EC       mov     eax, \[ebp-14\]  
00594E12 > .  E8 F5FDEAFF   call    00444C0C  ----------->ExtCtrls.TTimer.SetInterval(TTimer;Cardinal);  
00594E17   .  8B45 FC       mov     eax, \[ebp-4\]  
00594E1A   .  50            push    eax                                
00594E1B   .  68 1C485900   push    0059481C ---------------->关键的 TTimer 回调事件,它是产生 Nag 方法句柄!                          
00594E20   .  8B45 EC       mov     eax, \[ebp-14\]                     
00594E23 > .  E8 F4FDEAFF   call    00444C1C --------------->:TSplitter.\_PROC\_00444C1C()  
00594E28   .  B2 01         mov     dl, 1  
00594E2A   .  8B45 EC       mov     eax, \[ebp-14\]  
00594E2D > .  E8 CAFDEAFF   call    00444BFC                         ;  ->ExtCtrls.TTimer.SetEnabled(TTimer;Boolean);  
00594E32   >  8B45 FC       mov     eax, \[ebp-4\]  
00594E35 > .  8B80 F0030000 mov     eax, \[eax+3F0\]                   ;  \*TDeDeMainForm.ClearTimer:TTimer  
00594E3B   .  BA 88130000   mov     edx, 1388  
00594E40 > .  E8 C7FDEAFF   call    00444C0C                         ;  ->ExtCtrls.TTimer.SetInterval(TTimer;Cardinal);  
00594E45   .  8B45 FC       mov     eax, \[ebp-4\]  
00594E48 > .  8B80 F0030000 mov     eax, \[eax+3F0\]                   ;  \*TDeDeMainForm.ClearTimer:TTimer  
00594E4E   .  33D2          xor     edx, edx  
00594E50 > .  E8 A7FDEAFF   call    00444BFC                         ;  ->ExtCtrls.TTimer.SetEnabled(TTimer;Boolean);  
00594E55   .  C605 E8BF5A00>mov     byte ptr \[5ABFE8\], 0  
00594E5C   .  B2 01         mov     dl, 1  
00594E5E   .  A1 7CF64E00   mov     eax, \[4EF67C\]  
00594E63 > .  E8 90EFE6FF   call    00403DF8                         ;  ->System.TObject.Create(TObject;Boolean);  
00594E68   .  A3 F4F55A00   mov     \[5AF5F4\], eax  
00594E6D   .  33C0          xor     eax, eax  
00594E6F   .  5A            pop     edx  
00594E70   .  59            pop     ecx  
00594E71   .  59            pop     ecx  
00594E72   .  64:8910       mov     fs:\[eax\], edx  
00594E75   .  68 9A4E5900   push    00594E9A  
00594E7A   >  8D85 E8FDFFFF lea     eax, \[ebp-218\]  
00594E80   .  BA 0D000000   mov     edx, 0D  
00594E85 > .  E8 D2FDE6FF   call    00404C5C                         ;  ->System.@LStrArrayClr(void;void;Integer);  
00594E8A   .  8D45 F8       lea     eax, \[ebp-8\]  
00594E8D > .  E8 A6FDE6FF   call    00404C38                         ;  ->System.@LStrClr(void;void);  
00594E92   .  C3            retn  
00594E93 > .^ E9 24F7E6FF   jmp     004045BC                         ;  ->System.@HandleFinally;  
00594E98   .^ EB E0         jmp     short 00594E7A  
00594E9A   .  8BE5          mov     esp, ebp  
00594E9C   .  5D            pop     ebp  
00594E9D   .  C3            retn  
  
  
如果是重装了系统或者是改变了文件路径或者是文件名此时候 Dede 的 Nag 一定会再次询问 License Statement 声明,你同意与否.  
在<-TDeDeMainForm@FormCreate 事件中动态产生了 TTimer 组件,并且注册了 TTimer 对象的回调事件地址:0059481C.注意这是用 Dede 反编译自己得不出来的！  
  
下面看:TTimer 对象的回调事件地址:0059481C,这个事件将会产生 Nag 窗体的程序.  
  
0059481C  /.  55            push    ebp  
0059481D  |.  8BEC          mov     ebp, esp  
0059481F  |.  83C4 F0       add     esp, -10  
00594822  |.  8955 F8       mov     \[ebp-8\], edx  
00594825  |.  8945 FC       mov     \[ebp-4\], eax  
00594828  |.  8B45 F8       mov     eax, \[ebp-8\]  
0059482B  |.  8B15 2C204400 mov     edx, \[44202C\]  ------->TTimer  
00594831  |.  E8 A2F7E6FF   call    00403FD8  ---->类检测 System.@AsClass(TObject;TClass):TObject;  
00594836  |.  33D2          xor     edx, edx----->Boolean:=False;  
00594838  |.  E8 BF03EBFF   call    00444BFC    ---->注销这个 Timer ExtCtrls.TTimer.SetEnabled(TTimer;Boolean);  
0059483D  |.  33C9          xor     ecx, ecx  
0059483F  |.  B2 01         mov     dl, 1  
00594841  |.  A1 ECAA5500   mov     eax, \[55AAEC\]---->Nag 窗体的 TLSForm 类.下面产生 Nag 对象  
00594846  |.  E8 F958EDFF   call    0046A144 ------->TCustomForm.Create(TCustomForm;boolean;TComponent);  
0059484B  |.  8945 F4       mov     \[ebp-C\], eax  
0059484E  |.  33C0          xor     eax, eax  
00594850  |.  55            push    ebp  
00594851  |.  68 89485900   push    00594889  
00594856  |.  64:FF30       push    dword ptr fs:\[eax\]  
00594859  |.  64:8920       mov     fs:\[eax\], esp  
0059485C  |.  8B45 F4       mov     eax, \[ebp-C\]  
0059485F  |.  8B10          mov     edx, \[eax\]  
00594861  |.  FF92 E8000000 call    \[edx+E8\] ------------->显示 Nag 窗体 TLSForm.ShowModal  
00594867  |.  8B45 F4       mov     eax, \[ebp-C\]  
0059486A  |.  8B80 4C020000 mov     eax, \[eax+24C\] ---->Nag 窗体的 TModalResult  
00594870  |.  8945 F0       mov     \[ebp-10\], eax  
00594873  |.  33C0          xor     eax, eax  
00594875  |.  5A            pop     edx  
00594876  |.  59            pop     ecx  
00594877  |.  59            pop     ecx  
00594878  |.  64:8910       mov     fs:\[eax\], edx  
0059487B  |.  68 90485900   push    00594890  
00594880  |>  8B45 F4       mov     eax, \[ebp-C\]  
00594883  |.  E8 A0F5E6FF   call    00403E28                         ;  System.TObject.Free  
00594888  \\.  C3            retn  
00594889   .^ E9 2EFDE6FF   jmp     004045BC  
0059488E   .^ EB F0         jmp     short 00594880  
00594890   .  837D F0 01    cmp     dword ptr \[ebp-10\], 1---------------->关键比较  
00594894   .  75 1E         jnz     short 005948B4------->关键跳:如果 Nag 返回 ModalResult 不相等 1,就会结束程序的运行.  
00594896   .  A1 34C85A00   mov     eax, \[5AC834\]------>关键全局变量,保存 Dede SHA1 散列值  
0059489B   .  8B55 FC       mov     edx, \[ebp-4\]  
0059489E   .  8B92 2F050000 mov     edx, \[edx+52F\]                   ;  \* Reference to: System.@LStrAsg(void;void;void;void);  
005948A4   .  E8 E303E7FF   call    00404C8C  
005948A9   .  B2 01         mov     dl, 1  
005948AB   .  33C0          xor     eax, eax  
005948AD   .  E8 6AE3F5FF   call    004F2C1C---------------->把新计算 Dede SHA1 string\_\[5AC834\]散列值写回 ini 文件中.  
005948B2   .  EB 0C         jmp     short 005948C0  
005948B4   >  A1 D0CA5A00   mov     eax, \[5ACAD0\]  
005948B9   .  8B00          mov     eax, \[eax\]  
005948BB   .  E8 E0D0EDFF   call    004719A0------>如果不同意其 Licence,就会结束主程序.调用 Forms.TApplication.Terminate(TApplication);  
005948C0   >  8BE5          mov     esp, ebp  
005948C2   .  5D            pop     ebp  
005948C3   .  C3            retn  
  
  
到这里重要关键的地方分析完毕,你可以动一下鼠标就 game over 了.剩下的 Nag 等窗体上的事件流程用 dede 比较容易分析出来.在这里不分析了留给大家.  
  
  
  
Dede 采用的是 SHA1 计算 Licence 数据的.SHA1 的计算器很多,只要根据注册表中的 OS ID 号,Dede 的版本号,和 Dede 所在的本地路径全名为参数,就可以算出 Dede 的 Licence 数据来!  
现在论坛不是很流行算法就地取材么,同过导出本身的 exe,dll 关键算法函数,计算出 key 来.偶今天大体也作一下思路.  
  
给 Dede 加新区块：  
Name:qiweixue  
VitualAddress:00227000  
VitualSize:00000100  
RawOffset:0021C000  
RawSize:00000100  
Flags:F00000E0  
  
在新区块中加入导出函数表如下:  
Export Table  
   Characteristics:        0x00000000  
   TimeDateStamp:          0x00000000  (GMT: Thu Jan 01 00:00:00 1970)  
   MajorVersion:           0x0000  
   MinorVersion:           0x0000  -> 0.00  
   Name:                   0x00227030  ("dede.exe")  
   Base:                   0x00000001  
   NumberOfFunctions:      0x00000001  
   NumberOfNames:          0x00000001  
   AddressOfFunctions:     0x0022703C  
   AddressOfNames:         0x00227040  
   AddressOfNameOrdinals:  0x00227038  
  
   Ordinal RVA        Symbol Name  
   ------- ---------- ----------------------------------  
   0x0001  0x001A5862 "DDSHA1"  
  
内存地址如下:  
00627000  00 00 00 00 00 00 00 00 00 00 00 00 30 70 22 00  ............0p".  
00627010  01 00 00 00 01 00 00 00 01 00 00 00 3C 70 22 00  .........<p".  
00627020  40 70 22 00 38 70 22 00 00 00 00 00 00 00 00 00  @p".8p".........  
00627030  64 65 64 65 2E 65 78 65 00 00 00 00 62 58 1A 00  dede.exe....bX.  
00627040  44 70 22 00 44 44 53 48 41 31 00                 Dp".DDSHA1.  
  
导出函数 DDSHA1(stdcall 约定):  
           push    ebp  
           mov     ebp, esp  
           mov     eax, \[ebp+8\]//变寄存器约定,eax 为 out SHA1 散列字符串的地址.  
           call    005A20EC //如果采用进程间调用 DDSHA1 的话,这个函数里边需要很多重定位的数据.自己搞定.如果在同一个段调用就无须重定位,如自动弹出其 License 码!  
           pop     ebp  
           retn    4  
  
  
  
  
  
段外调用代码如下(注意 call 005A20EC 内部数据需要重定位):  
  
#include <windows.h>  
typedef string (\*CallDDSHA1)(LPSTR str )  
int main()  
{  
...  
HINSTANCE hInst;  
LPSTR outsha1;  
hInst=LoadLibrary("Dede.exe");  
CallDDSHA1 democall=(CallDDSHA1)GetProcAddress(hInst,"DDSHA1");  
democall(outsha1);  
...  
}  
  
如果在段内直接谈出它的 Key 或者是加个菜单调用这个函数则不需要特殊的重定位!  
完毕!  
有错之出,多谢指教! 

 4dd466d0e1e0cc123afbe504da2462ca