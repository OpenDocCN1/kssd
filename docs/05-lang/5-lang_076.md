
# .net 程序修改

> 原文：[`www.pediy.com/kssd/pediy06/pediy6581.htm`](https://www.pediy.com/kssd/pediy06/pediy6581.htm)

小弟最近一直在做 `.net` 下串行通信方面的程序，`.net` 下没有自带的串口控件，使用以前的 `com` 组件又不甘心，所以在网上找到了 `sax.net` 组件（破解）版本，使用的时候，比较满意，但是在使用的过程中，出现了

```
System.ArgumentOutOfRangeException: 长度不能小于 0。

参数名: newlength

   at System.Text.StringBuilder.set_Length(Int32 value)
   at Sax.Communications.SerialConnection.a(UInt32 A_0)
   at Sax.Communications.SerialConnection.a(IntPtr A_0, c& A_1)
   at Sax.Communications.SerialConnection.get_Available()
   at Sax.Communications.SerialConnection.b()
```

异常

尤其是使用 `ISP` 线与单片机通信的时候，几乎每次插拔 `ISP` 线都会出现异常。异常后，控件的串口数据监视线程退出以后再也不能收发数据，只能重新启动软件。经过仔细调试，发现

异常出现在插拔数据线的时候，有数据收到，产生了 `DataAvailable` 事件中

```csharp
private void serialConnection_DataAvailable(object sender, EventArgs e) // 受到数据时的处理函数
{
   // 在这之前出现了异常
   int length = serialConnection.Available;

//   Debug.WriteLine(length.ToString());

   if (length <= 0) return;

   Byte[] buffer = new byte[length]; // 输入缓冲区有多少数据？
   serialConnection.Read(buffer, 0, length);

   // 分析数据
   if (length > 0)
   {
    HandleDataReceive(buffer);
   }
}
```

就是在发生事件之后，在 `int length = serialConnection.Available;` 之前发生了异常。根据 `VS.net`

的提示信息，初步确定是在给一个 `StringBuilder` 对象 `builder` 设置其 `Length` 的时候，传入的参数 <0 导

致的。

好了，使用 `.net` 的反编译工具 `Reflector`（我用的版本 `4.0.18`）打开控件，找到 `SerialConnection` 类；由于此控件经过混淆，所以在反编译后出现了大量的比如 `a`，或者 `b` 这样的看不出作者意图的函数名。不过没有关系，因为异常出现在 `serialConnection.Available` 附近，直接找到 `Available` 的实现代码

```csharp
public override int get_Available()
{
     z.c c1;
     this.c();
     c1 = new z.c();
     this.a(this.a, ref c1);
     if (this.b != -1)
     {
         return ((int) (c1.b + 1));
     }
     return ((int) c1.b);
}
```

在里面寻找出错的函数

找到 `this.a(this.a, ref c1);` 这个

```csharp
private string a(IntPtr A_0, ref z.c A_1)
{
     uint num1 = 0;
     z.ClearCommError(A_0, ref num1, ref A_1);
     if (num1 != 0)
     {
         return this.a(num1);
     }
     return null;
}
```

有追踪到 `return this.a(num1);`

```csharp
private string a(uint A_0)
{
     StringBuilder builder1 = new StringBuilder("UART Error:", 60);
     if (builder1.Length <= 0)
     {
         goto Label_005F;
     }
     goto Label_00D0;

Label_001B:
     builder1 = builder1.Append(" Parity,");
     goto Label_00B1;

Label_002C:
     builder1 = builder1.Append(" Receive Overflow,");
     goto Label_0069;

Label_003D:
     builder1 = builder1.Append(" Framing,");
     goto Label_0095;

Label_004E:
     builder1 = builder1.Append(" Transmit Overflow,");
     goto Label_00A3;

Label_005F:
     if ((A_0 & 8) == 0)
     {
         goto Label_0095;
     }
     goto Label_003D;

Label_0069:
     if ((A_0 & 4) == 0)
     {
         goto Label_00B1;
     }
     goto Label_001B;

Label_0073:
     builder1 = builder1.Append(" <Unknown>,");
     goto Label_00FA;

Label_0084:
     builder1 = builder1.Append(" Overrun,");
     goto Label_00ED;

Label_0095:
     if ((A_0 & 1024) == 0)
     {
         goto Label_00E3;
     }
     goto Label_00BF;

Label_00A3:
     if ((A_0 & 1287) == 0)
     {
         goto Label_00FA;
     }
     goto Label_0073;

Label_00B1:
     if ((A_0 & 256) == 0)
     {
         goto Label_00A3;
     }
     goto Label_004E;

Label_00BF:
     builder1 = builder1.Append(" IO,");
     goto Label_00E3;

Label_00D0:
     builder1.Remove(0, builder1.Length);
     goto Label_005F;

Label_00E3:
     if ((A_0 & 2) != 0)
     {
         goto Label_0084;
     }

Label_00ED:
     if ((A_0 & 1) == 0)
     {
         goto Label_0069;
     }
     goto Label_002C;

Label_00FA:
     builder1.Length -= 1;//!!!!!!!!!!!!!!!!!! 错误的来源
     return builder1.ToString();
}
```

我靠，也没有想细读为什么要将 `builder1.Length` 减去 1，节省内存，没有必要？！！

反正是这句出错了，将后 `builder1.Length` 小于 0 了

查看前面的代码，没有对 `builder1.Length` 赋值的语句，看来将这句改掉就行了，

查看 `il` 代码，

最后的几句为

```il
L_00fa: ldloc.0
L_00fb: ldloc.0
L_00fc: callvirt StringBuilder.get_Length
L_0101: ldc.i4.1 // 装载常数 1 -- 将此处改为装载常数 0，即可
L_0102: sub
L_0103: callvirt StringBuilder.set_Length
L_0108: ldloc.0
L_0109: callvirt StringBuilder.ToString
L_010e: ret
```

查找 `il` 的汇编指令

发现（格式指令 --- 十六进制数）

```text
ldloc.0 ---- 06
ldloc.0 ---- 06
callvirt StringBuilder.get_Length ---- 6F X X X X (X 不知道具体的值，可以通过模糊匹配来确定要修改的指令在文件的位置)
ldc.i4.1 ---- 17
sub ----  59
L_0103: callvirt  ---- StringBuilder.set_Length 6F X X X X
L_0108: ldloc.0  ---- 06
L_0109: callvirt  ---- StringBuilder.ToString 6F X X X X
L_010e: ret  ----  28
```

使用 `WinHEx` 打开文件，使用 16 进制搜索，对于上面的 `X` 使用模糊匹配的原则

找到偏移 `0x34B0` 处，在 `0x34BD` 处的将 `0x17` 改为 `0x16` (`IL` 指令 `ldc.i4.0` ，就是将 `Length-0` 等于不减)

存盘退出，将修改好的 `dll` 文件覆盖原来的 `dll`，运行软件，怎么插拔 `ISP` 线都没有问题出现了！

## 参考资料

`Ｉｎｓｉｄｅ　Ｍｉｃｒｏｓｏｆｔ．ｎｅｔ　ＩＬ　Ａｓｓｅｓｍｂｌｅｒ`

`4dd466d0e1e0cc123afbe504da2462ca`
