
## 4. call `cyto-wj.0040E72E`

### 汇编代码

```
0040E72E - E9 17232F01 jmp 01700A4A
0040E72E FF7424 04 push dword ptr ss:[esp+4]
0040E732 E8 D1FFFFFF call cyto-wj.0040E708
0040E737 F7D8 neg eax
0040E739 1BC0 sbb eax,eax
0040E73B F7D8 neg eax
0040E73D 59 pop ecx
0040E73E 48 dec eax
0040E73F C3 retn
```

### 4. Stolen code After API

8 种 `Stolen code After API` 的分类处：

```
014DAA9B 8B55 F8 mov edx,dword ptr ss:[ebp-8]
014DAA9E 3A42 4A cmp al,byte ptr ds:[edx+4A]
014DAAA1 74 0B je short 014DAAAE
014DAAA3 8B55 F8 mov edx,dword ptr ss:[ebp-8]
014DAAA6 3A42 4B cmp al,byte ptr ds:[edx+4B]
014DAAA9 75 3E jnz short 014DAAE9
014DAAAB EB 01 jmp short 014DAAAE
```

寄存器映射：`0=eax, 1=ecx, 2=edx, 3=ebx, 4=esp, 5=ebp, 6=esi, 7=edi`

### 打开脚本运行后产生的文件 `SCafAPI.bin`

```
004D4000 >00401606 cyto-wj.00401606 // mov ecx,esi，隐藏
004D4004 >00401B75 cyto-wj.00401B75 // mov esi,eax,加密码
004D4008 >00402679 cyto-wj.00402679 //
004D400C >0040588C cyto-wj.0040588C //
004D4010 >00405F4C cyto-wj.00405F4C // mov ecx,edi，输入注册码
004D4014 >0040816B cyto-wj.0040816B //
004D4018 >00409AE4 cyto-wj.00409AE4 // mov ecx,esi，加密码
004D401C >00409C05 cyto-wj.00409C05 //
004D4020 >00411BCC cyto-wj.00411BCC //
004D4024 >0041286A cyto-wj.0041286A // mov eax,esi，程序启动
004D4028 >004167B2 cyto-wj.004167B2 //
004D402C >004167E9 cyto-wj.004167E9 //
004D4030 >00416F94 cyto-wj.00416F94 //
```

点击各个功能模块，有几个没能断下，可能要注册才能用到？

`4dd466d0e1e0cc123afbe504da2462ca`
