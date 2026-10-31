
# Vopt99另类破解实战录

(3千字)

## 用`Trw2000`破解`Vopt99`实战录

`Vopt99`是一个用作整理硬盘碎片的磁盘工具（用VB5.0编成的，为什么我会知道？因为我用了`UtrlEdit`打开过`Vopt99.exe`时看见的嘛！），而且有30天的使用限制，除之以外就没有其它限制。所以把时间限制去除，就算是破解完成了。`Loder32`或`Trw2000`的用法请浏览“看雪学宛”主页，那里有`Soft-Ice`、`Trw2000`的使用价绍和下载。本文就假设你已基本了解`Soft-Ice`、`Trw2000`这两个软件。

工件开始啦！

### 1. 首先，把`Vopt99`安装一次

输入你的大名、公司名和注册码，`Vopt99`提示你如没得到注册码，可以用`TRIAL`来作试用并按提示完成安装。安装完成后，把系统日期调至使`Vopt99`叫出时间限对话筐面的日期，先运行`Trw2000`，再运行`Vopt99`，接着`Vopt99`要求现在注册，但不要这个时候按注册键，`Ctrl-N`呼出`Trw2000`，按数次`F12`功能键，此时`Trw2000`返回`Vopt99`主程序，是时候按取消键啦！此时又返回`Trw2000`（因为不知道`Vopt99`调用的是什么函数所以只好这样做嘛……），接着按数次`F10`功能键，返回`VOPT99!TEXT+XXXXX`的领空，注意！看当前光标上一行！

```
0177:00437E1E  56                    PUSH ESI
XXXX:XXXX7E1F  FF97B0020000          CALL NEAR [EDI+02B0]
XXXX:XXXX7E25  85C0                  TEST EAX,EAX   ;返回时光标所在，可见上面那个CALL
XXXX:XXXX7E27  7D12                  JNL 00437E3B   ;个CALL的用途。
...
XXXX:XXXX7E3B  66833D4296450000      CMP WORD [00459642],BYTE+00
XXXX:XXXX7E43  7469                  JZ 00437EAE    ;JUMP
...
XXXX:XXXX7E16  C7458C00FA4000        MOV DWORD [EBP-74],0040FA00
;用D命令看一看0040FA00 ，你就看会见
;Thank you for your using our product!
;可见上面那个JZ 00437EAE 有问题!那把
;它修改为 90 90
```

按几下`F10`功能键来到这里

```
XXXX:XXXX7EXX CALL MSVBVM50!rtcMsgBox
            ;出现了注册成功的窗口
```

由此可见，就这个`CALL MSVBVM50!rtcMsgBox`带出`Vopt99`提示时间限制画面！原来`Vopt99`是用了这个函数！那么今后就要知道要拦截VB就以可`Bpx rtcMsgBox`来放置段点啦！

按`F5`功能键，看看结果......退出？？难道还有其它地方要修改？

### 2. 没办法只好再来一次!

当带过上面那`XXXX:XXXX7EXX CALL MSVBVM50!rtcMsgBox`时（此时已返回`Vopt99`主体程序），按确认键（返回`Trw2000`），第二次返回`Vopt99`的领空

```
0177:00431AE0  837D8800              CMP DWORD [EBP-78],BYTE+00
XXXX:XXXX1AE4  7D20                  JNL 00431B06
            ;此处Jump并会退出Vopt99，于是把它修改为90 90
...
XXXX:XXXXXXXX  0FBF0D42964500
XXXX:XXXX1B1E  85C9                  TEST ECX,ECX
XXXX:XXXX1B20  7536                  JNZ 00431B58
            ;此处不Jump并会退出Vopt99，于是把它修改为EB 36
XXXX:XXXXXXXX  C745FC2F000000
...
```

再一次按`F5`功能键看看...... Bingo !

## 整理：

运行16进制编辑器如：`UtrlEdit` 打开 `Vopt99.exe` 找刚才抄下的机器代码

1. 找 `56 FF 97 B0 02 00 00 85 C0`，把 `FF 97 B0 02 00 00` 改为 `90 90 90 90 90 90`
2. 找 `66 83 3D 42 96 45 00 00 74 69`，把 `74 69` 改为 `90 90`
3. 找 `83 7D 88 00 7d 20`，把 `7d 20` 改为 `90 90`
4. 找 `0F BF 0D 42 96 45 00 85 C9 75 36`，把 `75 36` 改为 `EB 36`

储存后退出并运行`Vopt99.exe`，试一试。看见了吗？这和已注册有何区别？

Cracker : BanhouseMaster  
E-Mail  : banhouse@21cn.com  
Phone  : (0755)13613070052
