
### 三、BL=00 XOR 47=47

①、`BL=00 XOR 47=47`

②、`BL=00 XOR 51=51`

使用 `WinHex` 修改 `BeeIcons.exe` 偏移 `000C5B86` 处的 `DB425551` 为：`EFB84751`

另存为 `fly.eXe`，第六步时需要在里面进行最终的修改

---

## 五、RegOpenKeyExA

得到上面的地址运算数据后，我们还要得到壳通过读取注册表中的信息判断是否过期的地方。

使用 `Ctrl+F` 在“整个段块”搜索命令：`push 20019`

```
00FD961D    68 19000200     push 20019
```
//找到这里

```
00FD9622    6A 00           push 0
00FD9624    8BC7            mov eax,edi
00FD9626    E8 31A2FFFF     call 00FD385C
00FD962B    50              push eax
00FD962C    56              push esi
00FD962D    E8 C6BAFFFF     call 00FD50F8 ; jmp to advapi32.RegOpenKeyExA
```
//Call `RegOpenKeyExA`

```
00FD9632    85C0            test eax,eax
00FD9634    0F85 84000000   jnz 00FD96BE
```

`call 00FD50F8` 就是 `call RegOpenKeyExA`

点右键，搜索全部命令：`call 00FD50F8`

| 地址 | 反汇编 | 注释 |
| :--- | :--- | :--- |
| `00FD962D` | `call 00FD50F8` | `jmp to advapi32.RegOpenKeyExA` |
| `00FD970A` | `call 00FD50F8` | `jmp to advapi32.RegOpenKeyExA` |
| `00FD9776` | `call 00FD50F8` | `jmp to advapi32.RegOpenKeyExA` |
| `00FD9829` | `call 00FD50F8` | `jmp to advapi32.RegOpenKeyExA` |
| `00FD9911` | `call 00FD50F8` | `jmp to advapi32.RegOpenKeyExA` |

一般修改后面 3 个 `CALL` 后的跳转就行了：

```
00FD9776    E8 7DB9FFFF     call 00FD50F8 ; jmp to advapi32.RegOpenKeyExA
00FD977B    85C0            test eax,eax
00FD977D    75 66           jnz short 00FD97E5
```
//修改①、 `jmp short 00FD97E5`

```
00FD9829    E8 CAB8FFFF     call 00FD50F8 ; jmp to advapi32.RegOpenKeyExA
00FD982E    85C0            test eax,eax
00FD9830    0F85 93000000   jnz 00FD98C9
```
//修改②、 `jmp 00FD98C9`

```
00FD9911    E8 E2B7FFFF     call 00FD50F8 ; jmp to advapi32.RegOpenKeyExA
00FD9916    85C0            test eax,eax
00FD9918    75 30           jnz short 00FD994A
```
//修改③、 `jmp 00FD994A`

记住这三个需要修改的地方：`00FD977D`、`00FD9830`、`00FD9918`

---

## 六、Patch 最终处理

用 `Ollydbg` 载入第四步保存的 `fly.eXe`，忽略所有异常

直接下断：`BP 400080`，`Shift+F9` 后中断下来

```
00400080    0000            add byte ptr ds:[eax],al
```
//中断

OK，`ASProtect` 成功到达我们指定的地点！现在写入 `Patch` 代码：

```
00400080    A1 07366100     mov eax,dword ptr ds:[613607]
```
//\[613607\]中是第三步得到的壳重定位后的基址

```
00400085    C680 7D970000 E>mov byte ptr ds:[eax+977D],0EB
```
//这三句是修改壳通过读取注册表中的信息判断是否过期的地方
//修改①、 `jmp 00FD97E5`

```
0040008C    C780 30980000 E>mov dword ptr ds:[eax+9830],94E9
```
//修改②、 `jmp 00FD98C9`

```
00400096    C680 18990000 E>mov byte ptr ds:[eax+9918],0E9
```
//修改③、 `jmp 00FD994A`

```
0040009D    55              push ebp
0040009E    8BEC            mov ebp,esp
```
//上面 2 句是把 `0052FAB4` 和 `0052FAB5` 代码挪到这里执行

```
004000A0    B8 C0004000     mov eax,004000C0 ; ASCII "fly"
```
//直接在 `004000C0` 处写入注册名：`fly`

```
004000A5    E9 10FA1200     jmp 0052FABA
```
//跳回去继续流程

从 `Ollydbg` 中“二进制复制”如下：

```
A1 07 36 61 00 C6 80 7D 97 00 00 EB C7 80 30 98 00 00 E9 94 00 00 C6 80 18 99 00 00 E9 55 8B EC
B8 C0 00 40 00 E9 10 FA 12 00
```

前面所有的工作只为了这里写入的几行代码，而这几行代码则使得这个使用 `ASProtect` 保护的程序变成了注册版！

至于制作 `Patch`，可以随意选择补丁工具，看你的喜好了。我选择 `t.p.e` 制作了通用的文件补丁。

**Game ！**

---

```
      ,     \_/ 
    /| \_.-~/          \\\_     ,        青春都一晌
   ( /~   /             \\~-.\_ |\\
   \`\\  \_/                \~ )          忍把浮名 
-_~~~-.)  )\_//;;,.        \\  //
/'\_,\ --~ \ ~~~- ,;;\_\_\_( (.-~~~-.       换了破解轻狂
`~ _( ,_..\ (     ,;''/    ~-- /.\`\\
  /~~//'  / `~\\       ) /--.._, )\ `~
  "  `~"  "      `     /~'\`\\    `\\~~\\
                  "    "   "~'  ""
         UnPacked By  :  fly
          2005-03-16 20:00 
```
`4dd466d0e1e0cc123afbe504da2462ca`
