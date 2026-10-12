
```
0040FE38    .  897E F4             mov dword ptr ds:[esi-C],edi   ；esi-C=001EA2F4，0=>1,保存计数器  

0040FE3B    .  C60437 00           mov byte ptr ds:[edi+esi],0    ；edi 为注册码长度+1，置字符串结尾符\0,esi 为注册码首地址  

0040FE3F    .^ 72 97              jb short AlertSpy.0040FDD8     ；往上跳  

END LOOP  
```

循环结束后：  

`EAX` `001EA300` `ASCII` `"7GFEGIF67WNAN5VW"`  ==\>>>>>`WooooWaaaaaa`,这就是注册码了  

```
0040FE41    .  8B4424 14           mov eax,dword ptr ss:[esp+14]  

0040FE45    .  50                 push eax  

0040FE46    .  E8 C5EB0200         call AlertSpy.0043EA10        ；F8 跳过再说  

0040FE4B    .  8B4E F0             mov ecx,dword ptr ds:[esi-10]  

0040FE4E    .  8B11               mov edx,dword ptr ds:[ecx]  

0040FE50    .  8D5E F0             lea ebx,dword ptr ds:[esi-10]  

0040FE53    .  83C4 04             add esp,4  

0040FE56    .  FF52 10             call dword ptr ds:[edx+10]    ；F8 跳过再说  

0040FE59    .  8B53 0C             mov edx,dword ptr ds:[ebx+C]  

0040FE5C    .  85D2               test edx,edx  

0040FE5E    .  8D4B 0C             lea ecx,dword ptr ds:[ebx+C]  

0040FE61    .  7C 1B               jl short AlertSpy.0040FE7E  

0040FE63    .  3B03               cmp eax,dword ptr ds:[ebx]  

0040FE65    .  75 17               jnz short AlertSpy.0040FE7E  

0040FE67    .  8BC3               mov eax,ebx  

0040FE69    .  BA 01000000         mov edx,1  

0040FE6E    .  F0:0FC111          lock xadd dword ptr ds:[ecx],edx  

0040FE72    .  EB 3D               jmp short AlertSpy.0040FEB1  

0040FE74    >  68 57000780         push 80070057  

0040FE79    .  E8 5218FFFF         call AlertSpy.004016D0  

0040FE7E    >  8B4B 04             mov ecx,dword ptr ds:[ebx+4]  

0040FE81    .  8B10               mov edx,dword ptr ds:[eax]  

0040FE83    .  6A 01               push 1  

0040FE85    .  51                 push ecx  

0040FE86    .  8BC8               mov ecx,eax  

0040FE88    .  FF12               call dword ptr ds:[edx]  

0040FE8A    .  85C0               test eax,eax  

0040FE8C    .  75 05               jnz short AlertSpy.0040FE93  

0040FE8E    .^ E9 0D19FFFF         jmp AlertSpy.004017A0  

0040FE93    >  8B53 04             mov edx,dword ptr ds:[ebx+4]  

0040FE96    .  8950 04             mov dword ptr ds:[eax+4],edx  

0040FE99    .  8B4B 04             mov ecx,dword ptr ds:[ebx+4]  

0040FE9C    .  41                 inc ecx  

0040FE9D    .  8BD1               mov edx,ecx  

0040FE9F    .  C1E9 02             shr ecx,2  

0040FEA2    .  8D73 10             lea esi,dword ptr ds:[ebx+10]  

0040FEA5    .  8D78 10             lea edi,dword ptr ds:[eax+10]  

0040FEA8    .  F3:A5               rep movs dword ptr es:[edi],dword ptr ds:[esi]  

0040FEAA    .  8BCA               mov ecx,edx  

0040FEAC    .  83E1 03             and ecx,3  

0040FEAF    .  F3:A4               rep movs byte ptr es:[edi],byte ptr ds:[esi]  

0040FEB1    >  8BB424 98000000     mov esi,dword ptr ss:[esp+98]  

0040FEB8    .  83C0 10             add eax,10  

0040FEBB    .  8906                mov dword ptr ds:[esi],eax  ；EAX 001EA300 ASCII "7GFEGIF67WNAN5VW"  ||  ESI 0012EDC0  

0040FEBD    .  C78424 90000000 FF>mov dword ptr ss:[esp+90],-1  

0040FEC8    .  8D43 0C             lea eax,dword ptr ds:[ebx+C]  

0040FECB    .  83C9 FF             or ecx,FFFFFFFF  

0040FECE    .  F0:0FC108           lock xadd dword ptr ds:[eax],ecx  

0040FED2    .  49                  dec ecx  

0040FED3    .  85C9               test ecx,ecx  

0040FED5    .  7F 08               jg short AlertSpy.0040FEDF  

0040FED7    .  8B0B               mov ecx,dword ptr ds:[ebx]  

0040FED9    .  8B11               mov edx,dword ptr ds:[ecx]  

0040FEDB    .  53                 push ebx  

0040FEDC    .  FF52 04            call dword ptr ds:[edx+4]  

0040FEDF    >  8B8C24 88000000     mov ecx,dword ptr ss:[esp+88]  

0040FEE6    .  5F                 pop edi  

0040FEE7    .  8BC6               mov eax,esi  

0040FEE9    .  5E                 pop esi  

0040FEEA    .  5D                 pop ebp  

0040FEEB    .  64:890D 00000000   mov dword ptr fs:[0],ecx  

0040FEF2    .  8B4C24 78           mov ecx,dword ptr ss:[esp+78]  

0040FEF6    .  5B                 pop ebx  

0040FEF7    .  E8 5FA60100         call AlertSpy.0042A55B  

0040FEFC    .  81C4 84000000       add esp,84  

0040FF02    .  C3                 retn    ；回到 00410A3F  见上面的代码  
```

===============================================================================================

【算法总结】 输入长度不为 0 的用户名，然后把用户名的每个字符 `XOR` `0x83`，然后采用标准 `MD5` 算法来 `XOR`，得到 `HASH` 值，  
每次取一个 `byte`，进行运算，然后查表`"ABCDEFGHIJKLMNPQRSTUVWXYZ2345679"`,`16` 轮循环完毕，便得到长为 `16` 的注册码。  
注册成功以后，注册码保存在注册表，`HKEY_LOCAL_MACHINE\SOFTWARE\Mandel Enterprises\AlertSpy`，`Registration Name`，  
`Registration Key` 下。  
注册机有 `WIN32ASM` 写的，采用 `Canterwood` 的模版，感谢！第一次用汇编写注册机，感觉真好。  
注册机还有 `VB6` 写的，`VC6` 写的，`Delphi 7` 版的。  
一点小说明，以前除 `VB6` 写的注册机对用户名长度没有小于 `64` 的限制外，其他都有限制，今天写文章再次调试纠正了这个错误，  
注册机里的错误我就不改了。

【Greetings】  看雪论坛,`FCG` 论坛,`DFCG` 论坛等  
注册机源码中的 `MD5` 算法以及 `Flat Button` 控件是从网上下载而来，稍加修改，感谢原作者！  
`VC` 版的注册机借用看雪加密解密 `2` 里面的例子，感谢！

【完稿时间等】`2005.04.29`，凌晨 `1:39`,天气:阴,广州

附件:`AlertSpyV1.08_WIN32ASM.rar`

附件:`KGAlertSpy_VB6.rar` 

附件:`KG_AlertSpy_Delphi 7.rar`

附件:`KG_Alertspy_VC6.rar`

`4dd466d0e1e0cc123afbe504da2462ca`
