
# RegRun2 V3.30

> 原文：[`www.pediy.com/kssd/pediy05/pediy50322.htm`](https://www.pediy.com/kssd/pediy05/pediy50322.htm)

`http://wycrack.51.net/learn/un_RegRun2.zib`

下载完后改后缀名为`ZIP`

## RegRun2 V3.30 脱壳说明

大家如对这篇文章有不懂之处，请先看`[DFCG]`脱壳教学之`aspr`引入表修复篇。`http://dfcg.9842.org/`

### Regrun2.exe 的脱壳过程

1. 用`loader`找到壳的入口点：`OEP=00401000`
2. 在入口点`dump`下程序保存为`un_regrun2.exe`
3. 用`W32dasm`反编译程序查找字符串：`jmp dword ptr [00`
   找到引入表的范围在：`0058a174`，大小为：`00000ec0h`
4. 在`un_regrun2.exe`中找一片全`00`地区。我找的是：`005e3000`，大小为：`000017000h`
5. 运行`[DFCG]`的`un_aspr1.0`，在窗口中输入`0058a174`和`005e3000`.
6. 按确定.在出现第二个提示窗口的时候，记下正确的引入表入口点：`001e3108`.
   用`lordpe`, `dump`出`0058a174`后的`ec0h`字节数据和`005e3000`后的`000017000h`的数据.
   保存为`0058a174~ec0.dmp`和`005e3000~17000.dmp`文件.
7. 用`dump`出的数据替换`un_regrun2.exe`中对应地址的数据.
8. 改正引入表入口点为`001e3108`.
9. 修复其他的几个`aspr`特殊`API`.如下表:

#### KERNEL32.DLL 中的未修复函数

`0018a434`    空函数        先让其指向`Getcommandlinea`函数,以确保函数的连续性，但修改`0053afe2`处的指令为:`RETN 04,`如下:

```
0053AFDC          $-FF25 30A45>JMP DWORD PTR DS:[<&KERNEL32.F>;  kernel32.FreeLibrary
0053AFE2          $ C2 0400     RETN 4
0053AFE5             90         NOP
0053AFE6             90         NOP
0053AFE7             90         NOP
0053AFE8          $-FF25 38A45>JMP DWORD PTR DS:[<&KERNEL32.G>;  kernel32.GetACP
```

`0018a440`    `GetCommandLineA`
`0018a444`    `GetCurrentProcess`
`0018a448`    `GetModuleHandleA`
`0018a488`    `GetProcAddress`

`0018a518`    空函数        先让其指向`Getcommandlinea`函数,以确保函数的连续性，但修改`0053b138`处的指令如下:

```
0053B132          $-FF25 14A55800     JMP DWORD PTR DS:[<&KERNEL32.L>;  kernel32.LocalFree
0053B138          $ C2 0400           RETN 4
0053B13B             90              NOP
0053B13C             90              NOP
0053B13D             90              NOP
```

#### COMCTR32.DLL 中的未修复函数

`0018a6bc`    `InitCommonControls`

### 结束语

附上脱壳后再用`upx1.20`压缩的`regrun2.exe`.其他的几个程序也可用这种方法脱壳。引入表的需修复函数相似.

-={完成}=- `4dd466d0e1e0cc123afbe504da2462ca`
