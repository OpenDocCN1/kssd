
```
0040DCE4   $- FF25 88BCBF01   JMP DWORD PTR DS:[1BFBC88]    ;  oleaut32.VariantInit  
0040DCEA     8BC0            MOV EAX,EAX  
0040DCEC   $- FF25 7CBCBF01   JMP DWORD PTR DS:[1BFBC7C]    ;  oleaut32.VariantClear  
0040DCF2     8BC0            MOV EAX,EAX  
0040DCF4   $- FF25 70BCBF01   JMP DWORD PTR DS:[1BFBC70]    ;  oleaut32.VariantCopy  
0040DCFA     8BC0            MOV EAX,EAX  
0040DCFC   $- FF25 64BCBF01   JMP DWORD PTR DS:[1BFBC64]    ;  oleaut32.VariantChangeType  
0040DD02     8BC0            MOV EAX,EAX  
0040DD04  /.  55              PUSH EBP  
0040DD05  |.  8BEC            MOV EBP,ESP  
```

壳将输入表 `IAT` 全部打烂，不仅不放在一起，而且相互夹杂，真是变态啊，这样 `ImportREC` 根本不能修复。这时我想到了“脚本”，虽然我从来没有写过，但脚本这两个字已是如雷贯耳。首先用 `zeroadd` 为 `test` 加一个节，大小 `2000`，然后写个脚本将所有的函数地址写入新加的节中，并将 `IAT` 改回，再就是大家熟悉的 `loader`，`ImportREC` 修复了。

至此，壳就脱了，当然还要优化，就留给感兴趣的人去做了。

`ps`：这个壳脱得很繁琐，输入表也没有优化，而且要找到所以的 `IAT` 地址，是很麻烦的。期待高手更好的方法。

## 上传的附件

![附件](img/0623e01110d741fd20f66acbc8ec6f9d.jpg)
`dump` 及 `脚本.rar`
