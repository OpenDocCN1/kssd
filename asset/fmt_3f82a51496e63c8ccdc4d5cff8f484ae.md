
```
004053D5    8B95 D8FEFFFF        mov edx,dword ptr ss:[ebp-128]

004053DB    52                   push edx

004053DC    8B85 D4FEFFFF        mov eax,dword ptr ss:[ebp-12C]

004053E2    50                   push eax

004053E3    E8 98E7FFFF          call QQLog.00403B80

004053E8    83C4 08              add esp,8

004053EB    8B85 D0FEFFFF        mov eax,dword ptr ss:[ebp-130]                             ;************

004053F1    8B4D F4              mov ecx,dword ptr ss:[ebp-C]

004053F4    64:890D 00000000     mov dword ptr fs:[0],ecx

004053FB    59                   pop ecx

004053FC    8B4D E4              mov ecx,dword ptr ss:[ebp-1C]

004053FF    33CD                 xor ecx,ebp

00405401    E8 05010200          call QQLog.0042550B

00405406    8BE5                 mov esp,ebp

00405408    5D                   pop ebp

00405409    C3                   retn

00405367    75 0F                jnz short QQLog.00405378                                 ;这里是关键原程序没有跳

00405369    E8 C2EBFFFF          call QQLog.00403F30

0040536E    C785 D0FEFFFF 010000>mov dword ptr ss:[ebp-130],1                            ;没有跳这里赋值为 1

004053EB    8B85 D0FEFFFF        mov eax,dword ptr ss:[ebp-130]                          ；没有跳这里也为 1
```

然后就可以跳过那个异常了。

我们同样可以发现 `00406357` 也是 `smc` 过的，同样是对应的字节加上 `A`，所以我们只需重新加载程序，在数据窗口 `ctrl + g` 来到 `00405367` 处，将 `6B` 改成 `6A`，然后保存。

程序可以运行了。

## 总结：

这个程序是通过比较脱壳前后文件大小来进行自校验的，另外还用了 `smc` 技术。

这里脱壳后有两种方法跳过错误：

**方法 1**
`ctrl + B` 搜索 `6B F7 C2` 找到三处 `0040604B`、`0040657E`、`00406842` 将其修改为：`6A F7 C2`。保存文件可以去校验。

**方法 2**
`ctrl + g` 来到 `00405367` 将其内容 `6B` 改成 `6A`。保存文件同样可以达到目的。

希望像我一样的菜鸟在遇到问题时，多多分析，多多思考，这样才有进步。
