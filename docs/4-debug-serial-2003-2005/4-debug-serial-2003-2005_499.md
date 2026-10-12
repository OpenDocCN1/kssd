
# 天音怒放

> 原文：[`www.pediy.com/kssd/pediy05/pediy50198.htm`](https://www.pediy.com/kssd/pediy05/pediy50198.htm)

作者：`HOOK[TTG]=钩子[TTG]`

目的：练习使用 `W32dsm`，`Trw 1.22`

（1）这个软件是用 `Aspack` 压缩的，但是我用 `UnAspack` 和 `procdum` 都不好使。用 `UnAspack` 解压说：不是压缩文件；用 `procdum` 总是截不住，“天音怒放”自己就运行了。。。。。。。(请问大虾，这是咋回事！！！！）

（2）没法子，只好现学现卖了！！手动脱壳了。。。。。。。

（3）好！废话少说，跟我一起来吧。。。。。。。。。

（4）用 `Trw 1.22` 载入“天音怒放”（将“天音怒放”的图标拖到 `Trw 1.22` 上，然后按载入调试按钮），此时会拦截在此

`XXXX:004F4001DB??`

`004F4002DB??`

`====================================TyPlayer!Aspack+XXXX=======================================`

此时，按一下 `F12` “天音怒放”就运行了，只好重来了。这回老实点，一路 `F10` 吧。到

`XXXX:004F44CCALL004F446D` 一带过“天音怒放”就运行了！！！

按 `F8` 追入！！

》》》`XXXX：004F4503JNZNEAR 004F44A1`

`XXXX：004F4509SUBEBX，C37EE801`

但是注意到：`XXXX：004F44A1JMP004F44C2` 是无条件跳转。。。。。

就在 `XXXX：004F4509SUBEBX，C37EE801` 设为断点，然后用 `G` 命令跳过循环。。。。。

一路 `F10`，因为在我这里一按 `F12` “天音怒放”程序就卡死了，所以。。。。。。。。

中间有很多环式跳转，我在每次有 `CALLxxxxxxxx` 的地方，都会记下此行的地址，以备后用。。。。。。

最后到：

`XXXX：004F4190CALLNEAR[EBP+00444000]` 按下 `F12`，“天音怒放”没卡死，真幸运！！！

接着会到此：

`XXXX：004F43BARET` 按 `F10` 来到。。。。。

`XXXX：004C0058PUSHEBP`

此时，注意到领空标志变为“`TyPlayer!CODE+XXXX`”，我知道，上帝同情我，让我走到了尽头！！！！！！我成功了！！！

`==============================================TyPlayer!CODE+XXXX========================================`

（5）好！！定位在 `XXXX：004C0058PUSHEBP`，下 `makePE HOOK[TTG].EXE` 就会得到脱了壳的“天音怒放”了。。。

文件应该在 `Trw 1.22` 目录内，改名为 `TyPlayer` 就可以了！！但是我发现，右键菜单编程英文的了。。。。。。

没关系，不管他，反正已经脱壳了。。。。。。。。。

下面进入寻找注册码阶段！！！

（1）现在就用 `W32dsm` 进行反汇编。

（2）查找字符串 `"注册码不正确"`。

```asm
* Possible StringData Ref from Code Obj ->"谢谢你选择了天音怒放"
                        |
:004BA8A9 B860A94B00             mov eax, 004BA960
:004BA8AE E89DFEF9FF             call 0045A750
:004BA8B3 A160334C00             mov eax, dword ptr [004C3360]
:004BA8B8 8B00                  mov eax, dword ptr [eax]
:004BA8BA C6809A04000001         mov byte ptr [eax+0000049A], 01
:004BA8C1 C645F701               mov [ebp-09], 01
:004BA8C5 EB0E                   jmp 004BA8D5

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004BA84B(C)
|
:004BA8C7 C645F700               mov [ebp-09], 00

* Possible StringData Ref from Code Obj ->"注册码不正确"
                        |
:004BA8CB B880A94B00             mov eax, 004BA980
:004BA8D0 E87BFEF9FF             call 0045A750
```

（3）转到调用此段的地方“`004BA84B`”

```asm
:004BA811 8945FC                 mov dword ptr [ebp-04], eax
:004BA814 8B45FC                 mov eax, dword ptr [ebp-04]**User Name
:004BA817 E85099F4FF             call 0040416C   ***********判断是否空
:004BA81C 8B45F8                 mov eax, dword ptr [ebp-08]**Serial No
:004BA81F E84899F4FF             call 0040416C   ***********判断是否空
:004BA824 33C0                   xor eax, eax
:004BA826 55                     push ebp
:004BA827 68FFA84B00             push 004BA8FF
:004BA82C 64FF30                 push dword ptr fs:[eax]
:004BA82F 648920                 mov dword ptr fs:[eax], esp
:004BA832 8B45F8                 mov eax, dword ptr [ebp-08]
:004BA835 50                     push eax
:004BA836 8B4DFC                 mov ecx, dword ptr [ebp-04]
:004BA839 B201                   mov dl, 01
:004BA83B A1289B4B00             mov eax, dword ptr [004B9B28]
:004BA840 E8EF4B0000             call 004BF434*************************关键！！！！
:004BA845 8BD8                   mov ebx, eax
:004BA847 807B1000               cmp byte ptr [ebx+10], 00
:004BA84B 747A                   je 004BA8C7  ****************************在此
:004BA84D A160334C00             mov eax, dword ptr [004C3360]
:004BA852 8B00                   mov eax, dword ptr [eax]
:004BA854 8B909C040000           mov edx, dword ptr [eax+0000049C]
:004BA85A 8D45F0                 lea eax, dword ptr [ebp-10]

* Possible StringData Ref from Code Obj ->"Reg.ini"************************************注意
                        |
:004BA85D B918A94B00             mov ecx, 004BA918
:004BA862 E89D97F4FF             call 00404004
:004BA867 8B4DF0                 mov ecx, dword ptr [ebp-10]***********
:004BA86A B201                   mov dl, 01

* Possible StringData Ref from Code Obj ->"\5G"
                        |
:004BA86C A1942E4700             mov eax, dword ptr [00472E94]
:004BA871 E8C686FBFF             call 00472F3C
:004BA876 8BF0                   mov esi, eax
:004BA878 8B45FC                 mov eax, dword ptr [ebp-04]*****
:004BA87B 50                     push eax

* Possible StringData Ref from Code Obj ->"User Name"
                        |
:004BA87C B928A94B00             mov ecx, 004BA928

* Possible StringData Ref from Code Obj ->"RegInfo"
                        |
:004BA881 BA3CA94B00             mov edx, 004BA93C
:004BA886 8BC6                   mov eax, esi
:004BA888 8B38                   mov edi, dword ptr [eax]
:004BA88A FF5704                 call [edi+04]
:004BA88D 8B45F8                 mov eax, dword ptr [ebp-08]*****
:004BA890 50                     push eax

* Possible StringData Ref from Code Obj ->"Serial No"
                        |
:004BA891 B94CA94B00             mov ecx, 004BA94C

* Possible StringData Ref from Code Obj ->"RegInfo"
                        |
:004BA896 BA3CA94B00             mov edx, 004BA93C
:004BA89B 8BC6                   mov eax, esi
:004BA89D 8B38                   mov edi, dword ptr [eax]
:004BA89F FF5704                 call [edi+04]
:004BA8A2 8BC6                   mov eax, esi
:004BA8A4 E82787F4FF             call 00402FD0

* Possible StringData Ref from Code Obj ->"谢谢你选择了天音怒放"
                        |
:004BA8A9 B860A94B00             mov eax, 004BA960
:004BA8AE E89DFEF9FF             call 0045A750
:004BA8B3 A160334C00             mov eax, dword ptr [004C3360]
:004BA8B8 8B00                   mov eax, dword ptr [eax]
:004BA8BA C6809A04000001         mov byte ptr [eax+0000049A], 01
:004BA8C1 C645F701               mov [ebp-09], 01
:004BA8C5 EB0E                   jmp 004BA8D5
```

可知，“天音怒放”将注册名和注册码存入文件 `Reg.ini` 中了。这就应该想到下次运行还得检查合法性。。。。。

也可以改相应的地方，使之变成破解版。但是，这次，我们要找出注册码，要注册！！！！

（4）从上可以看出`[ebp-04]`，`[ebp-08]`，`[ebp-10]`，应与注册有关系。

`[ebp-04]`〉`User Name`；`[ebp-08]`〉`Serial No`

看看，上面的注释和分析，可以确定

```asm
:004BA840 E8EF4B0000             call 004BF434*************************关键！！！！
```

（5）好！运行“天音怒放”，并注册。启动 `Trw 1。22`，设断 `bpx 004BA840`。确定拦截！！`F8` 追入！！！

```asm
:004BF434 55                     push ebp  ***************
:004BF435 8BEC                   mov ebp, esp
:004BF437 51                     push ecx
:004BF438 53                     push ebx
```



### （5）汇编代码片段

```
:004BF439 56                       push esi  
:004BF43A 84D2                     test dl, dl  
:004BF43C 7408                     je 004BF446  
:004BF43E 83C4F0                   add esp, FFFFFFF0  
:004BF441 E8CA3EF4FF               call 00403310  ***********带过  
:004BF446 894DFC                   mov dword ptr [ebp-04], ecx  
:004BF449 8BDA                     mov ebx, edx  
:004BF44B 8BF0                     mov esi, eax  
:004BF44D 8B45FC                   mov eax, dword ptr [ebp-04]  
:004BF450 E8174DF4FF               call 0040416C*********判断是否空  
:004BF455 8B4508                   mov eax, dword ptr [ebp+08]***Serial No  
:004BF458 E80F4DF4FF               call 0040416C*********判断是否空  
:004BF45D 33C0                     xor eax, eax  
:004BF45F 55                       push ebp  
:004BF460 689EF44B00               push 004BF49E  
:004BF465 64FF30                   push dword ptr fs:[eax]  
:004BF468 648920                   mov dword ptr fs:[eax], esp  
:004BF46B 8D460C                   lea eax, dword ptr [esi+0C]  
:004BF46E 8B5508                   mov edx, dword ptr [ebp+08]  
:004BF471 E81649F4FF               call 00403D8C  
:004BF476 8B55FC                   mov edx, dword ptr [ebp-04]*****User Name  
:004BF479 8BC6                     mov eax, esi  
:004BF47B E8E0000000               call 004BF560**************关键！！！追入！！！  
:004BF480 33C0                     xor eax, eax  
:004BF482 5A                       pop edx  
:004BF483 59                       pop ecx  
:004BF484 59                       pop ecx  
:004BF485 648910                   mov dword ptr fs:[eax], edx  
:004BF488 68A5F44B00               push 004BF4A5
```

### （6）继续

```
:004BF568 6A00                     push 00000000  
:004BF56A 6A00                     push 00000000  
:004BF56C 49                       dec ecx  
:004BF56D 75F9                     jne 004BF568  
:004BF56F 51                       push ecx  
:004BF570 53                       push ebx  
:004BF571 56                       push esi  
:004BF572 57                       push edi  
:004BF573 8BF2                     mov esi, edx*****User Name  
:004BF575 8BD8                     mov ebx, eax  
:004BF577 33C0                     xor eax, eax  
:004BF579 55                       push ebp  
:004BF57A 68D1F74B00               push 004BF7D1  
:004BF57F 64FF30                   push dword ptr fs:[eax]  
:004BF582 648920                   mov dword ptr fs:[eax], esp  
:004BF585 33C0                     xor eax, eax  
:004BF587 8945FC                   mov dword ptr [ebp-04], eax  
:004BF58A 8D55EC                   lea edx, dword ptr [ebp-14]  
:004BF58D 8BC6                     mov eax, esi  
:004BF58F E88893F4FF               call 0040891C  
:004BF594 8B55EC                   mov edx, dword ptr [ebp-14]*****User Name，小写变大写  
:004BF597 8D4308                   lea eax, dword ptr [ebx+08]  
:004BF59A E8ED47F4FF               call 00403D8C  
:004BF59F BE07000000               mov esi, 00000007  
:004BF5A4 BF08324C00               mov edi, 004C3208  
:004BF5A9 8B4308                   mov eax, dword ptr [ebx+08]  
:004BF5AC 8B17                     mov edx, dword ptr [edi]  
:004BF5AE E8154BF4FF               call 004040C8**************关键！！！追入！！！  
:004BF5B3 7511                     jne 004BF5C6  
:004BF5B5 8D4304                   lea eax, dword ptr [ebx+04]  
:004BF5B8 E87B47F4FF               call 00403D38  
:004BF5BD C6431000                 mov [ebx+10], 00  
:004BF5C1 E9F0010000               jmp 004BF7B6
```

### （7）

```
:004040C8 53                       push ebx  
:004040C9 56                       push esi  
:004040CA 57                       push edi  
:004040CB 89C6                     mov esi, eax  
:004040CD 89D7                     mov edi, edx  
:004040CF 39D0                     cmp eax, edx
```

以下，就不一一写了，免的有凑数之嫌！！！  

最后，会到此：  

```
:004BF70F 8B4304                   mov eax, dword ptr [ebx+04]********真的注册码  
:004BF712 E8A94AF4FF               call 004041C0     ********还得用‘-’分组  
:004BF717 8D45F4                   lea eax, dword ptr [ebp-0C]********分组规则嘛，望下看看  
:004BF71A 50                       push eax     ********作者会告诉你的。。。。。  
:004BF71B B904000000               mov ecx, 00000004  
:004BF720 BA04000000               mov edx, 00000004  
:004BF725 8B4304                   mov eax, dword ptr [ebx+04]  
:004BF728 E8934AF4FF               call 004041C0  
:004BF72D 8D45F0                   lea eax, dword ptr [ebp-10]  
:004BF730 50                       push eax  
:004BF731 B902000000               mov ecx, 00000002  
:004BF736 BA08000000               mov edx, 00000008  
:004BF73B 8B4304                   mov eax, dword ptr [ebx+04]  
:004BF73E E87D4AF4FF               call 004041C0  
:004BF743 8D45E4                   lea eax, dword ptr [ebp-1C]  
:004BF746 50                       push eax  
:004BF747 B903000000               mov ecx, 00000003  
:004BF74C BA01000000               mov edx, 00000001  
:004BF751 8B430C                   mov eax, dword ptr [ebx+0C]******假的注册码  
:004BF754 E8674AF4FF               call 004041C0  
```

```
:004BF759 8B45E4                   mov eax, dword ptr [ebp-1C]******假的注册码的前三位  
:004BF75C 8B55F8                   mov edx, dword ptr [ebp-08]******真的注册码的前三位  
:004BF75F E86449F4FF               call 004040C8****************进行比对！！！！
```

### （8）

整理为：

姓名：`HOOK[TTG]`

注册码（假）：`7123456`

注册码（真）：`142-9510-28`

### （9）

谢谢每一位帮助和关心我的人！！！！！！

### （10）

转载自由！！！但请保持完整！！！！！！！

Writenby `HOOK[TTG]` `4dd466d0e1e0cc123afbe504da2462ca`
