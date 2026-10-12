
# WMV to AVI VCD Converter v2.3 自校验解除

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-419.htm`](https://www.pediy.com/kssd/pediy07/pediy7-419.htm)

`WMV to AVI MPEG WMV VCD SVCD DVD Converter v2.3` 自校验解除

国外的一款 `WMV` 格式转 `mpeg` `vcd`... 等格式的软件，汉化新世纪一位朋友要做其汉化。发给我了要我看看。该程序为 `aspack2.12` 壳，`delphi` 编译的程序。高手们就不要看了。

脱壳后，运行几秒后程序自动退出，没有一点提示，比起那些不声不响的不运行的程序好多啦~Q~

拿出黑哥修改后的 `DEDE` 进行反编译，既然是在程序主界面显示后自动退出的，就来看看 `FromShow` 的事件吧，在 `DEDE` 中找到 `FromShow` 的事件反汇编得到如下代码：

```
004E0D20   8B833C040000           mov     eax, [ebx+$043C]  
004E0D26   8B08                 mov     ecx, [eax]  
004E0D28   FF5164                call    dword ptr [ecx+$64]  
004E0D2B   8D55FC                lea     edx, [ebp-$04]  
004E0D2E   8BC3                 mov     eax, ebx  
004E0D30   E8DBF8FFFF            call    004E0610           //获取文件创建日期 进入此 CALL 看一看  
```

### 0x4e0d30 CALL 处分析

这里只贴出关键代码

```
0040951B   E8A4D8FFFF           call    00406DC4           // 使用 findfirstfile 函数查找程序自身文件名  
00409520   83F8FF               cmp     eax, -$01  
00409523   7434                 jz      00409559  
00409525   50                   push    eax  
00409526   E891D8FFFF           call    00406DBC           //关闭由 FindFirstFile 函数创建的一个搜索句柄  
0040952B   F685B4FEFFFF10       test    byte ptr [ebp+$FFFFFEB4], $10  
00409532   7525                 jnz     00409559  
00409534   8D45F4               lea     eax, [ebp-$0C]  
00409537   50                   push    eax  
00409538   8D85C8FEFFFF         lea     eax, [ebp+$FFFFFEC8]  
0040953E   50                   push    eax  
0040953F   E870D8FFFF           call    00406DB4          //利用 FileTimeToLocalFileTime 函数将文件时间转换成本地时间  
00409544   8D45FC               lea     eax, [ebp-$04]  
00409547   50                   push    eax  
00409548   8D45FE               lea     eax, [ebp-$02]  
0040954B   50                   push    eax  
0040954C   8D45F4               lea     eax, [ebp-$0C]  
0040954F   50                   push    eax  
00409550   E857D8FFFF           call    00406DAC          //FileTimeToDosDateTime 将文件时间值转换成 DOS 日期时间值  
00409555   85C0                 test    eax, eax  
00409557   7507                 jnz     00409560  
00409559   C745FCFFFFFFFF       mov     dword ptr [ebp-$04], $FFFFFFFF  
```

```
004E0D35   8B45FC               mov     eax, [ebp-$04]        //将转换后的日期送到 EAX  
004E0D38   BA7C0D4E00           mov     edx, $004E0D7C        //原始程序（未脱壳的）的创建日期'2005-05-10'送到 EDX  
004E0D3D   E8223EF2FF           call    00404B64              //两个日期进行对比  
004E0D42   740D                 jz      004E0D51              //相等吗？相等就跳走，相等意味程序没有被脱壳或者没有被修改  
004E0D44   B201                 mov     dl, $01               //没有跳就下执行，执行就等死吧~Q~  
004E0D46   8B83C0030000         mov     eax, [ebx+$03C0]  
```

以上分析得出结果，程序首先查找自身，然后利用几个操作文件函数得到自身的创建日期并转换为本地时间，然后与内置的日期进行对比，如果相等就继续执行，否则退出程序。那么针对此我们可以更改脱壳后的文件时间来骗取它，使其日期相等....但.....

修改 `0x4e0d42` 处为无条件跳转后，程序是可以执行了，但是在添加转换文件功能执行的时候又退出了，那么说明程序还有校验。汗一个先，拿出 `DEDE` 继续分析。我们来看一下程序 `addfile` 按钮事件。在 `dede` 反汇编 `btnAddClick` 事件代码如下：

```
004DF1C3   E88045FBFF           call    00493748                           //在此下断点  
004DF1C8   A184BE4E00           mov     eax, dword ptr [$004EBE84]          // 添加后的文件指向 eax  
```

|
004DF1CD   E892130000           call    004E0564                          // 进入此 CALL  

此 CALL 关键代码如下：

```
00403283  |.  6A 00          PUSH    0                                  
00403285  |.  8B03          MOV     EAX, [EBX]                       
00403287  |.  50            PUSH    EAX                               
00403288  |.  E8 6BE0FFFF   CALL    004012F8                           //调用 GetFileSize 函数  
0040328D  |.  8BF0          MOV     ESI, EAX                           // 获取当前文件的大小  
0040328F  |.  83FE FF       CMP     ESI, -1  
00403292  |.  75 07         JNZ     SHORT 0040329B                     
00403294  |.  E8 E7F6FFFF   CALL    00402980                           
00403299  |.  EB 15         JMP     SHORT 004032B0                     
0040329B  |>  8BC6          MOV     EAX, ESI  
```

```
004DF1D2   3D00080700           cmp     eax, $00070800             // eax 中放的是当前文件的大小，0x70800 为原始文件大小  
004DF1D7   740D                 jz      004DF1E6                   // 判断文件大小是否相等，就继续执行... 修改 740D→EB0D  
004DF1D9   B201                 mov     dl, $01  
```

脱壳后的文件大小：`1708544 bytes` 原始程序文件大小：`0x70800` 转换十进制为 `460800 bytes`。

此程序在 `Convert file` 的时候还有校验，同上面一样，也是通过 `getfilesize` 来比较文件。在这里就不赘述了. 有兴趣的朋友可以反汇编 `btnConvertClick` 事件看一下。

ps：我发现我灌水能力特强的说，写起正经的文字显得十分的苍白无力。或许技术太菜了，呵呵，高手勿笑......

狗剩[`BCG`][`D.4S`][`DCM`]

学会了 2 个 anti：

1. 使用 `findfirstfile` 函数查找程序自身文件名
   利用 `FileTimeToLocalFileTime` 函数将文件时间转换成本地时间
   `FileTimeToDosDateTime` 将文件时间值转换成 `DOS` 日期时间值
   两个日期进行对比
2. 调用 `GetFileSize` 函数，判断文件大小

> 引用:
>
> *最初由 ttboy 发布*
>
> **好文章，真是及时雨，前两天正是分析一个自校验的没有搞定，看到这贴子又有一点信心去搞啦。**
>
> **顶！**

论坛里有很多关于自校验的文章：

[`bbs.pediy.com/showthread.php...p;threadid=1852`](http://bbs.pediy.com/showthread.php?s=&threadid=1852)
[`bbs.pediy.com/showthread.php...;threadid=17275`](http://bbs.pediy.com/showthread.php?s=&threadid=17275)
[`bbs.pediy.com/showthread.php...;threadid=16622`](http://bbs.pediy.com/showthread.php?s=&threadid=16622)
[`bbs.pediy.com/showthread.php...;threadid=15801`](http://bbs.pediy.com/showthread.php?s=&threadid=15801)
[`bbs.pediy.com/showthread.php...;threadid=15444`](http://bbs.pediy.com/showthread.php?s=&threadid=15444)
[`bbs.pediy.com/showthread.php...;threadid=15216`](http://bbs.pediy.com/showthread.php?s=&threadid=15216)
[`bbs.pediy.com/showthread.php...;threadid=14860`](http://bbs.pediy.com/showthread.php?s=&threadid=14860)
[`bbs.pediy.com/showthread.php...;threadid=14368`](http://bbs.pediy.com/showthread.php?s=&threadid=14368)
[`bbs.pediy.com/showthread.php...;threadid=14223`](http://bbs.pediy.com/showthread.php?s=&threadid=14223)
[`bbs.pediy.com/showthread.php...;threadid=14001`](http://bbs.pediy.com/showthread.php?s=&threadid=14001)
[`bbs.pediy.com/showthread.php...;threadid=13554`](http://bbs.pediy.com/showthread.php?s=&threadid=13554)
[`bbs.pediy.com/showthread.php...;threadid=11219`](http://bbs.pediy.com/showthread.php?s=&threadid=11219)

............（略）

请多用论坛搜索功能！![](img/d8960f6c7afa20ba0ca27920887f4540.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
