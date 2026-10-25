
# PE-explorer 1.4 的简要破解过程(1千字)

PE-explorer 1.4 的简要破解过程。

```
004DD69F: E838F6FFFF
  call        0004DCCDC
004DD6A4: E817D9FFFF
  call        0004DAFC0
004DD6A9: 8B45F0
    mov        eax,[ebp][-0010]
004DD6AC: 80B85402000000              cmp
      b,[eax][000000254],000 ;" "
004DD6B3: 0F85F5070000
jne        0004DDEAE
<<==修改 jne 为 je
004DD6B9: 8B45F0
    mov        eax,[ebp][-0010]
004DD6BC: 8B9834020000
mov        ebx,[eax][000000234]
004DD6C2: 83C305
    add        ebx,005 ;""
004DD6C5: 8D95C8FCFFFF
lea        edx,[ebp][0FFFFFCC8]
004DD6CB: 8BC3
      mov        eax,ebx
004DD6CD: E8DAA2F2FF
  call        0004079AC
004DD6D2: 8B95C8FCFFFF
mov        edx,[ebp][0FFFFFCC8]
004DD6D8: B8D4B94F00
  mov        eax,0004FB9D4
004DD6DD: E8D662F2FF
  call        0004039B8
004DD6E2: A1D4B94F00
  mov        eax,[0004FB9D4]
```

使用`Winhex`打开`pexplorer.exe`, 然后找出offset `Addreess : 000dcab3h`，

再修改`0F85F5070000` 为 `0F84F5070000`。

先用 “名字：444 注册号：FA7349671EE5C68E” 注册，跟着使用`Winhex`打开`pexplorer.exe`, 然后找出offset `Addreess : 000dcab3h`，再修改`0F85F5070000` 为 `0F84F5070000`。
