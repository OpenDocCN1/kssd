
# 立体画梦工厂专业版 V2.05

> 原文：[`www.pediy.com/kssd/pediy06/pediy6076.htm`](https://www.pediy.com/kssd/pediy06/pediy6076.htm)

## 破文信息

破文标题: 立体画梦工厂专业版-算法分析\[简单\]
破解成员: `LeNgHoSt\[DFCG\]\[CCG\]`
被破程序: `立体画梦工厂专业版.exe`
破解工具: `trw2000`
破解目的: 学习和交流加密解密技术, 仅供参考

## 软件信息

软件名称: 立体画梦工厂专业版
软件版本: `2.05`
软件大小: `1.85M`
应用平台: `Windows 9x/Me/2k/XP`
下载地址: `http://bmpto3d.51.net/`

## 软件简介

一幅画表面看上去平淡无奇，眼睛调节焦距后画中立刻呈现一方立体景象，显得玲珑剔透，栩栩如生。这就是被誉为“浮在纸面上的艺术”的电脑立体画。

《立体画梦工厂》将会激发您的无限创意，帮助您在数分钟内量身定做出属于自己的精美立体画。

## 破解过程

用 `trw2000` 加载程序，输入假注册码 `787878`，`ctrl+n` 呼出 `trw2000`，`bpx hmemcpy`。

回到程序，点击注册，程序被拦截，`f12`，`f10` 几次来到这里：

```
0046B2A3  |. E8 5886FCFF    CALL LTH.00433900      ;  我们来到这里
0046B2A8  |. 84C0           TEST AL,AL
0046B2AA  |. 0F84 99000000  JE LTH.0046B349        ;  一定不会跳
0046B2B0  |. 8D4D EC        LEA ECX,DWORD PTR SS:\[EBP-14\]
0046B2B3  |. 8BC6           MOV EAX,ESI;  看看 eax 是什么？哈哈是机器码的 16 进制
0046B2B5  |. C1E0 03        SHL EAX,3  ;  eax 左移 3 位，相当于乘以 8
0046B2B8  |. 2BC6           SUB EAX,ESI;  再减去原值
0046B2BA  |. BA 04000000    MOV EDX,4
0046B2BF  |. E8 14D6F9FF    CALL LTH.004088D8
0046B2C4  |. 8B4D EC        MOV ECX,DWORD PTR SS:\[EBP-14\];  注册码的后半部分
0046B2C7  |. 8D45 F8        LEA EAX,DWORD PTR SS:\[EBP-8\]
0046B2CA  |. BA B4B34600    MOV EDX,LTH.0046B3B4   ;  字符常量"Z"
0046B2CF  |. E8 8494F9FF    CALL LTH.00404758      ;  将"Z"和注册码连接起来
0046B2D4  |. 8B45 F8        MOV EAX,DWORD PTR SS:\[EBP-8\] ;  完整注册码
0046B2D7  |. 8B55 FC        MOV EDX,DWORD PTR SS:\[EBP-4\] ;  假注册码
0046B2DA  |. E8 7195F9FF    CALL LTH.00404850      ;  比较
0046B2DF  |. 74 1A          JE SHORT LTH.0046B2FB  ;  不等就 OVER
0046B2E1  |. 6A 00          PUSH 0
0046B2E3  |. B9 B8B34600    MOV ECX,LTH.0046B3B8
0046B2E8  |. BA C0B34600    MOV EDX,LTH.0046B3C0
0046B2ED  |. A1 3CD14600    MOV EAX,DWORD PTR DS:\[46D13C\]
0046B2F2  |. 8B00           MOV EAX,DWORD PTR DS:\[EAX\]
0046B2F4  |. E8 8FEDFEFF    CALL LTH.0045A088
0046B2F9  |. EB 4E          JMP SHORT LTH.0046B349
0046B2FB  |> BA 02000080    MOV EDX,80000002
0046B300  |. 8BC3           MOV EAX,EBX
0046B302  |. E8 ED49FFFF    CALL LTH.0045FCF4
0046B307  |. B1 01          MOV CL,1
```

### 算法总结

注册码 = `"Z"` + 机器码的 16 进制 * 7

如我的机器码为：`1026559`

转换为 16 进制为：`FA9FF`

乘以 7 得：`6DA5F9`

所以注册码为：`Z6DA5F9`

注册信息存放在 `HKEY\_LOCAL\_MACHINE\\Software\\Microsoft\\Windows\\CurrentVersion\\Cbmpreg`

删除此键值即变成未注册。

### KeyGen 代码

```c
2003-05-29 18:00      MaDe By:CrAcK aRtIcLe MaKeR

main()
{
    unsigned long i;
    printf("\n\nBmpto3D V2.10 KeyGen by *李逍遥[cschina]*\n\n 请输入你的 ID:");
    scanf("%lu", &i);
    i = i << 3;
    printf(" 你的注册码:Z%lX\n\n", i);
}
```

`\*\*\*\*\*\*\*\*\*\*\*\*\*TC2.0 编译通过\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

`4dd466d0e1e0cc123afbe504da2462ca`
