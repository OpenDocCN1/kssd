
# QQ2000c 0630

> 原文：[`www.pediy.com/kssd/pediy05/pediy50203.htm`](https://www.pediy.com/kssd/pediy05/pediy50203.htm)

**标题**：为 `QQ2000c 0630` 添加 IP 显示和去掉广告

**作者**：`soff[CoRAl]`

**主页**：`http://www.soff.net`

## (1) 用 `LordPE` 加入 `IPSearcher!_GetAddress`

如果手工做，可以保证 `exe` 大小无变化，既然 `LordPE` 已经有了这功能，我这种菜鸟就直接用了 :P

**下载地址**：

`http://www.pediy.com/temp/lordpe/lordpe.htm`

## (2) 去掉广告

修改 `QQAdManager.dll`

```
.1000C029: FF1590910210          call        GetLastError ;KERNEL32.dll
.1000C02F: 3DB7000000            cmp         eax,0000000B7
.1000C034: 7419                  je          .01000C04F ;此行改为：
.1000C034: EB19                  jmps        .01000C04F
.1000C036: 8BCE                  mov         ecx,esi
.1000C038: E82DD9FFFF            call        .01000996A
.1000C03D: FF7650                push        d,[esi][50]
.1000C040: 8B4E04                mov         ecx,[esi][04]
.1000C043: E8D852FFFF            call        .010001320
.1000C048: 57                    push        edi
.1000C049: FF1588910210          call        ReleaseMutex ;KERNEL32.dll
.1000C04F: 57                    push        edi
```

搜索 `QQ2000b.exe` 中的 `ad.gif` 字符串，修改为 `00 00 00 00 00 00`

## (3) 用 `ResHacker` 将原广告位置的 `STATIC` 改成具有 `ES_READONLY` | `ES_MULTILINE` 风格的 `EDIT`

修改之后，原来的 `PtInRect` 就不用修改了，因为 `DlgProc` 已经接受不到 `WM_LBUTTONDOWN` 消息了。

## (4) 添加显示 IP 程序段

```
.005D70E0: 51                    push        ecx
.005D70E1: 52                    push        edx
.005D70E2: 56                    push        esi
.005D70E3: 53                    push        ebx
.005D70E4: 8B86CC0A0000          mov         eax,[esi][00000ACC] ;数据
.005D70EA: 8B98CC030000          mov         ebx,[eax][000003CC] ;端口
.005D70F0: 8B80C8030000          mov         eax,[eax][000003C8] ;IP
.005D70F6: 803800                cmp         b,[eax],000
.005D70F9: 7437                  je          .0005D7132
.005D70FB: 53                    push        ebx
.005D70FC: 50                    push        eax
.005D70FD: 50                    push        eax
.005D70FE: FF151D007800          call        _GetAddress ;ipsearcher.dll
.005D7104: 83C404                add         esp,004
.005D7107: FF7004                push        d,[eax][04] ;地理位置
.005D710A: FF30                  push        d,[eax] ;连接方式
.005D710C: 6860EF6400            push        00064EF60 ;format
.005D7111: 90                    nop
.005D7112: 6870EF6400            push        00064EF70 ;buffer
.005D7117: FF1534875D00          call        d,[005D8734]
.005D711D: 83C418                add         esp,018
.005D7120: B870EF6400            mov         eax,00064EF70 ;buffer
.005D7125: 50                    push        eax
.005D7126: FFB6B0090000          push        d,[esi][000009B0] ;广告句柄
.005D712C: FF1564855D00          call        d,[005D8564]
.005D7132: 5B                    pop         ebx
.005D7133: 5E                    pop         esi
.005D7134: 5A                    pop         edx
.005D7135: 59                    pop         ecx
.005D7136: 8D86500A0000          lea         eax,[esi][00000A50]
.005D713C: E97006E9FF            jmp         .0004677B1
```

**数据**：

`02420576  25 73 25 73 0D 0A 25 73  3A 25 64 00 00 00 00 00  %s%s..%s:%d.....`
`02420592  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................`
`02420608  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ................`

## (5) 跳到显示 IP 程序段

```
.0046776E: 6896000000            push        000000096
.00467773: 57                    push        edi
.00467774: E81E911200            call        .000590897
.00467779: 8D860C0A0000          lea         eax,[esi][00000A0C]
.0046777F: 50                    push        eax
.00467780: 6824010000            push        000000124
.00467785: 57                    push        edi
.00467786: E80C911200            call        .000590897
.0046778B: 8D86480A0000          lea         eax,[esi][00000A48]
.00467791: 50                    push        eax
.00467792: 53                    push        ebx
.00467793: 57                    push        edi
.00467794: E8EA8C1200            call        .000590483
.00467799: 8D864C0A0000          lea         eax,[esi][00000A4C]
.0046779F: 50                    push        eax
.004677A0: 681D010000            push        00000011D
.004677A5: 57                    push        edi
.004677A6: E8D88C1200            call        .000590483
.004677AB: E930F91600            jmp         .0005D70E0 ;转去显示 IP
.004677B0: 90                    nop                    ;对齐
.004677B1: 50                    push        eax
.004677B2: 68DF000000            push        0000000DF
.004677B7: 57                    push        edi
.004677B8: E8C68C1200            call        .000590483
```

OK 了，呵呵～

**注**：本文参考最弱智的教程修改而成，在此表示感谢 :P

水平有限，错误之处还望指出 `4dd466d0e1e0cc123afbe504da2462ca`
