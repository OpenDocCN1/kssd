
# jslwl

> 原文：[`www.pediy.com/kssd/pediy01/forum088.htm`](https://www.pediy.com/kssd/pediy01/forum088.htm)

`http://ilike.myrice.com:8080/cgi-bin/dl.pl?url=ftp://ftp.zj.ilike.myrice.com/ilike/soft/soft_cai/jslwl.zip`

比较注册码的关键代码如下，又是浮点运算，记得用 `wf` 打开 `SoftICE` 的浮点寄存器窗口。

它是这样判断注册码的：

```
if ( (真注册码 减去 假注册码) < 10 的负 8 次方)
{
    good guy;
}
else
{
    bad guy;
}
```

这样的话在执行下面的 `FSUBR` 指令之前看一下 `ST0` 寄存器中的值，这个值就是真正的注册码（取 8 位小数的精度即可）。

一个可用的注册码（注意密码是个负小数）：

学校：`Internet`
姓名：`Cracker`
密码：`-0.93016765`

```
0167:00437207  FSUBR   REAL8 PTR [EBP-6C]  //寄存器 ST0 中的正确注册码减去你输入的假注册码
0167:0043720A  ADD     ESP,-08
0167:0043720D  FSTP    REAL8 PTR [ESP]
0167:00437210  CALL    004AAB94
0167:00437215  ADD     ESP,08
0167:00437218  FCOMP   REAL8 PTR [00437480]  //差应小于 10 的负 8 次方
0167:0043721E  FSTSW   AX
0167:00437220  SAHF
0167:00437221  JAE     00437408      //差大于 10 的负 8 次方就错了
```
