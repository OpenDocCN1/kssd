
上面那个`Disk`是你硬盘的`serialnumber`,他是如何来的呢?我简单跟踪一下,发现在

`005D114F` `call` `<Protecte.ZLIBArchiveGlobals2_sub_005A0>`里面,最终追踪到如下:

```
005A013F   .  E8 1068E6FF   call <Protecte.System_@UStrFromWStr>
005A0144   .  8B55 B4       mov edx,dword ptr ss:[ebp-0x4C]
005A0147   .  B8 38045A00   mov eax,Protecte.005A0438                    ;  UNICODE "SELECT"
005A014C   .  E8 C76FE6FF   call <Protecte.Pos_0>
005A0151   .  48            dec eax
005A0152   .  75 23         jnz short Protecte.005A0177
005A0154   .  8D45 F8       lea eax,dword ptr ss:[ebp-0x8]
005A0157   .  E8 3488E6FF   call <Protecte.System_@IntfClear>
005A015C   .  50            push eax
005A015D   .  6A 00         push 0x0
005A015F   .  6A 10         push 0x10
005A0161   .  68 4C045A00   push Protecte.005A044C                       ;  UNICODE "WQL"
005A0166   .  53            push ebx
005A0167   .  8B45 FC       mov eax,dword ptr ss:[ebp-0x4]
005A016A   .  50            push eax
005A016B   .  8B00          mov eax,dword ptr ds:[eax]
005A016D   .  FF50 3C       call dword ptr ds:[eax+0x3C]
005A0170   .  E8 2B89E6FF   call <Protecte.System_@CheckAutoResult>
005A0175   .  EB 1C         jmp short Protecte.005A0193
```

哈原来是`WQL`语言查询的(`WMI`中的查询语言).windows自带 `wbemtest.exe` `WQL`测试工具

通过查询语句`SELECT * FROM Win32_PhysicalMedia`可以看到一样的结果:

图................

![](https://bbs.kanxue.com/upload/attach/201501/342682_b18b90b639db36c1.png)

向下继续分析来到:

```
005D1441   .  8B85 0CFFFFFF mov eax,dword ptr ss:[ebp-0xF4]
005D1447   .  8B80 98030000 mov eax,dword ptr ds:[eax+0x398]
005D144D   .  8B8D 08FFFFFF mov ecx,dword ptr ss:[ebp-0xF8]
005D1453   .  8B55 FC       mov edx,dword ptr ss:[ebp-0x4]
005D1456   .  E8 E59AF8FF   call <Protecte.IdHTTP_TIdCustomHTTP_Post> //post数据,地址是784K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8X3i4K6u0W2K9h3!0T1K9i4c8Q4x3X3g2U0L8$3#2Q4x3V1k6X3N6h3&6U0N6r3W2G2L8Y4y4Q4x3V1k6U0K9r3g2U0K9#2)9J5k6i4m8Z5M7l9`.
005D145B   .  A1 54075F00   mov eax,dword ptr ds:[0x5F0754]
005D1460   .  8B00          mov eax,dword ptr ds:[eax]
005D1462   .  E8 C506F3FF   call <Protecte.Forms_TApplication_ProcessMessages>
005D1467   .  8B85 08FFFFFF mov eax,dword ptr ss:[ebp-0xF8]
005D146D   .  8B10          mov edx,dword ptr ds:[eax]
005D146F   .  FF52 44       call dword ptr ds:[edx+0x44]
005D1472   .  8B85 08FFFFFF mov eax,dword ptr ss:[ebp-0xF8]
005D1478   .  50            push eax
005D1479   .  8D95 B4FEFFFF lea edx,dword ptr ss:[ebp-0x14C]
005D147F   .  8B85 04FFFFFF mov eax,dword ptr ss:[ebp-0xFC]
005D1485   .  E8 7699EAFF   call <Protecte.Classes_TStringStream_GetDataString>//获取服务器传回的数据
005D148A   .  8B85 B4FEFFFF mov eax,dword ptr ss:[ebp-0x14C]
005D1490   .  E8 E352E3FF   call <Protecte.System_@UStrToPWChar>
005D1495   .  8BC8          mov ecx,eax
005D1497   .  BA 50205D00   mov edx,Protecte.005D2050
005D149C   .  B8 70205D00   mov eax,<Protecte.char_@>
005D14A1   .  E8 BA52EAFF   call <Protecte.ExtractStrings>//结压缩服务器数据,其实就是分割,因为服务器传回的数据是XXX&XXX&XXX形式的.
005D14A6   .  8B85 08FFFFFF mov eax,dword ptr ss:[ebp-0xF8]
005D14AC   .  8B10          mov edx,dword ptr ds:[eax]
005D14AE   .  FF52 14       call dword ptr ds:[edx+0x14]//这里指针调用的是Classes_TStringList_GetCount,即数据分割后得到的数据项数.
005D14B1   .  85C0          test eax,eax //必须大于一项,不然就是下面的出错提示了
005D14B3   .  75 6A         jnz short Protecte.005D151F
005D14B5   .  6A 30         push 0x30
005D14B7   .  8D85 B0FEFFFF lea eax,dword ptr ss:[ebp-0x150]
005D14BD   .  50            push eax
005D14BE   .  A1 84055F00   mov eax,dword ptr ds:[0x5F0584]
005D14C3   .  8B00          mov eax,dword ptr ds:[eax]
005D14C5   .  B9 9C205D00   mov ecx,<Protecte.aActiveError>                        ;  UNICODE "Active Error"
005D14CA   .  BA C4205D00   mov edx,<Protecte.aAcerr>                              ;  UNICODE "acerr"
005D14CF   .  E8 3CC1FBFF   call <Protecte.PLabelNote_sub_0058D610>
005D14D4   .  8B85 B0FEFFFF mov eax,dword ptr ss:[ebp-0x150]
005D14DA   .  E8 9952E3FF   call <Protecte.System_@UStrToPWChar>
005D14DF   .  50            push eax
005D14E0   .  8D85 ACFEFFFF lea eax,dword ptr ss:[ebp-0x154]
005D14E6   .  50            push eax
005D14E7   .  A1 84055F00   mov eax,dword ptr ds:[0x5F0584]
005D14EC   .  8B00          mov eax,dword ptr ds:[eax]
005D14EE   .  B9 DC205D00   mov ecx,<Protecte.aUnknownError>                       ;  UNICODE "Unknown Error!"
005D14F3   .  BA 08215D00   mov edx,<Protecte.aUnerr>                              ;  UNICODE "unerr"
005D14F8   .  E8 13C1FBFF   call <Protecte.PLabelNote_sub_0058D610>
005D14FD   .  8B85 ACFEFFFF mov eax,dword ptr ss:[ebp-0x154]
005D1503   .  E8 7052E3FF   call <Protecte.System_@UStrToPWChar>
005D1508   .  50            push eax
005D1509   .  8B85 0CFFFFFF mov eax,dword ptr ss:[ebp-0xF4]
005D150F   .  E8 2475F1FF   call <Protecte.Controls_TWinControl_GetHandle>
005D1514   .  50            push eax                                               ; |hOwner
005D1515   .  E8 E28FE3FF   call <Protecte.MessageBoxW>                            ; \MessageBoxW
005D151A   .  E9 10060000   jmp Protecte.005D1B2F
005D151F   >  8D8D A8FEFFFF lea ecx,dword ptr ss:[ebp-0x158]//服务器返回的数据项数大于一项会到这里
005D1525   .  33D2          xor edx,edx
005D1527   .  8B85 08FFFFFF mov eax,dword ptr ss:[ebp-0xF8]
005D152D   .  8B18          mov ebx,dword ptr ds:[eax]
005D152F   .  FF53 0C       call dword ptr ds:[ebx+0xC] //调用的是Classes_TStringList_Get,edx是获取的第几项,这里是第0项
005D1532   .  8B85 A8FEFFFF mov eax,dword ptr ss:[ebp-0x158]
005D1538   .  BA A01D5D00   mov edx,<Protecte.char_0_>
005D153D   .  E8 B658E3FF   call <Protecte.j_@UStrCmp> //也就是说,服务器返回数据的第一个是0才是正确的.
005D1542   .  0F85 8D010000 jnz Protecte.005D16D5
005D1548   .  8B85 08FFFFFF mov eax,dword ptr ss:[ebp-0xF8]
005D154E   .  8B10          mov edx,dword ptr ds:[eax]
005D1550   .  FF52 14       call dword ptr ds:[edx+0x14] //调用Classes_TStringList_GetCount,返回数据项数
005D1553   .  83F8 05       cmp eax,0x5                     //必须是5项
005D1556   .  0F85 79010000 jnz Protecte.005D16D5
005D155C   .  B8 CC965F00   mov eax,Protecte.005F96CC
005D1561   .  8B55 F8       mov edx,dword ptr ss:[ebp-0x8]
005D1564   .  E8 8351E3FF   call <Protecte.@UStrAsg>
005D1569   .  B8 D0965F00   mov eax,Protecte.005F96D0
005D156E   .  8B4D EC       mov ecx,dword ptr ss:[ebp-0x14]
005D1571   .  8B55 F0       mov edx,dword ptr ss:[ebp-0x10]
005D1574   .  E8 0756E3FF   call <Protecte.System_@UStrCat3>
005D1579   .  B8 D4965F00   mov eax,Protecte.005F96D4
005D157E   .  BA 20215D00   mov edx,<Protecte.aPro_0>                              ;  UNICODE "Pro"
005D1583   .  E8 6451E3FF   call <Protecte.@UStrAsg>
005D1588   .  8D8D A4FEFFFF lea ecx,dword ptr ss:[ebp-0x15C]
```

用`wireshark`拦截到发到服务器上去的数据:

`POST /functions/check.php HTTP/1.0`

`Connection: keep-alive`

`Content-Type: application/x-www-form-urlencoded`

`Content-Length: 90`

`Host: pf.iobit.com`

`Accept: text/html, */*`

`Accept-Encoding: identity`

`User-Agent: Mozilla/3.0 (compatible; Indy Library)`

`Code=22EAA-B9A86-4AC54-3FAAC&CPU=0003-06C3-BFEB-FBFF-7FFA-FBBF&Disk=J3390084J8V20D&Ver=100`

继续往下分析:
