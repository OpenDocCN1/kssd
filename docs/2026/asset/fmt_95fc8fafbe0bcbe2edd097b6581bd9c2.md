
```
005D26D0 >/$  55            push ebp                                     ;  regkeyValidate
005D26D1  |.  8BEC          mov ebp,esp
005D26D3  |.  B9 22000000   mov ecx,0x22
005D26D8  |>  6A 00         /push 0x0
005D26DA  |.  6A 00         |push 0x0
005D26DC  |.  49            |dec ecx
005D26DD  |.^ 75 F9         \jnz short Protecte.005D26D8
005D26DF  |.  51            push ecx
005D26E0  |.  53            push ebx
005D26E1  |.  8955 FC       mov [local.1],edx
005D26E4  |.  8B45 FC       mov eax,[local.1]
005D26E7  |.  E8 E83FE3FF   call <Protecte.j_System_@LStrAddRef>
005D26EC  |.  33C0          xor eax,eax
005D26EE  |.  55            push ebp
005D26EF  |.  68 112F5D00   push Protecte.005D2F11
005D26F4  |.  64:FF30       push dword ptr fs:[eax]
005D26F7  |.  64:8920       mov dword ptr fs:[eax],esp
005D26FA  |.  33DB          xor ebx,ebx
005D26FC  |.  8D55 E8       lea edx,[local.6]
005D26FF  |.  8B45 FC       mov eax,[local.1]
005D2702  |.  E8 BDA4E8FF   call <Protecte._Unit13_UpperCase> //将注册码转成大写
005D2707  |.  8B55 E8       mov edx,[local.6]
005D270A  |.  8D45 FC       lea eax,[local.1]
005D270D  |.  E8 2E40E3FF   call <Protecte.@UStrLAsg> //类似这些函数,是字符串赋值产生的(字符串都是对象)
005D2712  |.  0FB605 242F5D>movzx eax,byte ptr ds:[0x5D2F24]
005D2719  |.  50            push eax
005D271A  |.  8D45 E4       lea eax,[local.7]
005D271D  |.  50            push eax
005D271E  |.  B9 342F5D00   mov ecx,<Protecte.char_0>
005D2723  |.  BA 442F5D00   mov edx,<Protecte.char_O>//我这里已经做了注释,很重要的参数
005D2728  |.  8B45 FC       mov eax,[local.1]
005D272B  |.  E8 0C3FE9FF   call <Protecte.StringReplace> //看参数知道是将O换成0.
005D2730  |.  8B55 E4       mov edx,[local.7]
005D2733  |.  8D45 FC       lea eax,[local.1]
005D2736  |.  E8 0540E3FF   call <Protecte.@UStrLAsg>
005D273B  |.  0FB605 242F5D>movzx eax,byte ptr ds:[0x5D2F24]
005D2742  |.  50            push eax
005D2743  |.  8D45 E0       lea eax,[local.8]
005D2746  |.  50            push eax
005D2747  |.  B9 542F5D00   mov ecx,<Protecte.char_1>
005D274C  |.  BA 642F5D00   mov edx,<Protecte.char_L>
005D2751  |.  8B45 FC       mov eax,[local.1]
005D2754  |.  E8 E33EE9FF   call <Protecte.StringReplace>//将L换成1
005D2759  |.  8B55 E0       mov edx,[local.8]
005D275C  |.  8D45 FC       lea eax,[local.1]
005D275F  |.  E8 DC3FE3FF   call <Protecte.@UStrLAsg>
005D2764  |.  0FB605 242F5D>movzx eax,byte ptr ds:[0x5D2F24]
005D276B  |.  50            push eax
005D276C  |.  8D45 DC       lea eax,[local.9]
005D276F  |.  50            push eax
005D2770  |.  33C9          xor ecx,ecx
005D2772  |.  BA 742F5D00   mov edx,<Protecte.char_space>
005D2777  |.  8B45 FC       mov eax,[local.1]
005D277A  |.  E8 BD3EE9FF   call <Protecte.StringReplace>//清除空格
005D277F  |.  8B55 DC       mov edx,[local.9]
005D2782  |.  8D45 FC       lea eax,[local.1]
005D2785  |.  E8 B63FE3FF   call <Protecte.@UStrLAsg>
005D278A  |.  0FB605 242F5D>movzx eax,byte ptr ds:[0x5D2F24]
005D2791  |.  50            push eax
005D2792  |.  8D45 D8       lea eax,[local.10]
005D2795  |.  50            push eax
005D2796  |.  33C9          xor ecx,ecx
005D2798  |.  BA 842F5D00   mov edx,<Protecte.wchar_0>
005D279D  |.  8B45 FC       mov eax,[local.1]
005D27A0  |.  E8 973EE9FF   call <Protecte.StringReplace> //清除宽字符0....没用
005D27A5  |.  8B55 D8       mov edx,[local.10]
005D27A8  |.  8D45 FC       lea eax,[local.1]
005D27AB  |.  E8 903FE3FF   call <Protecte.@UStrLAsg>
005D27B0  |.  8B45 FC       mov eax,[local.1]
005D27B3  |.  85C0          test eax,eax
005D27B5  |.  74 16         je short Protecte.005D27CD
005D27B7  |.  8BD0          mov edx,eax
005D27B9  |.  83EA 0A       sub edx,0xA
005D27BC  |.  66:833A 02    cmp word ptr ds:[edx],0x2
005D27C0  |.  74 0B         je short Protecte.005D27CD
005D27C2  |.  8D45 FC       lea eax,[local.1]
005D27C5  |.  8B55 FC       mov edx,[local.1]
005D27C8  |.  E8 C331E3FF   call <Protecte.System_@InternalUStrFromLStr>
005D27CD  |>  85C0          test eax,eax
005D27CF  |.  74 05         je short Protecte.005D27D6
005D27D1  |.  83E8 04       sub eax,0x4
005D27D4  |.  8B00          mov eax,dword ptr ds:[eax]
005D27D6  |>  83F8 17       cmp eax,0x17 //比较字符串长度是不是23...包括"-"
005D27D9  |.  0F85 23060000 jnz Protecte.005D2E02 //不是就跳向 xor eax,eax....显然是让返回值为零
005D27DF  |.  8B45 FC       mov eax,[local.1]
005D27E2  |.  85C0          test eax,eax
005D27E4  |.  74 16         je short Protecte.005D27FC
005D27E6  |.  8BD0          mov edx,eax
005D27E8  |.  83EA 0A       sub edx,0xA
005D27EB  |.  66:833A 02    cmp word ptr ds:[edx],0x2 //这些看起来很奇怪的比较是检测字符串类型的
005D27EF  |.  74 0B         je short Protecte.005D27FC
005D27F1  |.  8D45 FC       lea eax,[local.1]
005D27F4  |.  8B55 FC       mov edx,[local.1]
005D27F7  |.  E8 9431E3FF   call <Protecte.System_@InternalUStrFromLStr>
005D27FC  |>  66:8378 0A 2D cmp word ptr ds:[eax+0xA],0x2D //比较第11个是不是"-"
005D2801  |.  74 04         je short Protecte.005D2807
005D2803  |.  B0 01         mov al,0x1
005D2805  |.  EB 25         jmp short Protecte.005D282C
005D2807  |>  8B45 FC       mov eax,[local.1]
005D280A  |.  85C0          test eax,eax
005D280C  |.  74 16         je short Protecte.005D2824
005D280E  |.  8BD0          mov edx,eax
005D2810  |.  83EA 0A       sub edx,0xA
005D2813  |.  66:833A 02    cmp word ptr ds:[edx],0x2
005D2817  |.  74 0B         je short Protecte.005D2824
005D2819  |.  8D45 FC       lea eax,[local.1]
005D281C  |.  8B55 FC       mov edx,[local.1]
005D281F  |.  E8 6C31E3FF   call <Protecte.System_@InternalUStrFromLStr>
005D2824  |>  66:8378 16 2D cmp word ptr ds:[eax+0x16],0x2D //比较第22个是不是"-"
005D2829  |.  0F95C0        setne al
005D282C  |>  84C0          test al,al
005D282E  |.  74 04         je short Protecte.005D2834
005D2830  |.  B0 01         mov al,0x1
005D2832  |.  EB 25         jmp short Protecte.005D2859
005D2834  |>  8B45 FC       mov eax,[local.1]
005D2837  |.  85C0          test eax,eax
005D2839  |.  74 16         je short Protecte.005D2851
005D283B  |.  8BD0          mov edx,eax
005D283D  |.  83EA 0A       sub edx,0xA
005D2840  |.  66:833A 02    cmp word ptr ds:[edx],0x2
005D2844  |.  74 0B         je short Protecte.005D2851
005D2846  |.  8D45 FC       lea eax,[local.1]
005D2849  |.  8B55 FC       mov edx,[local.1]
005D284C  |.  E8 3F31E3FF   call <Protecte.System_@InternalUStrFromLStr>
005D2851  |>  66:8378 22 2D cmp word ptr ds:[eax+0x22],0x2D //
005D2856  |.  0F95C0        setne al
005D2859  |>  84C0          test al,al
005D285B  |.  0F85 A1050000 jnz Protecte.005D2E02
005D2861  |.  0FB605 242F5D>movzx eax,byte ptr ds:[0x5D2F24]
005D2868  |.  50            push eax
005D2869  |.  8D45 D4       lea eax,[local.11]
005D286C  |.  50            push eax
005D286D  |.  33C9          xor ecx,ecx
005D286F  |.  BA 942F5D00   mov edx,<Protecte.char_2D>
005D2874  |.  8B45 FC       mov eax,[local.1]
005D2877  |.  E8 C03DE9FF   call <Protecte.StringReplace> //清除"-"
005D287C  |.  8B55 D4       mov edx,[local.11]
005D287F  |.  8D45 FC       lea eax,[local.1]
005D2882  |.  E8 B93EE3FF   call <Protecte.@UStrLAsg>
005D2887  |.  8B45 FC       mov eax,[local.1]
005D288A  |.  85C0          test eax,eax
005D288C  |.  74 16         je short Protecte.005D28A4
005D288E  |.  8BD0          mov edx,eax
005D2890  |.  83EA 0A       sub edx,0xA
005D2893  |.  66:833A 02    cmp word ptr ds:[edx],0x2
005D2897  |.  74 0B         je short Protecte.005D28A4
005D2899  |.  8D45 FC       lea eax,[local.1]
005D289C  |.  8B55 FC       mov edx,[local.1]
005D289F  |.  E8 EC30E3FF   call <Protecte.System_@InternalUStrFromLStr>
005D28A4  |>  85C0          test eax,eax
005D28A6  |.  74 05         je short Protecte.005D28AD
005D28A8  |.  83E8 04       sub eax,0x4
005D28AB  |.  8B00          mov eax,dword ptr ds:[eax]
005D28AD  |>  83F8 14       cmp eax,0x14
005D28B0  |.  0F85 4C050000 jnz Protecte.005D2E02
005D28B6  |.  B8 01000000   mov eax,0x1
005D28BB  |>  8B55 FC       /mov edx,[local.1]
005D28BE  |.  0FB75442 FE   |movzx edx,word ptr ds:[edx+eax*2-0x2]
005D28C3  |.  83C2 D0       |add edx,-0x30
005D28C6  |.  66:83EA 0A    |sub dx,0xA
005D28CA  |.  72 0D         |jb short Protecte.005D28D9
005D28CC  |.  83C2 F9       |add edx,-0x7
005D28CF  |.  66:83EA 06    |sub dx,0x6
005D28D3  |.  0F83 29050000 |jnb Protecte.005D2E02 //这一句又是跳向xor eax,eax
005D28D9  |>  40            |inc eax             //这个小循环只是检测输入的字符是否小于 0x3a
005D28DA  |.  83F8 15       |cmp eax,0x15       //或者是否小于0x47....说白了输入范围就是 0-9,A-F(不管大小)
005D28DD  |.^ 75 DC         \jnz short Protecte.005D28BB
005D28DF  |.  8D45 CC       lea eax,[local.13]
005D28E2  |.  8B55 FC       mov edx,[local.1]
005D28E5  |.  E8 7E40E3FF   call <Protecte.System_@WStrFromUStr>
005D28EA  |.  8B45 CC       mov eax,[local.13]
005D28ED  |.  8D4D D0       lea ecx,[local.12]
005D28F0  |.  BA 04000000   mov edx,0x4
005D28F5  |.  E8 7A89E8FF   call <Protecte._Unit13_LeftStr> //取注册码左4位...到时候要用的.
005D28FA  |.  8B55 D0       mov edx,[local.12]
005D28FD  |.  8D45 EC       lea eax,[local.5]
005D2900  |.  E8 4F40E3FF   call <Protecte.System_@UStrFromWStr>
005D2905  |.  8D45 C4       lea eax,[local.15]
005D2908  |.  8B55 FC       mov edx,[local.1]
005D290B  |.  E8 5840E3FF   call <Protecte.System_@WStrFromUStr>
005D2910  |.  8B45 C4       mov eax,[local.15]
005D2913  |.  8D4D C8       lea ecx,[local.14]
005D2916  |.  BA 10000000   mov edx,0x10
005D291B  |.  E8 7089E8FF   call <Protecte.RightStr> //取注册码右16位
005D2920  |.  8B55 C8       mov edx,[local.14]
005D2923  |.  8D45 FC       lea eax,[local.1]
005D2926  |.  E8 2940E3FF   call <Protecte.System_@UStrFromWStr>
005D292B  |.  8D45 BC       lea eax,[local.17]
005D292E  |.  8B55 FC       mov edx,[local.1]
005D2931  |.  E8 3240E3FF   call <Protecte.System_@WStrFromUStr>
005D2936  |.  8B45 BC       mov eax,[local.17]
005D2939  |.  8D4D C0       lea ecx,[local.16]
005D293C  |.  BA 05000000   mov edx,0x5
005D2941  |.  E8 2E89E8FF   call <Protecte._Unit13_LeftStr>//取注册码右16位的左5位
005D2946  |.  8B55 C0       mov edx,[local.16]
005D2949  |.  8D45 F0       lea eax,[local.4]
005D294C  |.  E8 0340E3FF   call <Protecte.System_@UStrFromWStr>
005D2951  |.  8D45 B4       lea eax,[local.19]
005D2954  |.  8B55 FC       mov edx,[local.1]
005D2957  |.  E8 0C40E3FF   call <Protecte.System_@WStrFromUStr>
005D295C  |.  8B45 B4       mov eax,[local.19]
005D295F  |.  8D4D B8       lea ecx,[local.18]
005D2962  |.  BA 0B000000   mov edx,0xB
005D2967  |.  E8 2489E8FF   call <Protecte.RightStr> //取注册码右11位
005D296C  |.  8B55 B8       mov edx,[local.18]
005D296F  |.  8D45 FC       lea eax,[local.1]
005D2972  |.  E8 DD3FE3FF   call <Protecte.System_@UStrFromWStr>
005D2977  |.  8D45 AC       lea eax,[local.21]
005D297A  |.  8B55 FC       mov edx,[local.1]
005D297D  |.  E8 E63FE3FF   call <Protecte.System_@WStrFromUStr>
005D2982  |.  8B45 AC       mov eax,[local.21]
005D2985  |.  8D4D B0       lea ecx,[local.20]
005D2988  |.  BA 04000000   mov edx,0x4
005D298D  |.  E8 E288E8FF   call <Protecte._Unit13_LeftStr>//取注册码右11位的左4位
005D2992  |.  8B55 B0       mov edx,[local.20]
005D2995  |.  8D45 F8       lea eax,[local.2]
005D2998  |.  E8 B73FE3FF   call <Protecte.System_@UStrFromWStr>
005D299D  |.  8D45 A4       lea eax,[local.23]
005D29A0  |.  8B55 FC       mov edx,[local.1]
005D29A3  |.  E8 C03FE3FF   call <Protecte.System_@WStrFromUStr>
005D29A8  |.  8B45 A4       mov eax,[local.23]
005D29AB  |.  8D4D A8       lea ecx,[local.22]
005D29AE  |.  BA 07000000   mov edx,0x7
005D29B3  |.  E8 D888E8FF   call <Protecte.RightStr>//取注册码右7位
005D29B8  |.  8B55 A8       mov edx,[local.22]
005D29BB  |.  8D45 FC       lea eax,[local.1]
005D29BE  |.  E8 913FE3FF   call <Protecte.System_@UStrFromWStr>
005D29C3  |.  8D45 9C       lea eax,[local.25]
005D29C6  |.  8B55 FC       mov edx,[local.1]
005D29C9  |.  E8 9A3FE3FF   call <Protecte.System_@WStrFromUStr>
005D29CE  |.  8B45 9C       mov eax,[local.25]
005D29D1  |.  8D4D A0       lea ecx,[local.24]
005D29D4  |.  BA 05000000   mov edx,0x5
005D29D9  |.  E8 9688E8FF   call <Protecte._Unit13_LeftStr>//取注册码右7位左5位
005D29DE  |.  8B55 A0       mov edx,[local.24]
005D29E1  |.  8D45 F4       lea eax,[local.3]
005D29E4  |.  E8 6B3FE3FF   call <Protecte.System_@UStrFromWStr>
005D29E9  |.  8D45 94       lea eax,[local.27]
005D29EC  |.  8B55 FC       mov edx,[local.1]
005D29EF  |.  E8 743FE3FF   call <Protecte.System_@WStrFromUStr>
005D29F4  |.  8B45 94       mov eax,[local.27]
005D29F7  |.  8D4D 98       lea ecx,[local.26]
005D29FA  |.  BA 02000000   mov edx,0x2
005D29FF  |.  E8 8C88E8FF   call <Protecte.RightStr>//取注册码右2位
005D2A04  |.  8B55 98       mov edx,[local.26]
005D2A07  |.  8D45 FC       lea eax,[local.1]
005D2A0A  |.  E8 453FE3FF   call <Protecte.System_@UStrFromWStr>
005D2A0F  |.  8D85 7CFFFFFF lea eax,[local.33]
005D2A15  |.  8B55 EC       mov edx,[local.5]
005D2A18  |.  B9 00000000   mov ecx,0x0
005D2A1D  |.  E8 0E3FE3FF   call <Protecte.System_@LStrFromUStr>
005D2A22  |.  8B85 7CFFFFFF mov eax,[local.33]      //左4位
005D2A28  |.  8D55 80       lea edx,[local.32]
005D2A2B  |.  E8 A8B6FCFF   call <Protecte.calcMD5> //计算MD5...当初也分析了一会儿才知道,早该用Kypto analyzer的
005D2A30  |.  8D45 80       lea eax,[local.32]
005D2A33  |.  8D55 90       lea edx,[local.28]
005D2A36  |.  E8 35B7FCFF   call <Protecte.MD5Hex2Ustr>//MD5算出来是hex数值,要转成str...很简单的分析,不说了
005D2A3B  |.  8B45 90       mov eax,[local.28]
005D2A3E  |.  50            push eax
005D2A3F  |.  8D85 60FFFFFF lea eax,[local.40]
005D2A45  |.  8B55 F8       mov edx,[local.2]
005D2A48  |.  B9 00000000   mov ecx,0x0
005D2A4D  |.  E8 DE3EE3FF   call <Protecte.System_@LStrFromUStr>
005D2A52  |.  8B85 60FFFFFF mov eax,[local.40]
005D2A58  |.  8D55 80       lea edx,[local.32]
005D2A5B  |.  E8 78B6FCFF   call <Protecte.calcMD5> //计算右11位再取左四位的MD5
005D2A60  |.  8D45 80       lea eax,[local.32]
005D2A63  |.  8D95 64FFFFFF lea edx,[local.39]
005D2A69  |.  E8 02B7FCFF   call <Protecte.MD5Hex2Ustr>
005D2A6E  |.  8B85 64FFFFFF mov eax,[local.39]
005D2A74  |.  8D95 68FFFFFF lea edx,[local.38]
005D2A7A  |.  E8 45A1E8FF   call <Protecte._Unit13_UpperCase>
005D2A7F  |.  8B95 68FFFFFF mov edx,[local.38]
005D2A85  |.  8D85 6CFFFFFF lea eax,[local.37]
005D2A8B  |.  E8 D83EE3FF   call <Protecte.System_@WStrFromUStr>
005D2A90  |.  8B85 6CFFFFFF mov eax,[local.37]
005D2A96  |.  8D8D 70FFFFFF lea ecx,[local.36]
005D2A9C  |.  BA 04000000   mov edx,0x4
005D2AA1  |.  E8 EA87E8FF   call <Protecte.RightStr>
005D2AA6  |.  8B95 70FFFFFF mov edx,[local.36]
005D2AAC  |.  8D85 74FFFFFF lea eax,[local.35]
005D2AB2  |.  B9 00000000   mov ecx,0x0
005D2AB7  |.  E8 5C30E3FF   call <Protecte.System_@LStrFromWStr>
005D2ABC  |.  8B85 74FFFFFF mov eax,[local.35]
005D2AC2  |.  8D55 80       lea edx,[local.32]
005D2AC5  |.  E8 0EB6FCFF   call <Protecte.calcMD5>//计算右11位再取左4位的MD5的右4位的MD5....有点绕了
005D2ACA  |.  8D45 80       lea eax,[local.32]
005D2ACD  |.  8D95 78FFFFFF lea edx,[local.34]
005D2AD3  |.  E8 98B6FCFF   call <Protecte.MD5Hex2Ustr>
005D2AD8  |.  8B95 78FFFFFF mov edx,[local.34]
005D2ADE  |.  58            pop eax
005D2ADF  |.  E8 1443E3FF   call <Protecte.j_@UStrCmp>//比较上面计算的MD5值...换一种表达就是key[0-3]_md5, key[9-12]_md5_r4_md5
005D2AE4  |.  0F85 18030000 jnz Protecte.005D2E02
005D2AEA  |.  FF75 EC       push [local.5]
005D2AED  |.  FF75 F0       push [local.4]
005D2AF0  |.  FF75 F8       push [local.2]
005D2AF3  |.  FF75 F4       push [local.3]
005D2AF6  |.  FF75 FC       push [local.1]
005D2AF9  |.  8D45 FC       lea eax,[local.1]
005D2AFC  |.  BA 05000000   mov edx,0x5
005D2B01  |.  E8 4A41E3FF   call <Protecte.System_@UStrCatN>
005D2B06  |.  B8 01000000   mov eax,0x1
005D2B0B  |>  8B55 FC       /mov edx,[local.1]
005D2B0E  |.  0FB75442 FE   |movzx edx,word ptr ds:[edx+eax*2-0x2]
005D2B13  |.  83C2 D0       |add edx,-0x30
005D2B16  |.  66:83EA 0A    |sub dx,0xA
005D2B1A  |.  72 0D         |jb short Protecte.005D2B29
005D2B1C  |.  83C2 F9       |add edx,-0x7
005D2B1F  |.  66:83EA 09    |sub dx,0x9
005D2B23  |.  0F83 D9020000 |jnb Protecte.005D2E02
005D2B29  |>  40            |inc eax
005D2B2A  |.  83F8 15       |cmp eax,0x15 //这个小循环和上面是一样的.限制范围的
005D2B2D  |.^ 75 DC         \jnz short Protecte.005D2B0B
005D2B2F  |.  8D85 58FFFFFF lea eax,[local.42]
005D2B35  |.  8B55 FC       mov edx,[local.1]
005D2B38  |.  E8 2B3EE3FF   call <Protecte.System_@WStrFromUStr>
005D2B3D  |.  8B85 58FFFFFF mov eax,[local.42]
005D2B43  |.  8D8D 5CFFFFFF lea ecx,[local.41]
005D2B49  |.  BA 04000000   mov edx,0x4
005D2B4E  |.  E8 2187E8FF   call <Protecte._Unit13_LeftStr> //好吧,又开始取了,左4位
005D2B53  |.  8B95 5CFFFFFF mov edx,[local.41]
005D2B59  |.  8D45 EC       lea eax,[local.5]
005D2B5C  |.  E8 F33DE3FF   call <Protecte.System_@UStrFromWStr>
005D2B61  |.  8D85 50FFFFFF lea eax,[local.44]
005D2B67  |.  8B55 FC       mov edx,[local.1]
005D2B6A  |.  E8 F93DE3FF   call <Protecte.System_@WStrFromUStr>
005D2B6F  |.  8B85 50FFFFFF mov eax,[local.44]
005D2B75  |.  8D8D 54FFFFFF lea ecx,[local.43]
005D2B7B  |.  BA 10000000   mov edx,0x10
005D2B80  |.  E8 0B87E8FF   call <Protecte.RightStr> //右16位
005D2B85  |.  8B95 54FFFFFF mov edx,[local.43]
005D2B8B  |.  8D45 FC       lea eax,[local.1]
005D2B8E  |.  E8 C13DE3FF   call <Protecte.System_@UStrFromWStr>
005D2B93  |.  8D85 48FFFFFF lea eax,[local.46]
005D2B99  |.  8B55 FC       mov edx,[local.1]
005D2B9C  |.  E8 C73DE3FF   call <Protecte.System_@WStrFromUStr>
005D2BA1  |.  8B85 48FFFFFF mov eax,[local.46]
005D2BA7  |.  8D8D 4CFFFFFF lea ecx,[local.45]
005D2BAD  |.  BA 05000000   mov edx,0x5
005D2BB2  |.  E8 BD86E8FF   call <Protecte._Unit13_LeftStr> //右16位的左5位
005D2BB7  |.  8B95 4CFFFFFF mov edx,[local.45]
005D2BBD  |.  8D45 F0       lea eax,[local.4]
005D2BC0  |.  E8 8F3DE3FF   call <Protecte.System_@UStrFromWStr>
005D2BC5  |.  8D85 40FFFFFF lea eax,[local.48]
005D2BCB  |.  8B55 FC       mov edx,[local.1]
005D2BCE  |.  E8 953DE3FF   call <Protecte.System_@WStrFromUStr>
005D2BD3  |.  8B85 40FFFFFF mov eax,[local.48]
005D2BD9  |.  8D8D 44FFFFFF lea ecx,[local.47]
005D2BDF  |.  BA 0B000000   mov edx,0xB
005D2BE4  |.  E8 A786E8FF   call <Protecte.RightStr> //右11位
005D2BE9  |.  8B95 44FFFFFF mov edx,[local.47]
005D2BEF  |.  8D45 FC       lea eax,[local.1]
005D2BF2  |.  E8 5D3DE3FF   call <Protecte.System_@UStrFromWStr>
005D2BF7  |.  8D85 38FFFFFF lea eax,[local.50]
005D2BFD  |.  8B55 FC       mov edx,[local.1]
005D2C00  |.  E8 633DE3FF   call <Protecte.System_@WStrFromUStr>
005D2C05  |.  8B85 38FFFFFF mov eax,[local.50]
005D2C0B  |.  8D8D 3CFFFFFF lea ecx,[local.49]
005D2C11  |.  BA 04000000   mov edx,0x4
005D2C16  |.  E8 5986E8FF   call <Protecte._Unit13_LeftStr> //右11位的左4位
005D2C1B  |.  8B95 3CFFFFFF mov edx,[local.49]
005D2C21  |.  8D45 F8       lea eax,[local.2]
005D2C24  |.  E8 2B3DE3FF   call <Protecte.System_@UStrFromWStr>
005D2C29  |.  8D85 30FFFFFF lea eax,[local.52]
005D2C2F  |.  8B55 FC       mov edx,[local.1]
005D2C32  |.  E8 313DE3FF   call <Protecte.System_@WStrFromUStr>
005D2C37  |.  8B85 30FFFFFF mov eax,[local.52]
005D2C3D  |.  8D8D 34FFFFFF lea ecx,[local.51]
005D2C43  |.  BA 07000000   mov edx,0x7
005D2C48  |.  E8 4386E8FF   call <Protecte.RightStr> //右7位
005D2C4D  |.  8B95 34FFFFFF mov edx,[local.51]
005D2C53  |.  8D45 FC       lea eax,[local.1]
005D2C56  |.  E8 F93CE3FF   call <Protecte.System_@UStrFromWStr>
005D2C5B  |.  8D85 28FFFFFF lea eax,[local.54]
005D2C61  |.  8B55 FC       mov edx,[local.1]
005D2C64  |.  E8 FF3CE3FF   call <Protecte.System_@WStrFromUStr>
005D2C69  |.  8B85 28FFFFFF mov eax,[local.54]
005D2C6F  |.  8D8D 2CFFFFFF lea ecx,[local.53]
005D2C75  |.  BA 05000000   mov edx,0x5
005D2C7A  |.  E8 F585E8FF   call <Protecte._Unit13_LeftStr>//右7位的左5位
005D2C7F  |.  8B95 2CFFFFFF mov edx,[local.53]
005D2C85  |.  8D45 F4       lea eax,[local.3]
005D2C88  |.  E8 C73CE3FF   call <Protecte.System_@UStrFromWStr>
005D2C8D  |.  8D85 20FFFFFF lea eax,[local.56]
005D2C93  |.  8B55 FC       mov edx,[local.1]
005D2C96  |.  E8 CD3CE3FF   call <Protecte.System_@WStrFromUStr>
005D2C9B  |.  8B85 20FFFFFF mov eax,[local.56]
005D2CA1  |.  8D8D 24FFFFFF lea ecx,[local.55]
005D2CA7  |.  BA 02000000   mov edx,0x2
005D2CAC  |.  E8 DF85E8FF   call <Protecte.RightStr> //右两位
005D2CB1  |.  8B95 24FFFFFF mov edx,[local.55]
005D2CB7  |.  8D45 FC       lea eax,[local.1]
005D2CBA  |.  E8 953CE3FF   call <Protecte.System_@UStrFromWStr>
005D2CBF  |.  8D85 10FFFFFF lea eax,[local.60]
005D2CC5  |.  8B55 F0       mov edx,[local.4]
005D2CC8  |.  B9 00000000   mov ecx,0x0
005D2CCD  |.  E8 5E3CE3FF   call <Protecte.System_@LStrFromUStr>
005D2CD2  |.  8B85 10FFFFFF mov eax,[local.60]
005D2CD8  |.  8D55 80       lea edx,[local.32]
005D2CDB  |.  E8 F8B3FCFF   call <Protecte.calcMD5> //计算右16位的左5位MD5
005D2CE0  |.  8D45 80       lea eax,[local.32]
005D2CE3  |.  8D95 14FFFFFF lea edx,[local.59]
005D2CE9  |.  E8 82B4FCFF   call <Protecte.MD5Hex2Ustr>
005D2CEE  |.  8B95 14FFFFFF mov edx,[local.59]
005D2CF4  |.  8D85 18FFFFFF lea eax,[local.58]
005D2CFA  |.  B9 00000000   mov ecx,0x0
005D2CFF  |.  E8 2C3CE3FF   call <Protecte.System_@LStrFromUStr>
005D2D04  |.  8B85 18FFFFFF mov eax,[local.58]
005D2D0A  |.  8D55 80       lea edx,[local.32]
005D2D0D  |.  E8 C6B3FCFF   call <Protecte.calcMD5>//计算右16位的左5位MD5的MD5
005D2D12  |.  8D45 80       lea eax,[local.32]
005D2D15  |.  8D95 1CFFFFFF lea edx,[local.57]
005D2D1B  |.  E8 50B4FCFF   call <Protecte.MD5Hex2Ustr>
005D2D20  |.  8B85 1CFFFFFF mov eax,[local.57]
005D2D26  |.  50            push eax
005D2D27  |.  8D85 ECFEFFFF lea eax,[local.69]
005D2D2D  |.  8B55 F4       mov edx,[local.3]
005D2D30  |.  B9 00000000   mov ecx,0x0
005D2D35  |.  E8 F63BE3FF   call <Protecte.System_@LStrFromUStr>
005D2D3A  |.  8B85 ECFEFFFF mov eax,[local.69]
005D2D40  |.  8D55 80       lea edx,[local.32]
005D2D43  |.  E8 90B3FCFF   call <Protecte.calcMD5> //计算右7位的左5位MD5
005D2D48  |.  8D45 80       lea eax,[local.32]
005D2D4B  |.  8D95 F0FEFFFF lea edx,[local.68]
005D2D51  |.  E8 1AB4FCFF   call <Protecte.MD5Hex2Ustr>
005D2D56  |.  8B85 F0FEFFFF mov eax,[local.68]
005D2D5C  |.  8D95 F4FEFFFF lea edx,[local.67]
005D2D62  |.  E8 5D9EE8FF   call <Protecte._Unit13_UpperCase>
005D2D67  |.  8B95 F4FEFFFF mov edx,[local.67]
005D2D6D  |.  8D85 F8FEFFFF lea eax,[local.66]
005D2D73  |.  E8 F03BE3FF   call <Protecte.System_@WStrFromUStr>
005D2D78  |.  8B85 F8FEFFFF mov eax,[local.66]
005D2D7E  |.  8D8D FCFEFFFF lea ecx,[local.65]
005D2D84  |.  BA 05000000   mov edx,0x5
005D2D89  |.  E8 E684E8FF   call <Protecte._Unit13_LeftStr>//取右7位的左5位MD5的左5位
005D2D8E  |.  8B95 FCFEFFFF mov edx,[local.65]
005D2D94  |.  8D85 00FFFFFF lea eax,[local.64]
005D2D9A  |.  B9 00000000   mov ecx,0x0
005D2D9F  |.  E8 742DE3FF   call <Protecte.System_@LStrFromWStr>
005D2DA4  |.  8B85 00FFFFFF mov eax,[local.64]
005D2DAA  |.  8D55 80       lea edx,[local.32]
005D2DAD  |.  E8 26B3FCFF   call <Protecte.calcMD5>//计算右7位的左5位MD5的左5位的MD5
005D2DB2  |.  8D45 80       lea eax,[local.32]
005D2DB5  |.  8D95 04FFFFFF lea edx,[local.63]
005D2DBB  |.  E8 B0B3FCFF   call <Protecte.MD5Hex2Ustr>
005D2DC0  |.  8B95 04FFFFFF mov edx,[local.63]
005D2DC6  |.  8D85 08FFFFFF lea eax,[local.62]
005D2DCC  |.  B9 00000000   mov ecx,0x0
005D2DD1  |.  E8 5A3BE3FF   call <Protecte.System_@LStrFromUStr>
005D2DD6  |.  8B85 08FFFFFF mov eax,[local.62]
005D2DDC  |.  8D55 80       lea edx,[local.32]
005D2DDF  |.  E8 F4B2FCFF   call <Protecte.calcMD5>计算右7位的左5位MD5的左5位的MD5的MD5....我自己看着都晕了
005D2DE4  |.  8D45 80       lea eax,[local.32]          //简化一下就是key_r7_L5_md5_L5_md5_md5
005D2DE7  |.  8D95 0CFFFFFF lea edx,[local.61]
005D2DED  |.  E8 7EB3FCFF   call <Protecte.MD5Hex2Ustr>
005D2DF2  |.  8B95 0CFFFFFF mov edx,[local.61]
005D2DF8  |.  58            pop eax
005D2DF9  |.  E8 FA3FE3FF   call <Protecte.j_@UStrCmp>//比较上述计算出来的MD5,即key[0-3]_md5, key[4-8]_MD5_MD5 ,key[13-17]_MD5_L5_MD5_MD5比较
005D2DFE  |.  75 02         jnz short Protecte.005D2E02
005D2E00  |.  B3 01         mov bl,0x1
005D2E02  |>  33C0          xor eax,eax //不等就让返回值为0了...
005D2E04  |.  5A            pop edx
005D2E05  |.  59            pop ecx
005D2E06  |.  59            pop ecx
005D2E07  |.  64:8910       mov dword ptr fs:[eax],edx
005D2E0A  |.  68 1B2F5D00   push Protecte.005D2F1B
005D2E0F  |>  8D85 ECFEFFFF lea eax,[local.69]
005D2E15  |.  E8 8A29E3FF   call <Protecte.System_@LStrClr>
005D2E1A  |.  8D85 F0FEFFFF lea eax,[local.68]
005D2E20  |.  BA 02000000   mov edx,0x2
005D2E25  |.  E8 BA38E3FF   call <Protecte.j_@LStrArrayClr>
005D2E2A  |.  8D85 F8FEFFFF lea eax,[local.66]
005D2E30  |.  BA 02000000   mov edx,0x2
005D2E35  |.  E8 6A34E3FF   call <Protecte.System_@WStrArrayClr>
005D2E3A  |.  8D85 00FFFFFF lea eax,[local.64]
005D2E40  |.  E8 5F29E3FF   call <Protecte.System_@LStrClr>
005D2E45  |.  8D85 04FFFFFF lea eax,[local.63]
005D2E4B  |.  E8 8C38E3FF   call <Protecte.j_System_@LStrClr>
005D2E50  |.  8D85 08FFFFFF lea eax,[local.62]
005D2E56  |.  E8 4929E3FF   call <Protecte.System_@LStrClr>
005D2E5B  |.  8D85 0CFFFFFF lea eax,[local.61]
005D2E61  |.  E8 7638E3FF   call <Protecte.j_System_@LStrClr>
005D2E66  |.  8D85 10FFFFFF lea eax,[local.60]
005D2E6C  |.  E8 3329E3FF   call <Protecte.System_@LStrClr>
005D2E71  |.  8D85 14FFFFFF lea eax,[local.59]
005D2E77  |.  E8 6038E3FF   call <Protecte.j_System_@LStrClr>
005D2E7C  |.  8D85 18FFFFFF lea eax,[local.58]
005D2E82  |.  E8 1D29E3FF   call <Protecte.System_@LStrClr>
005D2E87  |.  8D85 1CFFFFFF lea eax,[local.57]
005D2E8D  |.  E8 4A38E3FF   call <Protecte.j_System_@LStrClr>
005D2E92  |.  8D85 20FFFFFF lea eax,[local.56]
005D2E98  |.  BA 10000000   mov edx,0x10
005D2E9D  |.  E8 0234E3FF   call <Protecte.System_@WStrArrayClr>
005D2EA2  |.  8D85 60FFFFFF lea eax,[local.40]
005D2EA8  |.  E8 F728E3FF   call <Protecte.System_@LStrClr>
005D2EAD  |.  8D85 64FFFFFF lea eax,[local.39]
005D2EB3  |.  BA 02000000   mov edx,0x2
005D2EB8  |.  E8 2738E3FF   call <Protecte.j_@LStrArrayClr>
005D2EBD  |.  8D85 6CFFFFFF lea eax,[local.37]
005D2EC3  |.  BA 02000000   mov edx,0x2
005D2EC8  |.  E8 D733E3FF   call <Protecte.System_@WStrArrayClr>
005D2ECD  |.  8D85 74FFFFFF lea eax,[local.35]
005D2ED3  |.  E8 CC28E3FF   call <Protecte.System_@LStrClr>
005D2ED8  |.  8D85 78FFFFFF lea eax,[local.34]
005D2EDE  |.  E8 F937E3FF   call <Protecte.j_System_@LStrClr>
005D2EE3  |.  8D85 7CFFFFFF lea eax,[local.33]
005D2EE9  |.  E8 B628E3FF   call <Protecte.System_@LStrClr>
005D2EEE  |.  8D45 90       lea eax,[local.28]
005D2EF1  |.  E8 E637E3FF   call <Protecte.j_System_@LStrClr>
005D2EF6  |.  8D45 94       lea eax,[local.27]
005D2EF9  |.  BA 10000000   mov edx,0x10
005D2EFE  |.  E8 A133E3FF   call <Protecte.System_@WStrArrayClr>
005D2F03  |.  8D45 D4       lea eax,[local.11]
005D2F06  |.  BA 0B000000   mov edx,0xB
005D2F0B  |.  E8 D437E3FF   call <Protecte.j_@LStrArrayClr>
005D2F10  \.  C3            retn
005D2F11   .^ E9 B21EE3FF   jmp <Protecte.@HandleFinally>
005D2F16   .^ E9 F4FEFFFF   jmp Protecte.005D2E0F
005D2F1B   .  8BC3          mov eax,ebx //返回值在ebx
005D2F1D   .  5B            pop ebx
005D2F1E   .  8BE5          mov esp,ebp
005D2F20   .  5D            pop ebp
005D2F21   .  C3            retn
```
