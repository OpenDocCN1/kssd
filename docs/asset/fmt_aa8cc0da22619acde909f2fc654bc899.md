
```
005E314A    E8 711CE2FF      call JyCrm.00404DC0  

005E314F    8B4D D4         mov ecx,dword ptr ss:\[ebp\-2C\]  

005E3152    BA 9C325E00     mov edx,JyCrm.005E329C           //ASCII "SignCode"  

                                                     //注册表内的产品编号（硬盘号）  

005E3157    8BC3            mov eax,ebx  

005E3159    E8 6602E9FF     call JyCrm.004733C4  

005E315E    8B45 F8         mov eax,dword ptr ss:\[ebp\-8\]  

005E3161    E8 1A1FE2FF     call JyCrm.00405080  

005E3166    50              push eax  

005E3167    E8 6012FFFF     call <jmp.&PunUnitLib.SavePass>  

005E316C    8BD0            mov edx,eax  

005E316E    8D45 CC         lea eax,dword ptr ss:\[ebp\-34\]  

005E3171    E8 4A1CE2FF     call JyCrm.00404DC0  

005E3176    8B4D CC         mov ecx,dword ptr ss:\[ebp\-34\]  

005E3179    BA B0325E00     mov edx,JyCrm.005E32B0           //ASCII "RegCode"  

                                                     //注册表内的授权编号（注册码）  

005E317E    8BC3            mov eax,ebx  

005E3180    E8 3F02E9FF     call JyCrm.004733C4  

005E3185    8BC3            mov eax,ebx  

005E3187    E8 500BE2FF     call JyCrm.00403CDC  

005E318C    6A 00           push 0  

005E318E    68 B8325E00     push JyCrm.005E32B8             //ASCII "系统注册成功，欢迎你使用本软件！"  

005E3193    E8 0C12FFFF     call <jmp.&PunUnitLib.ShowMess>  

005E3198    A1 28B56100     mov eax,dword ptr ds:\[61B528\]  

005E319D    C700 02000000   mov dword ptr ds:\[eax\],2  

005E31A3    A1 F0B26100     mov eax,dword ptr ds:\[61B2F0\]  

005E31A8    8B00            mov eax,dword ptr ds:\[eax\]  

005E31AA    E8 C5D8E8FF     call JyCrm.00470A74  

005E31AF    33C0            xor eax,eax  

005E31B1    5A              pop edx  

005E31B2    59              pop ecx  

005E31B3    59              pop ecx  

005E31B4    64:8910         mov dword ptr fs:\[eax\],edx  

005E31B7    EB 20           jmp short JyCrm.005E31D9  

005E31B9  ^ E9 FE0FE2FF     jmp JyCrm.004041BC  

005E31BE    8B45 FC         mov eax,dword ptr ss:\[ebp\-4\]  

005E31C1    E8 FE9FE8FF     call JyCrm.0046D1C4  

005E31C6    E8 1D14E2FF     call JyCrm.004045E8  

005E31CB    EB 0C           jmp short JyCrm.005E31D9  

005E31CD    6A 03           push 3  

005E31CF    68 DC325E00     push JyCrm.005E32DC             //ASCII "系统注册失败，请检查注册是否有误！"  

005E31D4    E8 CB11FFFF     call <jmp.&PunUnitLib.ShowMess>  

005E31D9    33C0            xor eax,eax  

005E31DB    5A              pop edx  

005E31DC    59              pop ecx  

005E31DD    59              pop ecx  

005E31DE    64:8910         mov dword ptr fs:\[eax\],edx  

005E31E1    68 45325E00     push JyCrm.005E3245  

005E31E6    8D45 CC         lea eax,dword ptr ss:\[ebp\-34\]  

005E31E9    E8 E219E2FF     call JyCrm.00404BD0  

005E31EE    8D45 D0         lea eax,dword ptr ss:\[ebp\-30\]  

005E31F1    E8 DA19E2FF     call JyCrm.00404BD0  

005E31F6    8D45 D4         lea eax,dword ptr ss:\[ebp\-2C\]  

005E31F9    E8 D219E2FF     call JyCrm.00404BD0  

005E31FE    8D45 D8         lea eax,dword ptr ss:\[ebp\-28\]  

005E3201    BA 02000000     mov edx,2  

005E3206    E8 E919E2FF     call JyCrm.00404BF4  

005E320B    8D45 E0         lea eax,dword ptr ss:\[ebp\-20\]  

005E320E    E8 BD19E2FF     call JyCrm.00404BD0  

005E3213    8D45 E4         lea eax,dword ptr ss:\[ebp\-1C\]  

005E3216    BA 02000000     mov edx,2  

005E321B    E8 D419E2FF     call JyCrm.00404BF4  

005E3220    8D45 EC         lea eax,dword ptr ss:\[ebp\-14\]  

005E3223    E8 A819E2FF     call JyCrm.00404BD0  

005E3228    8D45 F0         lea eax,dword ptr ss:\[ebp\-10\]  

005E322B    E8 A019E2FF     call JyCrm.00404BD0  

005E3230    8D45 F4         lea eax,dword ptr ss:\[ebp\-C\]  

005E3233    BA 02000000     mov edx,2  

005E3238    E8 B719E2FF     call JyCrm.00404BF4  

005E323D    C3              retn  

005E323E  ^ E9 2D12E2FF     jmp JyCrm.00404470  

005E3243  ^ EB A1           jmp short JyCrm.005E31E6  

005E3245    5F              pop edi  

005E3246    5E              pop esi  

005E3247    5B              pop ebx  

005E3248    8BE5            mov esp,ebp  

005E324A    5D              pop ebp  

005E324B    C3              retn  
```

## 跟进：005E3093 call <jmp.&PunUnitLib.GetRegPass>

```
005D43D4  - FF25 48FB6100   jmp dword ptr ds:\[<&PunUnitLib.GetRegPass>\]   //跳向算法 CALL，F8 跟进★

005D43DA    8BC0            mov eax,eax  

005D43DC    FFFF            ???                                             ; 未知命令  

005D43DE    FFFF            ???                                             ; 未知命令  

005D43E0    0C 00           or al,0  

005D43E2    0000            add byte ptr ds:\[eax\],al  

005D43E4    CE              into  

005D43E5    F7B0 B2BCD1D2   div dword ptr ds:\[eax+D2D1BCB2\]  

005D43EB    CB              retf  
```

## 跟进：005D43D4 jmp dword ptr ds:\[<&PunUnitLib.GetRegPass>\]

```
003E9024 >   55              push ebp  

003E9025      8BEC            mov ebp,esp  

003E9027      B9 06000000     mov ecx,6                                   //检查注册内容是否填写完整，并循环 6 次  

003E902C      6A 00           push 0  

003E902E      6A 00           push 0  

003E9030      49              dec ecx  

003E9031  ^  75 F9           jnz short PunUnitL.003E902C                 //向上循环检查 6 次  

003E9033      53              push ebx  

003E9034      56              push esi  

003E9035      33C0            xor eax,eax                                 //EAX 清零  

003E9037      55              push ebp  

003E9038      68 F2913E00     push PunUnitL.003E91F2  

003E903D      64:FF30         push dword ptr fs:\[eax\]  

003E9040      64:8920         mov dword ptr fs:\[eax\],esp  

003E9043      8D45 EC         lea eax,dword ptr ss:\[ebp\-14\]  

003E9046      E8 65B5F8FF     call PunUnitL.003745B0                     //取产品编号，ASCII "3HSCCTSM"  

003E904B      8D45 F0         lea eax,dword ptr ss:\[ebp\-10\]  

003E904E      8B55 08         mov edx,dword ptr ss:\[ebp+8\]  

003E9051      E8 4AB7F8FF     call PunUnitL.003747A0  

003E9056      8B45 F0         mov eax,dword ptr ss:\[ebp\-10\]          //移入 EAX，准备开始计算  

003E9059      E8 0AB8F8FF     call PunUnitL.00374868  

003E905E      8BF0            mov esi,eax                               //取产品编号位数，EAX=8  

003E9060      85F6            test esi,esi                              //检查产品编号位数是否合法  

003E9062      7E 26           jle short PunUnitL.003E908A  

003E9064      BB 01000000     mov ebx,1                                 //运算开始  

003E9069      8D4D E8         lea ecx,dword ptr ss:\[ebp\-18\]  

003E906C      8B45 F0         mov eax,dword ptr ss:\[ebp\-10\]  

003E906F      0FB64418 FF     movzx eax,byte ptr ds:\[eax+ebx\-1\]      //依次取产品编号的 HEX 值  

003E9074      33D2            xor edx,edx                               //异或清零  

003E9076      E8 F905F9FF     call PunUnitL.00379674  

003E907B      8B55 E8         mov edx,dword ptr ss:\[ebp\-18\]          //产品编号的 HEX 值  

                                                     //1、EDX=33“3”  

                                                     //2、EDX=48“H”  

                                                     //3、EDX=53“S”  

                                                     //4、EDX=43“C”  

                                                     //5、EDX=43“C”  

                                                     //6、EDX=54“T”  

                                                     //7、EDX=53“S”  

                                                     //8、EDX=4D“M”  

003E907E      8D45 FC         lea eax,dword ptr ss:\[ebp\-4\]  

003E9081      E8 EAB7F8FF     call PunUnitL.00374870  

003E9086      43              inc ebx                                     //EBX 自加一，指向下一位  

003E9087      4E              dec esi  
```
