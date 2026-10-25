
# Easy Video to iPod MP4 PSP 3GP Converter 另类破解

> 原文：[`www.pediy.com/kssd/pediy10/58933.html`](https://www.pediy.com/kssd/pediy10/58933.html)

**【文章标题】**: Easy Video to iPod MP4 PSP 3GP Converter 另类破解

**【文章作者】**: `xPLK[0GiNr]`

**【作者主页】**: `http://hi.baidu.com/zoo_` `http://www.0GiNr.com`

**【软件名称】**: Easy Video to MP4 Converter

**【下载地址】**: `http://www.divxtodvd.net/`

**【加壳方式】**: 无

**【编写语言】**: Microsoft Visual C++ 6.0

**【使用工具】**: `OllyDbg`

**【软件介绍】**: 转换器

**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

***

## 【详细过程】

很久以前就破解了，发在 Blog 上，刚才看到看雪有位兄弟说做掉了一个很容易破解的外国程序，我也来凑凑热闹，来个最容易写注册机的外国程序，而且是一锅端。主要目的是为了使大家在破解中能开阔视野，找到多条路子。

高手可以略过。

下面开始。

这回的目标是 Easy Video to iPod MP4 PSP 3GP Converter，一个转换器，外国的。

初步看了下，注册方法是采用注册名+算法的形式的，实时比较，不是那些重启比较的。

`PEiD` 查壳，嗯，无壳，`VC6` 写的。`OD` 载入主程序，搜索字符串 `"invalid user name or registeration code"`，容易找到下面这段代码：

```
004075A5   > \6A 40         push    40
004075A7   . 68 A49F4100    push    00419FA4                     ;  sorry
004075AC   . 68 7C9F4100    push    00419F7C                     ;  invalid user name or registeration code
004075B1   . 8BCB           mov     ecx, ebx
004075B3   . C705 8CAE4100 >mov     dword ptr [41AE8C], 0
004075BD   . E8 0AAB0000    call    <jmp.&MFC42.#4224>
```

`OD` 显示 `004075A5` 由 `004074C0` 的指令 `. /0F85 DF000000 jnz 004075A5` 跳过。所以往上看。

然后的然后，就到这里：

```
00407440   . 81EC C0020000  sub     esp, 2C0
00407446   . 53             push    ebx
00407447   . 56             push    esi
00407448   . 6A 01          push    1
0040744A   . 8BD9           mov     ebx, ecx
0040744C   . E8 2FAD0000    call    <jmp.&MFC42.#6334>
00407451   . 8B43 64        mov     eax, dword ptr [ebx+64]      ;  eax=注册名
00407454   . 8D5424 08      lea     edx, dword ptr [esp+8]
00407458   . 2BD0           sub     edx, eax
0040745A   > 8A08           mov     cl, byte ptr [eax]
0040745C   . 880C02         mov     byte ptr [edx+eax], cl
0040745F   . 40             inc     eax
00407460   . 84C9           test    cl, cl
00407462   . ^ 75 F6        jnz     short 0040745A
00407464   . 8B43 60        mov     eax, dword ptr [ebx+60]
00407467   . 8D5424 48      lea     edx, dword ptr [esp+48]
0040746B   . 2BD0           sub     edx, eax
0040746D   > 8A08           mov     cl, byte ptr [eax]
0040746F   . 880C02         mov     byte ptr [edx+eax], cl
00407472   . 40             inc     eax
00407473   . 84C9           test    cl, cl
00407475   . ^ 75 F6        jnz     short 0040746D
00407477   . 68 5C914100    push    0041915C                     ; /ether.dll    加载 DLL 名为"ether.dll"的 DLL
0040747C   . FF15 C0404100  call    dword ptr [<&KERNEL32.LoadLibrar>; \LoadLibraryA
00407482   . 8BF0           mov     esi, eax
00407484   . 68 A0954100    push    004195A0                     ; /reg\_code      有个函数叫"reg\_code"
00407489   . 56             push    esi                           ; |hModule
0040748A   . FF15 BC404100  call    dword ptr [<&KERNEL32.GetProcAdd>; \GetProcAddress
00407490   . 8D8C24 880000 >lea     ecx, dword ptr [esp+88]
00407497   . 8D5424 08      lea     edx, dword ptr [esp+8]        ;  EDX=注册名
0040749B   . 51             push    ecx
0040749C   . 52             push    edx
0040749D   . FFD0           call    eax                           ;  Call DLL 的导出函数
0040749F   . 83C4 08        add     esp, 8
004074A2   . 56             push    esi                           ; /hLibModule
004074A3   . FF15 B8404100  call    dword ptr [<&KERNEL32.FreeLibrar>; \FreeLibrary
004074A9   . 8D8424 880000 >lea     eax, dword ptr [esp+88]       ;  eax=明文注册码
004074B0   . 8D4C24 48      lea     ecx, dword ptr [esp+48]       ;  ecx=我们输入的注册码
004074B4   . 50             push    eax
004074B5   . 51             push    ecx
004074B6   . E8 55F3FFFF    call    00406810                     ;  比较函数
004074BB   . 83C4 08        add     esp, 8
004074BE   . 85C0           test    eax, eax
004074C0   . 0F85 DF000000  jnz     004075A5                     ;  如果就挂了，所以这里可以爆
004074C6   > 8A4C04 08      mov     cl, byte ptr [esp+eax+8]
004074CA   . 8888 F0AA4100  mov     byte ptr [eax+41AAF0], cl
```

整个过程十分常规，而且出现了明文。

相信大家已经注意到了，注册码是由 `ether.dll` 的导出函数 `reg_code` 产生的，程序通过这个 `reg_code` 得到明文注册码，然后和输入的注册码进行比较，从而判断注册是否正确。

显然，十分显然，我们也可以 Call 这个 `reg_code` 来获取注册码(弄到这里我真的不知道软件作者怎么想的，他似乎以为世界上的人都不知道有导出函数这种东西!-.-)

我们用 `Depneds` 看一下 `ether.dll` 就可以知道。

![](img/da4853f45f39def6f56baa6c6122cba4.jpg)

而且，导出函数的形式可以从反汇编代码得出，就是这样：

```
typedef int (*REG)(char *p,char *b);
```

其中第一个为注册名，第二个是注册码，返回时输出就可以。

所以，C 代码就出来了：

```
///
typedef int (*REG)(char *p,char *b);

int main()
{
    char name[100];
    char reg[10];
    printf("Easy Video to iPod MP4 PSP 3GP Converter v1.3.7\n");
    printf("KeyGen\nPowered By XiaoWei[0GiNr]\n\n");
    printf("0GiNr Studio: http://www.0GiNr.com\n");
    printf("XiaoWei's Zoo: http://hi.baidu.com/Zoo_\n\n");

    HMODULE h = ::LoadLibrary("ether.dll");
    if(!h)
    {
        printf("注册机应和 Easy Video to iPod MP4 PSP 3GP Converter 在同一目录下！\n");
        return 0;
    }

    REG Addr = (REG)::GetProcAddress(h,"reg\_code");
    if(Addr != NULL)
    {
        printf("输入用户名:\n");
        scanf("%s",&name);
        Addr(name,reg);
        printf("注册码：\n%s\n",reg);
    }

    ::FreeLibrary(h);
    printf("Remember to copy.....⁰^\nHave a good time.\n");
    system("pause");
    return 0;
}
```

还有，这种方法据说可以通过该工作室的全部软件。。

效果图：

![](img/6055d3d3d058aa19e6d300daac7935d8.jpg)
![](img/63a2b466b026021182c6a6521e81de37.jpg)

![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷") ![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

***

## 【经验总结】

编程其实也可以破解。大牛就分析下算法吧，毕竟这样可以锻炼在自己的反汇编能力。

***

## 【版权声明】

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2008 年 01 月 26 日 PM 02:28:42
