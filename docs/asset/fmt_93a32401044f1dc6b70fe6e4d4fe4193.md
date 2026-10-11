
# 关于 war3 中的随机数获取

> 原文：[`www.pediy.com/kssd/pediy11/125424.html`](https://www.pediy.com/kssd/pediy11/125424.html)

初学破解，水平不是很高，想找一些既简单自己又感兴趣的东西来做。前几天在看雪上看见有人讨论 `war3` 中随机数相关的东西，但是不是特别全，我按照他们的思路在我的机器上研究了几天，成功的弄清了 `war3` 的随机数机制，发上来分享一下，技术含量不是特别高，如果能获得邀请码就最好不过了。

先说环境，我用的是 `war3 1.24e`，`dota` 版本也是最新的 `6.69c`。玩过 `dota` 的都知道，输入 `-roll` 命令，会在 `1-100` 之间 `roll` 出一个随机数，其实 `dota` 地图是直接调用 `war3` 的获取随机数函数来实现这件事情的。这个函数叫 `GetRandomInit(int a, int b)`，`a` 和 `b` 分别是下限和上限，在 `game.dll` 中，加载到内存中后地址为 `0x6f3b40a0`。种子地址存放在 `0x6FACE230` 处。

以下是反汇编出的二进制代码。

```
6F3B40A0    8B4424 08       mov     eax, dword ptr [esp+8]  //b,上限  
6F3B40A4    57              push    edi  
6F3B40A5    8B7C24 08       mov     edi, dword ptr [esp+8]  //a,下限  
6F3B40A9    3BF8            cmp     edi, eax  
6F3B40AB    75 04           jnz     short 6F3B40B1//如果上下限相等,则直接返回  
6F3B40AD    8BC7            mov     eax, edi  
6F3B40AF    5F              pop     edi  
6F3B40B0    C3              retn  
6F3B40B1    3BF8            cmp     edi, eax  
6F3B40B3    56              push    esi  
6F3B40B4    7E 09           jle     short 6F3B40BF //if ( a <= b ) range = b-a  
6F3B40B6    8BF7            mov     esi, edi    //if ( a > b ) range = a-b  
6F3B40B8    2BF0            sub     esi, eax  
6F3B40BA    83C6 01         add     esi, 1  
6F3B40BD    EB 07           jmp     short 6F3B40C6  
6F3B40BF    2BC7            sub     eax, edi  
6F3B40C1    83C0 01         add     eax, 1  
6F3B40C4    8BF0            mov     esi, eax  
6F3B40C6    8B0D 30E2AC6F   mov     ecx, dword ptr [6FACE230] //取种子地址  
6F3B40CC    E8 FFD5C5FF     call    6F0120D0        //调用随机计算函数  
6F3B40D1    F7E6            mul     esi  
6F3B40D3    B1 20           mov     cl, 20  
6F3B40D5    E8 96DE4200     call    6F7E1F70        //进行一些简单的移位处理  
6F3B40DA    5E              pop     esi  
6F3B40DB    03C7            add     eax, edi    //将结果放在 eax 中返回  
6F3B40DD    5F              pop     edi  
6F3B40DE    C3              retn  
```

可以看出进来后，对上下限进行一个简单的比较，如果相等直接返回，如果不相等，计算出差值为 `range`，然后调用 `6F0120D0` 处的函数，根据种子（地址存放在 `ecx` 中）和 `range` 计算出一个随机数，最后调用 `6F7E1F70` 处的函数进行一些简单的处理后返回。我们只需要关注 `6F0120D0` 处的函数，里面有一些重要的信息。

```
6F0120D0    83EC 08         sub     esp, 8  
6F0120D3    8B01            mov     eax, dword ptr [ecx]  //取种子  
6F0120D5    53              push    ebx  
6F0120D6    8B59 04         mov     ebx, dword ptr [ecx+4]  //取种子  
6F0120D9    55              push    ebp  
6F0120DA    56              push    esi  
6F0120DB    8BF3            mov     esi, ebx  
6F0120DD    8BD3            mov     edx, ebx  
6F0120DF    0FB6EF          movzx   ebp, bh  
6F0120E2    C1EB 10         shr     ebx, 10  
6F0120E5    C1EA 18         shr     edx, 18  
6F0120E8    81E3 FF000000   and     ebx, 0FF  
6F0120EE    83EB 0C         sub     ebx, 0C  
6F0120F1    83EA 04         sub     edx, 4  
6F0120F4    81E6 FF000000   and     esi, 0FF  
6F0120FA    85D2            test    edx, edx  
6F0120FC    57              push    edi  
6F0120FD    8BFB            mov     edi, ebx  
6F0120FF    895C24 10       mov     dword ptr [esp+10], ebx  
6F012103    7D 06           jge     short 6F01210B  
6F012105    81C2 BC000000   add     edx, 0BC  
6F01210B    83ED 18         sub     ebp, 18  
6F01210E    85FF            test    edi, edi  
6F012110    7D 0A           jge     short 6F01211C  
6F012112    81C7 D4000000   add     edi, 0D4  
6F012118    897C24 10       mov     dword ptr [esp+10], edi  
6F01211C    8B9A A857966F   mov     ebx, dword ptr [edx+6F9657A8]  
6F012122    83EE 1C         sub     esi, 1C  
6F012125    85ED            test    ebp, ebp  
6F012127    7D 06           jge     short 6F01212F  
6F012129    81C5 EC000000   add     ebp, 0EC  
6F01212F    8BBF A857966F   mov     edi, dword ptr [edi+6F9657A8]  
6F012135    D1C3            rol     ebx, 1  
6F012137    85F6            test    esi, esi  
6F012139    895C24 14       mov     dword ptr [esp+14], ebx  
6F01213D    7D 06           jge     short 6F012145  
6F01213F    81C6 F4000000   add     esi, 0F4  
6F012145    8B9D A857966F   mov     ebx, dword ptr [ebp+6F9657A8]  
6F01214B    C1C3 03         rol     ebx, 3  
6F01214E    C1E2 08         shl     edx, 8  
6F012151    0B5424 10       or      edx, dword ptr [esp+10]  
6F012155    C1C7 02         rol     edi, 2  
6F012158    33DF            xor     ebx, edi  
6F01215A    339E A857966F   xor     ebx, dword ptr [esi+6F9657A8]  
6F012160    C1E2 08         shl     edx, 8  
6F012163    335C24 14       xor     ebx, dword ptr [esp+14]  
6F012167    0BD5            or      edx, ebp  
6F012169    03C3            add     eax, ebx  
6F01216B    5F              pop     edi  
6F01216C    C1E2 08         shl     edx, 8  
6F01216F    0BD6            or      edx, esi  
6F012171    5E              pop     esi  
6F012172    5D              pop     ebp  
6F012173    8951 04         mov     dword ptr [ecx+4], edx  //更新种子  
6F012176    8901            mov     dword ptr [ecx], eax  //更新种子  
6F012178    5B              pop     ebx  
6F012179    83C4 08         add     esp, 8  
6F01217C    C3              retn  
```

重点需要关心的是以下几句代码，每次调用该函数之后都会导致种子更新。

```
6F0120D3    8B01            mov     eax, dword ptr [ecx]  //取种子  
6F0120D6    8B59 04         mov     ebx, dword ptr [ecx+4]  //取种子  
6F012173    8951 04         mov     dword ptr [ecx+4], edx  //更新种子  
6F012176    8901            mov     dword ptr [ecx], eax  //更新种子  
```

至于其他的计算过程，应该比较好理解。至此，我们已经可以动手写一个获取随机数的小程序了。用远程线程注入的方法，把一段代码注入到 `war3` 中，先保存种子，调用 `GetRandomInit(1,100)` 若干次，获取随机数后将种子写回。然后你在 `dota` 里输入 `-roll`，就可以看到出来的数与你获得的随机数完全一致。

获取随机数的代码用内联汇编可以写成：

```
for(int i=0;i<100;i++)
{
  __asm
  {
    MOV EAX,0x64  //上限 100
    PUSH EAX
    MOV EAX,0x1  //下限 1
    PUSH EAX
    MOV EAX,0x6f3b40a0
    CALL  EAX  //调用 GetRandomInit
    MOV temp,EAX  //将结果保存到 temp 变量,然后可以将 temp 值写入文件
    POP EAX
    POP EAX  //平衡调用堆栈
  }
  fout<<dec<<temp<<endl;
}
```

接下来就可以使用远程线程注入方法，将相关代码注入到 `war3` 进程中执行即可。附件里是我已经实现的程序，把两个 `dll` 拷贝到能找到的目录，推荐 `C:\WINDOWS\system32` 下。首先打开 `war3`，创建游戏进入 `dota`，游戏开始后先暂停（这是因为游戏创建过程中和出兵后都要用到大量的随机数，为了避免干扰，先暂停以便进行试验），然后运行附件中的程序 `test1.exe`，在 `d:` 盘根目录下会看到一个 `test.txt` 文件，打开，然后返回 `war3`，输入命令 `-roll`，对比一下，可以发现 `roll` 出来的数都在刚才的文件中。另外，既然能获得随机数，那么也能控制随机数，一个最简单的想法是直接改写种子。我尝试了直接把种子改成 `0`。同样，在游戏刚开始时暂停，运行 `test2.exe`，返回游戏输入 `-rd`，你会看到随机出来的阵容无论多少次都是一样的，应该有影魔，屠夫和火枪。

简单的破解至此结束，望高人指正。之前已经有牛人做出来了，不过他在自己空间里没有说的很明白，我就尝试着把这件事给做完了，虽然不记得名字了，还是要向他表示感谢。就这样吧。

上传的附件

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || `war3_rnd_test.zip` |
|---|---|
