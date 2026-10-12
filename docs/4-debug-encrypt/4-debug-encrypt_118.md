
# A 变速器算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-180.htm`](https://www.pediy.com/kssd/pediy09/pediy09-180.htm)

> 【破文作者】风间仁  
> 【作者邮箱】fenjianzhun@gmail.com  
> 【破解工具】OD  
> 【软件名称】A 变速器 V2006.9  
> ------------------------------------------------------------------------  
> 【破解过程】

### 一、查壳

`UPX` 壳，`UPX ShellEx` 脱之

机器码：`3C036EA7`

注册码：`7ubG16kAb$CySLN5ESps0$Gv9Zl1797Q9lnkGvEZc8OYbHX$8ASaMC8UcFhIJIxy`

这是一组可用注册信息，调试的时候可以在 `004028B1` 处将你本机的机器码改为上面的机器码

### 二、算法分析

```
004028AC    E8 17FEFFFF       call aspeeder.004026C8                   ; 计算机器码
004028B1    A1 E82C5C00       mov eax,dword ptr ds:[5C2CE8]           
004028B6    85C0              test eax,eax
004028B8    74 05             je short aspeeder.004028BF
004028BA    A3 E42C5C00       mov dword ptr ds:[5C2CE4],eax           ;机器码存于此处
```

#### 关键部分

```
00404631    B8 1B115300       mov eax,aspeeder.0053111B               ; ASCII "D0330A59"
00404636    E8 F9A50000       call aspeeder.0040EC34
0040463B    8D55 94           lea edx,dword ptr ss:[ebp-6C]
0040463E    B8 24115300       mov eax,aspeeder.00531124               ; ASCII "A7456C12309EAF6BEF610A5B1F408D62B4AF7775E167656C236BC3B8D77F587E92D80DB14AC83281"
00404643    E8 ECA50000       call aspeeder.0040EC34
00404648    8B15 E42C5C00     mov edx,dword ptr ds:[5C2CE4]           ; 机器码 3C036EA7
0040464E    8D8D 84F7FFFF     lea ecx,dword ptr ss:[ebp-87C]
00404654    52                push edx
00404655    68 75115300       push aspeeder.00531175                 ; ASCII "%08X"
0040465A    51                push ecx
0040465B    E8 18C11000       call aspeeder.00510778
00404660    83C4 0C           add esp,0C
00404663    8D55 84           lea edx,dword ptr ss:[ebp-7C]
00404666    8D85 84F7FFFF     lea eax,dword ptr ss:[ebp-87C]          ; "3C036EA7"
0040466C    E8 A7A70000       call aspeeder.0040EE18
```

; 经过 `fun1`(解密函数)得 `"0CC00318E287"`

```
00404671    8D4D 8C           lea ecx,dword ptr ss:[ebp-74]
00404674    51                push ecx
00404675    8B4D 94           mov ecx,dword ptr ss:[ebp-6C]
00404678    8B55 90           mov edx,dword ptr ss:[ebp-70]
0040467B    8B45 84           mov eax,dword ptr ss:[ebp-7C]
0040467E    E8 2D9D0000       call aspeeder.0040E3B0
```

; `N＝A7456C12309EAF6BEF610A5B1F408D62B4AF7775E167656C236BC3B8D77F587E92D80DB14AC83281`  
; `E＝D0330A59`  
; `C＝0CC00318E287`

; RSA 解密得

`5531FB43A5AE9C6DB8246A0D4D07C2CA050FA1CE97BD05C5E0CCB20635AC8D581B9EAC785458278B`

```
00404683    8D85 84F7FFFF     lea eax,dword ptr ss:[ebp-87C]
00404689    8B55 8C           mov edx,dword ptr ss:[ebp-74]
0040468C    E8 2FA80000       call aspeeder.0040EEC0
```

; 经过 `fun2`(加密函数)得

`1LCVj3fQwSRRWaQWrD1yBA1G$XpfUz1SNWpB86DQoDM1kUh7XKM2UB`

```
00404691    68 7A115300       push aspeeder.0053117A                 ; ASCII "ASPEEDER"
00404696    8D8D 84F7FFFF     lea ecx,dword ptr ss:[ebp-87C]
0040469C    51                push ecx
0040469D    E8 82881000       call aspeeder.0050CF24                 ; 前者连接在后者
004046A2    83C4 08           add esp,8
004046A5    8D55 8C           lea edx,dword ptr ss:[ebp-74]
004046A8    8D85 84F7FFFF     lea eax,dword ptr ss:[ebp-87C]
```

; `1LCVj3fQwSRRWaQWrD1yBA1G$XpfUz1SNWpB86DQoDM1kUh7XKM2UBASPEEDER`

```
004046AE    E8 65A70000       call aspeeder.0040EE18
```

; `fun1`(解密函数),结果存于`[ebp-74]`,关键比较值之 1

`05531FB43A5AE9C6DB8246A0D4D07C2CA050FA1CE97BD05C5E0CCB20635AC8D581B9EAC785458278B29C64E38D39B`

```
004046B3    8D55 94           lea edx,dword ptr ss:[ebp-6C]
004046B6    B8 242D5400       mov eax,aspeeder.00542D24              ; ASCII "C985F97A3C4E0D3BF7D35DC4148E5C47749076D668CA8464A6D2CCFB1B26183623315E5450610784D974ED5E9702A451"
004046BB    E8 74A50000       call aspeeder.0040EC34
004046C0    8D55 84           lea edx,dword ptr ss:[ebp-7C]
004046C3    B8 EC2C5C00       mov eax,aspeeder.005C2CEC              
```

; ASCII `"7ubG16kAb$CySLN5ESps0$Gv9Zl1797Q9lnkGvEZc8OYbHX$8ASaMC8UcFhIJIxy"`

```
004046C8    E8 4BA70000       call aspeeder.0040EE18
```

; 经过 `fun1`(解密函数)得

`1F8950046B8A97E33C7155C539CCF603E439263BC11C91DA26FC6E4393A398862295187E20A72458C21E98FAD24D2EFC`

```
004046CD    8D4D 88           lea ecx,dword ptr ss:[ebp-78]
004046D0    51                push ecx
004046D1    8B4D 94           mov ecx,dword ptr ss:[ebp-6C]
004046D4    8B55 90           mov edx,dword ptr ss:[ebp-70]
004046D7    8B45 84           mov eax,dword ptr ss:[ebp-7C]
004046DA    E8 D19C0000       call aspeeder.0040E3B0
```

; `N＝C985F97A3C4E0D3BF7D35DC4148E5C47749076D668CA8464A6D2CCFB1B26183623315E5450610784D974ED5E9702A451`  
; `E＝D0330A59`  
; `C＝1F8950046B8A97E33C7155C539CCF603E439263BC11C91DA26FC6E4393A398862295187E20A72458C21E98FAD24D2EFC`

; RSA 解密得

`5531FB43A5AE9C6DB8246A0D4D07C2CA050FA1CE97BD05C5E0CCB20635AC8D581B9EAC785458278B29C64E38D39B`

存于`[ebp-78]`,关键比较值之 2

```
004046DF    8D85 84F7FFFF     lea eax,dword ptr ss:[ebp-87C]
004046E5    8B55 88           mov edx,dword ptr ss:[ebp-78]
004046E8    E8 D3A70000       call aspeeder.0040EEC0
004046ED    8B55 88           mov edx,dword ptr ss:[ebp-78]
004046F0    8B45 8C           mov eax,dword ptr ss:[ebp-74]
004046F3    E8 48870000       call aspeeder.0040CE40                 ;两个关键值比较
004046F8    85C0              test eax,eax
004046FA    8D45 94           lea eax,dword ptr ss:[ebp-6C]
004046FD    0F94C3            sete bl
00404700    83E3 01           and ebx,1
00404703    E8 08850000       call aspeeder.0040CC10
00404708    8D45 90           lea eax,dword ptr ss:[ebp-70]
0040470B    E8 00850000       call aspeeder.0040CC10
00404710    8D45 8C           lea eax,dword ptr ss:[ebp-74]
00404713    E8 F8840000       call aspeeder.0040CC10
00404718    8D45 88           lea eax,dword ptr ss:[ebp-78]
0040471B    E8 F0840000       call aspeeder.0040CC10
00404720    8D45 84           lea eax,dword ptr ss:[ebp-7C]
00404723    E8 E8840000       call aspeeder.0040CC10
00404728    84DB              test bl,bl
0040472A    74 27             je short aspeeder.00404753             ;关键跳转，不跳走则注册成功
```

程序中共有 3 处比较，修改即可爆破

```
0040472A   /74 27            je short aspeeder.00404753 //nop
00404B66   /75 3F            jnz short aspeeder.00404BA7 //jmp
004071B5   /74 3C            je short aspeeder.004071F3  //nop
```

#### `fun1` 与 `fun2` 两个函数用 C 表示成

```c
unsigned char Encode64[]="0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz$#";
unsigned char Tab[]="0123456789ABCDEF";

unsigned char search(unsigned char ch,unsigned char dst[])
{ /*从 dst 中查找 ch 字符，若未找到，则返回 dst 长度-1*/
  unsigned index=0,len;
  int i,j;
  len=strlen(dst);
  for (i=0;i<len;i++) {
    if (ch==dst[i]) break;
  }
  index=i;
  if (index>=len) index=len-1;
  return index;
}

void fun1(unsigned char src[],unsigned len,unsigned char dst[])
{  
  unsigned i,j;
  unsigned ebx=0,esi=0;
  unsigned eax,ecx,edx;

  if ((len&1)!=0) ebx=2;
  for (i=0,j=0;i<len;i++) {
     eax=search(src[i],Encode64);
     esi=esi<<6;
     ebx=ebx+6;
     eax=eax+esi;
     esi=eax;
     while (ebx>=4) {
        ebx=ebx-4;
        edx=esi;
        ecx=ebx;
        edx=edx>>ecx;
        ecx=ebx;
        dst[j]=Tab[edx];
```



```
edx=1;
edx=edx<<ecx;
edx--;
esi=esi&edx;
j++;
}
dst[j]='\0';
}
}

void fun2(unsigned char src[], unsigned len, unsigned char dst[])
{
    unsigned i, j;
    unsigned ebx = 0, edi = 0;
    unsigned eax, ecx, edx, esi;
    esi = search(src[0], Tab);
    eax = len / 3;
    edx = len % 3;
    if (edx == 1) ebx = 2;
    if (edx == 2) {
        if ((esi & 0x0c) == 0) ebx = -2;
        else ebx = 4;
    }
    for (i = 0, j = 0; i < len; i++) {
        esi = 0;
        esi = search(src[i], Tab);
        edi = edi << 4;
        ebx = ebx + 4;
        esi = esi + edi;
        edi = esi;
        while (ebx >= 6) {
            ebx = ebx - 6;
            eax = edi;
            ecx = ebx;
            eax = eax >> ecx;
            ecx = ebx;
            dst[j] = Encode64[eax];
            eax = 1;
            eax = eax << ecx;
            eax--;
            edi = edi & eax;
            j++;
        }
        dst[j] = '\0';
    }
}
```

## 三、算法总结

1. 机器码经过 `fun1` 函数得 `X`
   - RSA 解密：
     - `E` = `D0330A59`
     - `N` = `A7456C12309EAF6BEF610A5B1F408D62B4AF7775E167656C236BC3B8D77F587E92D80DB14AC83281`
     - `C` = `X`
   - 结果记为串 1

2. 串 1 通过 `fun2` 得串 2，串 2 后接 `ASPEEDER` 再通过 `fun1` 得比较值 1

3. 注册码通过 `fun1` 得串 3，串 3 的 `Hex` 形式记为 `Y`
   - RSA 解密：
     - `E` = `D0330A59`
     - `N` = `C985F97A3C4E0D3BF7D35DC4148E5C47749076D668CA8464A6D2CCFB1B26183623315E5450610784D974ED5E9702A451`
     - `C` = `Y`
   - 结果为关键比较值 2

4. 若两个关键比较值相等则注册成功
