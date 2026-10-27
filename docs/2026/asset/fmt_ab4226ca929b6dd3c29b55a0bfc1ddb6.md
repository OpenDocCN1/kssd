
修改完Magic Jump和自校验后，直接F4到`00475449`处。

下断:`HE GetModuleHandleA` F9运行，中断后取消断点。Alt+F9返回

```
00471C95     E8 00000000         call lm.00471C9A

00471C9A     59                  pop ecx

00471C9B     81C1 72F4FFFF       add ecx,-0B8E

00471CA1     0301                add eax,dword ptr ds:[ecx]

//EAX=00400000+00043B6E=00443B6E 这就是OEP值 ★

00471CA3     8BF8                mov edi,eax  ; lm.00443B6E

00471CA5     50                  push eax

00471CA6     E8 00000000         call lm.00471CAB
```

下面经过几个ret后就可以返回OEP了

当然，可以使用第2区段内存断点大法直接暂停在OEP处。

我们下命令：`G 00443B6E` 回车之后就直达OEP啦

```
00443B6E     55                  push ebp

//在这儿用LordPE完全Dump此进程

00443B6F     8BEC                mov ebp,esp

00443B71     6A FF               push -1

00443B73     68 90E74400         push lm.0044E790

00443B78     68 D83C4400         push lm.00443CD8 ; jmp to msvcrt.\_except\_handler3

00443B7D     64:A1 00000000      mov eax,dword ptr fs:[0]

00443B83     50                  push eax

00443B84     64:8925 00000000    mov dword ptr fs:[0],esp

00443B8B     83EC 68             sub esp,68

00443B8E     53                  push ebx

00443B8F     56                  push esi

00443B90     57                  push edi

00443B91     8965 E8             mov dword ptr ss:[ebp-18],esp

00443B94     33DB                xor ebx,ebx

00443B96     895D FC             mov dword ptr ss:[ebp-4],ebx

00443B99     6A 02               push 2

00443B9B     FF15 9C9E4400       call dword ptr ds:[449E9C]; msvcrt.\_\_set\_app\_type
```

运行ImportREC，选择这个进程。把OEP改为`00043B6E`，点IT AutoSearch，很明显ImportREC自动得到的数据不正确。

可以手动定位IAT RVA和大小，但是这个壳有更简单的方法。

用LordPE打开原来的`lm.exe`，看到IAT RVA=`00049000`、Size=`000010F8`，呵呵，壳没有加密IAT信息。直接在ImportREC里填入这个数据，点“Get Import”。我们已经避开输入表加密了，所以直接CUT掉填充在DLL名之间的垃圾数据，就可以得到完整的输入表了。

可以“新增区段”来修复，不过我选择放在原来的输入表处。New Import Infors填入RVA=`000547D0`，把ImportREC新建输入表的3个选项清空，FixDump！

简单优化一下。用LordPE删除最后的`.Shield`区段，重建PE即可。

希望作者能够继续完善这个壳，成长为新的猛壳。

---

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

               2005-01-18  零点
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
