
# 我的第一个脚本-- krypton 的脱壳脚本

> 原文：[`www.pediy.com/kssd/pediy10/88551.html`](https://www.pediy.com/kssd/pediy10/88551.html)

前些天我全程分析了 `krypton` 这个老壳。

文章论坛地址：

[`bbs.pediy.com/showthread.php?t=87760&highlight=`](http://bbs.pediy.com/showthread.php?t=87760&highlight=)

另外逛论坛的时候我发现 `CCDebuger` 大牛已经在一个求助帖写出来了，对比一下发现我写的脚本和大牛的只是定位代码有些不一样，有的抄的嫌疑![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸") 我写了详细的注释，希望像我一样的菜鸟能结合我前面分析壳的文章熟悉一下写脚本的思路。

我最近在学习脚本，这是我第一个脚本：

（试炼品见附件）

```
var temp

bphwcall

dbh

/*
壳程序先分配内存然后进行解码，分配内存地址存放到 esi 中
这句 jmp esi 是为了跳到这段代码中
*/

findop eip, #FFE6#                         ;找到    FFE6            jmp esi
mov temp, $RESULT
bp temp
esto
bc
sto                     ;f8

/*
如果不对下面的代码下断的话，我发现定位加密输入表的指令定位的不对，如果过了下面的代码之后，就可以定位到
下面代码就是下面的代码：
003A28A6     FF95 50BE4100          call dword ptr ss:\[ebp+41BE50\]            ; kernel32.GetProcAddress
003A28AC     83BD 6F714000 00       cmp dword ptr ss:\[ebp+40716F\],0
*/

find eip, #FF95????????83BD#
mov temp, $RESULT
bp temp
esto
bc

find eip, #8B85????????EB#
mov temp, $RESULT
add temp, 6

/*查找以下命令：
003a30D5    8B85 A6714000   MOV EAX,DWORD PTR SS:\[EBP+4071A6\]     ; 输入表地址写入的，NOP 掉
*/

find temp, #8B85????????EB#
mov temp, $RESULT
mov [temp], #909090909090#

/*查找以下命令：
003A3A5E  - FFE2            jmp edx                                  ; KRYPTON.00401000 跳向 oep
003A3A60     51            push ecx
*/

find temp, #FFE251#
mov temp, $RESULT
bphws temp, "x"
esto
bphwc
sto
cmt eip, " OEP，请用 IR 重建输入表"
ret
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `KRYPTON.rar` \[解压密码：`PEDIY`\] |
| :--- | :--- |
