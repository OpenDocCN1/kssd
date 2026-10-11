
# DeDe 用法

> 原文：[`www.pediy.com/kssd/pediy05/pediy50313.htm`](https://www.pediy.com/kssd/pediy05/pediy50313.htm)

题目： Cracking using DeDe （用 `dede` 来破解）   
作者： Fulgore   
翻译：WiNrOOt 2002.10.7   

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   

引   

这是一篇介绍 `dede` 的基础文章，在这里我将向你展示如何用 `dede` 分析软件，将软件爆破，并写出他的 `keygen` 的问题.   

工具:   

Hmmm... 用什么呢？可能有 `dede`？:-)还有 `Hiew`，还有 `Delphi`（用来写 `keygen`）   

文件：   

我自己特地为这片教程写了一个 `test` 的文件用来破解   

下载地址：`http://www.balbaro.com/dede/files/test.zip`   

*在最后我们将爆破这个文件~~~~~   

开工~~~~~~~~~~~   

*1。运行 `dede`   

*2。选择文件，单击 `Process`。   

*3。`test.exe` 将被运行，来让我们认识一下他（`name`，`company`&`serial`)   

  当然你也可以尝试一下注册按钮（`winroot`:千万不要把 `test` 关了)   

*4。点击 `OK`，`dede` 开始工作~~~~~~~~读取完成~~再点 `ok`.读取工作完成，分析正式开始   

*5。点选 `dede` 中的 `procedures` 项   

*6。单击左边窗口的 `TForm1`，你将会看到左边出现 `Button1Click`（双击它）————噢！发生了什么？   

*7。开始浏览代码……一会你见会看到这样的信息（`winroot`:这些东西不会在同一个地方显示！废话！！）   

   * Possible reference to control 'Edit2':`TEdit`   

     ………………………………   

     * Possible reference to control 'Edit1':`TEdit`   

     这些就是调用我们输入 `Name`/`Company` 的地方。   

*8。我么继续看代码：   

* Reference to: `System..LStrCat3()`   

|   

```
0044437D  E82AF8FBFF               call    00403BAC
00444382  8D55F0                  lea     edx, [ebp-$10]
```

T 他的意思是将两个串连起来.   

9\. 下面你将看到 `edit3.text` 的调用. 很有趣, huh? :)   

```
00444390  8B55F0                  mov     edx, [ebp-$10]
00444393  8B45FC                  mov     eax, [ebp-$04]
```

* Reference to: `System..LStrCmp()`   

|   

```
00444396  E8D5F8FBFF               call    00403C70
0044439B  7407                    jz      004443A4
```

`uhuh`. Weeeell 这意味着, `edx` 与 `eax` 比较, 就象输入值与序列号比较一样。不会是明码比较这么简单把? 我们来看   

如果我们执行到这里:   

```
0044439B  7407                    jz      004443A4
```

如果 `string` 相等程序将什么也不会做（程序自己会判断）所以绝对不能让他跳，   

我们继续往下看;   

10\. `0044439F  E85C010000               call    00444500`   

我觉的有点可疑！ :) 双击跟进取!   

我们发现这里好像是一函数的开始部分, 把两个值连起来.   

?   

```
00444550  E87FFEFFFF               call    004443D4
```

这个子程序是干吗的？跟进去看到   

|   or: `System..LStrLen()`   

```
004443F8  E863F7FBFF               call    00403B60
```

取字符串长度   

```
004443FF  83FB0A                  cmp     ebx, +$0A
00444402  7F12                    jnle    00444416
```

一直循环到 `$0A`, 就是 10.   

所以它等于 `"for i:=length(s) to 10"`   

Oki. 我们能在循环中做些什么呢?   

* Reference to: `System..LStrCat()`   

在字符串后加个 `'A'`.   

结果怎么样呢? 程序调用字符串, 如果字符个数小于 10 就在后面加 `'A'`.   

11\. 单击右上的 "<" 键推出上面那个 `call` 来到:   

* Possible String Reference to: `"123-"`   

|   

```
00444555  683C464400               push    $0044463C
```

oki. 我们看到 `'123-'` 很好.   

继续看. 这个子程序将创建一个串:   

a) `LStrCopy(something,from,to)`   

b) 存储结果   

c) 测试结果 `call 00444444`   

12\. 看这里这里有两次变化:   

```
0044455E  B905000000               mov     ecx, $00000005
00444563  BA01000000               mov     edx, $00000001
```

调用 `eax` 的值，复制 `$1` 到 `$5`. 如果串是 `'aaabbbccc'` 则结果是 `'aaabb'`;   

```
00444587  B904000000               mov     ecx, $00000004
0044458C  BA05000000               mov     edx, $00000005
```

同样, 从第五个字母开始取四个. 所以结果是: `'bbbc'`   

13\. 在这两个 `call` 之间, 你会看到   

* Possible String Reference to: `"-321-"`   

|   

```
0044457E  684C464400               push    $0044464C
```

记住他，他使中间的那个.   

* Possible String Reference to: `"-123"`   

|   

```
004445A7  685C464400               push    $0044465C
```

and another one in the end.   

现在我们知道 `serial` 的结构: `123-xxx-321-xxx-123`   

14\. 现在我们要找 `xxx` 的值, 进入 `00444444 call`. 看到:   

* Possible String Reference to: `"zZz"`   

在起始和结束保存,   

然后:   

```
00444488  BA03000000               mov     edx, $00000003
```

* Reference to: `System..LStrCatN()`   

从字符串中调用第三个字母 像这样复制他:   

`zZz + char + zZz`   

然后, 同样的, 他们调用调用第一和第二个字母.   

如果输入 `'Hello'`, 结果是:   

`'HzZzlzZze'`   

cool.   

15. 这就是算法!   

好让我们来算自己的 `serial`!   

`Name`: Ful   

`Company`: gore   

a) `Temp String`: Fulgore   

b) 第一个函数得到 我们得到: FulgoreAAA   

c) 然后第一个规则得到 `'Fulgo'`, 用第二个函数得到 `uzZzlzZzF`   

d) 用第 2 个得到 `'oreA'`, 用第二个函数得到 `rzZzezZzo`   

e) 用第 3 个函数我们得到: `123-FzZzlzZzu-321-ozZzezZzr-123`   

f) 让我们检测以下！成功!!!   

g) 用到下面的函数:   

`1st`:   

```
procedure prepare(var s:string);

var z:string; i:integer;

begin

z:=s;

for i:=length(z) to 10 do

s:=s+'A';

end;
```

`2nd`

```
function Cripple(s:String):string;

var kk:string;

begin

kk:='zZz'+s[3]+'zZz';

kk:=s[1]+kk+s[2];

result:=kk;

end;
```

`3rd.`

```
function generate(name,comp:string):string;

var temp:string;

begin

temp:=name+comp;

prepare(temp);

result:='123-'+cripple(copy(temp,1,5))+'-321'+cripple(copy(temp,5,4))+'-123';

end;
```

爆破   

Well, 这一步非常简单, 看 `Button1Click` 的代码,   

双击 `call    00444500` 进入我们会看到：   

```
004445D2  740C                    jz      004445E0
```

* Possible String Reference to: `"screw you!"`   

```
004445D4  B86C464400               mov     eax, $0044466C
```

* Reference to: `Dialogs.ShowMessage(System.AnsiString)`   

```
004445D9  E896F9FFFF               call    00443F74
004445DE  EB0A                    jmp     004445EA
```

* Possible String Reference to: `"good for you!"`   

```
004445E0  B880464400               mov     eax, $00444680
```

很明显的可以看到我们下一步将干什么，只需将 `jz` 变成 `jmp`.   

运行 `RVA converter` （在 `dede` 的 `tools` 里）将 `004445D2` 变成 `offset`: `000439D2`   

运行 `hiew`, 进入 `hex` 模式, 按 `F5` 到 `000439D2`;   

这里将出现 `JE` 操作码 ，把它变成 `JMP`, `save` 然后运行   

收工;)   

爆破已经完成, 相信你能写出他的 `keygen` :)   

结束语:   

谢谢你读完我这篇垃圾（谁说老外不谦虚？), 我希望你能从中体会点什么 ;)   

非常感谢 `DaFixer` 提供的的软件, 也要谢谢 `Borland` 的 `RAD`.   

谢谢观看 !(老外怎么那么多废话？？）   

byebye   

fulgore   

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   

winroot 的结束语：   

  我是一名初学者，难免翻译有不准确错误的地方，请各位高手指正！   

我的 `mail`: `ymlianming@etang.com` 欢迎交流~ `4dd466d0e1e0cc123afbe504da2462ca`
