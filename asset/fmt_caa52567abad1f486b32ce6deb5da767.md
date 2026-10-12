
```
00575703    E8 D0000000     call ETDirLoc.005757D8  

00575708    8B55 F0         mov edx,dword ptr ss:[ebp-10]  

0057570B    B8 68EC5900     mov eax,ETDirLoc.0059EC68  

00575710    E8 B3F2F8FF     call ETDirLoc.005049C8  

00575715    8BC3            mov eax,ebx  

00575717    8B15 68EC5900   mov edx,dword ptr ds:[59EC68]  

0057571D    E8 A6F2F8FF     call ETDirLoc.005049C8  

00575722    33C0            xor eax,eax  

00575724    5A              pop edx  

00575725    59              pop ecx  

00575726    59              pop ecx  

00575727    64:8910         mov dword ptr fs:[eax],edx  

0057572A    68 44575700     push ETDirLoc.00575744  

0057572F    8D45 EC         lea eax,dword ptr ss:[ebp-14]  

00575732    BA 05000000     mov edx,5  

00575737    E8 5CF2F8FF     call ETDirLoc.00504998  

0057573C    C3              retn  

0057573D  ^ E9 D2EBF8FF     jmp ETDirLoc.00504314  

00575742  ^ EB EB          jmp short ETDirLoc.0057572F  

00575744    5B              pop ebx  

00575745    8BE5            mov esp,ebp  

00575747    5D              pop ebp  

00575748    C3              retn  
```

`***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***`

### 算法 call（3）

```
005757D8    55              push ebp  

005757D9    8BEC            mov ebp,esp  

005757DB    83C4 EC         add esp,-14  

005757DE    53              push ebx  

005757DF    56              push esi  

005757E0    57              push edi  

005757E1    33DB            xor ebx,ebx  

005757E3    895D EC         mov dword ptr ss:[ebp-14],ebx  

005757E6    895D F0         mov dword ptr ss:[ebp-10],ebx  

005757E9    894D F4         mov dword ptr ss:[ebp-C],ecx  

005757EC    8955 F8         mov dword ptr ss:[ebp-8],edx  

005757EF    8945 FC         mov dword ptr ss:[ebp-4],eax  

005757F2    33C0            xor eax,eax  

005757F4    55              push ebp  

005757F5    68 8A585700     push ETDirLoc.0057588A  

005757FA    64:FF30         push dword ptr fs:[eax]  

005757FD    64:8920         mov dword ptr fs:[eax],esp  

00575800    8B45 FC         mov eax,dword ptr ss:[ebp-4]  

00575803    E8 24F4F8FF     call ETDirLoc.00504C2C                  ; WD-WCAD19885326=eax  

00575808    8BF8            mov edi,eax  

0057580A    85FF            test edi,edi  

0057580C    7E 61           jle short ETDirLoc.0057586F  

0057580E    BE 01000000     mov esi,1  

00575813    8B45 FC         mov eax,dword ptr ss:[ebp-4]         ; WD-WCAD19885326=ebp-4 送 eax  

00575816    0FB64430 FF     movzx eax,byte ptr ds:[eax+esi-1]      ; 逐位取“WD-WCAD19885326”，送 eax  

0057581B    99              cdq  

0057581C    F77D F8         idiv dword ptr ss:[ebp-8]             ; WD-WCAD19885326 逐位取，每位 ascii 码除 12，结果整数放 eax，余数放 edx  

0057581F    8BDA            mov ebx,edx                               ; 余数放 edx 并送 ebx  

00575821    33DE            xor ebx,esi                               ; 余数逐位与 1，2，3……15 进行 xor 运算，结果记和起来就是机器码  

00575823    837D F4 01      cmp dword ptr ss:[ebp-C],1  

00575827    75 2A           jnz short ETDirLoc.00575853  

00575829    8BC3            mov eax,ebx  

0057582B    B9 24000000     mov ecx,24  

00575830    99              cdq  

00575831    F7F9            idiv ecx  

00575833    8BDA            mov ebx,edx  

00575835    8D45 F0         lea eax,dword ptr ss:[ebp-10]  

00575838    8A93 90C75900   mov dl,byte ptr ds:[ebx+59C790]  

0057583E    E8 11F3F8FF     call ETDirLoc.00504B54  

00575843    8B55 F0         mov edx,dword ptr ss:[ebp-10]  

00575846    8B45 08         mov eax,dword ptr ss:[ebp+8]  

00575849    E8 E6F3F8FF     call ETDirLoc.00504C34  

0057584E    8B45 08         mov eax,dword ptr ss:[ebp+8]  

00575851    EB 18           jmp short ETDirLoc.0057586B  

00575853    8D55 EC         lea edx,dword ptr ss:[ebp-14]  

00575856    8BC3            mov eax,ebx                               ; E 送 eax  

00575858    E8 DB38F9FF     call ETDirLoc.00509138  

0057585D    8B55 EC         mov edx,dword ptr ss:[ebp-14]             ; 由前面的 call 将 codeA 逐个由 16 位转成十进制，即 14121011813951089292015  

00575860    8B45 08         mov eax,dword ptr ss:[ebp+8]  

00575863    E8 CCF3F8FF     call ETDirLoc.00504C34  

00575868    8B45 08         mov eax,dword ptr ss:[ebp+8]  

0057586B    46              inc esi  

0057586C    4F              dec edi  

0057586D  ^ 75 A4          jnz short ETDirLoc.00575813  

0057586F    33C0            xor eax,eax  

00575871    5A              pop edx  

00575872    59              pop ecx  

00575873    59              pop ecx  

00575874    64:8910         mov dword ptr fs:[eax],edx  

00575877    68 91585700     push ETDirLoc.00575891  

0057587C    8D45 EC         lea eax,dword ptr ss:[ebp-14]  

0057587F    BA 02000000     mov edx,2  

00575884    E8 0FF1F8FF     call ETDirLoc.00504998  

00575889    C3              retn  
```

`***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***` `***`

算法思路就是硬盘固定码`WD-WCAD19885326`经过一定运算得到机器码`14121011813951089292015`，之后与用户名经过运算得到注册码“`O8L70-8C1IO-YFV08-6YK9S`”，这里没有做详细分析，软件主要的加密特点是通过对`dll`文件进行注册加密，调试主程序时须调用`ETDirLock.dll`。希望对刚入门的朋友有帮助!

`4dd466d0e1e0cc123afbe504da2462ca`
