
```
0040F3FD    8918            MOV DWORD PTR DS:[EAX],EBX  

0040F3FF    83C0 06         ADD EAX,6  

0040F402    8078 03 00      CMP BYTE PTR DS:[EAX+3],0  

0040F406    74 EE           JE SHORT safeguar.0040F3F6  

0040F408    61              POPAD  

0040F409    90              NOP  

60 B8 1C F4 40 00 8B 18 8B 1B 8B 5B 02 89 18 83 C0 06 80 78 03 00 74 EE 61 90  
```

\*/

//重建 iat 调用地址

`mov eip,0040F3F0`

`mov [eip],#60B81CF440008B188B1B8B5B02891883C0068078030074EE6190#`

//pause

`bp 0040F409`

`eob setep`

`run`

//处理 stolen code

`setep:`

`bc eip`

`mov eip,0040F3F0`

`mov [eip],#6A00E841000000A3622041006A006847F240006A006A65FF3562204100E87A0000006A00E813000000#`

`msg "safeguard v1.01 脱壳完成:-), 感谢 simonzh !"`

`ret`

```
fxyang
2005.6.19
```

希望这个脚本对`simonzh`有所帮助，特别是对`iat`的处理，虽然加密函数很复杂，但是传递给这个函数参数时漏洞太大：

```
0041E195    E8 79DBFFFF     CALL safeguar.0041BD13  

0012FF98    003A004D  |Arg1 = 003A004D  

0012FF9C    77E5D8B4  |Arg2 = 77E5D8B4  

0012FFA0    00000000  \Arg3 = 00000000  
```

```
EAX 003A004D  

ECX 77E5D8B4  kernel32.LoadLibraryA  

EDX 7FFE0304  

EBX 00413000  safeguar.00413000  

ESP 0012FF98  

EBP 00013000  

ESI 00416633  safeguar.00416633  

EDI 0072C794  

EIP 0041E195  safeguar.0041E195  
```

修改`iat`处理代码：

```
0041BD13    55              PUSH EBP  

0041BD14    8BEC            MOV EBP,ESP  

0041BD16    60              PUSHAD  

0041BD17    8B7D 08         MOV EDI,DWORD PTR SS:[EBP+8]         ; 003A0000  

0041BD1A    8B75 0C         MOV ESI,DWORD PTR SS:[EBP+C]         ; Stack SS:[0012FF9C]=77E5B285 (kernel32.GetProcAddress)  
```

很容易修改成自动恢复的

`4dd466d0e1e0cc123afbe504da2462ca`
