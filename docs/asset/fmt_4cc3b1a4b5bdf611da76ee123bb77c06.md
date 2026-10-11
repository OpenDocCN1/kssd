
# 注册表写入逻辑

**注释**: 被无条件或条件跳转引用
`|:0050C62D(C)`

|

`:0050C6A1 8B45E0` `mov eax, dword ptr [ebp-20]`

**注释**: 可能的字符串数据引用自代码对象 -> `"wlm"`

|

`:0050C6A4 BA10CC5000` `mov edx, 0050CC10`

`:0050C6A9 E83E190000` `call 0050DFEC`

`:0050C6AE B201` `mov dl, 01`

`:0050C6B0 A144304700` `mov eax, dword ptr [00473044]`

`:0050C6B5 E88A6AF6FF` `call 00473144`

`:0050C6BA 8945E8` `mov dword ptr [ebp-18], eax`

`:0050C6BD 807DFF00` `cmp byte ptr [ebp-01], 00`

`:0050C6C1 0F8552020000` `jne 0050C919`

`:0050C6C7 BA02000080` `mov edx, 80000002`

`:0050C6CC 8B45E8` `mov eax, dword ptr [ebp-18]`

`:0050C6CF E8106BF6FF` `call 004731E4`

`:0050C6D4 B101` `mov cl, 01`

**注释**: 可能的字符串数据引用自代码对象 -> `"Software\学生评语管理系统"` //写入注册表

|

`:0050C6D6 BA1CCC5000` `mov edx, 0050CC1C`

`:0050C6DB 8B45E8` `mov eax, dword ptr [ebp-18]`

`:0050C6DE E8656BF6FF` `call 00473248`

`:0050C6E3 84C0` `test al, al`

`:0050C6E5 7413` `je 0050C6FA`

`:0050C6E7 8B0D70445600` `mov ecx, dword ptr [00564470]`

**注释**: 可能的字符串数据引用自代码对象 -> `"installpath"` //写入注册表

|

`:0050C6ED BA40CC5000` `mov edx, 0050CC40`

`:0050C6F2 8B45E8` `mov eax, dword ptr [ebp-18]`

`:0050C6F5 E8EA6CF6FF` `call 004733E4`

**注释**: 被无条件或条件跳转引用
`|:0050C6E5(C)`

|

`:0050C6FA 8B45E8` `mov eax, dword ptr [ebp-18]`

`:0050C6FD E8B26AF6FF` `call 004731B4`

`:0050C702 BA02000080` `mov edx, 80000002`

`:0050C707 8B45E8` `mov eax, dword ptr [ebp-18]`

`:0050C70A E8D56AF6FF` `call 004731E4`

`:0050C70F B101` `mov cl, 01`

**注释**: 可能的字符串数据引用自代码对象 -> `"Software\Microsoft\Windows\CurrentVersion"` ////写入注册表中的位置

|

`:0050C711 BA54CC5000` `mov edx, 0050CC54`

`:0050C716 8B45E8` `mov eax, dword ptr [ebp-18]`

`:0050C719 E82A6BF6FF` `call 00473248`

`:0050C71E 84C0` `test al, al`

`:0050C720 0F84E3010000` `je 0050C909`

**注释**: 可能的字符串数据引用自代码对象 -> `"ProductWRemark"` //注册表中关键性的键值，是否注册，是否到使用次数等都决定于它，后面还会出现多处。

|

`:0050C726 BA88CC5000` `mov edx, 0050CC88`

...《略》...

`:0050C74C 50` `push eax`

`:0050C74D 8D4DD0` `lea ecx, dword ptr [ebp-30]`

**注释**: 可能的字符串数据引用自代码对象 -> `"ProductWRemark"`

|

`:0050C750 BA88CC5000` `mov edx, 0050CC88`

...《略》...

`:0050C787 8D4DC4` `lea ecx, dword ptr [ebp-3C]`

**注释**: 可能的字符串数据引用自代码对象 -> `"ProductWRemark"`

|

`:0050C78A BA88CC5000` `mov edx, 0050CC88`

...《略》...

`:0050C880 66BA97FF` `mov dx, FF97`

`:0050C884 E88FD5FFFF` `call 00509E18`

`:0050C889 8B4DC0` `mov ecx, dword ptr [ebp-40]`

**注释**: 可能的字符串数据引用自代码对象 -> `"ProductWRemark"`

|

`:0050C88C BA88CC5000` `mov edx, 0050CC88`

`:0050C891 8B45E8` `mov eax, dword ptr [ebp-18]`

...《略》...

`:0050C8F4 E81FD5FFFF` `call 00509E18`

`:0050C8F9 8B4DB0` `mov ecx, dword ptr [ebp-50]`

**注释**: 可能的字符串数据引用自代码对象 -> `"ProductWRemark"`

|

`:0050C8FC BA88CC5000` `mov edx, 0050CC88`

...《略》...

`:0050C92F E8B068F6FF` `call 004731E4`

`:0050C934 B101` `mov cl, 01`

**注释**: 可能的字符串数据引用自代码对象 -> `"Software\Microsoft\Windows\CurrentVersion"`

|

`:0050C936 BA54CC5000` `mov edx, 0050CC54`

`:0050C93B 8B45E8` `mov eax, dword ptr [ebp-18]`

`:0050C93E E80569F6FF` `call 00473248`

`:0050C943 84C0` `test al, al`

`:0050C945 0F84AF010000` `je 0050CAFA`

`:0050C94B 8D45E4` `lea eax, dword ptr [ebp-1C]`

`:0050C94E 8B55F4` `mov edx, dword ptr [ebp-0C]`

`:0050C951 E88A7FEFFF` `call 004048E0`

`:0050C956 8D55A4` `lea edx, dword ptr [ebp-5C]`

`:0050C959 8B45E4` `mov eax, dword ptr [ebp-1C]` //送入 `register` 值

`:0050C95C E843CAEFFF` `call 004093A4`

`:0050C961 8B55A4` `mov edx, dword ptr [ebp-5C]`

`:0050C964 8B45F8` `mov eax, dword ptr [ebp-08]` //送入 `userid` 值

`:0050C967 E8C8DDFFFF` `call 0050A734` //关键判断 `CALL`，想知道算法的话，就跟进去。我比较懒。

`:0050C96C 84C0` `test al, al`

`:0050C96E 0F8441010000` `je 0050CAB5` //此处不让它跳就成功一半了！！

`:0050C974 33C0` `xor eax, eax`

`:0050C976 55` `push ebp`

`:0050C977 68E9C95000` `push 0050C9E9`

`:0050C97C 64FF30` `push dword ptr fs:[eax]`

`:0050C97F 648920` `mov dword ptr fs:[eax], esp`

`:0050C982 E8012C0000` `call 0050F588`

`:0050C987 8D4598` `lea eax, dword ptr [ebp-68]`

`:0050C98A B9C4CC5000` `mov ecx, 0050CCC4`

`:0050C98F 8B1580445600` `mov edx, dword ptr [00564480]`

`:0050C995 E8B281EFFF` `call 00404B4C`

`:0050C99A 8B4598` `mov eax, dword ptr [ebp-68]`

`:0050C99D 8D4D9C` `lea ecx, dword ptr [ebp-64]`

`:0050C9A0 66BA14E2` `mov dx, E214`

`:0050C9A4 E86FD4FFFF` `call 00509E18`

`:0050C9A9 8B459C` `mov eax, dword ptr [ebp-64]`

`:0050C9AC 8D4DA0` `lea ecx, dword ptr [ebp-60]`

`:0050C9AF 66BA97FF` `mov dx, FF97`

`:0050C9B3 E860D4FFFF` `call 00509E18`

`:0050C9B8 8B4DA0` `mov ecx, dword ptr [ebp-60]`

**注释**: 可能的字符串数据引用自代码对象 -> `"ProductWRemark"`

|

`:0050C9BB BA88CC5000` `mov edx, 0050CC88`

`:0050C9C0 8B45E8` `mov eax, dword ptr [ebp-18]`

`:0050C9C3 E81C6AF6FF` `call 004733E4`

`:0050C9C8 C6056C44560001` `mov byte ptr [0056446C], 01` //此处赋注册正确的值。

`:0050C9CF 8B45E8` `mov eax, dword ptr [ebp-18]`

`:0050C9D2 E8DD67F6FF` `call 004731B4`

`:0050C9D7 8B45E8` `mov eax, dword ptr [ebp-18]`

`:0050C9DA E81D70EFFF` `call 004039FC`

`:0050C9DF 33C0` `xor eax, eax`

`:0050C9E1 5A` `pop edx`

`:0050C9E2 59` `pop ecx`

`:0050C9E3 59` `pop ecx`

`:0050C9E4 648910` `mov dword ptr fs:[eax], edx`

`:0050C9E7 EB0A` `jmp 0050C9F3`

`:0050C9E9 E9EE74EFFF` `jmp 00403EDC`

`:0050C9EE E81579EFFF` `call 00404308`

### 总结

在 `0050C96E` 处下断点，改跳转，就显示注册成功了，将注册表中值导出成注册文件。

```regedit
REGEDIT4
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion]
"ProductWRemark"="BB21C32E08E9F15F750A"
```

### （2）隐藏的检测

> 大家好，上次的破解留了一个小尾巴，就是作者放了一个暗桩，导致程序运行三分钟之后自动退出。这回解决了这个问题，看下面：

```asm
:0050C961 8B55A4    mov edx, dword ptr [ebp-5C]
:0050C964 8B45F8    mov eax, dword ptr [ebp-08]
:0050C967 E8C8DDFFFF call 0050A734
```
