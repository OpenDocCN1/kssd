
## 六、输出表修复

由于上面输入表的修复覆盖了输出表，所以我们要把输出表重新挪个位置。

打开`dumped.dll`，把`0020EC10`处输出表数据复制写入到`dumped_.dll`的`0X001B5300`处。

![](http://bbs.pediy.com/upload/2005/8/image/exporttable.gif)

修正Export Table各指针，再修正`dumped_.dll`的Export Table RVA。

也可以用看雪老师的`PeMove`挪移输出表。复制`dumped.dll`，用`PeMove`打开，Export Table's File Offset填入`001B5300`，点击Move后把`001B5300`处的输出表复制写入到`dumped_.dll`中，修正Export Table RVA。

![](http://bbs.pediy.com/upload/2005/8/image/pemove.gif)

修正输出表所在区段`0019E000`段的VSize和RSize为`001B6000-0019E000＝00018000`。

---

## 七、重定位表修复

Armadillo对于DLL比较友好，没有加密重定位表，因此就没有再调试时跟踪此重定位处理了。

LordPE察看区段，`.reloc`段RVA即是重定位表RVA，Size看看其结尾的00就知道了。

```
00297590   A3 3A B8 3A D7 3A DC 3D  E0 3D E4 3D E8 3D EC 3D   ????????
002975A0   F0 3D F4 3D 00 00 00 00  00 00 00 00 00 00 00 00   ??............
```

修正`dumped_.dll`的Relocation RVA=`00276000`，Size=`002975A4-00276000=215A4`。

---

## 八、Armadillo脱壳文件的优化

其实优化已经在《ArmInline――Armadillo客户版Code Splicing+Import Table Elimination的简便修复方法》中说过了，再重复一次吧。

用LordPE删除`.reloc`区段下的所有壳区段，记住第一个壳区段`.text1`段的RVA=`002A6000`。

![](http://bbs.pediy.com/upload/2005/8/image/cutsections.gif)

用WinHex移除`0X002A6000`至末尾的所有数据，另存为`UnPacKed.dll`。好了，壳数据基本清理完了。

用`DT_FixRes`修复资源。DT_FixRes打开修复前的`dumped.dll`。

![](http://bbs.pediy.com/upload/2005/8/image/dt_fixres.gif_891.gif)

`NewRva=002A6000`，`FileAlignment=1000`，Dump Resource，获得`rsrc.bin`。

用LordPE把`rsrc.bin`载入`UnPacKed.dll`，修正Resource RVA=`002A6000`。

---

## 九、破解

启动VC时会弹出“License Error”的提示，无法使用。以这个为线索就很容易定位了。

---

### 1、License Error

```
1ED411D3      385C24 13          cmp byte ptr ss:[esp+13],bl
1ED411D7      74 47              je short 1ED41220
//修改为：jmp 1ED41220
1ED411D9      A1 C44CF21E        mov eax,dword ptr ds:[1EF24CC4]
1ED411DE      53                 push ebx
1ED411DF      68 38AFEB1E        push 1EEBAF38 ; ASCII "License"
1ED411E4      68 30AFEB1E        push 1EEBAF30 ; ASCII "Error"
1ED411E9      50                 push eax
1ED411EA      FF15 2C69EB1E      call dword ptr ds:[1EEB692C] ; USER32.MessageBoxA
```

---

### 2、过期

```
1ED41384      8B5424 4C          mov edx,dword ptr ss:[esp+4C]
1ED41388      8B4424 50          mov eax,dword ptr ss:[esp+50]
1ED4138C      8B4C24 54          mov ecx,dword ptr ss:[esp+54]
1ED41390      52                 push edx
1ED41391      50                 push eax
1ED41392      51                 push ecx
1ED41393      E8 58D80900        call 1EDDEBF0
1ED41398      83C4 1C            add esp,1C
1ED4139B      E8 90DC0900        call 1EDDF030
1ED413A0      3BC3               cmp eax,ebx
1ED413A2      0F84 33010000      je 1ED414DB
//修改为：jmp 1ED414DB
```

---

### 3、局域网验证

```
1ED673FF      68 E495EB1E        push 1EEB95E4 ; ASCII "UserName"
1ED67404      68 BC95EB1E        push 1EEB95BC ; ASCII "Software\Whole Tomato\Visual Assist X"
1ED67409      8D4424 10          lea eax,dword ptr ss:[esp+10]
1ED6740D      68 01000080        push 80000001
1ED67412      50                 push eax
1ED67413      C74424 24 00000000 mov dword ptr ss:[esp+24],0
1ED6741B      E8 602F0000        call 1ED6A380
1ED67420      8B4C24 18          mov ecx,dword ptr ss:[esp+18]
1ED67424      51                 push ecx
1ED67425      8D5424 18          lea edx,dword ptr ss:[esp+18]
1ED67429      68 E8EDEB1E        push 1EEBEDE8 ; ASCII "All instances of the license ",LF,"""%s""",LF,"are in use.  Visual Assist X will be disabled.  You must unload or uninstall Visual Assist X ",LF,"and restart your IDE to prevent this PC from being counted by other license checks."
```

向上回溯：

```
1ED67480     55                 push ebp
1ED67481     8BEC               mov ebp,esp
1ED67483     6A FF              push -1
1ED67485     68 0AA5E81E        push 1EE8A50A
1ED6748A     64:A1 00000000     mov eax,dword ptr fs:[0]
1ED67490     50                 push eax
1ED67491     64:8925 00000000   mov dword ptr fs:[0],esp

//修改为：
1ED67480     B8 01000000        mov eax,1
1ED67485     C3                 retn
```

Game Over

```
         ,     \_/
        /| \_.-~/            \\_     ,        青春都一晌
       ( /~   /              \~-.\_ |\
       `\\  \_/                \   ~\ )          忍把浮名
   \_-~~~-.)  )\_\_/;;,.          \\_  //'
  /'\_,\   --~   \ ~~~-  ,;;\\_\_\_(  (.-~~~-.        换了破解轻狂
`~ \_( ,\_..--\ (     ,;'' /    ~--   /.\_`\
  /~~//'   /' `~\         ) /--..\_, )\_  `~
  "  `~"  "      `"      /~'`\    `\\~~\
                         "     "   "~'  ""

              UnPacked By :  fly
               2005-12-12 00:00
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
