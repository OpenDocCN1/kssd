
```
      002C04C8            8B45 AC           mov eax,dword ptr ss:[ebp-54]  

      002C04CB            0385 D1FEFFFF     add eax,dword ptr ss:[ebp-12F]  

      002C04D1            50              push eax  

      002C04D2            8D85 7CFFFFFF     lea eax,dword ptr ss:[ebp-84]  

      002C04D8            50              push eax  

      002C04D9            FFD6            call esi  

      002C04DB            EB 5B           jmp short       002C0538  

      002C04DD            8BFB            mov edi,ebx  

      002C04DF            037D 8C         add edi,dword ptr ss:[ebp-74]  

      002C04E2            0FB685 D0FEFFFF  movzx eax,byte ptr ss:[ebp-130]  

      002C04E9            50              push eax  

      002C04EA            57              push edi  

      002C04EB            E8 9B000000      call       002C058B  

      002C04F0            8945 80         mov dword ptr ss:[ebp-80],eax  

      002C04F3            8B45 80         mov eax,dword ptr ss:[ebp-80]  

      002C04F6            0145 8C         add dword ptr ss:[ebp-74],eax  

      002C04F9            E8 D1000000      call       002C05CF  

      002C04FE            8945 A8         mov dword ptr ss:[ebp-58],eax  

      002C0501            50              push eax  

      002C0502            8B45 A8         mov eax,dword ptr ss:[ebp-58]  

      002C0505            C700 33C033C0     mov dword ptr ds:[eax],C033C033  

      002C050B            C740 04 33C0C300  mov dword ptr ds:[eax+4],0C3C033  

      002C0512            58              pop eax  

      002C0513            83BD D5FEFFFF 00  cmp dword ptr ss:[ebp-12B],0  

      002C051A            75 1C           jnz short       002C0538  

      002C051C            8BFB            mov edi,ebx  

      002C051E            037D 8C         add edi,dword ptr ss:[ebp-74]  

      002C0521            0FB685 D0FEFFFF  movzx eax,byte ptr ss:[ebp-130]  

      002C0528            50              push eax  

      002C0529            57              push edi  

      002C052A            E8 5C000000      call       002C058B  

      002C052F            8945 80         mov dword ptr ss:[ebp-80],eax  

      002C0532            8B45 80         mov eax,dword ptr ss:[ebp-80]  

      002C0535            0145 8C         add dword ptr ss:[ebp-74],eax  

      002C0538            6A 2C           push 2C                         

      002C053A            8D85 D0FEFFFF     lea eax,dword ptr ss:[ebp-130]  

      002C0540            50              push eax  

      002C0541            8BC3            mov eax,ebx  

      002C0543            0345 8C         add eax,dword ptr ss:[ebp-74]  

      002C0546            50              push eax  

      002C0547            FFD6            call esi                       //将下一个 ss 数据结构 copy 入堆栈  

      002C0549            83BD D1FEFFFF 00  cmp dword ptr ss:[ebp-12F],0         //通过检测 ss.IATRva 看是否全部处理完毕  

      002C0550          ^ 0F85 27FDFFFF    jnz       002C027D                 //ss.IATRva 不为 0 则跳上去处理  

      002C0556            8B85 40FFFFFF     mov eax,dword ptr ss:[ebp-C0]  

      002C055C            0345 AC           add eax,dword ptr ss:[ebp-54]  

      002C055F            5F              pop edi  

      002C0560            5E              pop esi  

      002C0561            5B              pop ebx  

      002C0562            8BE5            mov esp,ebp  

      002C0564            5D              pop ebp  

      002C0565            C3              retn  
```

经过分析后，修复起来很方便，因为数据结构我们需要的信息都有

### 1. 还原 `IAT` 的真实名字

```
      002C034B          /E9 B0060000      jmp 002C0A00                             //跳到自己的空间  

      002C0350          |90              nop  

      002C0A00           FF55 9C         call dword ptr ss:[ebp-64]                //取得导入函数的地址  

      002C0A03           8945 84         mov dword ptr ss:[ebp-7C],eax              //原句照抄  

      002C0A06           8B95 D1FEFFFF    mov edx,dword ptr ss:[ebp-12F]            //ss.IATRva 取出  

      002C0A0C           0355 AC         add edx,dword ptr ss:[ebp-54]             //加上基址得到 IAT 的地址  

      002C0A0F           8902            mov dword ptr ds:[edx],eax                //导入函数地址写入 IAT  

      002C0A11        ^ E9 3AF9FFFF      jmp 002C0350                             //跳回去  
```

### 2. 还原变形的代码

```
      002C0414           66:C785 F4FEFFFF F>mov word ptr ss:[ebp-10C],15FF           //原来的 90 e8 改成 ff 15  

      002C041D           8B85 D1FEFFFF     mov eax,dword ptr ss:[ebp-12F]            //取出相应的 ss.IATRva  

      002C0423           0345 AC           add eax,dword ptr ss:[ebp-54]             //得到 IAT 地址  

      002C0426           90              nop  

      002C0427           90              nop  

      002C0428           90              nop  

      002C0429           90              nop  

      002C042A           90              nop  

      002C042B           90              nop  

      002C042C           90              nop  

      002C042D           90              nop  

      002C042E           90              nop  

      002C042F           90              nop  

      002C0430           90              nop  

      002C0431           90              nop  

      002C0432           90              nop  

      002C0433           90              nop  

      002C0434           90              nop  

      002C03B6           66:C785 EEFEFFFF E>mov word ptr ss:[ebp-112], 25FF          //e9 改成 ff 25 也是一样，不过要多动几行  

                                                                                      因为后面是 inc eax，我们需要 eax+2  
```

经过这样处理后，代码段都修复完成，输入表只有一个未能识别，那个就是 `MessageBoxW`。

由于 `netwowell` 大侠没有把 `mov` `IAT` 的部分打开，所以这里就没有仔细分析。

谢谢大家看完

`4dd466d0e1e0cc123afbe504da2462ca`
