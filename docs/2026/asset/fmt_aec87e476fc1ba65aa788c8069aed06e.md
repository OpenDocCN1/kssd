
```assembly
00407F1B  |.  8D45 FC               lea eax,[local.1]
00407F1E  |.  BA 02000000           mov edx,2
00407F23  |.  E8 8C5E1100           call btengine.0051DDB4
00407F28  |.  58                    pop eax
00407F29  |.  8B16                  mov edx,dword ptr ds:[esi]
00407F2B  |.  64:8915 00000000      mov dword ptr fs:[0],edx
00407F32  |.  EB 2D                 jmp short btengine.00407F61
00407F34  |>  33C0                  xor eax,eax
00407F36  |.  BA 02000000           mov edx,2
00407F3B  |.  50                    push eax
00407F3C  |.  8D45 F8               lea eax,[local.2]
00407F3F  |.  FF4E 1C               dec dword ptr ds:[esi+1C]
00407F42  |.  E8 6D5E1100           call btengine.0051DDB4
00407F47  |.  FF4E 1C               dec dword ptr ds:[esi+1C]
00407F4A  |.  8D45 FC               lea eax,[local.1]
00407F4D  |.  BA 02000000           mov edx,2
00407F52  |.  E8 5D5E1100           call btengine.0051DDB4
00407F57  |.  58                    pop eax
00407F58  |.  8B16                  mov edx,dword ptr ds:[esi]
00407F5A  |.  64:8915 00000000      mov dword ptr fs:[0],edx
00407F61  |>  5E                    pop esi
00407F62  |.  5B                    pop ebx
00407F63  |.  8BE5                  mov esp,ebp
00407F65  |.  5D                    pop ebp
00407F66  \.  C3                    retn
```

### 注册算法总结

只要满足`StrToStr(注册码1-6位)＋StrToStr(注册码7-12位)＝$160A61`即可，用户名不参加注册计算。

`(16进制)160A61＝(十进制)1444449＝800000＋644449`

一个可用的注册码：

用户名：`Baby2008`

注册码：`800000644449`

用BT下载的朋友可以试试，省30大洋的注册费！

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
