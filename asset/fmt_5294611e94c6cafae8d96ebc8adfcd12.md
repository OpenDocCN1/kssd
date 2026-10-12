
### 汇编代码分析

```assembly
0051ADFE  |.  FF75 DC       push dword ptr ss:[ebp-24]      ;  Pic4=ebp-24，即注册码第一部分  
0051AE01  |.  68 7CAE5100  push Down.0051AE7C               ;  -，“-”连接符号引入  
0051AE06  |.  FF75 F8      push dword ptr ss:[ebp-8]        ;  ebp-8=56D6，与固定值组合得到注册码第二部分  
0051AE09  |.  8D45 D8      lea eax,dword ptr ss:[ebp-28]  
0051AE0C  |.  50          push eax  
0051AE0D  |.  B9 05000000 mov ecx,5  
0051AE12  |.  BA 05000000 mov edx,5  
0051AE17  |.  8B45 F0      mov eax,dword ptr ss:[ebp-10]    ;  "Pic4ei8espr"=ebp-10  
0051AE1A  |.  E8 8DA1EEFF call Down.00404FAC  
0051AE1F  |.  FF75 D8      push dword ptr ss:[ebp-28]      ;  "ei8es"部分提出来  
0051AE22  |.  68 7CAE5100  push Down.0051AE7C               ;  -，“-”连接符号引入  
0051AE27  |.  FF75 F4      push dword ptr ss:[ebp-C]        ;  ebp-c=3796，即注册码第三部分  
0051AE2A  |.  8BC7         mov eax,edi  
0051AE2C  |.  BA 06000000  mov edx,6  
0051AE31  |.  E8 D69FEEFF  call Down.00404E0C  
0051AE36  |.  33C0         xor eax,eax  
0051AE38  |.  5A          pop edx  
0051AE39  |.  59          pop ecx  
0051AE3A  |.  59          pop ecx  
0051AE3B  |.  64:8910     mov dword ptr fs:[eax],edx  
0051AE3E  |.  68 58AE5100 push Down.0051AE58  
0051AE43  |>  8D45 D8     lea eax,dword ptr ss:[ebp-28]  
0051AE46  |.  BA 0A000000 mov edx,0A  
0051AE4B  |.  E8 609CEEFF call Down.00404AB0  
0051AE50  \\. C3          retn  
0051AE51    .^ E9 B695EEFF jmp Down.0040440C  
0051AE56    .^ EB EB       jmp short Down.0051AE43  
0051AE58    .  5F          pop edi  
0051AE59    .  5E          pop esi  
0051AE5A    .  5B          pop ebx  
0051AE5B    .  8BE5        mov esp,ebp  
0051AE5D    .  5D          pop ebp  
0051AE5E    .  C3          retn  
```

************************************************************************************************

### 算法小结

这里利用了软件第一次注册后信息保存的特点，通过这样进而查找注册码；

这里的注册码只与注册名及固定值`"Pic4ei8espr"`有关，即注册码=`Pic4`+`“-”`+`codeB`前四位及`ei8es`组合+`“-”`+`codeB`第5至8位；

`codeB`为注册名逐位转成十六进制`ascii`码并倒转得来，我这里是`tigerisme`，逐位字符转成十六进制得到`746967657269736D65`，再倒转得到`56D637962756769647`，即`codeB`；

### 特别说明

本文仅做学习使用，是一些破解的心得和思路，完全是个人对程序的研究，无其他目的。
![](img/45777b5dbd6b901752ab7df42fce6892.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
