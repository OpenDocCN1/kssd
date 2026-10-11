
# 黑客字典 II

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-167.htm`](https://www.pediy.com/kssd/pediy07/pediy7-167.htm)

## 破解信息

【破解作者】 落魄浪子
【作者邮箱】 `zxy223_szb@21cn.net`
【使用工具】 `FlyOD1.1`
【破解平台】 `Win9x/NT/2000/XP`
【软件名称】 `黑客字典 II`
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------------------------

## 破解内容

今天由于要用字典，就把过去的光盘拿出来找到了这个软件，它的注册机也可能满天飞了，呵呵。

程序在注册不正确时会弹出错误提示“注册名或注册码不正确!”。用 `OD` 载入程序，`F9` 运行，查找出错字符串，找到后下断，并看看是从哪里跳来的。找到关键点下断。

点注册，输入注册名：`Crack`，注册码输入：`987654321`.

```
00401BF5    .   E8 1CE40100 call UltraDic.00420016
00401BFA    .   8D7C24 20   lea edi,dword ptr ss:[esp+20]    注册名入 EDI
00401BFE    .   83C9 FF     or ecx,FFFFFFFF
00401C01    .   33C0        xor eax,eax
00401C03    .   F2:AE       repne scas byte ptr es:[edi]     注册名是否为空
00401C05    .   F7D1        not ecx
00401C07    .   49          dec ecx
00401C08    .   0F84 160200>je UltraDic.00401E24             为空则按钮无反应，不响应任何事件
00401C0E    .   8DBC24 8400>lea edi,dword ptr ss:[esp+84]   注册码入 EDI
00401C15    .   83C9 FF     or ecx,FFFFFFFF
00401C18    .   F2:AE       repne scas byte ptr es:[edi]    注册码是否为空
00401C1A    .   F7D1        not ecx
00401C1C    .   49          dec ecx
00401C1D    .   0F84 010200>je UltraDic.00401E24            为空则按钮无反应，不响应任何事件
00401C23    .   8D7C24 20   lea edi,dword ptr ss:[esp+20]   注册名入 EDI
00401C27    .   83C9 FF     or ecx,FFFFFFFF
00401C2A    .   33F6        xor esi,esi
00401C2C    .   F2:AE       repne scas byte ptr es:[edi]
00401C2E    .   F7D1        not ecx
00401C30    .   49          dec ecx
00401C31    .   74 79       je short UltraDic.00401CAC   再次检查
00401C33    >   0FBE7C34 20 movsx edi,byte ptr ss:[esp+esi+20]  取注册名的第一位 C 的 HEX---43
00401C38    .   8BC7        mov eax,edi   43 入 EAX
00401C3A    .   B9 0A000000 mov ecx,0A    A 入 ECX
00401C3F    .   99          cdq           EDX 清空
00401C40    .   F7F9        idiv ecx      43 除 A  商 AL 等于 6，余数 DL 等于 7
00401C42    .   8BCA        mov ecx,edx   余数 7 入 ECX
00401C44    .   81E2 010000>and edx,80000001
00401C4A    .   79 05       jns short UltraDic.00401C51
00401C4C    .   4A          dec edx
00401C4D    .   83CA FE     or edx,FFFFFFFE
00401C50    .   42          inc edx
00401C51    >   75 16       jnz short UltraDic.00401C69
00401C53    .   8BC7        mov eax,edi
00401C55    .   B9 1A000000 mov ecx,1A
00401C5A    .   99          cdq
00401C5B    .   F7F9        idiv ecx
00401C5D    .   80C2 41     add dl,41
00401C60    .   889434 4801>mov byte ptr ss:[esp+esi+148],dl
00401C67    .   EB 2E       jmp short UltraDic.00401C97
00401C69    >   8BC1        mov eax,ecx                   余数 7 入 ECX
00401C6B    .   BB 03000000 mov ebx,3
00401C70    .   99          cdq
00401C71    .   F7FB        idiv ebx   余数 7 除 3 ，商 AL 等于 2，余数 DL 等于 1
00401C73    .   85D2        test edx,edx
00401C75    .   75 16       jnz short UltraDic.00401C8D  能够被整除这里则不跳
00401C77    .   8BC7        mov eax,edi
00401C79    .   B9 1A000000 mov ecx,1A
00401C7E    .   99          cdq
00401C7F    .   F7F9        idiv ecx    上面不跳则把这位注册码的十进六制除 1A
00401C81    .   80C2 61     add dl,61   余数加 61
00401C84    .   889434 4801>mov byte ptr ss:[esp+esi+148],dl
00401C8B    .   EB 0A       jmp short UltraDic.00401C97
00401C8D    >   80C1 31     add cl,31                   余数 7 加 31 等于 38
00401C90    .   888C34 4801>mov byte ptr ss:[esp+esi+148],cl
00401C97    >   8D7C24 20   lea edi,dword ptr ss:[esp+20]    注册名入 EDI
00401C9B    .   83C9 FF     or ecx,FFFFFFFF
00401C9E    .   33C0        xor eax,eax
00401CA0    .   46          inc esi
00401CA1    .   F2:AE       repne scas byte ptr es:[edi]
00401CA3    .   F7D1        not ecx
00401CA5    .   49          dec ecx
00401CA6    .   3BF1        cmp esi,ecx
00401CA8    . ^ 72 89       jb short UltraDic.00401C33    比较是否已计算完，没有则继续
00401CAA    .   33DB        xor ebx,ebx
00401CAC    >   55          push ebp
00401CAD    .   8D8C24 EC00>lea ecx,dword ptr ss:[esp+EC]
00401CB4    .   889C34 4C01>mov byte ptr ss:[esp+esi+14C],bl
00401CBB    .   E8 F0010000 call UltraDic.00401EB0
00401CC0    .   899C24 B401>mov dword ptr ss:[esp+1B4],ebx
00401CC7    .   8DB424 8400>lea esi,dword ptr ss:[esp+84]   注册码入 ESI
00401CCE    .   8D8424 4801>lea eax,dword ptr ss:[esp+148]  用注册名计算出的注册码，这里可以做内存注册机
00401CD5    >   8A10        mov dl,byte ptr ds:[eax]   真注册码的一位入 DL
00401CD7    .   8ACA        mov cl,dl
00401CD9    .   3A16        cmp dl,byte ptr ds:[esi] 与假注册码作比较，不对则出错
00401CDB    .   75 1C       jnz short UltraDic.00401CF9
00401CDD    .   3ACB        cmp cl,bl
00401CDF    .   74 14       je short UltraDic.00401CF5
00401CE1    .   8A50 01     mov dl,byte ptr ds:[eax+1] 真注册码第二位入 DL
00401CE4    .   8ACA        mov cl,dl                   比较真注册码的第二位是否等于第一位
00401CE6    .   3A56 01     cmp dl,byte ptr ds:[esi+1]
00401CE9    .   75 0E       jnz short UltraDic.00401CF9
00401CEB    .   83C0 02     add eax,2
00401CEE    .   83C6 02     add esi,2
00401CF1    .   3ACB        cmp cl,bl
00401CF3    . ^ 75 E0       jnz short UltraDic.00401CD5  是否比较完毕，没有则继续。
00401CF5    >   33C0        xor eax,eax
00401CF7    .   EB 05       jmp short UltraDic.00401CFE
00401CF9    >   1BC0        sbb eax,eax
00401CFB    .   83D8 FF     sbb eax,-1
00401CFE    >   3BC3        cmp eax,ebx
00401D00    .   0F85 DF0000>jnz UltraDic.00401DE5    注册码正确则不跳并写入 C:\regbanyet.dat 文件中。
00401D06    .   68 01100000 push 1001
00401D0B    .   68 F4704300 push UltraDic.004370F4                  ;  ASCII "C:\regbanyet.dat"
00401D10    .   8D4C24 18   lea ecx,dword ptr ss:[esp+18]
00401D14    .   E8 8FEF0100 call UltraDic.00420CA8
00401D19    .   8D7C24 20   lea edi,dword ptr ss:[esp+20]
00401D1D    .   83C9 FF     or ecx,FFFFFFFF
00401D20    .   33C0        xor eax,eax
00401D22    .   B2 0D       mov dl,0D
00401D24    .   F2:AE       repne scas byte ptr es:[edi]
00401D26    .   F7D1        not ecx
00401D28    .   49          dec ecx
00401D29    .   8DBC24 8400>lea edi,dword ptr ss:[esp+84]   注册码
00401D30    .   C68424 B401>mov byte ptr ss:[esp+1B4],1
00401D38    .   88540C 20   mov byte ptr ss:[esp+ecx+20],dl
00401D3C    .   41          inc ecx
00401D3D    .   C6440C 20 0>mov byte ptr ss:[esp+ecx+20],0A
00401D42    .   885C0C 21   mov byte ptr ss:[esp+ecx+21],bl
00401D46    .   83C9 FF     or ecx,FFFFFFFF
00401D49    .   F2:AE       repne scas byte ptr es:[edi]
00401D4B    .   F7D1        not ecx
00401D4D    .   49          dec ecx
00401D4E    .   8D7C24 20   lea edi,dword ptr ss:[esp+20]
00401D52    .   88940C 8400>mov byte ptr ss:[esp+ecx+84],dl
00401D59    .   41          inc ecx
00401D5A    .   C6840C 8400>mov byte ptr ss:[esp+ecx+84],0A
00401D62    .   889C0C 8500>mov byte ptr ss:[esp+ecx+85],bl
00401D69    .   83C9 FF     or ecx,FFFFFFFF
00401D6C    .   F2:AE       repne scas byte ptr es:[edi]
00401D6E    .   F7D1        not ecx
00401D70    .   49          dec ecx
00401D71    .   8D4424 20   lea eax,dword ptr ss:[esp+20]
00401D75    .   51          push ecx                              ; /Arg2
00401D76    .   50          push eax                              ; |Arg1
00401D77    .   8D4C24 18   lea ecx,dword ptr ss:[esp+18]         ; |
00401D7B    .   E8 DCF10100 call UltraDic.00420F5C                ; \UltraDic.00420F5C
00401D80    .   8DBC24 8400>lea edi,dword ptr ss:[esp+84]
00401D87    .   83C9 FF     or ecx,FFFFFFFF
00401D8A    .   33C0        xor eax,eax
00401D8C    .   F2:AE       repne scas byte ptr es:[edi]
00401D8E    .   F7D1        not ecx
00401D90    .   49          dec ecx
00401D91    .   51          push ecx                              ; /Arg2
```



```
00401D92    .  8D8C24 8800>lea ecx,dword ptr ss:\[esp+88\]               ; |  

00401D99    .  51          push ecx                                       ; |Arg1  

00401D9A    .  8D4C24 18   lea ecx,dword ptr ss:\[esp+18\]               ; |  

00401D9E    .  E8 B9F10100 call UltraDic.00420F5C                         ; \\UltraDic.00420F5C  

00401DA3    .  6A 03       push 3                                         ; /FileAttributes = READONLY|HIDDEN  

00401DA5    .  68 F4704300 push UltraDic.004370F4                         ; |FileName = "C:\\regbanyet.dat"  

00401DAA    .  FF15 8CB242>call dword ptr ds:\[<&KERNEL32.SetFileAttrib>; \\SetFileAttributesA  

00401DB0    .  68 D8704300 push UltraDic.004370D8                         ;  非常感谢您的支持,注册成功!  

00401DB5    .  8D8C24 EC00>lea ecx,dword ptr ss:\[esp+EC\]  

00401DBC    .  E8 AF010000 call UltraDic.00401F70  

00401DC1    .  8D4C24 10   lea ecx,dword ptr ss:\[esp+10\]  

00401DC5    .  889C24 B401>mov byte ptr ss:\[esp+1B4\],bl  

00401DCC    .  E8 77EF0100 call UltraDic.00420D48  

00401DD1    .  8BCD        mov ecx,ebp  

00401DD3    .  E8 44B50100 call UltraDic.0041D31C  

00401DD8    .  C78424 B401>mov dword ptr ss:\[esp+1B4\],3  

00401DE3    .  EB 1C       jmp short UltraDic.00401E01  

00401DE5    >  68 C0704300 push UltraDic.004370C0                         ;  注册名或注册码不正确!  
```

### 破解总结

这个软件的注册算法很简单，就是取注册名的每一位的十六进制与`A`相除，余数与`3`相除，如果余数为`0`，也就是能整除则与`1A`相除，余数加`61`。不能整除则余数加`31`为注册码。

注册名：`落魄浪子`

注册码：`7?;\*5\`0,`

---

### 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
