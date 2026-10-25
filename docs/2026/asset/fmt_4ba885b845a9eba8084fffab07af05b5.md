
``` 
00435F3A  |. 33D6          |XOR EDX,ESI
<edx=edx ^ esi>
00435F3C  |. C1E0 08        |SHL EAX,8
<eax=eax * 0x100>
00435F3F  |. 83C1 04        |ADD ECX,4
<ecx=ecx +0x4>
00435F42  |. 0BC2           |OR EAX,EDX
<eax=eax | edx>
00435F44  |. 3BCF           |CMP ECX,EDI
<循环条件判断>
00435F46  |.^72 C8          \JB SHORT WinRip.00435F10
<跳转，取下一个待解码数据>
00435F48  |> 5F             POP EDI
```

补丁原理:

1、根据上面解码原理编写一个解码过程 `DeCode(Byte buff[])`;

2、打开`WinRip.exe`，把`436200-436f50`这一段读入`Byte buff[0xD50]`，并对其用`DeCode(Byte buff[])`进行解码；

3、修改`buff[0x998]`、`buff[0x999]`中数据的值分别为`eb`、`07`;

4、重新对`buff[]`用`DeCode(Byte buff[])`进行编码(编码、解码是对称的);

5、把`buff[]`写回文件；

至此，程序已经完全破解。

youth

2002-8-23
