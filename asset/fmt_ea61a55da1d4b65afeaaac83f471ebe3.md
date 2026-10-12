
```assembly
003806DC   ^ 0F85 87FEFFFF   JNZ 00380569                     ; 没处理完则继续  

00380569    03C1             ADD EAX,ECX  

0038056B    68 033C0101      PUSH 1013C03  

00380570    8946 FC          MOV DWORD PTR DS:[ESI-4],EAX  

00380573    68 2CBF3C57      PUSH 573CBF2C  

00380578    25 FFFFFF7F      AND EAX,7FFFFFFF                 ; 去掉前面的 80xxxx  

0038057D    68 62A73AA1      PUSH A13AA762  

00380582    8BC8             MOV ECX,EAX  

00380584    68 6204EF04      PUSH 4EF0462  

00380589    0385 120F4100    ADD EAX,DWORD PTR SS:[EBP+410F12] ; 转换为 VA401256  

0038058F    52               PUSH EDX                         ; xxx.00400100  
```

现在又进堆栈里执行代码:

```assembly
0012FF94    89FB             MOV EBX,EDI                      ; 把那个让代码放到 ESP 中的人给 B 了^\_^  

0012FF96    29C3             SUB EBX,EAX                      ; 清除存放表中的数据  

0012FF98    8958 FC          MOV DWORD PTR DS:[EAX-4],EBX     ; 取 esi+4 的地址到[EBX]中  

0012FF9B    66:C740 FA 90E8 MOV WORD PTR DS:[EAX-6],0E890    ; 把第一层的地址放到目标程序中  

0012FFA1    C3               RETN  
```

```assembly
0038086A    03B5 120F4100    ADD ESI,DWORD PTR SS:[EBP+410F12] ; 401000(开始地址)  

00380870    8B8D 45174100    MOV ECX,DWORD PTR SS:[EBP+411745] ; 198c(SIZE)  

00380876    E8 5F050000      CALL 00380DDA                    ; 又进行 CRC 检测  

0038087B    8985 2A0F4100    MOV DWORD PTR SS:[EBP+410F2A],EAX; 检测的值为 2A954338(每次不一定相等)  

00380881    8BC5             MOV EAX,EBP                      ; EBP=FFFF715A2  

00380883    8DB5 6D184100    LEA ESI,DWORD PTR SS:[EBP+41186D] ; 382E0F  

00380889    0146 04          ADD DWORD PTR DS:[ESI+4],EAX  

0038088C    0146 08          ADD DWORD PTR DS:[ESI+8],EAX  

0038088F    83C6 20          ADD ESI,20  

00380892    0146 04          ADD DWORD PTR DS:[ESI+4],EAX  

00380895    83C6 20          ADD ESI,20  

00380898    0146 04          ADD DWORD PTR DS:[ESI+4],EAX  

0038089B    0146 08          ADD DWORD PTR DS:[ESI+8],EAX  

0038089E    83C6 20          ADD ESI,20  

003808A1    0146 04          ADD DWORD PTR DS:[ESI+4],EAX  

003808A4    83C6 20          ADD ESI,20  

003808A7    0146 04          ADD DWORD PTR DS:[ESI+4],EAX  

003808AA    83C6 20          ADD ESI,20  

003808AD    0146 04          ADD DWORD PTR DS:[ESI+4],EAX  

003808B0    83C6 20          ADD ESI,20  

003808B3    0146 04          ADD DWORD PTR DS:[ESI+4],EAX  

003808B6    83C6 20          ADD ESI,20  

003808B9    0146 04          ADD DWORD PTR DS:[ESI+4],EAX  

003808BC    8DB5 69184100    LEA ESI,DWORD PTR SS:[EBP+411869]  

003808C2    0106             ADD DWORD PTR DS:[ESI],EAX  

003808C4    8D85 8D194100    LEA EAX,DWORD PTR SS:[EBP+41198D]  
```

```assembly
00380A1A    64:FF35 0000000>PUSH DWORD PTR FS:[0]            ; 又来 int3 异常了  

00380A21    64:8925 0000000>MOV DWORD PTR FS:[0],ESP  

00380A28    33C0             XOR EAX,EAX  

00380A2A    8B00             MOV EAX,DWORD PTR DS:[EAX]  

00380A2C    90               NOP  

00380A2D    90               NOP  

00380A2E    CC               INT3  
```

到了这里后，还有几个异常，不过感觉没什么有价值的东西，再则，可能这段时间又要忙了，所以先写个上篇，有空再写下篇。现在我们可以直接在抽代码处下断就行了。

`OEP` 代码的找回和文件的修复，如果有时间就尽快写，没时间我也没有办法了:-(.

在最后，我要特别感谢 `xxx`（名字就不说了），如果没有他写的那篇文章的话，我可能就没有信心跟这个壳，还有就是在壳的很多部分不懂的东西，也是从他那里学到的，真心感谢!!!

上篇完

`Greetz`:

`Fly`, `Jingulong`, `yock`, `tDasm`, `David`, `ahao`, `vcasm`, `UFO`(`brother`), `alan`(`sister`), `heXpe`, `hexer`, all of my friends and you!

```text
By loveboom[DFCG][FCG]
Email:bmd2chen#tom.com
 4dd466d0e1e0cc123afbe504da2462ca
```
