
```
:00495F64 3BD1               cmp edx, ecx   

:00495F66 75CD               jne 00495F35   

:00495F68 48                 dec eax   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:00495F4F(C), :00495F56(C)   

|   

:00495F69 5F                 pop edi   

:00495F6A 5E                 pop esi   

:00495F6B C3                 ret   
```

不能乱改，否则崩溃，程序其他部分还会调用此处，此乃程序高明之处。

===========================================================

### 4、程序被修改提示框的去除

我用 `R!SC'sPatcher v1.4` 做 `PATCH`，非常简单，按 `1,2` 改。软件可在 `TOYE` 工具处下载。

我的 `bbrk32.rpp` 如下：

`F=bbrk32.exe:`

`O=cr_bbrk32.exe:`

`P=426d9c/14,31/10,90:`

`P=427589/8b,c1,66,8b,7d,0c,99,f7,7d,fc,8b,45/c7,45,fc,00,00,00,00,e9,3f,00,00,00:`

```
$
```

===========================================================

### 5、运行生成的 `cr_bbrk32.exe`

We have a fully functional `REGISTERED` copy of `BrainsBreaker3.0[07b]!`

===========================================================

```
/zest//2001.3.8  4dd466d0e1e0cc123afbe504da2462ca
```
