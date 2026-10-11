
# Delphi 中参数的传递和函数值的返回

> 原文：[`www.pediy.com/kssd/pediy06/pediy6572.htm`](https://www.pediy.com/kssd/pediy06/pediy6572.htm)

关于 Delphi 中参数的传递和函数值的返回

**前言：**

高手们应该早知道了，不屑于写出来而已。真正的高手一个比一个潜的深，只剩下偶这样的小菜写些菜文给更小的菜。高手看时还请捂好大牙，多多指点。

不知各位小菜同胞对破解 `DELPHI` 程序有什么看法，反正我的感觉就一个字：怪。各位最先遇到的问题恐怕都是：我下了 `GetDlgItemInt`、`GetDlgItemText`、`GetWindowText`....怎么什么也断不下来，甚至连 `Hmemcpy` 都不起作用？呵呵，从这里就能看出宝蓝的那批人成心想跟 `M$` 对着干，非搞出些新鲜的东东不可。

这回我们就来看看 `DLEPHI` 中对函数（过程）参数的传递是如何进行的。

我们知道 `WinAPI` 采用的调用约定是 `StdCall`，也就是调用一个函数 `Func(arg1, agr2, agr3, arg4)`，你需要 `push arg4, push arg3, push arg2, push arg1, call Func`。在 `VC++` 里也是这种形式，所以一个函数有几个参数，可以非常直观地看出来。可是在 `DELPHI` 中就很奇怪了，在一个 `CALL` 前面你可能一个 `PUSH` 也看不到，怎么回事呢？听我慢慢道来。

`DELPHI` 中的调用约定有 `StdCall`、`Cdecl`、`Safecall`、`Pascal` 和 `Register` 等几种方式，而 `DELPHI` 的默认方式是 `Register`（为什么不是 `Pascal`？）`Register` 方式就是尽可能地使用寄存器来传递参数，减少堆栈的操作来提高速度。具体情况是怎样呢，看个例子先：

在 `FORM` 上放一个 `BUTTON`，双击写代码如下：

```
代码:
function add1(a:Integer):Integer;    //一个参数
begin
    add1:=a+a;
end;

function add2(a,b:Integer):Integer;    //两个参数
begin
    add2:=a+b;
end;

function add3(a,b,c:Integer):Integer;    //三个参数
begin
    add3:=a+b+c;
end;

function add4(a,b,c,d:Integer):Integer;    //四个参数
begin
    add4:=a+b+c+d;
end;

function add5(a,b,c,d,e:Integer):Integer;  //五个参数
begin
    add5:=a+b+c+d+e;
end;

function add6:Integer;        //加入一些局部变量
var local1,local2,local3,local4,local5:Integer;
begin
    local1:=1;
    local2:=2;
    local3:=3;
    local4:=4;
    local5:=5;
    add6:=local1+local2+local3+local4+local5;
end;

function add7(a,b,c,d,e:Integer):Integer;  //利用 result 来返回
begin
    result:=a+b+c+d+e;
end;

function add8(a,b,c,d,e:Integer):Integer;StdCall;//StdCall 调用方式
begin
    add8:=a+b+c+d+e;
end;

procedure TForm1.Button1Click(Sender: TObject);
var a,b,c,d,e:Integer;
    s1,s2,s3,s4,s5,s6,s7,s8,s:Integer;
begin
    a:=1; b:=2; c:=3; d:=4; e:=5;
    s1:=add1(a);
    s2:=add2(a,b);
    s3:=add3(a,b,c);
    s4:=add4(a,b,c,d);
    s5:=add5(a,b,c,d,e);
    s6:=add6;
    s7:=add7(a,b,c,d,e);
    s8:=add8(a,b,c,d,e);
    s:=s1+s2+s3+s4+s5+s6+s7+s8;       //必须要有这么几句
    MessageDlg(IntToStr(s),mtConfirmation,\[mbOK\],0);  //不然编译器根本不去处理返回值
end;
```

用 `DEDE` 反一下看看，这个 `Button1Click` 的内容：
