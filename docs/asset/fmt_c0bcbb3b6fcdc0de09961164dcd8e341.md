
# 汇编代码分析

## 关键循环逻辑

```asm
004E397A  idiv ecx
004E397C  mov edi,edx
004E397E  cmp edi,9
004E3981  jle short gwyinter.004E3986
004E3983  inc dword ptr ss:[ebp-20]
004E3986  lea edx,dword ptr ss:[ebp-24]
004E3989  mov eax,edi
004E398B  call gwyinter.00409578    ; 每次循环计算得到注册码的 2 位数
004E3990  mov edx,dword ptr ss:[ebp-24]
004E3993  lea eax,dword ptr ss:[ebp-1C]
004E3996  call gwyinter.00403F54    ; 将 004E398B 计算值一个个连接起来,注意堆栈的值
004E399B  inc ebx
004E399C  dec esi                 ; 循环指针
004E399D  jnz gwyinter.004E38C4
004E39A3  mov eax,dword ptr ss:[ebp+8]
004E39A6  mov edx,dword ptr ss:[ebp-1C]
004E39A9  call gwyinter.00403D20
004E39AE  xor ecx,ecx
004E39B0  mov edx,5
004E39B5  mov eax,dword ptr ss:[ebp-4]
004E39B8  call gwyinter.004E05CC
004E39BD  xor eax,eax
004E39BF  pop edx
004E39C0  pop ecx
004E39C1  pop ecx
004E39C2  mov dword ptr fs:[eax],edx
004E39C5  push gwyinter.004E39EF
004E39CA  lea eax,dword ptr ss:[ebp-24]
004E39CD  call gwyinter.00403CCC
004E39D2  lea eax,dword ptr ss:[ebp-1C]
004E39D5  call gwyinter.00403CCC
004E39DA  lea eax,dword ptr ss:[ebp-C]
004E39DD  mov edx,2
004E39E2  call gwyinter.00403CF0
004E39E7  retn
004E39E8  jmp gwyinter.004036E0
004E39ED  jmp short gwyinter.004E39CA
004E39EF  pop edi                 ; 0012FC6C
004E39F0  pop esi
004E39F1  pop ebx
004E39F2  mov esp,ebp
004E39F4  pop ebp
004E39F5  retn 4
```

## 小结

该循环对两个值进行计算：一个是机器码：`"CD48047047390"`；一个是字符串：`"SHou0597!@"`。

堆栈得到一个值：
`0012FA54` `00E62E84` `ASCII "2044854858838"`

## 6.4 退出 6.3 的 call 后 F8 继续

```asm
004CBC1C  mov eax,dword ptr ss:[ebp-10]
```
堆栈 `ss:[0012FC70]=014A2150`, `(ASCII "636877907888216")`

```asm
004CBC1F  mov edx,dword ptr ss:[ebp-14]
```
堆栈 `ss:[0012FC6C]=00E62E84`, `(ASCII "2044854858838")`

```asm
004CBC22  call gwyinter.0040405C    ; 估计是注册码比较的地方
004CBC27  jnz short gwyinter.004CBC75    ; 不等就完了吧？
```

这个注册码的判断太像自校验的判断。

## 7. 试验

将计算得到的注册码（？）：`2044854858838` 替换掉注册表里的`"636877907888216"`，运行程序，之前不能点的都可以点了。

另外脱壳爆破后的提示画面好像有麻痹作用。

## 8. 算法总结

- 程序先将输入的注册码转换成数字存在注册表里。
- 运行的时候没有对注册表进行读取注册码比较，一直到点击需注册才能看到的项目时，它才进行关键比较。
- 注册码的计算涉及到两个值，一个机器码，一个内置的字符串 `"SHou0597!@"`。
- 计算过程复杂，不过最后还是明码比较，将计算得到的注册码与注册表里的比较，相等就可以了。

```
4dd466d0e1e0cc123afbe504da2462ca
```
