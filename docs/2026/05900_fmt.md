
# [原创]ASProtect 1.22 - 1.23 手脱

asprotect 1.22的壳比较简单，仅仅是个人学习记录，没啥技术含量，大牛略过。

## 一。找oep

利用最后一次异常到达oep

只忽略kernel32异常。不停shift+f9到最后一次异常。

```
009F00E1     3100             xor dword ptr ds:[eax],eax

009F00E3     64:8F05 00000000 pop dword ptr fs:[0]

009F00EA     58               pop eax

009F00EB     833D DC399F00 00 cmp dword ptr ds:[9F39DC],0
```

alt+m打开内存镜像，在`00401000`段下断。shift+f9 到达oep

```
00442CE8     55               push ebp

00442CE9     8BEC             mov ebp,esp

00442CEB     83C4 F4          add esp,-0C

00442CEE     E8 D506FCFF      call ASPack.004033C8
```

`loadpe`修正镜像大小，完全脱壳。

## 二。修复iat

`importrec`输入oep `42ce8`,获取输入表，显示无效，等级1修复。

能修复一部份。剩下的需要手动修复。我找了半天也没找到那几个iat.

正好发现那几个加密iat都在`009e0000`段，所以就把`9e0000` dump下来。添加到dump文件中。

`loadpe` 区段脱壳。选择`009e0000`.保存`Region009E0000-009EC000.dmp`

pe编辑，选择dump后的程序->区段，选择最后一个，右键->从磁盘加载区段。选择刚才脱壳的区段。

选择刚加载的区段，右键->编辑区段头。

名称可以随便取，rva 为`009E0000 - 00400000 = 005e0000`.关闭，保存。

选项中右边只能选择验证pe，重建pe.选择dump后的文件。

剪切掉`importrec`中无效的指针，修复dump后的文件。ok

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [ASPack.rar](javascript:void(0);)

  （733.85kb，62次下载）
