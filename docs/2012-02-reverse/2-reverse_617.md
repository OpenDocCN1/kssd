
# Cracked for Myself--伪移植网际飞音

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-307.htm`](https://www.pediy.com/kssd/pediy07/pediy7-307.htm)

## 目标软件
网际飞音（如果是云南的朋友在看贴就不用多说了。不过还是要介绍一下这一款软件是很好的多媒体播放软件，但是只有该软件才可以播放 `music.km169.net` 上的歌曲。）

## 逆向目的
跟踪出她的解密方法，用 `C` 语言实现在 `linux` 下的伪移植^.^

## 必杀武器
`Ollydgb`，`VI`，`GCC`

## 看 T 须知
请大家不要将此帖转发到任何其他网站，或论坛。大家还要注意看帖要回帖，上完厕所要冲水，路边捡到钱要交给警察叔叔^.^

这是一篇逆向跟踪网际飞音编码过的歌曲 url 地址的解码过程，从而编写出一个解码器，使 `music.km169.net` 上面的歌曲能够在 `linux` 下用 `Mplayer` 来在线播放。

```
00402810   /$  83EC 08             sub esp,8
00402813   |.  8D4424 00          lea eax,dword ptr ss:[esp]
00402817   |.  56                push esi
00402818   |.  8B7424 10         mov esi,dword ptr ss:[esp+10]
0040281C   |.  57                push edi
0040281D   |.  6A 02             push 2
0040281F   |.  6A 07             push 7
00402821   |.  50                push eax
00402822   |.  8BCE             mov ecx,esi
00402824   |.  E8 F7C90200      call donor.0042F220
00402829   |.  8B3D 28F24300    mov edi,dword ptr ds:[<&KERNEL32.lstrcmpA>]   ;  kernel32.lstrcmpA
0040282F   |.  8D4C24 08        lea ecx,dword ptr ss:[esp+8]
00402833   |.  68 BC514400      push donor.004451BC                       ; /String2 = "01"
00402838   |.  51                push ecx                               ; |String1
00402839   |.  FFD7             call edi                               ; \lstrcmpA
0040283B   |.  85C0             test eax,eax
0040283D   |.  75 0F            jnz short donor.0040284E
0040283F   |.  56                push esi
00402840   |.  E8 5B060000      call donor.00402EA0    ； 这一句就是关键的 call，跟进对解密详细的分析
00402845   |.  83C4 04          add esp,4
00402848   |.  5F                pop edi
00402849   |.  5E                pop esi
0040284A   |.  83C4 08          add esp,8
0040284D   |.  C3                retn
00402EA0   /$  6A FF            push -1
00402EA2   |.  68 A8BD4300      push donor.0043BDA8                    
00402EA7   |.  64:A1 00000000   mov eax,dword ptr fs:[0]
00402EAD   |.  50                push eax
00402EAE   |.  64:8925 00000000 mov dword ptr fs:[0],esp
00402EB5   |.  83EC 0C          sub esp,0C
00402EB8   |.  53                push ebx
00402EB9   |.  55                push ebp
00402EBA   |.  56                push esi
00402EBB   |.  57                push edi
00402EBC   |.  8D4C24 10        lea ecx,dword ptr ss:[esp+10]
00402EC0   |.  E8 8BC00200      call donor.0042EF50
00402EC5   |.  68 00080000      push 800          
00402ECA   |.  C74424 28 00000000 mov dword ptr ss:[esp+28],0
00402ED2   |.  E8 34E80200      call donor.0043170B      ；这一部分作用是在系统中申请一部分内存空间以备后用
00402ED7   |.  68 00040000      push 400
00402EDC   |.  8BF8             mov edi,eax
00402EDE   |.  E8 28E80200      call donor.0043170B
00402EE3   |.  8B7424 34        mov esi,dword ptr ss:[esp+34]
00402EE7   |.  83C4 08          add esp,8
00432AD8    > \8B45 08          mov eax,dword ptr ss:[ebp+8]      从 43170B 跟进就可以发现在这里是分配堆空间的
00432ADB    .  85C0             test eax,eax
00432ADD    .  75 03            jnz short donor.00432AE2
00432ADF    .  6A 01            push 1
00432AE1    .  58                pop eax
00432AE2    >  83C0 0F           add eax,0F
00432AE5    .  24 F0            and al,0F0
00432AE7    .  50                push eax
00432AE8    >  6A 00            push 0                             ; |Flags = 0
00432AEA    .  FF35 E0E84400   push dword ptr ds:[44E8E0]                  ; |hHeap = 003F0000
00432AF0    .  FF15 A8F14300   call dword ptr ds:[<&KERNEL32.HeapAlloc>]    ; \HeapAlloc
00432AF6    >  8B4D F0          mov ecx,dword ptr ss:[ebp-10]
00432AF9    .  64:890D 00000000 mov dword ptr fs:[0],ecx
00432B00    .  5F                pop edi
00432B01    .  5E                pop esi
00432B02    .  5B                pop ebx
00432B03    .  C9                leave
00432B04    .  C3                retn        ；不用细看，可以直接返回到 43170B 的下一条指令继续跟踪
```

这里一看就知道肯定是对字符处理的部分了，不过这还只是第一部分，不能高兴得太早。先跟进 `42F3A0` 看看

```
00402EEE   |.  6A 30             push 30                       0<====>C
00402EF0   |.  6A 43             push 43
00402EF2   |.  E8 A9C40200      call donor.0042F3A0
00402EF7   |.  6A 31             push 31         1<====>D
00402EF9   |.  6A 44             push 44
00402EFB   |.  8BCE             mov ecx,esi
00402EFD   |.  E8 9EC40200      call donor.0042F3A0
00402F02   |.  6A 32             push 32         2<====>E
00402F04   |.  6A 45             push 45
00402F06   |.  8BCE             mov ecx,esi
00402F08   |.  E8 93C40200      call donor.0042F3A0
00402F0D   |.  6A 33             push 33         3<====>F
00402F0F   |.  6A 46             push 46
00402F11   |.  8BCE             mov ecx,esi
00402F13   |.  E8 88C40200      call donor.0042F3A0
00402F18   |.  6A 34             push 34         4<====>G
00402F1A   |.  6A 47             push 47                     

00402F1C   |.  8BCE             mov ecx,esi      
00402F1E   |.  E8 7DC40200      call donor.0042F3A0
00402F23   |.  6A 35             push 35         5<====>A
00402F25   |.  6A 41             push 41
00402F27   |.  8BCE             mov ecx,esi
00402F29   |.  E8 72C40200      call donor.0042F3A0
00402F2E   |.  6A 36             push 36         6<====>B
00402F30   |.  6A 42             push 42
00402F32   |.  8BCE             mov ecx,esi
00402F34   |.  E8 67C40200      call donor.0042F3A0
00402F39   |.  6A 37             push 37         7<====>Z
00402F3B   |.  6A 5A             push 5A
00402F3D   |.  8BCE             mov ecx,esi
00402F3F   |.  E8 5CC40200      call donor.0042F3A0
00402F44   |.  6A 38             push 38         8<====>I
00402F46   |.  6A 49             push 49
00402F48   |.  8BCE             mov ecx,esi
00402F4A   |.  E8 51C40200      call donor.0042F3A0
00402F4F   |.  6A 39             push 39         9<====>P
00402F51   |.  6A 50             push 50
00402F53   |.  8BCE             mov ecx,esi
00402F55   |.  E8 46C40200      call donor.0042F3A0
00402F5A   |.  6A 41             push 41         A<====>O
00402F5C   |.  6A 4F             push 4F
00402F5E   |.  8BCE             mov ecx,esi
00402F60   |.  E8 3BC40200      call donor.0042F3A0
00402F65   |.  6A 42             push 42         B<====>R
00402F67   |.  6A 52             push 52
00402F69   |.  8BCE             mov ecx,esi
00402F6B   |.  E8 30C40200      call donor.0042F3A0
00402F70   |.  6A 43             push 43         C<====>N
00402F72   |.  6A 4E             push 4E
00402F74   |.  8BCE             mov ecx,esi
00402F76   |.  E8 25C40200      call donor.0042F3A0
00402F7B   |.  6A 44             push 44         D<====>L
00402F7D   |.  6A 4C             push 4C
00402F7F   |.  8BCE             mov ecx,esi
00402F81   |.  E8 1AC40200      call donor.0042F3A0
00402F86   |.  6A 45             push 45         E<====>K
00402F88   |.  6A 4B             push 4B
00402F8A   |.  8BCE             mov ecx,esi
00402F8C   |.  E8 0FC40200      call donor.0042F3A0
00402F91   |.  6A 46             push 46         F<====>T
00402F93   |.  6A 54             push 54
00402F95   |.  8BCE             mov ecx,esi
00402F97   |.  E8 04C40200      call donor.0042F3A0
0042F3A0   /$  8B41 08          mov eax,dword ptr ds:[ecx+8]      ；将字符串的个数放入 eax 寄存器中
0042F3A3   |.  56                push esi
0042F3A4   |.  57                push edi
```



0042F3A5   |.  33FF                   xor edi,edi  

0042F3A7   |.  33F6                   xor esi,esi  

0042F3A9   |.  85C0                   test eax,eax            ；检测字符串是否为空   

0042F3AB   |.  7E 2A                  jle short donor.0042F3D7      ；是就跳了  

0042F3AD   |.  8A4424 10             mov al,byte ptr ss:[esp+10]      ；目标字符放入 eax 寄存器待用  

0042F3B1   |.  53                     push ebx  

0042F3B2   |.  55                     push ebp  

0042F3B3   |.  8B6C24 14             mov ebp,dword ptr ss:[esp+14]  

0042F3B7   |>  8B51 04               /mov edx,dword ptr ds:[ecx+4]                ; 得到待处理字符串的地址   

0042F3BA   |.  03D6                   |add edx,esi  

0042F3BC   |.  0FBE1A                 |movsx ebx,byte ptr ds:[edx]      ；取出字符串的每个字符，edx 相当于一个累加器  

0042F3BF   |.  3BDD                   |cmp ebx,ebp            ；与特征字符作比较。  

0042F3C1   |.  75 03                 |jnz short donor.0042F3C6      ；如果不同则跳到下面继续  

0042F3C3   |.  8802                   |mov byte ptr ds:[edx],al      ；如果相同，则用目标字符替换该特征字符  

0042F3C5   |.  47                     |inc edi            ；edi 相当于一个计数器，记录被转换了的字符个数  

0042F3C6   |>  8B51 08               |mov edx,dword ptr ds:[ecx+8]    ；取得字符串字符个数，检测是否到了字符串尾，  

0042F3C9   |.  46                     |inc esi            ；是则跳出，否则就循环  

0042F3CA   |.  3BF2                   |cmp esi,edx  

0042F3CC   |.^ 7C E9                 \\jl short donor.0042F3B7  

0042F3CE   |.  5D                     pop ebp  

0042F3CF   |.  8BC7                   mov eax,edi                             ;  eax 为互换后的个数  

0042F3D1   |.  5B                     pop ebx  

0042F3D2   |.  5F                     pop edi  

0042F3D3   |.  5E                     pop esi  

0042F3D4   |.  C2 0800               retn 8  

00402FE5   |> \\8BCE                   mov ecx,esi  

00402FE7   |.  E8 C4C70200           call donor.0042F7B0  

00402FEC   |.  B9 F6FFFFFF           mov ecx,-0A  

00402FF1   |.  2BCB                   sub ecx,ebx  

00402FF3   |.  83C3 09               add ebx,9  

00402FF6   |.  03C1                   add eax,ecx  

00402FF8   |.  8BCE                   mov ecx,esi  

00402FFA   |.  50                     push eax  

00402FFB   |.  53                     push ebx  

00402FFC   |.  57                     push edi  

00402FFD   |.  E8 1EC20200           call donor.0042F220        ；第二次解码的入口  

00403002   |.  57                     push edi  

00403003   |.  8BCE                   mov ecx,esi  

00403005   |.  E8 76C00200           call donor.0042F080  

0040300A   |.  8BCE                   mov ecx,esi  

0040300C   |.  E8 CFC30200           call donor.0042F3E0        ；在这里进行具体的解码工作  

00403011   |.  68 88D04400           push donor.0044D088  

00403016   |.  8D4C24 14             lea ecx,dword ptr ss:[esp+14]  

0040301A   |.  E8 61C00200           call donor.0042F080  

0040301F   |.  33DB                   xor ebx,ebx  

0042F220   |/$  53                     push ebx  

0042F221   |.  56                     push esi            ；这一部分依然是对字符串的长度做出判断。  

0042F222   |.  8B7424 10             mov esi,dword ptr ss:[esp+10]     

0042F226   |.  57                     push edi  

0042F227   |.  8BF9                   mov edi,ecx  

0042F229   |.  3B77 08               cmp esi,dword ptr ds:[edi+8]  

0042F22C   |.  7E 08                 jle short donor.0042F236  

0042F22E   |.  5F                     pop edi  

0042F22F   |.  5E                     pop esi  

0042F230   |.  33C0                   xor eax,eax  

0042F232   |.  5B                     pop ebx  

0042F233   |.  C2 0C00               retn 0C  

0042F236   |>  8B5C24 18             mov ebx,dword ptr ss:[esp+18]  

0042F23A   |.  83FB FF               cmp ebx,-1  

0042F23D   |.  75 1C                 jnz short donor.0042F25B  

0042F25B   |> \\8B47 04               mov eax,dword ptr ds:[edi+4]      ；这里用于去掉字符串中无意义的字符串  

0042F25E   |.  8B4C24 10             mov ecx,dword ptr ss:[esp+10]    ；也就是 dnmp://这些，具体的有 17 个无用字符  

0042F262   |.  8D53 01               lea edx,dword ptr ds:[ebx+1]  

0042F265   |.  03C6                   add eax,esi  

0042F267   |.  52                     push edx                             ; /n  

0042F268   |.  50                     push eax                             ; |String2  

0042F269   |.  51                     push ecx                             ; |String1  

0042F26A   |.  FF15 C8F04300         call dword ptr ds:[<&KERNEL32.lstrcpynA>]     ; \\lstrcpynA  

0042F3E0   |/$  55                     push ebp  

0042F3E1   |.  56                     push esi  

0042F3E2   |.  57                     push edi  

0042F3E3   |.  8B79 08               mov edi,dword ptr ds:[ecx+8]  

0042F3E6   |.  4F                     dec edi  

0042F3E7   |.  33F6                   xor esi,esi  

0042F3E9   |.  8BC7                   mov eax,edi  

0042F3EB   |.  99                     cdq  

0042F3EC   |.  2BC2                   sub eax,edx  

0042F3EE   |.  8BE8                   mov ebp,eax  

0042F3F0   |.  D1FD                   sar ebp,1  

0042F3F2   |.  78 20                 js short donor.0042F414  

0042F3F4   |.  53                     push ebx  

0042F3F5   |>  8B51 04               /mov edx,dword ptr ds:[ecx+4]    ; 字符串取反  

0042F3F8   |.  8BDA                   |mov ebx,edx                        

0042F3FA   |.  0FBE0432               |movsx eax,byte ptr ds:[edx+esi]    ；  

0042F3FE   |.  2BDE                   |sub ebx,esi  

0042F400   |.  8A1C3B                 |mov bl,byte ptr ds:[ebx+edi]    ；循环得到下一个字符  

0042F403   |.  881C32                 |mov byte ptr ds:[edx+esi],bl    ；放入目标字符串中  

0042F406   |.  8B51 04               |mov edx,dword ptr ds:[ecx+4]  

0042F409   |.  2BD6                   |sub edx,esi  

0042F40B   |.  46                     |inc esi  

0042F40C   |.  3BF5                   |cmp esi,ebp  

0042F40E   |.  88043A                 |mov byte ptr ds:[edx+edi],al  

0042F411   |.^ 7E E2                 \\jle short donor.0042F3F5  

0042F413   |.  5B                     pop ebx  

0042F414   |>  5F                     pop edi  

0042F415   |.  5E                     pop esi  

0042F416   |.  5D                     pop ebp  

0042F417   |\.  C3                     retn  

0040302C   |.  48                     dec eax          ；来到这里也到了解码的最后一步了  

0040302D   |.  85C0                   test eax,eax  

0040302F   |.  7E 4F                 jle short donor.00403080  

00403031   |>  6A 02                 /push 2  

00403033   |.  53                     |push ebx  

00403034   |.  57                     |push edi  

00403035   |.  8BCE                   |mov ecx,esi  

00403037   |.  E8 E4C10200           |call donor.0042F220  

0040303C   |.  8A07                   |mov al,byte ptr ds:[edi]      ；循环取字符串的下一个奇数字符  

0040303E   |.  3C 41                 |cmp al,41            ；比较是否大于‘A’  

00403040   |.  0FBEC0                 |movsx eax,al        ；先将字符保存  

00403043   |.  7C 05                 |jl short donor.0040304A      ；如果小于就减去 30，大于等于就减去 37。也就相  

00403045   |.  83E8 37               |sub eax,37            ；当于 ascii 编码的数字被转化为相应的 Hex 值  

00403048   |.  EB 03                 |jmp short donor.0040304D  

0040304A   |>  83E8 30               |sub eax,30  

0040304D   |>  8A4F 01               |mov cl,byte ptr ds:[edi+1]      ；循环取字符串的下一个偶数字符  

00403050   |.  80F9 41               |cmp cl,41  

00403053   |.  0FBEC9                 |movsx ecx,cl          

00403056   |.  7C 05                 |jl short donor.0040305D   

00403058   |.  83E9 37               |sub ecx,37  

0040305B   |.  EB 03                 |jmp short donor.00403060  

0040305D   |>  83E9 30               |sub ecx,30            ；同上作相应处理  

00403060   |>  C0E0 04               |shl al,4                             ；左移 4 位，也就是乘于 2 的 4 次方等于乘于 16   

00403063   |.  02C1                   |add al,cl            ；将两个字符组合为一个新的 ascii 字符  

00403065   |.  8B4C24 2C             |mov ecx,dword ptr ss:[esp+2C]  

00403069   |.  83C3 02               |add ebx,2  

0040306C   |.  880429                 |mov byte ptr ds:[ecx+ebp],al    ；之后就是判断循环的条件了。  

0040306F   |.  41                     |inc ecx  

00403070   |.  894C24 2C             |mov dword ptr ss:[esp+2C],ecx  

00403074   |.  8BCE                   |mov ecx,esi  



# 解码过程完成说明

## 汇编代码片段

```
00403076   |.  E8 35C70200          |call donor.0042F7B0  
0040307B   |.  48                   |dec eax  
0040307C   |.  3BD8                 |cmp ebx,eax  
0040307E   |.^ 7C B1                \\jl short donor.00403031  
```

至此，整个解码过程已经最终完成了。但是这里解码出来的字符串里面除了包含歌曲的 url 以外，还包括了一些我们所不需要的信息。不过这些无用的信息都可以编程将其去掉。以下就是我的 C 语言源代码。注意：使用这个程序的前提是你的 linux 系统上已经安装有了 Mplayer。

以下是程序源代码,写得很乱,也没有做任何的优化,先凑合着用.考了试再进一步做修改,如果有用 linux 的朋友有兴趣可以一起研究研究.

## 程序源代码

```c
/ *  music online v0.1    */

/ *    code by xeno     */

/*email: xeno2cc@sohu.com*/

/ *       2005.7.7       */

#include "stdio.h"

#include "stdlib.h"

int GetString (char ts\[7\], FILE \* s);

void DeCode (char txt\[230\], int n);

char \*ab;

int count = 0, lg;

main (int argc, char \*argv\[\])

{

  char cds\[230\] = "", ts\[7\] = "<media", mp\[200\] = "mplayer ";

  FILE \*s;

  int n = 1, i;

  if ((s = fopen (argv\[1\], "r")) == NULL)

    printf ("file open failed\\n");

  while (n != 0)

    {

      rewind (s);

      n = GetString (ts, s);

      if (n == 0)

  break;

      strcpy (cds, ab);

      for (i = 0; i <= lg; i++)

  mp\[8 + i\] = cds\[i\];

      printf ("%s\\n", mp);

      system (mp);

    }

  fclose (s);

  exit (0);

}

int

GetString (char ts\[7\], FILE \* s)

{

  int j, n = 0, flag = 0, chs = 1;

  char cd, cs\[7\] = "", name\[230\] = "";

  fseek (s, 0, 2);

  j = ftell (s);

  rewind (s);

  if (count < j)

    {

      while (chs != 0)

  {

    fseek (s, count, 0);

    j = fgets (cs, 7, s);

    if (j == 0)

      return 0;

    count++;

    if (strcmp (cs, ts) == 0)  /\*取得指定字符串 */

      {

        while (flag != 2)

    {

      cd = fgetc (s);

      if (cd == 34)

        flag++;

      while (flag == 1)

        {

          cd = fgetc (s);

          if (cd == 34)

      {

        flag++;

        break;

      }

          name\[n\] = cd;

          n++;

        }

    }

        if (n % 2 == 1)

    name\[n\] = 0;

        chs = 0;

        flag = 0;

      }

  }

    }

  DeCode (name, n);

  return j;

}

void

DeCode (char txt\[230\], int n)  /\*字符对比代换 */

{

  char a\[230\], m\[76\] = "";

  int i, ah, d = 0, al, z;

  for (i = 0; i <= n; i++)

    {

      switch (txt\[i\])

  {

  case 'C':

    txt\[i\] = '0';

    break;

  case 'D':

    txt\[i\] = '1';

    break;

  case 'E':

    txt\[i\] = '2';

    break;

  case 'F':

    txt\[i\] = '3';

    break;

  case 'G':

    txt\[i\] = '4';

    break;

  case 'A':

    txt\[i\] = '5';

    break;

  case 'B':

    txt\[i\] = '6';

    break;

  case 'Z':

    txt\[i\] = '7';

    break;

  case 'I':

    txt\[i\] = '8';

    break;

  case 'P':

    txt\[i\] = '9';

    break;

  case 'O':

    txt\[i\] = 'A';

    break;

  case 'R':

    txt\[i\] = 'B';

    break;

  case 'N':

    txt\[i\] = 'C';

    break;

  case 'L':

    txt\[i\] = 'D';

    break;

  case 'K':

    txt\[i\] = 'E';

    break;

  case 'T':

    txt\[i\] = 'F';

    break;

  }

    }

  for (z = 0; z < 230; z++)  /\*数组倒序 */

    {

      a\[z\] = txt\[i - 3\];

      i--;

    }

  for (z = 8; z < 230; z++)  /\*解码处理 */

    {

      if (a\[z\] < 65)

  al = a\[z\] - 48;

      else

  al = a\[z\] - 55;

      if ((z % 2) == 1)

  {

    m\[d\] = (ah \* 16 + al);

    d++;

  }

      ah = al;

    }

  for (i = 0; i < n; i++)  /\*提取有效地址 */

    {

      if (m\[i\] == '|')

  {

    m\[i\] = NULL;

    break;

  }

      else

  lg = i;

      continue;

    }

  ab = m;

} 
```

`4dd466d0e1e0cc123afbe504da2462ca`
