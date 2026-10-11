
# 分析

:00494156 `E8A9F10100` call `004B3304`  

:0049415B `83C404` add `esp`, `00000004` 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`|:00494153(C)`  

`|`  

:0049415E `83BD40FFFFFF00` cmp `dword ptr [ebp+FFFFFF40]`, `00000000`  

:00494165 `0F8407000000` je `00494172` 

\> 跳则 OVER！ 

:0049416B `B801000000` mov `eax`, `00000001`  

:00494170 `EB02` jmp `00494174` 

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

`|:00493E14(C), :00493F09(C), :00494028(C), :00494165(C)`  

`|`  

:00494172 `33C0` xor `eax`, `eax` 

\> 跳到这里清 0 就 OVER 了！ 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

`|:00494170(U)`  

`|`  

:00494174 `85C0` test `eax`, `eax`  

:00494176 `0F84782C0000` je `00496DF4` 

…… ……省略…… ……  唉，很长一段代码都是保存注册信息的。有必要如此吗？:-( 

:00495C14 `E803D70100` call `004B331C` 

\> 呵呵，胜利女神！ 

:00497273 `E8A4C00100` call `004B331C` 

\> BAD BOY！ 

—————————————————————————————————   

进入关键 CALL：`00493D9F` call `004B331C` 取后 4 位字符 

\* Referenced by a CALL at Addresses:  

`|:0046873A   , :00468846   , :004688FB   , :0046891F   , :004689A6`  

…… …… 省 略 …… ……  如此多的地方来这里！ 

再进入：`004B331C` jmp `dword ptr [0000000C]`  这里面是易语言的代码了  ^O^  

再进入：`100257ED` call `dword ptr ds:[ebx]` 

```
10052730       56                 push esi                     ; KRNLN.100D6BD0  
10052731       8B7424 10          mov esi,dword ptr ss:[esp+10]  
10052735       57                 push edi  
10052736       83C9 FF            or ecx,FFFFFFFF  
10052739       8B16               mov edx,dword ptr ds:[esi]  
                          ====>EDX=328587 
1005273B       33C0               xor eax,eax  
1005273D       8BFA               mov edi,edx  
1005273F       F2:AE              repne scas byte ptr es:[edi]  
10052741       8B46 0C            mov eax,dword ptr ds:[esi+C]  
                          ====>EAX=[esi+C]=4           这个是“法定长度”了   
10052744       F7D1               not ecx  
10052746       49                 dec ecx  
                          ====>和 1005273F 处取 328587 长度  ECX=6 
10052747       3BC8               cmp ecx,eax  
                          ====>和 4 比较 
10052749       8BF1               mov esi,ecx  
1005274B       7C 02              jl short KRNLN.1005274F  
1005274D       8BF0               mov esi,eax  
1005274F       5F                 pop edi  
10052750       85F6               test esi,esi  
10052752       5E                 pop esi  
10052753       7D 16              jge short KRNLN.1005276B  
10052755       33C0               xor eax,eax  
10052757       2BD0               sub edx,eax  
10052759       03D1               add edx,ecx  
1005275B       52                 push edx  
1005275C       E8 6F97FFFF        call KRNLN.1004BED0  
10052761       8B4C24 08          mov ecx,dword ptr ss:[esp+8]  
10052765       83C4 04            add esp,4  
10052768       8901               mov dword ptr ds:[ecx],eax  
1005276A       C3                 retn 
1005276B       3BC8               cmp ecx,eax  
1005276D       7D 02              jge short KRNLN.10052771  
1005276F       8BC1               mov eax,ecx  
10052771       2BD0               sub edx,eax  
10052773       03D1               add edx,ecx  
                          ====>呵呵，这 2 步就简单的完成取后 4 位了 
10052775       52                 push edx  
                          ====>EDX=8587 
10052776       E8 5597FFFF        call KRNLN.1004BED0  
1005277B       8B4C24 08          mov ecx,dword ptr ss:[esp+8]  
1005277F       83C4 04            add esp,4  
10052782       8901               mov dword ptr ds:[ecx],eax  
10052784       C3                 retn 
```

其他几部分取后 4 位的步骤是相同的。 

—————————————————————————————————  

### 算法总结

1. 参考码：`561584633321`  分为 4 段：`A=561`  `B=584`  `C=633`  `D=321` 
2. 注册码 1=`A * B + D * 3=328587`  的后 4 为数字：`8587` 
3. 注册码 2=`B * D + C * 4=189996`  的后 4 为数字：`9996` 
4. 注册码 3=`D * C + B * 2=204361`  的后 4 为数字：`4361` 
5. 注册码 4=`C * A + A * 5=357918`  的后 4 为数字：`7918` 

  得出注册码：`8587-9996-4361-7918` 

注：乘以 3、4、2、5 这几个数我不清楚是否是固定值，哪位朋友看了的话麻烦告之。 

—————————————————————————————————   

### KeyMake 之{110th}内存注册机

中断地址：`00493DD6`  

中断次数：`1`  

第一字节：`50`  

指令长度：`1`  

内存方式：`EAX`   结尾插入 - 

中断地址：`00493ECB`  

中断次数：`1`  

第一字节：`50`  

指令长度：`1`  

内存方式：`EAX`   结尾插入 - 

中断地址：`00493FDB`  

中断次数：`1`  

第一字节：`50`  

指令长度：`1`  

内存方式：`EAX`   结尾插入 - 

中断地址：`00494118`  

中断次数：`1`  

第一字节：`50`  

指令长度：`1` 

内存方式：`EAX` 

修改内存：  

1. `00493DEA 0F94C0` 改为：`B00190`  
2. `00493EDF 0F94C0` 改为：`B00190`   
3. `00493FF2 0F94C0` 改为：`B00190`   

—————————————————————————————————   

### 注册信息保存

呵呵，算是服了他了，分开几处保存注册标志。 ^O^  ^O^ 

1. `[HKEY_CURRENT_USER\Software\STREG\cankaoma]`  

   `"Fvbhryh-jk"="inwWBoUgJJ"` 

2. `[HKEY_CURRENT_USER\Software\Afghfg-76\cankaoma]`  

   `"Fvbhryh-jk"="jYx7SifwkB"` 

3. `[HKEY_CURRENT_USER\Software\Homecctv\cankaoma]`  

   `"Fvbhryh-jk"="EluvrWjFLj"` 

4. `[HKEY_CURRENT_USER\Software\086KGJFt-56\cankaoma]`  

   `"Fvbhryh-jk"="HYhiJzFMt"` 

—————————————————————————————————   

### 整理

参考码：`561584633321`  

注册码：`8587-9996-4361-7918` 

—————————————————————————————————  

        ,     _/   

       /| \_.-~/           \\\_     ,       青春都一饷  
       ( /~   /             \\~.\_ |\  
       \`\\\\  _/               \\   ~\\ )          忍把浮名   
       _-~~~-.)  )_\_/;;,.           \\\_  //'  
       '/\_,\   --~   \\ ~~~-  ,;;\_\_\_( (.-~~~-.        换了破解轻狂  
       \`~ _ ( ,\_.--\\ (     ,;'' /    ~--   /.\_\`\\   
       /~~//'   /' \`~\          ) /--..\_, )\_  \`~  
       "  \`~"  "       \`"       /~'\`\\   \`\\\\~~\   



                   "       "   "~'  "" 

                   Cracked By 巢水工作坊——fly `[OCN][FCG]` 

`4dd466d0e1e0cc123afbe504da2462ca`
