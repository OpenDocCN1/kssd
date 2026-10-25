
# LDM 木马检测程序(反木马) VI Build 1500 算法分析(2007 年 3 月 18 日发布)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-89.htm`](https://www.pediy.com/kssd/pediy09/pediy09-89.htm)

> 【文章标题】: LDM 木马检测程序(反木马) VI Build 1500 算法分析(2007 年 3 月 18 日发布)  
> 【文章作者】: 壹只老虎  
> 【作者邮箱】: tiger..tiger@163.com  
> 【详细过程】  

1：安装程序，发现有 5 个可执行程序，一看就知道是他们写的一套东西。看了看软件，做得很华丽，有点心虚了，不过还是继续，运行后发现 `lavs.exe` 需要注册，其他的都不需要注册的，`PEID` 查壳，`nSpack V2.9 -> LiuXingPing`，`esp` 定律脱壳后用 `importrec` 修复下就可以了。看看是什么写的？`Microsoft Visual Basic 5.0 / 6.0`，晕，我，我怕这个东西，试试看吧，也许会有收获，注册试试看，它里面有生成好的机器码，我们只需要写注册码就可以了，我来写，`asdsadsa,asfsaf,asfsaf` 错误了，(后来吃亏了才发现需要数字的！汗！)呵呵。这里需要输入数字才行，小数也可以的。至于为什么，因为，作者采用了字符串转浮点的函数进行转化，发现不是合法的数字就错误。好了，输入试验码，`111111`，`2222222`，`33333333`，不急，先把天网开起来，怕是网络验证的，注册，黑黑，天网没有报警，说明是本地验证的，好了，有错误提示，非常好，“你的注册码错误，请确认清楚”。虽然有这个字符串，不过对于 `vb` 的东西，我不指望能哪来怎么用，(不是不行，是替换字符串比较麻烦)。碰下运气，下函数断点看看。（这里有件比较有意思的事情需要说一下，你看那个脱壳前的 `lavs.exe` 大小是 `1.36M`，脱壳后的程序是 `15.0M` 我晕，这也太夸张了吧，不佩服不行啊，都不知道里面加了些什么，写的这么大，不过这个程序的界面做的是相当美观阿，估计是界面用去了很多空间，。注意下。`od` 脱壳的时候需要有耐心，发现 `od` 窗口假死个 `20-30s` 也不要急，这东西是有点大了，可以去泡杯茶，泡好了也差不多脱完了，黑黑）。修复下就行乐。

2：`od` 载入修复后的程序，不要字符串参考了，没用，直接运行程序，好了，注册提示出来了，要我们写注册码。我写上 `111111`，`2222222`，`33333333`，不要忙着点按钮，回 `od` 里面 `ctrl+n`，找到 `msvbvm60.rtcMsgBox` 这个函数，在每个参考点上设置断点，`ok`，注册把。

3：好了，断下来了，这里。

```
011907FC    .  FF15 C8104000     call dword ptr ds:[<&msvbvm60.rtcM>;  注册失败
```

往上面找到头，好长的代码的，灰心了，不过还好，里面垃圾代码太多，看完了你就知道了，下面是分析过程。

```
|01190090    > \ 55               push ebp
|01190091    .  8BEC             mov ebp,esp
|01190093    .  83EC 14          sub esp,14
|01190096    .  68 362C4000      push <jmp.&msvbvm60.__vbaExceptHan>;  SE handler installation
|0119009B    .  64:A1 00000000   mov eax,dword ptr fs:[0]
|011900A1    .  50               push eax
|011900A2    .  64:8925 00000000 mov dword ptr fs:[0],esp
|011900A9    .  81EC D4000000    sub esp,0D4
|011900AF    .  53               push ebx
|011900B0    .  56               push esi
|011900B1    .  57               push edi
|011900B2    .  8965 EC          mov dword ptr ss:[ebp-14],esp
|011900B5    .  C745 F0 602B4000 mov dword ptr ss:[ebp-10],lavss1_.>
|011900BC    .  8B75 08          mov esi,dword ptr ss:[ebp+8]
|011900BF    .  8BC6             mov eax,esi                     ;  EAX=ESI=1A3B41
|011900C1    .  83E0 01          and eax,1                       ;  EAX=1
|011900C4    .  8945 F4          mov dword ptr ss:[ebp-C],eax
|011900C7    .  83E6 FE          and esi,FFFFFFFE
|011900CA    .  8975 08          mov dword ptr ss:[ebp+8],esi
|011900CD    .  33DB             xor ebx,ebx
|011900CF    .  895D F8          mov dword ptr ss:[ebp-8],ebx
|011900D2    .  8B0E             mov ecx,dword ptr ds:[esi]
|011900D4    .  56               push esi
|011900D5    .  FF51 04          call dword ptr ds:[ecx+4]
|011900D8    .  895D E0          mov dword ptr ss:[ebp-20],ebx
|011900DB    .  895D DC          mov dword ptr ss:[ebp-24],ebx
|011900DE    .  895D D8          mov dword ptr ss:[ebp-28],ebx
|011900E1    .  895D D4          mov dword ptr ss:[ebp-2C],ebx
|011900E4    .  895D D0          mov dword ptr ss:[ebp-30],ebx
|011900E7    .  895D CC          mov dword ptr ss:[ebp-34],ebx
|011900EA    .  895D C8          mov dword ptr ss:[ebp-38],ebx
|011900ED    .  895D B8          mov dword ptr ss:[ebp-48],ebx
|011900F0    .  895D A8          mov dword ptr ss:[ebp-58],ebx
|011900F3    .  895D 98          mov dword ptr ss:[ebp-68],ebx
|011900F6    .  895D 88          mov dword ptr ss:[ebp-78],ebx
|011900F9    .  899D 78FFFFFF    mov dword ptr ss:[ebp-88],ebx
|011900FF    .  899D 68FFFFFF    mov dword ptr ss:[ebp-98],ebx
|01190105    .  6A 01            push 1
|01190107    .  FF15 CC104000    call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaOnError
|0119010D    .  8B16             mov edx,dword ptr ds:[esi]
|0119010F    .  56               push esi
|01190110    .  FF92 0C030000    call dword ptr ds:[edx+30C]        
|01190116    .  50               push eax
|01190117    .  8D45 D0          lea eax,dword ptr ss:[ebp-30]
|0119011A    .  50               push eax
|0119011B    .  FF15 C4104000    call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaObjSet
|01190121    .  8BF8             mov edi,eax
|01190123    .  8B0F             mov ecx,dword ptr ds:[edi]
|01190125    .  8D55 DC          lea edx,dword ptr ss:[ebp-24]
|01190128    .  52               push edx
|01190129    .  57               push edi
|0119012A    .  FF91 A0000000    call dword ptr ds:[ecx+A0]
|01190130    .  DBE2             fclex
|01190132    .  3BC3             cmp eax,ebx
|01190134    .  7D 12            jge short lavss1_.01190148
|01190136    .  68 A0000000      push 0A0
|0119013B    .  68 D4564600      push lavss1_.004656D4
|01190140    .  57               push edi
|01190141    .  50               push eax
|01190142    .  FF15 98104000    call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaHresultCheckObj
|01190148    >  8B06             mov eax,dword ptr ds:[esi]
|0119014A    .  56               push esi
|0119014B    .  FF90 FC020000    call dword ptr ds:[eax+2FC]
|01190151    .  50               push eax
|01190152    .  8D4D CC          lea ecx,dword ptr ss:[ebp-34]
|01190155    .  51               push ecx
|01190156    .  FF15 C4104000    call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaObjSet
|0119015C    .  8BF8             mov edi,eax
|0119015E    .  8B17             mov edx,dword ptr ds:[edi]
|01190160    .  8D45 D8          lea eax,dword ptr ss:[ebp-28]
|01190163    .  50               push eax
|01190164    .  57               push edi
|01190165    .  FF92 A0000000    call dword ptr ds:[edx+A0]
|0119016B    .  DBE2             fclex
|0119016D    .  3BC3             cmp eax,ebx
|0119016F    .  7D 12            jge short lavss1_.01190183
|01190171    .  68 A0000000      push 0A0
|01190176    .  68 D4564600      push lavss1_.004656D4
|0119017B    .  57               push edi
|0119017C    .  50               push eax
|0119017D    .  FF15 98104000    call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaHresultCheckObj
```



> `|01190183    >  8B0E               mov ecx,dword ptr ds:\[esi\]`  
>
> `|01190185    .  56                 push esi`  
>
> `|01190186    .  FF91 08030000      call dword ptr ds:\[ecx+308\]`  
>
> `|0119018C    .  50                 push eax`  
>
> `|0119018D    .  8D55 C8            lea edx,dword ptr ss:\[ebp-38\]`  
>
> `|01190190    .  52                 push edx`  
>
> `|01190191    .  FF15 C4104000      call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaObjSet`  
>
> `|01190197    .  8BF8              mov edi,eax`  
>
> `|01190199    .  8B07              mov eax,dword ptr ds:\[edi\]`  
>
> `|0119019B    .  8D4D D4            lea ecx,dword ptr ss:\[ebp-2C\]`  
>
> `|0119019E    .  51                 push ecx`  
>
> `|0119019F    .  57                 push edi`  
>
> `|011901A0    .  FF90 A0000000      call dword ptr ds:\[eax+A0\]`  
>
> `|011901A6    .  DBE2              fclex`  
>
> `|011901A8    .  3BC3              cmp eax,ebx`  
>
> `|011901AA    .  7D 12             jge short lavss1\_.011901BE`  
>
> `|011901AC    .  68 A0000000       push 0A0`  
>
> `|011901B1    .  68 D4564600       push lavss1\_.004656D4`  
>
> `|011901B6    .  57                 push edi`  
>
> `|011901B7    .  50                 push eax`  
>
> `|011901B8    .  FF15 98104000      call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaHresultCheckObj`  
>
> `|011901BE    >  8B55 D4            mov edx,dword ptr ss:\[ebp-2C\]      ;  EBX=试验码第三部分`  
>
> `|011901C1    .  52                 push edx`  
>
> `|011901C2    .  68 98684600       push lavss1\_.00466898`  
>
> `|011901C7    .  8B1D 24114000      mov ebx,dword ptr ds:\[<&msvbvm60.\_>;  msvbvm60.\_\_vbaStrCmp`  
>
> `|011901CD    .  FFD3              call ebx                    ;  <&msvbvm60.\_\_vbaStrCmp>`  
>
> `|011901CF    .  8BF8              mov edi,eax`  
>
> `|011901D1    .  F7DF              neg edi`  
>
> `|011901D3    .  1BFF              sbb edi,edi`  
>
> `|011901D5    .  47                 inc edi`  
>
> `|011901D6    .  F7DF              neg edi`  
>
> `|011901D8    .  8B45 D8            mov eax,dword ptr ss:\[ebp-28\]      ;  EAX=试验码第二部分`  
>
> `|011901DB    .  50                 push eax`  
>
> `|011901DC    .  68 98684600       push lavss1\_.00466898`  
>
> `|011901E1    .  FFD3              call ebx`  
>
> `|011901E3    .  F7D8              neg eax`  
>
> `|011901E5    .  1BC0              sbb eax,eax`  
>
> `|011901E7    .  40                 inc eax`  
>
> `|011901E8    .  F7D8              neg eax`  
>
> `|011901EA    .  0BF8              or edi,eax`  
>
> `|011901EC    .  8B4D DC            mov ecx,dword ptr ss:\[ebp-24\]      ;  ECX=试验码第一部分`  
>
> `|011901EF    .  51                 push ecx`  
>
> `|011901F0    .  68 98684600       push lavss1\_.00466898`  
>
> `|011901F5    .  FFD3              call ebx`  
>
> `|011901F7    .  F7D8              neg eax`  
>
> `|011901F9    .  1BC0              sbb eax,eax`  
>
> `|011901FB    .  40                 inc eax`  
>
> `|011901FC    .  F7D8              neg eax`  
>
> `|011901FE    .  0BF8              or edi,eax`  
>
> `|01190200    .  8D55 D4            lea edx,dword ptr ss:\[ebp-2C\]`  
>
> `|01190203    .  52                 push edx`  
>
> `|01190204    .  8D45 D8            lea eax,dword ptr ss:\[ebp-28\]`  
>
> `|01190207    .  50                 push eax`  
>
> `|01190208    .  8D4D DC            lea ecx,dword ptr ss:\[ebp-24\]`  
>
> `|0119020B    .  51                 push ecx`  
>
> `|0119020C    .  6A 03              push 3`  
>
> `|0119020E    .  FF15 30124000      call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaFreeStrList`  
>
> `|01190214    .  8D55 C8            lea edx,dword ptr ss:\[ebp-38\]`  
>
> `|01190217    .  52                 push edx`  
>
> `|01190218    .  8D45 CC            lea eax,dword ptr ss:\[ebp-34\]`  
>
> `|0119021B    .  50                 push eax`  
>
> `|0119021C    .  8D4D D0            lea ecx,dword ptr ss:\[ebp-30\]`  
>
> `|0119021F    .  51                 push ecx`  
>
> `|01190220    .  6A 03              push 3`  
>
> `|01190222    .  FF15 54104000      call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaFreeObjList`  
>
> `|01190228    .  83C4 20            add esp,20`  
>
> `|0119022B    .  66:85FF            test di,di`  
>
> `|0119022E    .  0F85 E0050000      jnz lavss1\_.01190814`  
>
> `|01190234    .  8B0E              mov ecx,dword ptr ds:\[esi\]`  
>
> `|01190236    .  56                 push esi`  
>
> `|01190237    .  FF91 04030000      call dword ptr ds:\[ecx+304\]`  
>
> `|0119023D    .  50                 push eax`  
>
> `|0119023E    .  8D55 D0            lea edx,dword ptr ss:\[ebp-30\]`  
>
> `|01190241    .  52                 push edx`  
>
> `|01190242    .  8B1D C4104000      mov ebx,dword ptr ds:\[<&msvbvm60.\_>;  msvbvm60.\_\_vbaObjSet`  
>
> `|01190248    .  FFD3              call ebx                    ;  <&msvbvm60.\_\_vbaObjSet>`  
>
> `|0119024A    .  8BF8              mov edi,eax`  
>
> `|0119024C    .  8B07              mov eax,dword ptr ds:\[edi\]`  
>
> `|0119024E    .  8D4D DC            lea ecx,dword ptr ss:\[ebp-24\]`  
>
> `|01190251    .  51                 push ecx`  
>
> `|01190252    .  57                 push edi`  
>
> `|01190253    .  FF90 A0000000      call dword ptr ds:\[eax+A0\]`  
>
> `|01190259    .  DBE2              fclex`  
>
> `|0119025B    .  85C0              test eax,eax`  
>
> `|0119025D    .  7D 12             jge short lavss1\_.01190271`  
>
> `|0119025F    .  68 A0000000       push 0A0`  
>
> `|01190264    .  68 D4564600       push lavss1\_.004656D4`  
>
> `|01190269    .  57                 push edi`  
>
> `|0119026A    .  50                 push eax`  
>
> `|0119026B    .  FF15 98104000      call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaHresultCheckObj`  
>
> `|01190271    >  8B45 DC            mov eax,dword ptr ss:\[ebp-24\]      ;  EAX="1882370974"(我的机器码)`  
>
> `|01190274    .  C745 DC 00000000  mov dword ptr ss:\[ebp-24\],0`  
>
> `|0119027B    .  8945 C0            mov dword ptr ss:\[ebp-40\],eax`  
>
> `|0119027E    .  C745 B8 08000000  mov dword ptr ss:\[ebp-48\],8`  
>
> `|01190285    .  6A 03              push 3`  
>
> `|01190287    .  8D55 B8            lea edx,dword ptr ss:\[ebp-48\]`  
>
> `|0119028A    .  52                 push edx`  
>
> `|0119028B    .  8D45 A8            lea eax,dword ptr ss:\[ebp-58\]`  
>
> `|0119028E    .  50                 push eax`  
>
> `|0119028F    .  FF15 8C124000      call dword ptr ds:\[<&msvbvm60.rtcL>;  msvbvm60.rtcLeftCharVar`  
>
> `|01190295    .  8D4D A8            lea ecx,dword ptr ss:\[ebp-58\]`  
>
> `|01190298    .  51                 push ecx`  
>
> `|01190299    .  FF15 38104000      call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaStrVarMove`  
>
> `|0119029F    .  8BD0              mov edx,eax                ;  EDX=机器码的前 3 位，我这里是"188"`  
>
> `|011902A1    .  8D4D E0            lea ecx,dword ptr ss:\[ebp-20\]`



```text
            |011902A4    .  8B3D 9C124000     mov edi,dword ptr ds:[<&msvbvm60.__>;  msvbvm60.__vbaStrMove  

            |011902AA    .  FFD7              call edi                               ;  <&msvbvm60.__vbaStrMove>  

            |011902AC    .  8D4D D0           lea ecx,dword ptr ss:[ebp-30]  

            |011902AF    .  FF15 D0124000     call dword ptr ds:[<&msvbvm60.__>;  msvbvm60.__vbaFreeObj  

            |011902B5    .  8D55 A8           lea edx,dword ptr ss:[ebp-58]  

            |011902B8    .  52                push edx  

            |011902B9    .  8D45 B8           lea eax,dword ptr ss:[ebp-48]  

            |011902BC    .  50                push eax  

            |011902BD    .  6A 02             push 2  

            |011902BF    .  FF15 40104000     call dword ptr ds:[<&msvbvm60.__>;  msvbvm60.__vbaFreeVarList  

            |011902C5    .  83C4 0C           add esp,0C  

            |011902C8    .  8B0E              mov ecx,dword ptr ds:[esi]  

            |011902CA    .  56                push esi  

            |011902CB    .  FF91 08030000     call dword ptr ds:[ecx+308]  

            |011902D1    .  50                push eax  

            |011902D2    .  8D55 D0           lea edx,dword ptr ss:[ebp-30]  

            |011902D5    .  52                push edx  

            |011902D6    .  FFD3             call ebx  

            |011902D8    .  8BD8             mov ebx,eax  

            |011902DA    .  8B03             mov eax,dword ptr ds:[ebx]  

            |011902DC    .  8D4D DC          lea ecx,dword ptr ss:[ebp-24]  

            |011902DF    .  51                push ecx  

            |011902E0    .  53                push ebx  

            |011902E1    .  FF90 A0000000     call dword ptr ds:[eax+A0]  

            |011902E7    .  DBE2             fclex  

            |011902E9    .  85C0             test eax,eax  

            |011902EB    .  7D 12             jge short lavss1_.011902FF  

            |011902ED    .  68 A0000000      push 0A0  

            |011902F2    .  68 D4564600      push lavss1_.004656D4  

            |011902F7    .  53                push ebx  

            |011902F8    .  50                push eax  

            |011902F9    .  FF15 98104000     call dword ptr ds:[<&msvbvm60.__>;  msvbvm60.__vbaHresultCheckObj  

            |011902FF    >  8B55 DC           mov edx,dword ptr ss:[ebp-24]       ;  EDX=试验码的第三部分  

            |01190302    .  52                push edx  

            |01190303    .  FF15 04124000     call dword ptr ds:[<&msvbvm60.__>;  EDX 字符串转换双精度型  

            |01190309    .  DD9D 10FFFFFF     fstp qword ptr ss:[ebp-F0]  

            |0119030F    .  8B45 E0           mov eax,dword ptr ss:[ebp-20]       ;  EAX="188"  

            |01190312    .  50                push eax  

            |01190313    .  FF15 04124000     call dword ptr ds:[<&msvbvm60.__>;  msvbvm60.__vbaR8Str  

            |01190319    .  DC0D D8204000     fmul qword ptr ds:[4020D8]          ;  ST0=ST0*3  

            |0119031F    .  DC05 E0204000     fadd qword ptr ds:[4020E0]          ;  STO=STO+85562317  

            |01190325    .  DFE0             fstsw ax  

            |01190327    .  A8 0D             test al,0D  

            |01190329    .  0F85 E7050000     jnz lavss1_.01190916               ;  AL!=D 就跳  

            |0119032F    .  FF15 E8104000     call dword ptr ds:[<&msvbvm60.__>;  msvbvm60.__vbaFpR8  

            |01190335    .  DC9D 10FFFFFF     fcomp qword ptr ss:[ebp-F0]        ;  STO 和试验码第三部分的数据比较  

            |0119033B    .  DFE0             fstsw ax  

            |0119033D       F6C4 40           test ah,40                         ;  如果等于就 ah=40  

            |01190340       74 07             je short lavss1_.01190349          ;  不能跳走  

            |01190342    .  B8 01000000      mov eax,1  

            |01190347    .  EB 02             jmp short lavss1_.0119034B  

            |01190349    >  33C0             xor eax,eax  

            |0119034B    >  F7D8             neg eax  

            |0119034D    .  8BD8             mov ebx,eax                        ;  EBX=EAX  

            |0119034F    .  8D4D DC          lea ecx,dword ptr ss:[ebp-24]  

            |01190352    .  FF15 D4124000     call dword ptr ds:[<&msvbvm60.__>;  msvbvm60.__vbaFreeStr  

            |01190358    .  8D4D D0          lea ecx,dword ptr ss:[ebp-30]  

            |0119035B    .  FF15 D0124000     call dword ptr ds:[<&msvbvm60.__>;  msvbvm60.__vbaFreeObj  

            |01190361    .  66:85DB          test bx,bx                         ;  关键比较  

            |01190364    .  B9 04000280      mov ecx,80020004                   ;  ECX=80020004  

            |01190369    .  894D 90          mov dword ptr ss:[ebp-70],ecx  

            |0119036C    .  B8 0A000000      mov eax,0A                         ;  EAX=A  

            |01190371    .  8945 88          mov dword ptr ss:[ebp-78],eax  

            |01190374    .  894D A0          mov dword ptr ss:[ebp-60],ecx  

            |01190377    .  8945 98          mov dword ptr ss:[ebp-68],eax  

            |0119037A       0F84 2C040000    je lavss1_.011907AC                ;  跳向死亡，关键跳  

            |01190380    .  C785 70FFFFFF 68E>mov dword ptr ss:[ebp-90],lavss1_.>  

            |0119038A    .  C785 68FFFFFF 080>mov dword ptr ss:[ebp-98],8  

            |01190394    .  8D95 68FFFFFF     lea edx,dword ptr ss:[ebp-98]  

            |0119039A    .  8D4D A8          lea ecx,dword ptr ss:[ebp-58]  

            |0119039D    .  8B1D 70124000     mov ebx,dword ptr ds:[<&msvbvm60._>;  msvbvm60.__vbaVarDup  

            |011903A3    .  FFD3             call ebx                           ;  <&msvbvm60.__vbaVarDup>  

            |011903A5    .  C745 80 18E64600  mov dword ptr ss:[ebp-80],lavss1_.>  

            |011903AC    .  C785 78FFFFFF 080>mov dword ptr ss:[ebp-88],8  

            |011903B6    .  8D95 78FFFFFF     lea edx,dword ptr ss:[ebp-88]  

            |011903BC    .  8D4D B8          lea ecx,dword ptr ss:[ebp-48]  

            |011903BF    .  FFD3             call ebx  

            |011903C1    .  8D4D 88          lea ecx,dword ptr ss:[ebp-78]  

            |011903C4    .  51                push ecx  

            |011903C5    .  8D55 98          lea edx,dword ptr ss:[ebp-68]  

            |011903C8    .  52                push edx  

            |011903C9    .  8D45 A8          lea eax,dword ptr ss:[ebp-58]  

            |011903CC    .  50                push eax  

            |011903CD    .  6A 40             push 40  

            |011903CF    .  8D4D B8          lea ecx,dword ptr ss:[ebp-48]  

            |011903D2    .  51                push ecx  

            |011903D3    .  FF15 C8104000     call dword ptr ds:[<&msvbvm60.rtcM>;  注册成功。  

            |011903D9    .  8D55 88          lea edx,dword ptr ss:[ebp-78]  

            |011903DC    .  52                push edx  

            |011903DD    .  8D45 98          lea eax,dword ptr ss:[ebp-68]  

            |011903E0    .  50                push eax  

            |011903E1    .  8D4D A8          lea ecx,dword ptr ss:[ebp-58]  

            |011903E4    .  51                push ecx  

            |011903E5    .  8D55 B8          lea edx,dword ptr ss:[ebp-48]  

            |011903E8    .  52                push edx  

            |011903E9    .  6A 04             push 4  

            |011903EB    .  FF15 40104000     call dword ptr ds:[<&msvbvm60.__>;  msvbvm60.__vbaFreeVarList  
```



```
011903F1    .  83C4 14           add esp,14  
011903F4    .  68 94E64600       push lavss1\_.0046E694              ;  UNICODE "4817241"  
011903F9    .  68 78E64600       push lavss1\_.0046E678              ;  UNICODE "visionzero"  
011903FE    .  68 B4504600       push lavss1\_.004650B4              ;  UNICODE "SOFTWARE\\LinDirMicro\\LDM Anti Trojan"  
01190403    .  68 3C5D4600       push lavss1\_.00465D3C              ;  UNICODE "\\options"  
01190408    .  8B1D 80104000     mov ebx,dword ptr ds:\[<&msvbvm60.\_>;  msvbvm60.\_\_vbaStrCat  
0119040E    .  FFD3              call ebx                            ;  <&msvbvm60.\_\_vbaStrCat>  
01190410    .  8BD0             mov edx,eax  
01190412    .  8D4D DC          lea ecx,dword ptr ss:\[ebp-24\]  
01190415    .  FFD7             call edi  
01190417    .  50               push eax  
01190418    .  68 02000080       push 80000002  
0119041D    .  E8 6EEEFEFF       call lavss1\_.0117F290  
01190422    .  8D4D DC          lea ecx,dword ptr ss:\[ebp-24\]  
01190425    .  FF15 D4124000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaFreeStr  
0119042B    .  8B06             mov eax,dword ptr ds:\[esi\]  
0119042D    .  56               push esi  
0119042E    .  FF90 0C030000     call dword ptr ds:\[eax+30C\]  
01190434    .  50               push eax  
01190435    .  8D4D D0          lea ecx,dword ptr ss:\[ebp-30\]  
01190438    .  51               push ecx  
01190439    .  FF15 C4104000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaObjSet  
0119043F    .  8985 44FFFFFF     mov dword ptr ss:\[ebp-BC\],eax  
01190445    .  8B10             mov edx,dword ptr ds:\[eax\]  
01190447    .  8D4D D8          lea ecx,dword ptr ss:\[ebp-28\]  
0119044A    .  51               push ecx  
0119044B    .  50               push eax  
0119044C    .  FF92 A0000000     call dword ptr ds:\[edx+A0\]  
01190452    .  DBE2             fclex  
01190454    .  85C0             test eax,eax  
01190456    .  7D 18            jge short lavss1\_.01190470  
01190458    .  68 A0000000       push 0A0  
0119045D    .  68 D4564600       push lavss1\_.004656D4  
01190462    .  8B95 44FFFFFF     mov edx,dword ptr ss:\[ebp-BC\]  
01190468    .  52               push edx  
01190469    .  50               push eax  
0119046A    .  FF15 98104000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaHresultCheckObj  
01190470    >  8B45 D8          mov eax,dword ptr ss:\[ebp-28\]  
01190473    .  50               push eax  
01190474    .  68 20A24600       push lavss1\_.0046A220              ;  UNICODE "visionone"  
01190479    .  68 B4504600       push lavss1\_.004650B4              ;  UNICODE "SOFTWARE\\LinDirMicro\\LDM Anti Trojan"  
0119047E    .  68 3C5D4600       push lavss1\_.00465D3C              ;  UNICODE "\\options"  
01190483    .  FFD3             call ebx  
01190485    .  8BD0             mov edx,eax  
01190487    .  8D4D DC          lea ecx,dword ptr ss:\[ebp-24\]  
0119048A    .  FFD7             call edi  
0119048C    .  50               push eax  
0119048D    .  68 02000080       push 80000002  
01190492    .  E8 F9EDFEFF       call lavss1\_.0117F290  
01190497    .  8D4D D8          lea ecx,dword ptr ss:\[ebp-28\]  
0119049A    .  51               push ecx  
0119049B    .  8D55 DC          lea edx,dword ptr ss:\[ebp-24\]  
0119049E    .  52               push edx  
0119049F    .  6A 02             push 2  
011904A1    .  FF15 30124000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaFreeStrList  
011904A7    .  83C4 0C          add esp,0C  
011904AA    .  8D4D D0          lea ecx,dword ptr ss:\[ebp-30\]  
011904AD    .  FF15 D0124000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaFreeObj  
011904B3    .  8B06             mov eax,dword ptr ds:\[esi\]  
011904B5    .  56               push esi  
011904B6    .  FF90 FC020000     call dword ptr ds:\[eax+2FC\]  
011904BC    .  50               push eax  
011904BD    .  8D4D D0          lea ecx,dword ptr ss:\[ebp-30\]  
011904C0    .  51               push ecx  
011904C1    .  FF15 C4104000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaObjSet  
011904C7    .  8985 44FFFFFF     mov dword ptr ss:\[ebp-BC\],eax  
011904CD    .  8B10             mov edx,dword ptr ds:\[eax\]  
011904CF    .  8D4D D8          lea ecx,dword ptr ss:\[ebp-28\]  
011904D2    .  51               push ecx  
011904D3    .  50               push eax  
011904D4    .  FF92 A0000000     call dword ptr ds:\[edx+A0\]  
011904DA    .  DBE2             fclex  
011904DC    .  85C0             test eax,eax  
011904DE    .  7D 18            jge short lavss1\_.011904F8  
011904E0    .  68 A0000000       push 0A0  
011904E5    .  68 D4564600       push lavss1\_.004656D4  
011904EA    .  8B95 44FFFFFF     mov edx,dword ptr ss:\[ebp-BC\]  
011904F0    .  52               push edx  
011904F1    .  50               push eax  
011904F2    .  FF15 98104000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaHresultCheckObj  
011904F8    >  8B45 D8          mov eax,dword ptr ss:\[ebp-28\]  
011904FB    .  50               push eax  
011904FC    .  68 50A24600       push lavss1\_.0046A250              ;  UNICODE "visiontwo"  
01190501    .  68 B4504600       push lavss1\_.004650B4              ;  UNICODE "SOFTWARE\\LinDirMicro\\LDM Anti Trojan"  
01190506    .  68 3C5D4600       push lavss1\_.00465D3C              ;  UNICODE "\\options"  
0119050B    .  FFD3             call ebx  
0119050D    .  8BD0             mov edx,eax  
0119050F    .  8D4D DC          lea ecx,dword ptr ss:\[ebp-24\]  
01190512    .  FFD7             call edi  
01190514    .  50               push eax  
01190515    .  68 02000080       push 80000002  
0119051A    .  E8 71EDFEFF       call lavss1\_.0117F290  
0119051F    .  8D4D D8          lea ecx,dword ptr ss:\[ebp-28\]  
01190522    .  51               push ecx  
01190523    .  8D55 DC          lea edx,dword ptr ss:\[ebp-24\]  
01190526    .  52               push edx  
01190527    .  6A 02             push 2  
01190529    .  FF15 30124000     call dword ptr ds:\[<&msvbvm60.\_\_vb>;  msvbvm60.\_\_vbaFreeStrList  
0119052F    .  83C4 0C          add esp,0C
```



```
    01190532    .  8D4D D0        lea ecx,dword ptr ss:[ebp-30]  
    01190535    .  FF15 D0124000    call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeObj  
    0119053B    .  8B06           mov eax,dword ptr ds:[esi]  
    0119053D    .  56            push esi  
    0119053E    .  FF90 08030000    call dword ptr ds:[eax+308]  
    01190544    .  50            push eax  
    01190545    .  8D4D D0        lea ecx,dword ptr ss:[ebp-30]  
    01190548    .  51            push ecx  
    01190549    .  FF15 C4104000    call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaObjSet  
    0119054F    .  8985 44FFFFFF mov dword ptr ss:[ebp-BC],eax  
    01190555    .  8B10           mov edx,dword ptr ds:[eax]  
    01190557    .  8D4D D8        lea ecx,dword ptr ss:[ebp-28]  
    0119055A    .  51            push ecx  
    0119055B    .  50            push eax  
    0119055C    .  FF92 A0000000    call dword ptr ds:[edx+A0]  
    01190562    .  DBE2           fclex  
    01190564    .  85C0           test eax,eax  
    01190566    .  7D 18          jge short lavss1_.01190580  
    01190568    .  68 A0000000      push 0A0  
    0119056D    .  68 D4564600      push lavss1_.004656D4  
    01190572    .  8B95 44FFFFFF mov edx,dword ptr ss:[ebp-BC]  
    01190578    .  52            push edx  
    01190579    .  50            push eax  
    0119057A    .  FF15 98104000    call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaHresultCheckObj  
    01190580    >  8B45 D8        mov eax,dword ptr ss:[ebp-28]  
    01190583    .  50            push eax  
    01190584    .  68 04A24600      push lavss1_.0046A204            ;  UNICODE "visionthree"  
    01190589    .  68 B4504600      push lavss1_.004650B4            ;  UNICODE "SOFTWARE\\LinDirMicro\\LDM Anti Trojan"  
    0119058E    .  68 3C5D4600      push lavss1_.00465D3C            ;  UNICODE "\\options"  
    01190593    .  FFD3           call ebx  
    01190595    .  8BD0           mov edx,eax  
    01190597    .  8D4D DC        lea ecx,dword ptr ss:[ebp-24]  
    0119059A    .  FFD7           call edi  
    0119059C    .  50            push eax  
    0119059D    .  68 02000080      push 80000002  
    011905A2    .  E8 E9ECFEFF      call lavss1_.0117F290  
    011905A7    .  8D4D D8        lea ecx,dword ptr ss:[ebp-28]  
    011905AA    .  51            push ecx  
    011905AB    .  8D55 DC        lea edx,dword ptr ss:[ebp-24]  
    011905AE    .  52            push edx  
    011905AF    .  6A 02           push 2  
    011905B1    .  FF15 30124000    call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeStrList  
    011905B7    .  83C4 0C        add esp,0C  
    011905BA    .  8D4D D0        lea ecx,dword ptr ss:[ebp-30]  
    011905BD    .  FF15 D0124000    call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeObj  
    011905C3    .  68 0C5F4600      push lavss1_.00465F0C  
    011905C8    .  68 A0C84600      push lavss1_.0046C8A0            ;  UNICODE "vican"  
    011905CD    .  68 B4504600      push lavss1_.004650B4            ;  UNICODE "SOFTWARE\\LinDirMicro\\LDM Anti Trojan"  
    011905D2    .  68 3C5D4600      push lavss1_.00465D3C            ;  UNICODE "\\options"  
    011905D7    .  FFD3           call ebx  
    011905D9    .  8BD0           mov edx,eax  
    011905DB    .  8D4D DC        lea ecx,dword ptr ss:[ebp-24]  
    011905DE    .  FFD7           call edi  
    011905E0    .  50            push eax  
    011905E1    .  68 02000080      push 80000002  
    011905E6    .  E8 A5ECFEFF      call lavss1_.0117F290  
    011905EB    .  8D4D DC        lea ecx,dword ptr ss:[ebp-24]  
    011905EE    .  FF15 D4124000    call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeStr  
    011905F4    .  68 BC8A4600      push lavss1_.00468ABC            ;  UNICODE "iuyhjgtbcmakf76521jkcn"  
    011905F9    .  68 CC884600      push lavss1_.004688CC            ;  UNICODE "showa"  
    011905FE    .  68 B4504600      push lavss1_.004650B4            ;  UNICODE "SOFTWARE\\LinDirMicro\\LDM Anti Trojan"  
    01190603    .  68 A48A4600      push lavss1_.00468AA4            ;  UNICODE "\\sandk"  
    01190608    .  FFD3           call ebx  
    0119060A    .  8BD0           mov edx,eax  
    0119060C    .  8D4D DC        lea ecx,dword ptr ss:[ebp-24]  
    0119060F    .  FFD7           call edi  
    01190611    .  50            push eax  
    01190612    .  68 02000080      push 80000002  
    01190617    .  E8 74ECFEFF      call lavss1_.0117F290  
    0119061C    .  8D4D DC        lea ecx,dword ptr ss:[ebp-24]  
    0119061F    .  FF15 D4124000    call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeStr  
    01190625    .  68 BCE64600      push lavss1_.0046E6BC            ;  UNICODE "nirvashtypezerospec2"  
    0119062A    .  68 A8E64600      push lavss1_.0046E6A8            ;  UNICODE "lookreg"  
    0119062F    .  68 B4504600      push lavss1_.004650B4            ;  UNICODE "SOFTWARE\\LinDirMicro\\LDM Anti Trojan"  
    01190634    .  68 3C5D4600      push lavss1_.00465D3C            ;  UNICODE "\\options"  
    01190639    .  FFD3           call ebx  
    0119063B    .  8BD0           mov edx,eax  
    0119063D    .  8D4D DC        lea ecx,dword ptr ss:[ebp-24]  
    01190640    .  FFD7           call edi  
    01190642    .  50            push eax  
    01190643    .  68 02000080      push 80000002  
    01190648    .  E8 43ECFEFF      call lavss1_.0117F290  
    0119064D    .  8D4D DC        lea ecx,dword ptr ss:[ebp-24]  
    01190650    .  FF15 D4124000    call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeStr  
    01190656    .  68 68CD4600      push lavss1_.0046CD68            ;  UNICODE "dopod596FORWM5XTOO2"  
    0119065B    .  68 34CF4600      push lavss1_.0046CF34            ;  UNICODE "dopod"  
    01190660    .  68 B4504600      push lavss1_.004650B4            ;  UNICODE "SOFTWARE\\LinDirMicro\\LDM Anti Trojan"  
    01190665    .  68 3C5D4600      push lavss1_.00465D3C            ;  UNICODE "\\options"  
    0119066A    .  FFD3           call ebx  
    0119066C    .  8BD0           mov edx,eax  
    0119066E    .  8D4D DC        lea ecx,dword ptr ss:[ebp-24]  
    01190671    .  FFD7           call edi  
    01190673    .  50            push eax  
    01190674    .  68 02000080      push 80000002  
    01190679    .  E8 12ECFEFF      call lavss1_.0117F290  
```



```asm
0119067E    .  8D4D DC        lea ecx,dword ptr ss:[ebp-24]  
01190681    .  FF15 D4124000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeStr  
01190687    .  68 FCE64600    push lavss1_.0046E6FC              ;  UNICODE "INZCKINGMAX"  
0119068C    .  68 ECE64600    push lavss1_.0046E6EC              ;  UNICODE "finza"  
01190691    .  68 B4504600    push lavss1_.004650B4              ;  UNICODE "SOFTWARE\\LinDirMicro\\LDM Anti Trojan"  
01190696    .  68 3C5D4600    push lavss1_.00465D3C              ;  UNICODE "\\options"  
0119069B    .  FFD3           call ebx  
0119069D    .  8BD0           mov edx,eax  
0119069F    .  8D4D DC        lea ecx,dword ptr ss:[ebp-24]  
011906A2    .  FFD7           call edi  
011906A4    .  50            push eax  
011906A5    .  68 02000080    push 80000002  
011906AA    .  E8 E1EBFEFF    call lavss1_.0117F290  
011906AF    .  8D4D DC        lea ecx,dword ptr ss:[ebp-24]  
011906B2    .  FF15 D4124000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeStr  
011906B8    .  A1 78431901    mov eax,dword ptr ds:[1194378]  
011906BD    .  85C0           test eax,eax  
011906BF    .  75 10         jnz short lavss1_.011906D1  
011906C1    .  68 78431901    push lavss1_.01194378  
011906C6    .  68 7C564600    push lavss1_.0046567C  
011906CB    .  FF15 0C124000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaNew2  
011906D1    >  8B3D 78431901  mov edi,dword ptr ds:[1194378]  
011906D7    .  8B1F           mov ebx,dword ptr ds:[edi]  
011906D9    .  56            push esi  
011906DA    .  8D45 D0        lea eax,dword ptr ss:[ebp-30]  
011906DD    .  50            push eax  
011906DE    .  FF15 D4104000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaObjSetAddref  
011906E4    .  50            push eax  
011906E5    .  57            push edi  
011906E6    .  FF53 10        call dword ptr ds:[ebx+10]  
011906E9    .  DBE2           fclex  
011906EB    .  85C0           test eax,eax  
011906ED    .  7D 0F         jge short lavss1_.011906FE  
011906EF    .  6A 10         push 10  
011906F1    .  68 6C564600    push lavss1_.0046566C  
011906F6    .  57            push edi  
011906F7    .  50            push eax  
011906F8    .  FF15 98104000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaHresultCheckObj  
011906FE    >  8D4D D0        lea ecx,dword ptr ss:[ebp-30]  
01190701    .  FF15 D0124000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeObj  
01190707    .  A1 10301901    mov eax,dword ptr ds:[1193010]  
0119070C    .  85C0           test eax,eax  
0119070E    .  75 10         jnz short lavss1_.01190720  
01190710    .  68 10301901    push lavss1_.01193010  
01190715    .  68 58CC4500    push lavss1_.0045CC58  
0119071A    .  FF15 0C124000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaNew2  
01190720    >  8B35 10301901  mov esi,dword ptr ds:[1193010]  
01190726    .  B8 04000280    mov eax,80020004  
0119072B    .  8985 70FFFFFF  mov dword ptr ss:[ebp-90],eax  
01190731    .  B9 0A000000    mov ecx,0A  
01190736    .  898D 68FFFFFF  mov dword ptr ss:[ebp-98],ecx  
0119073C    .  8BD0           mov edx,eax  
0119073E    .  8955 80        mov dword ptr ss:[ebp-80],edx  
01190741    .  898D 78FFFFFF  mov dword ptr ss:[ebp-88],ecx  
01190747    .  8B3E           mov edi,dword ptr ds:[esi]  
01190749    .  83EC 10        sub esp,10  
0119074C    .  8BDC           mov ebx,esp  
0119074E    .  890B           mov dword ptr ds:[ebx],ecx  
01190750    .  8B8D 6CFFFFFF  mov ecx,dword ptr ss:[ebp-94]  
01190756    .  894B 04        mov dword ptr ds:[ebx+4],ecx  
01190759    .  8943 08        mov dword ptr ds:[ebx+8],eax  
0119075C    .  8B85 74FFFFFF  mov eax,dword ptr ss:[ebp-8C]  
01190762    .  8943 0C        mov dword ptr ds:[ebx+C],eax  
01190765    .  83EC 10        sub esp,10  
01190768    .  8BCC           mov ecx,esp  
0119076A    .  8B85 78FFFFFF  mov eax,dword ptr ss:[ebp-88]  
01190770    .  8901           mov dword ptr ds:[ecx],eax  
01190772    .  8B85 7CFFFFFF  mov eax,dword ptr ss:[ebp-84]  
01190778    .  8941 04        mov dword ptr ds:[ecx+4],eax  
0119077B    .  8951 08        mov dword ptr ds:[ecx+8],edx  
0119077E    .  8B55 84        mov edx,dword ptr ss:[ebp-7C]  
01190781    .  8951 0C        mov dword ptr ds:[ecx+C],edx  
01190784    .  56            push esi  
01190785    .  FF97 B0020000  call dword ptr ds:[edi+2B0]  
0119078B    .  DBE2           fclex  
0119078D    .  85C0           test eax,eax  
0119078F    .  0F8D 06010000 jge lavss1_.0119089B  
01190795    .  68 B0020000    push 2B0  
0119079A    .  68 6C514600    push lavss1_.0046516C  
0119079F    .  56            push esi  
011907A0    .  50            push eax  
011907A1    .  FF15 98104000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaHresultCheckObj  
011907A7    .  E9 EF000000    jmp lavss1_.0119089B  
011907AC    >  C785 70FFFFFF 08E>mov dword ptr ss:[ebp-90],lavss1_.>  
011907B6    .  BF 08000000    mov edi,8  
011907BB    .  89BD 68FFFFFF  mov dword ptr ss:[ebp-98],edi  
011907C1    .  8D95 68FFFFFF  lea edx,dword ptr ss:[ebp-98]  
011907C7    .  8D4D A8        lea ecx,dword ptr ss:[ebp-58]  
011907CA    .  8B35 70124000  mov esi,dword ptr ds:[<&msvbvm60._>;  msvbvm60.__vbaVarDup  
011907D0    .  FFD6           call esi                            ;  <&msvbvm60.__vbaVarDup>  
011907D2    .  C745 80 E4E54600 mov dword ptr ss:[ebp-80],lavss1_.>  
011907D9    .  89BD 78FFFFFF  mov dword ptr ss:[ebp-88],edi  
011907DF    .  8D95 78FFFFFF  lea edx,dword ptr ss:[ebp-88]  
011907E5    .  8D4D B8        lea ecx,dword ptr ss:[ebp-48]  
011907E8    .  FFD6           call esi  
011907EA    .  8D45 88        lea eax,dword ptr ss:[ebp-78]  
011907ED    .  50            push eax  
011907EE    .  8D4D 98        lea ecx,dword ptr ss:[ebp-68]  
011907F1    .  51            push ecx  
```



```asm
011907F2    .  8D55 A8           lea edx,dword ptr ss:[ebp-58]
011907F5    .  52              push edx
011907F6    .  6A 30          push 30
011907F8    .  8D45 B8        lea eax,dword ptr ss:[ebp-48]
011907FB    .  50              push eax
011907FC    .  FF15 C8104000  call dword ptr ds:[<&msvbvm60.rtcM];  注册失败
01190802    .  8D4D 88        lea ecx,dword ptr ss:[ebp-78]
01190805    .  51              push ecx
01190806    .  8D55 98        lea edx,dword ptr ss:[ebp-68]
01190809    .  52              push edx
0119080A    .  8D45 A8        lea eax,dword ptr ss:[ebp-58]
0119080D    .  50              push eax
0119080E    .  8D4D B8        lea ecx,dword ptr ss:[ebp-48]
01190811    .  51              push ecx
01190812    .  EB 7C          jmp short lavss1_.01190890
01190814    >  B9 04000280    mov ecx,80020004
01190819    .  894D 90        mov dword ptr ss:[ebp-70],ecx
0119081C    .  B8 0A000000    mov eax,0A
01190821    .  8945 88        mov dword ptr ss:[ebp-78],eax
01190824    .  894D A0        mov dword ptr ss:[ebp-60],ecx
01190827    .  8945 98        mov dword ptr ss:[ebp-68],eax
0119082A    .  C785 70FFFFFF 08E>mov dword ptr ss:[ebp-90],lavss1_.>
01190834    .  BF 08000000    mov edi,8
01190839    .  89BD 68FFFFFF  mov dword ptr ss:[ebp-98],edi
0119083F    .  8D95 68FFFFFF  lea edx,dword ptr ss:[ebp-98]
01190845    .  8D4D A8        lea ecx,dword ptr ss:[ebp-58]
01190848    .  8B35 70124000  mov esi,dword ptr ds:[<&msvbvm60._>;  msvbvm60.__vbaVarDup
0119084E    .  FFD6          call esi                                ;  <&msvbvm60.__vbaVarDup>
01190850    .  C745 80 E4E54600  mov dword ptr ss:[ebp-80],lavss1_.>
01190857    .  89BD 78FFFFFF  mov dword ptr ss:[ebp-88],edi
0119085D    .  8D95 78FFFFFF  lea edx,dword ptr ss:[ebp-88]
01190863    .  8D4D B8        lea ecx,dword ptr ss:[ebp-48]
01190866    .  FFD6          call esi
01190868    .  8D55 88        lea edx,dword ptr ss:[ebp-78]
0119086B    .  52              push edx
0119086C    .  8D45 98        lea eax,dword ptr ss:[ebp-68]
0119086F    .  50              push eax
01190870    .  8D4D A8        lea ecx,dword ptr ss:[ebp-58]
01190873    .  51              push ecx
01190874    .  6A 30          push 30
01190876    .  8D55 B8        lea edx,dword ptr ss:[ebp-48]
01190879    .  52              push edx
0119087A    .  FF15 C8104000  call dword ptr ds:[<&msvbvm60.rtcM>;  msvbvm60.rtcMsgBox
01190880    .  8D45 88        lea eax,dword ptr ss:[ebp-78]
01190883    .  50              push eax
01190884    .  8D4D 98        lea ecx,dword ptr ss:[ebp-68]
01190887    .  51              push ecx
01190888    .  8D55 A8        lea edx,dword ptr ss:[ebp-58]
0119088B    .  52              push edx
0119088C    .  8D45 B8        lea eax,dword ptr ss:[ebp-48]
0119088F    .  50              push eax
01190890    >  6A 04          push 4
01190892    .  FF15 40104000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeVarList
01190898    .  83C4 14        add esp,14
0119089B    >  FF15 BC104000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaExitProc
011908A1    .  9B              wait
011908A2    .  68 F7081901    push lavss1_.011908F7
011908A7    .  EB 44          jmp short lavss1_.011908ED
011908A9    .  8D4D D4        lea ecx,dword ptr ss:[ebp-2C]
011908AC    .  51              push ecx
011908AD    .  8D55 D8        lea edx,dword ptr ss:[ebp-28]
011908B0    .  52              push edx
011908B1    .  8D45 DC        lea eax,dword ptr ss:[ebp-24]
011908B4    .  50              push eax
011908B5    .  6A 03          push 3
011908B7    .  FF15 30124000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeStrList
011908BD    .  8D4D C8        lea ecx,dword ptr ss:[ebp-38]
011908C0    .  51              push ecx
011908C1    .  8D55 CC        lea edx,dword ptr ss:[ebp-34]
011908C4    .  52              push edx
011908C5    .  8D45 D0        lea eax,dword ptr ss:[ebp-30]
011908C8    .  50              push eax
011908C9    .  6A 03          push 3
011908CB    .  FF15 54104000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeObjList
011908D1    .  8D4D 88        lea ecx,dword ptr ss:[ebp-78]
011908D4    .  51              push ecx
011908D5    .  8D55 98        lea edx,dword ptr ss:[ebp-68]
011908D8    .  52              push edx
011908D9    .  8D45 A8        lea eax,dword ptr ss:[ebp-58]
011908DC    .  50              push eax
011908DD    .  8D4D B8        lea ecx,dword ptr ss:[ebp-48]
011908E0    .  51              push ecx
011908E1    .  6A 04          push 4
011908E3    .  FF15 40104000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeVarList
011908E9    .  83C4 34        add esp,34
011908EC    .  C3              retn
011908ED    >  8D4D E0        lea ecx,dword ptr ss:[ebp-20]
011908F0    .  FF15 D4124000  call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFreeStr
011908F6    .  C3              retn
011908F7    .  8B45 08        mov eax,dword ptr ss:[ebp+8]
011908FA    .  8B10          mov edx,dword ptr ds:[eax]
011908FC    .  50              push eax
011908FD    .  FF52 08        call dword ptr ds:[edx+8]
01190900    .  8B45 F4        mov eax,dword ptr ss:[ebp-C]
01190903    .  8B4D E4        mov ecx,dword ptr ss:[ebp-1C]
01190906    .  64:890D 00000000  mov dword ptr fs:[0],ecx
0119090D    .  5F              pop edi
0119090E    .  5E              pop esi
0119090F    .  5B              pop ebx
01190910    .  8BE5          mov esp,ebp
01190912    .  5D              pop ebp
01190913    .  C2 0400        retn 4
01190916    >^ E9 212327FF    jmp <jmp.&msvbvm60.__vbaFPExceptio>
0119091B      90              nop
0119091C      90              nop
```



# 汇编代码分析

其他不用看，主要是这里。

```
|011902FF  >  8B55 DC          mov edx,dword ptr ss:[ebp-24]    ;  EDX=试验码的第三部分
|01190302  .  52              push edx
|01190303  .  FF15 04124000   call dword ptr ds:[<&msvbvm60.__vb>;  EDX 字符串转换双精度型
|01190309  .  DD9D 10FFFFFF   fstp qword ptr ss:[ebp-F0]
|0119030F  .  8B45 E0         mov eax,dword ptr ss:[ebp-20]    ;  EAX=机器码的前 3 位
|01190312  .  50              push eax
|01190313  .  FF15 04124000   call dword ptr ds:[<&msvbvm60.__vb>;  ST0=机器码前 3 位的数字
|01190319  .  DC0D D8204000   fmul qword ptr ds:[4020D8]      ;  ST0=ST0*3
|0119031F  .  DC05 E0204000   fadd qword ptr ds:[4020E0]      ;  STO=STO+85562317
|01190325  .  DFE0           fstsw ax
|01190327  .  A8 0D          test al,0D
|01190329  .  0F85 E7050000 jnz lavss1_.01190916           ;  AL!=D 就跳
|0119032F  .  FF15 E8104000 call dword ptr ds:[<&msvbvm60.__vb>;  msvbvm60.__vbaFpR8
|01190335  .  DC9D 10FFFFFF fcomp qword ptr ss:[ebp-F0]     ;  STO 和试验码第三部分的数据比较
|0119033B  .  DFE0           fstsw ax
|0119033D     F6C4 40        test ah,40                     ;  如果等于就 ah=40
|01190340     74 07        je short lavss1_.01190349       ;  不能跳走
```

## 4：算法总结

注册码和机器码都是数字，可以使小数的。

`注册码第三部分` = `机器码前3位` * 3 + 85562317，第一部分和第二部分随便，因为他们没用到。

这是我遇到的最简单的算法吧，前两天都遇到 `bt` 的算法，这次运气比较好。

嘿嘿，注册成功了。就这样。

软件做的的确是很漂亮，功能也不错，可是注册算法太简单了，可惜。

--------------------------------------------------------------------------------

# 【经验总结】

今天心情不错，等下就快天亮了，电脑快没电了！

--------------------------------------------------------------------------------

# 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 03 月 19 日 5:13:25

`4dd466d0e1e0cc123afbe504da2462ca`
