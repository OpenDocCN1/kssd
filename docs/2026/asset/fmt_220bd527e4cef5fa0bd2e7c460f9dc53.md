
# 耐心的体验——手脱SVK1.43加壳记事本（两种方法）

## 文章信息
【文章标题】: 耐心的体验——手脱SVK1.43加壳记事本（两种方法）
【文章作者】: zyqex
【软件名称】: 加壳NotePad98
【下载地址】: 附件
【加壳方式】: `SVK1.43`
【使用工具】: `ollyice`
【操作平台】: `xp`
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

### 前语
唔~又是记事本！！呵呵，没办法，水平菜，最近忙着找工作，好歹脱了四五天，又晾了一个周，还是拿出来给部分人分享一下（高手就飘过吧）。我将用两种方法演示：1.常规方法 2.以壳解壳

### Begin

#### 方法一

一.先大致介绍一下壳用到的技术：
1. 加密`iat`
2. `stolen code` `oep` 处
3. `replace code` `mov` 类的
4. 还有一个替换`api`的，不知该归为什么（我首次接触`stolen code`等概念，失误之处敬请诸位赐教）

二.
1. `peid`查看，`SVKP 1.3x` -> `Pavol Cerven`（其实是`1.43`，我用论坛里的那个`svk1.43`的版本）
`od`载入，开始的时候提示打开文件错误，不用管点确定就是，`Alt+M`看看，只看得到一个区段（至少在我的机器上是这样）。

地址=`00400000`
大小=`0002D000` `184320.`
属主=`NOTEPAD_` `00400000` `自身`
区段=
包含=`PE` 文件头
类型=`Imag 01001002`
访问=`R`
初始访问=`RWE`

2. 设置一下`od`:
`olly advance1.26 beta 12`插件里勾选`IsDebuggerPresent`及`Anti-RdTSC`下的`Enable`
异常设置，只勾选“忽略在`kernel32`中的内存访问异常”（忽略所有异常，按下面的做法我的`od`会死，所以就麻烦一点吧）

下面的跟`chasgone`《某外挂 `SVKP 1.4x` -> `Pavol Cerven`脱壳》的部分类似，我就直接引用一下了（呵呵，应该不会犯错吧）

【引用】
搞定`iat` //避过`IAT`加密的处理，这样修复时就容易多了
`od`载入 下断点`bp GetModuleHandleA+5` //到达处理`IAT`的地方，随意跟踪过壳的不会不知道
`shift＋f9`断下，取消断点，`ctrl＋f9`返回主程序。
`ctrl＋f`搜索特征码
`cmp dword ptr ds:[ebx],251097CC`
把这下面的所有的比较和跳转全部`nop`掉
然后在搜索特征码
`mov dword ptr ds:[edi],eax` //此处处理甚妙，改过后`iat`里写入的直接就是调用的函数的地址了
`popad`
改成：
`popad`
`mov dword ptr ds:[edi],eax`
【引用结束】

接着上面

```
0B755B4F    8907            mov     dword ptr [edi], eax        //即上面要修改的地方
0B755B51    61              popad
0B755B52    8385 43010200 0>add     dword ptr [ebp+20143], 4
0B755B59  ^ E9 ADFBFFFF     jmp     0B75570B
0B755B5E    83C6 14         add     esi, 14
0B755B61    8B95 871C0300   mov     edx, dword ptr [ebp+31C87]
0B755B67    C685 32550200 0>mov     byte ptr [ebp+25532], 0
0B755B6E    C685 33550200 0>mov     byte ptr [ebp+25533], 0
0B755B75  ^ E9 9DFAFFFF     jmp     0B755617
0B755B7A    33C0            xor     eax, eax
0B755B7C    64:8F00         pop     dword ptr fs:[eax]
0B755B7F    83C4 04         add     esp, 4
0B755B82    C3              retn       //在这里下断（这时iat就处理完了,下断后F9运行）
```

断下后，取消`0B755B82`处断点。
接着`Alt+M`在那个主程序唯一的区段上`F2`下断，`F9`运行
断在：
```
0013FF17    8B89 78010000   mov     ecx, dword ptr [ecx+178]
0013FF1D    8BAD 42210000   mov     ebp, dword ptr [ebp+2142]
```
`f8`步过`0013FF17`处后（不步过待会下断运行后会原地停留），`Alt+M`在那个主程序唯一的区段上`F2`下断，`F9`运行
断在：
```
0B7DF5F4    FF15 E4634000   call    dword ptr [4063E4]           //其实这两句是被偷的完整的两句代码
0B7DF5FA    89C6            mov     esi, eax
```
`F7`步进一下（`F8`或`F9`就跑飞），`Alt+M`在那个主程序唯一的区段上`F2`下断，`F9`运行
断在：
```
004010DD    3C 22           cmp     al, 22                   //foep 处
004010DF    75 1B           jnz     short 004010FC
004010E1    56              push    esi
004010E2    FF15 AC644000   call    dword ptr [4064AC]
```

很明显的一个大跨段的跳跃了^\_^
向上翻看：
```
004010C7    000D 0A000090   add     byte ptr [9000000A], cl
004010CD    90              nop
004010CE    90              nop
004010CF    90              nop
004010D0    90              nop
004010D1    90              nop
004010D2    90              nop
004010D3    90              nop
004010D4    90              nop
004010D5    90              nop
004010D6    90              nop
004010D7    90              nop
004010D8    90              nop
004010D9    90              nop
004010DA    90              nop
004010DB    90              nop
004010DC    90              nop
004010CC--004010DC 被偷了。
```

拿起`importrec` 填`oep=10dd`，自动搜索`iat` 全部有效 保存为树文件 后面有用

好了，现在我们来搞定`stolen code`

对于`sloen code` 的处理有（又一篇文章里的引用^\_^）
【引用】
找回抽掉的代码的方法一般有：
1、`TC`跟踪得到
2、`tDasm`老兄介绍的还原`stolen code`的分析法
3、`FLY`老兄介绍的`cdi`猜测法，这个我还是未明白
4、`blowfish`老兄介绍的建立`stack`法，这个方法非常好。
【引用结束】

关于2：看雪精华里搜`tDasm`相关文章吧，没搜到合适的
关于3：很遗憾我没搜索到（发了个求助帖，好像没人知道，知道的补充一下吧。）
关于4：看雪精华6里搜《重建`stolen bytes`的一般方法》（建议去好好的看看，非常不错的一篇文章）

----------------------------------------------------------------------------------------------

下面我大致介绍一下我自己找的过程中用的方法（可能有些愚笨，见谅！），上面的1、2、4的混用。
先到`foep`后记下寄存器的值（我们可以注意到`foep`开始以后的代码里用到了寄存器`eax,esi,`所以在跟踪`stolen code`时要注意这里个寄存器的变化。），以及堆栈内`ebp`到`esp`的内容。

先看看代码的入口特征，`vc`的，大致的跟踪一遍，看程序在什么地方跳向`foep`处，这个地方以上的一些代码一般就是模拟被偷代码的功能部分了，在跟踪过程中我们注意什么时候`ebp`变为到`foep`后`ebp`的值，注意寄存器的变化，我自己琢磨的`tc`跟踪法应该是这样使用：重载程序，按二的步骤到达地址`0B755B82`处，然后在`foep(4010dd)`处下断，`Ctrl+F11`(步进跟踪)，停下后看记录。

引用`blowfish`一句话：“重建`stolen bytes`的目的，是为了让局部变量、全局变量、寄存器的初值在执行到`fake OEP`时和未加壳前的保持一致，并非要精确还原这些`stolen`的指令，只要功能相同即可。 ” 加壳程序要恢复被偷处的功能也是同样的道理。（不清处见谅，水平太菜了！亲自跟上几遍也许就明白了^\_^）

依照这样一个思路，跟踪到下面的代码(只列出了关键部分)：
```
0B7E0095   主  sub     esp, 44  FL=A, ESP=0013FF7C              //被偷代码
0B7E0098   主  push    eax  ESP=0013FF78                    //被偷代码（注意到esp的值）
0B7E0099   主  add     esi, 8B8A3F66  FL=S, ESI=8B9DFB7A
0B7E009F   主  jmp     0B7DE2B3
……
0B7DF5ED   主  sub     dword ptr [esp], eax  FL=P
0B7DF5F0   主  pop     esi  ESP=0013FF74, ESI=0013BC14
0B7DF5F1   主  mov     edi, edi
0B7DF5F3   主  pop     eax  EAX=0B7DE159, ESP=0013FF78
0B7DF5F4   主  call    dword ptr [4063E4]  ESP=0013FF74      //被偷代码
0B7DF5FA   主  mov     esi, eax  ESI=00152398             //被偷代码（注意esi值）
0B7DF5FC   主  mov     al, byte ptr [eax]  EAX=00152322     //被偷代码
0B7DF5FE   主  push    74B5D177  ESP=0013FF74
0B7DF603   主  add     eax, 8B8A3F66  FL=PS, EAX=8B9F6288
0B7DF608   主  push    eax  ESP=0013FF70
0B7DF609   主  mov     eax, 8B8A3F66  EAX=8B8A3F66
0B7DF60E   主  sub     dword ptr [esp], eax  FL=P
0B7DF611   主  jmp     0B7DE83A
0B7DE83A   主  mov     eax, dword ptr [esp]  EAX=00152322
0B7DE83D   主  mov     eax, 8B8A3F66  EAX=8B8A3F66
0B7DE842   主  add     dword ptr [esp+4], eax  FL=CP
0B7DE846   主  pop     eax  EAX=00152322, ESP=0013FF74
0B7DE847   主  add     esp, 4  FL=P, ESP=0013FF78
0B7DE84A   主  jmp     dword ptr [esp-4]                //这里跳向foep
```
`c`程序的入口为`push ebp` `mov ebp,esp`这两句在上面的代码里被变形了，罗列一下`stolen code`:
`push ebp`
`mov ebp,esp`
`sub esp,44`
`push eax`
`call dword ptr ds:[4063E4]`
`mov esi,eax`
`mov al,byte ptr ds:[eax]`

补到`00401010cc`处并保存到可执行文件，然后重新用`od`打开，运行到`oep`处后，用`LoadPe dump`程序，改入口为`004010CC`。

当然使用`blowfish`介绍的建立`stack`法会容易一些，有兴趣的可以自己试试。

**************************************************************************

三.修复被替换的`api`

上面的步骤完成后，如果使用步骤二保存的树文件修复`iat`，运行程序没反应。用`od`加入看看，很多调用`MessageBox`和`ExitProcess`的地方。

现在我们知道了了`foep--004010dd,od`载入“加壳记事本”， `bp GetModuleHandleA+5`断下后直接在`foep`处下断。

断下后，我们看到程序里很多`call dword ptr [4064AC]`和`call dword ptr [4063A0]`。
随便找一个调用的地方`f7`进去看看（可以在下面的语句设断，然后`Ctr+F11`），查看记录：

会发现如下的关键地方：
******对于`call dword ptr [4064AC]`******
```
00F2A078    9C              pushfd               //004064ac
00F2A079    55              push    ebp
00F2A07A    E8 00000000     call    00F2A07F
--------------------------------------------------------------------
0013FE64    833C31 00       cmp     dword ptr [ecx+esi], 0             //关键循环1
0013FE68    74 64           je      short 0013FECE
0013FE6A    50              push    eax
0013FE6B    807D 36 01      cmp     byte ptr [ebp+36], 1
0013FE6F    75 0C           jnz     short 0013FE7D
0013FE71    2B85 3E010000   sub     eax, dword ptr [ebp+13E]
0013FE77    0385 42010000   add     eax, dword ptr [ebp+142]
0013FE7D    390431          cmp     dword ptr [ecx+esi], eax         ; NOTEPAD.004010E4
0013FE80    58              pop     eax
0013FE81    74 07           je      short 0013FE8A
0013FE83    83E9 09         sub     ecx, 9
0013FE86    74 46           je      short 0013FECE
0013FE88  ^ EB DA           jmp     short 0013FE64
eax=004010E4 (NOTEPAD.004010E4)
ds:[00F2CB14]=0040353F (NOTEPAD.0040353F)
------------------------------------------------------------------------
```
