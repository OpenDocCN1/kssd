
# NetAnts 1.22

> 原文：[`www.pediy.com/kssd/pediy03/forum1051.htm`](https://www.pediy.com/kssd/pediy03/forum1051.htm)

网络蚂蚁广告窗去除 NetAnts 1.22

下面程序调用 `CreateWindowExA` 函数创建网络蚂蚁所用到的窗口

`* Reference To: USER32.SetMenu, Ord:0235h`

```
|
:0041A807 FF15ECC34300           Call dword ptr [0043C3EC]
:0041A80D 57                     push edi
:0041A80E E863DA0000             call 00428276
:0041A813 3BC3                   cmp eax, ebx
:0041A815 59                     pop ecx
:0041A816 A36C854400             mov dword ptr [0044856C], eax
:0041A81B 0F84A51E0000           je 0041C6C6
:0041A821 50                     push eax
:0041A822 E8C9DB0000             call 004283F0 //创建 ToolBar
:0041A827 3BC3                   cmp eax, ebx
:0041A829 59                     pop ecx
:0041A82A A3B8854400             mov dword ptr [004485B8], eax
:0041A82F 0F84911E0000           je 0041C6C6
:0041A835 FF356C854400           push dword ptr [0044856C]
:0041A83B E810170100             call 0042BF50
:0041A840 3BC3                   cmp eax, ebx
:0041A842 59                     pop ecx
:0041A843 A398854400             mov dword ptr [00448598], eax
:0041A848 0F84781E0000           je 0041C6C6
:0041A84E 57                     push edi
:0041A84F E8007AFEFF             call 00402254 //创建 广告条
:0041A854 3BC3                   cmp eax, ebx
:0041A856 59                     pop ecx
:0041A857 A3B0854400             mov dword ptr [004485B0], eax //保存窗口句柄
:0041A85C 0F84641E0000           je 0041C6C6
:0041A862 57                     push edi
:0041A863 E860CA0000             call 004272C8
:0041A868 3BC3                   cmp eax, ebx
:0041A86A 59                     pop ecx
:0041A86B A3C4864400             mov dword ptr [004486C4], eax
:0041A870 0F84501E0000           je 0041C6C6
:0041A876 57                     push edi
:0041A877 E8FB18FFFF             call 0040C177
:0041A87C 3BC3                   cmp eax, ebx
:0041A87E 59                     pop ecx
:0041A87F A384854400             mov dword ptr [00448584], eax
:0041A884 0F843C1E0000           je 0041C6C6
:0041A88A 57                     push edi
:0041A88B E84CA2FFFF             call 00414ADC //创建 SysListView32
:0041A890 3BC3                   cmp eax, ebx
:0041A892 59                     pop ecx
:0041A893 A390854400             mov dword ptr [00448590], eax
:0041A898 0F84281E0000           je 0041C6C6
:0041A89E 57                     push edi
:0041A89F E84EBFFEFF             call 004067F2
:0041A8A4 3BC3                   cmp eax, ebx
:0041A8A6 59                     pop ecx
:0041A8A7 A3A0854400             mov dword ptr [004485A0], eax
:0041A8AC 0F84141E0000           je 0041C6C6
```

关键在于 `41A84F` 处的 `Call`，将 `41A84F` 到 `0041A862` 处的代码改成如下

### 改前

```
:0041A84F E8007AFEFF             call 00402254 //创建 广告条
:0041A854 3BC3                   cmp eax, ebx
:0041A856 59                     pop ecx
:0041A857 A3B0854400             mov dword ptr [004485B0], eax //保存窗口句柄
:0041A85C 0F84641E0000           je 0041C6C6
:0041A862 57                     push edi
```

### 改后

```
:0041A84F 31C0909090             xor eax,eax //清 eax=0
:0041A854 3BC3                   cmp eax, ebx
:0041A856 59                     pop ecx
:0041A857 A3B0854400             mov dword ptr [004485B0], eax //保存窗口句柄
:0041A85C 90                     nop
:0041A85D 90                     nop
:0041A85E 90                     nop
:0041A85F 90                     nop
:0041A860 90                     nop
:0041A861 90                     nop
:0041A862 57                     push edi
```

自此将不会再出现广告条，而是好大一面墙。下面将通过该 `SysListView32` 的高度去除这面墙

`* Possible Reference to String Resource ID=00001: "Invalid retry delay"`

```
|
:0041A659 6A01                   push 00000001
:0041A65B 2B45F0                 sub eax, dword ptr [ebp-10]
:0041A65E 6A42                   push 00000042
:0041A660 50                     push eax
:0041A661 FF75F4                 push [ebp-0C]
:0041A664 FF75F0                 push [ebp-10]
:0041A667 FF35B0854400           push dword ptr [004485B0]
:0041A66D FFD7                   call edi
:0041A66F 8345F443               add dword ptr [ebp-0C], 00000043 //确定 SysListView32 位置
                                 //00000043 是墙的高度,也就是广告条的高度
:0041A673 391D80854400           cmp dword ptr [00448580], ebx
:0041A679 0F84F5000000           je 0041A774
```

`* Possible Reference to String Resource ID=00005: "Invalid maximum speed"`

```
|
:0041A67F 6A05                   push 00000005
```

根据上面分析只要将 `0041a66F` 除的 `00000043` 改成 `00000000` 就可以了

### 改前

`0041A66F 8345F443 add dword ptr [ebp-0C], 00000043`

### 改后

`0041A66F 8345F400 add dword ptr [ebp-0C], 00000000`

到此网络蚂蚁 1.22 版去广告条成功。
`4dd466d0e1e0cc123afbe504da2462ca`
