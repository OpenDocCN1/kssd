
# My Flasher Play 1.4

> 原文：[`www.pediy.com/kssd/pediy04/kanxue360.htm`](https://www.pediy.com/kssd/pediy04/kanxue360.htm)

My Flasher Play 1.4 (破解+算法分析+注册机)

crack by fwnl

**下载地址:**

`http://ln.skycn.net/down/14.exe`

`http://www.skycn.com/down1.php?id=6936`

**软件简介:**

这是一个具有变体换肤、播放列表、进度条、`Exe2Swf`、自动搜索文件夹中的动画等功能的 Flash 动画播放器。它可使你随心所欲的控制 Flash 动画的播放：）此外界面简单，操作方便，几乎不需要帮助，就可掌握使用：）在视图上，它有三种视图，特别是精简视图和使窗口‘总在最上’的功能给合，可满足那些喜欢一边工作一边看动画的朋友：）此版本吸收了一些网友的建议，改进了播放列表的功能，可直接打开 `Exe` 格式的动画，同时修改了一些 bug，欢迎下载使用

**破解工具:** `trw2000` `1.22` `W32DASM` `8.93`

用 `trw2000` 载入软件后,按 `F5` 返回程序,点注册并填入任意数字,按 `ctrl+n` 呼出 `trw2000` 下 `bpx hmemcpy`,`F5` 返回，点确定,程序中断后下 `bd *`,按几下(我按了 14 下) `F10` 可来到下面:

```
:00472E71 BA842F4700               mov edx, 00472F84
:00472E76 E8CD1CF9FF               call 00404B48
:00472E7B 0F84D2000000             je 00472F53
:00472E81 33D2                     xor edx, edx
:00472E83 55                       push ebp
:00472E84 683F2F4700               push 00472F3F
:00472E89 64FF32                   push dword ptr fs:[edx]
:00472E8C 648922                   mov dword ptr fs:[edx], esp
:00472E8F 8D55F4                   lea edx, dword ptr [ebp-0C]
:00472E92 8B83F8020000             mov eax, dword ptr [ebx+000002F8]
:00472E98 E847A3FCFF               call 0043D1E4
:00472E9D 8B45F4                   mov eax, dword ptr [ebp-0C]
:00472EA0 E88B60F9FF               call 00408F30
:00472EA5 8BF0                     mov esi, eax                   <===把你输入的注册码送入 esi
:00472EA7 A1487F4700               mov eax, dword ptr [00477F48]<===把你的机器码送入 eax
:00472EAC 05671B3B07               add eax, 073B1B67 <==eax 与 73b1b67 相加,结果送入 eax
:00472EB1 25AB702E01               and eax, 012E70AB <==eax 与 12e70ab 作与运算,结果送入 eax
:00472EB6 0578030000               add eax, 00000378 <==eax 加上 378,结果送入 eax
:00472EBB 3BC6                     cmp eax, esi        <==真假注册码比较  ? eax 可得真码
:00472EBD 7446                     je 00472F05
:00472EBF 81FEBD56B800             cmp esi, 00B856BD
:00472EC5 7532                     jne 00472EF9
```

`* Possible StringData Ref from Code Obj ->"感谢注册！"`

哈哈，算法是不是太简单了，那就正适合我这菜鸟做注册机了:)

**算法分析:** 由上面的代码可知正解的注册码是用你的机器码加上 `73b1b67` 的结果和 `12e70ab` 作与运算再加上 `378`。

`73b1b67` 转十进制为 `121314151`

`12e70ab` 转十进制为 `19820715`(哈哈，可能是作者生日)

`378` 转十进制为 `888`

看到这里，我想大家都能写出注册机了。:)

下面是我用 c++写的一个

```cpp
#include<iostream.h>

void main()
{
    unsigned long int a;
    cout<<"Please input your  ID :";
    cin>>a;
    cout<<"Your Register code is :"<<((a+121314151)&19820715)+888<<endl;
}
```

哈哈，是不是非常简单.

祝各位玩得开心!! fwnl
2002.1.24
长沙

```
:00472EA0 E88B60F9FF               call 00408F30
:00472EA5 8BF0                     mov esi, eax                   <===把你输入的注册码送入 esi
:00472EA7 A1487F4700               mov eax, dword ptr [00477F48]<===把你的机器码送入 eax
:00472EAC 05671B3B07               add eax, 073B1B67 <==eax 与 73b1b67 相加,结果送入 eax
:00472EB1 25AB702E01               and eax, 012E70AB <==eax 与 12e70ab 作与运算,结果送入 eax
:00472EB6 0578030000               add eax, 00000378 <==eax 加上 378,结果送入 eax
:00472EBB 3BC6                     cmp eax, esi        <==真假注册码比较  ? eax 可得真码
:00472EBD 7446                     je 00472F05
:00472EBF 81FEBD56B800             cmp esi, 00B856BD   ==> 这里还有一个比较，将“B856BD”跟你输入的注册码相比，如过它也不等才跳！下 ? B856bd 得到 12080829 试试吧！
:00472EC5 7532                     jne 00472EF9
```

\* \* \*

`4dd466d0e1e0cc123afbe504da2462ca`
