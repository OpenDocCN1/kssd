
`INC`    `EBX`

0187:006ACF5D 3803

`CMP`    `[EBX]`,`AL`

0187:006ACF5F

75F9              `JNZ`    006ACF5A

0187:006ACF61 83859AAF400004    `ADD`    `DWord Ptr [EBP+0040AF9A]`,00000004

0187:006ACF68 E9D4FDFFFF        `JMP`    006ACD41

0187:006ACF6D 83C614            `ADD`

`ESI`,00000014

0187:006ACF70 8B95AEAF4000      `MOV`

`EDX`,`[EBP+0040AFAE]`

0187:006ACF76 E9B1FCFFFF        `JMP`

006ACC2C    //循环

0187:006ACF7B 61

        `POPAD`

0187:006ACF7C C3

        `RET`

按我的方法把`006ACE7A`和`006ACE80`改成这个样子，就可以用`ImportRECf`了。

0187:006ACE7A 89BD9AB04000      `MOV`    `[EBP+0040B09A]`,`EDI`

//`MOV`    `[EBP+0040B09A]`,`EBX`

899D9AB04000

0187:006ACE80 8BBD9AB04000

`MOV`    `EDI`,`[EBP+0040B09A]`  //`MOV`    `EDI`,`[EBP+0040B0A2]`

8BBDA2B04000

这样做完所有的函数都可以认出，但还有一个问题，就是该分段的地址还是被telock写入了东西，这样`ImportRECf`就无法识别，还好按我的改法认不出来的地方都是该分段的地方。

类似这样

```
1    001AB24C    KERNEL32.dll
  0181    GetFileType
1    001AB250
  KERNEL32.dll    00BA    CreateFileA
1    001AB254    KERNEL32.dll    00A1
  CloseHandle
0    001AB258    ?
  0000    009D01C2
1    001AB25C
  USER32.dll    0115    GetKeyboardType
1    001AB260    USER32.dll    01AB
  LoadStringA
1    001AB264    USER32.dll
  01BA    MessageBoxA
1    001AB268
  USER32.dll    0027    CharNextA
0    001AB26C    ?    0000
  009E0028
1    001AB270    ADVAPI32.dll
  00F8    RegQueryValueExA
1    001AB274
  ADVAPI32.dll    00F0    RegOpenKeyExA
1    001AB278    ADVAPI32.dll    00D9
  RegCloseKey
```

只需手工把它改成如下就可以了，比原来的方法要快些:)，具体的动手试试吧。

```
1    001AB24C    KERNEL32.dll    0181
  GetFileType
1    001AB250    KERNEL32.dll
  00BA    CreateFileA
1    001AB254
  KERNEL32.dll    00A1    CloseHandle
 FThunk: 001AB25C    NbFunc: 00000004
1    001AB25C
  USER32.dll    0115    GetKeyboardType
1    001AB260    USER32.dll    01AB
  LoadStringA
1    001AB264    USER32.dll
  01BA    MessageBoxA
1    001AB268
  USER32.dll    0027    CharNextA
 FThunk: 001AB270    NbFunc: 00000003
1    001AB270
  ADVAPI32.dll    00F8    RegQueryValueExA
1    001AB274    ADVAPI32.dll    00F0
  RegOpenKeyExA
1    001AB278    ADVAPI32.dll
  00D9    RegCloseKey
```

做完这些后用`ImportRECf`修复后的程序可以正常运行了。

完活，哪位大哥有其他好方法可用的话，千万记得告诉我呀，谢啦。

另外可以参考`hying`和`fpc`大哥关于`telock`的脱文。

感谢你能够坚持下来hoho:)

`telock`的壳除了手动找入口比较难外，其它的还是比较简单的。

关于`sss`的注册：

这是我对付以前用`telock`加过壳的那个`sss`的`patch`，`sss`任意版都只需改2个字节变成无限制版

```
#Process Patcher Configuration File

 Version=3.93

 DisplayName=My

Test Program

 Filename=sss.exe

 Filesize=1043968

 Arguments=/quiet

 Address=0x58ec5d:0x75:0xeb

 #End of Configuration File
```

再有用这个`key`强制注册后，就是注册版。

`UserName=SSSRussian`

`DigitalSignature=Vu3xB7ar69mR3Vt6hru4piTtDpPOVALxT+SXLg5nALwNVGDHbzWyEA4UAxQktR2arkd5klsvjHCTQVvO0zz9RU`

2001.9.27

`zombieys[CCG]`

―――――――――――――――――――――――――――――――>

                 .-"

  "-.      unpacked by zombieys[CCG] >

            /　      　　\

  qq:1789655                >

              |      ★　　

|    http://zombieys.yeah.net  >

      |,  .-.  .-.  ,|    http://zombieys.126.com

>

                 |)(\_\_/  \\_\_)(|

    zombieys.cn.hongnet.com  >

        |/    /\    \|

  >

       (@\_@)    (\_    ^^

  \_)      Thanks for your supports  >

\_  )\\_\_\_\_\_\_\_\\_\_|IIIIII|\_\_/\_\_\_\_\_

              >

 \_)@8@8{}<\_\_\_\_\_\_\_\_|-\IIIIII/-|\_\_\_\_China

Crack Group\_zombieys\_\_\_>
