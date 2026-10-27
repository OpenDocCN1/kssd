
### 调试分析

`012B2A4F`  `|.  8B48 04    mov ecx,dword ptr ds:[eax+4]`

`012B2A52`  `|.  8B51 04    mov edx,dword ptr ds:[ecx+4]`

`012B2A55`  `|.  8B45 0C    mov eax,dword ptr ss:[ebp+C]`

`012B2A58`  `|.  8B48 04    mov ecx,dword ptr ds:[eax+4]`

`012B2A5B`  `|.  8B41 04    mov eax,dword ptr ds:[ecx+4]`

`012B2A5E`  `|.  8B4D 0C    mov ecx,dword ptr ss:[ebp+C]`

`012B2A61`  `|.  8D4401 04  lea eax,dword ptr ds:[ecx+eax+>`

`012B2A65`  `|.  8B4D 0C    mov ecx,dword ptr ss:[ebp+C]`

`012B2A68`  `|.  8B5411 04  mov edx,dword ptr ds:[ecx+edx+>`

`012B2A6C`  `|.  50         push eax`

`012B2A6D`  `|.  8B42 30    mov eax,dword ptr ds:[edx+30]`

`012B2A70`  `|.  FFD0       call eax;调用getMethodModule`

`012B2A72`  `|.  8945 F0    mov dword ptr ss:[ebp-10],eax`

`012B2A75`  `|.  8D4D F0    lea ecx,dword ptr ss:[ebp-10]`

`012B2A78`  `|.  51         push ecx`

`012B2A79`  `|.  8D55 F8    lea edx,dword ptr ss:[ebp-8]`

`012B2A7C`  `|.  52         push edx`

`012B2A7D`  `|.  8B8D 7CFFF>mov ecx,dword ptr ss:[ebp-84]`

`012B2A83`  `|.  E8 88FEFFF>call CliSec\_1.012B2910`

`012B2A88`  `|.  8D45 98    lea eax,dword ptr ss:[ebp-68]`

`012B2A8B`  `|.  50         push eax`

`012B2A8C`  `|.  8B8D 7CFFF>mov ecx,dword ptr ss:[ebp-84]`

`012B2A92`  `|.  E8 89FAFFF>call CliSec\_1.012B2520`

`012B2A97`  `|.  50         push eax`

`012B2A98`  `|.  8D4D F8    lea ecx,dword ptr ss:[ebp-8]`

`012B2A9B`  `|.  E8 80F7FFF>call CliSec\_1.012B2220`

`012B2AA0`  `|.  0FB6C8     movzx ecx,al`

`012B2AA3`  `|.  85C9       test ecx,ecx`

`012B2AA5`  `|.  0F84 35020>je CliSec\_1.012B2CE0;正常（需要解密）时这里不会跳`

#### 紧接着代码取方法的名称：

`012B2AAB`  `|.  8D55 B4    lea edx,dword ptr ss:[ebp-4C]`

`012B2AAE`  `|.  52         push edx`

`012B2AAF`  `|.  8B45 10    mov eax,dword ptr ss:[ebp+10]`

`012B2AB2`  `|.  8B08       mov ecx,dword ptr ds:[eax]`

`012B2AB4`  `|.  51         push ecx`

`012B2AB5`  `|.  8B55 0C    mov edx,dword ptr ss:[ebp+C]`

`012B2AB8`  `|.  8B42 04    mov eax,dword ptr ds:[edx+4]`

`012B2ABB`  `|.  8B48 04    mov ecx,dword ptr ds:[eax+4]`

`012B2ABE`  `|.  8B55 0C    mov edx,dword ptr ss:[ebp+C]`

`012B2AC1`  `|.  8B42 04    mov eax,dword ptr ds:[edx+4]`

`012B2AC4`  `|.  8B50 04    mov edx,dword ptr ds:[eax+4]`

`012B2AC7`  `|.  8B45 0C    mov eax,dword ptr ss:[ebp+C]`

`012B2ACA`  `|.  8D5410 04  lea edx,dword ptr ds:[eax+edx+>`

`012B2ACE`  `|.  8B45 0C    mov eax,dword ptr ss:[ebp+C]`

`012B2AD1`  `|.  8B4C08 04  mov ecx,dword ptr ds:[eax+ecx+>`

`012B2AD5`  `|.  52         push edx`

`012B2AD6`  `|.  8B11       mov edx,dword ptr ds:[ecx]`

`012B2AD8`  `|.  FFD2       call edx    ;这里调用.text:7A0A2503 CEEInfo::getMethodName`

这两大段代码其实就是我在上一篇调试随记中讲的取vtable的过程，这里略过。最后一句`call edx`后，看一下`eax`的值：`EAX 11095453 ASCII "UxQ="`。这说明了显示关于窗口的方法名称是”UxQ=”，显示，这是混淆过的。下面的代码调用`getMethodDefFormMethod`：

`012B2ADA`  `|.  8945 B0    mov dword ptr ss:[ebp-50],eax`

`012B2ADD`  `|.  8B45 10    mov eax,dword ptr ss:[ebp+10]`

`012B2AE0`  `|.  8B08       mov ecx,dword ptr ds:[eax]`

`012B2AE2`  `|.  51         push ecx`

`012B2AE3`  `|.  8B55 0C    mov edx,dword ptr ss:[ebp+C]`

`012B2AE6`  `|.  8B42 04    mov eax,dword ptr ds:[edx+4]`

`012B2AE9`  `|.  8B48 0C    mov ecx,dword ptr ds:[eax+C]`

`012B2AEC`  `|.  8B55 0C    mov edx,dword ptr ss:[ebp+C]`

`012B2AEF`  `|.  8B42 04    mov eax,dword ptr ds:[edx+4]`

`012B2AF2`  `|.  8B50 0C    mov edx,dword ptr ds:[eax+C]`

`012B2AF5`  `|.  8B45 0C    mov eax,dword ptr ss:[ebp+C]`

`012B2AF8`  `|.  8D5410 04  lea edx,dword ptr ds:[eax+edx+>`

`012B2AFC`  `|.  8B45 0C    mov eax,dword ptr ss:[ebp+C]`

`012B2AFF`  `|.  8B4C08 04  mov ecx,dword ptr ds:[eax+ecx+>`

`012B2B03`  `|.  52         push edx`

`012B2B04`  `|.  8B51 38    mov edx,dword ptr ds:[ecx+38]`

`012B2B07`  `|.  FFD2       call edx;调用.text:7A0A27FB CEEInfo::getMethodDefFromMethod`

`CEEInfo::getMethodDefFromMethod`是没有见过的函数，在遇到这种情况时，对于`.net 2.0`我们可以参考`sscli`的源代码或直接看反汇编代码。这里直接看返回值，答案就明了了。`EAX 060000B0`，这个函数返回的就是方法的`token`。

紧接着的代码比较是否是有效的`token`值，如果是，则继续运行。代码如下：

`012B2B09`  `|.  8945 B8    mov dword ptr ss:[ebp-48],eax`

`012B2B0C`  `|.  8B45 B8    mov eax,dword ptr ss:[ebp-48]`

`012B2B0F`  `|.  25 FFFFFF0>and eax,0FFFFFF`

`012B2B14`  `|.  0F84 C6010>je CliSec\_1.012B2CE0`

紧接着代码如下：

`012B2B1A`  `|.  8B4D B8    mov ecx,dword ptr ss:[ebp-48]`

`012B2B1D`  `|.  81E1 FFFFF>and ecx,0FFFFFF`

`012B2B23`  `|.  894D A4    mov dword ptr ss:[ebp-5C],ecx`

`012B2B26`  `|.  8D4D F8    lea ecx,dword ptr ss:[ebp-8]`

`012B2B29`  `|.  E8 72F9FFF>call CliSec\_1.012B24A0`

这里`012B2B26`处的`ecx=0013E890`，指向了椎栈。调试到现在，我们并没有跟踪这个椎栈值的变化，更不可能知道它的作用。因此，重新调试（这就是选择一个固定方法的好处，可重复性），并观察该处值的改变。

首先记下此时的椎栈值：

`0013E890   012D45AC  CliSec\_1.012D45AC`

`0013E894   012F2178`

重新运行后再次中断在`hook`方法的入口处，注意这时观察椎栈的值，特别是`0013E890`处的变化。

![](http://bbs.pediy.com/upload/2006/4/image/33.jpg)

入口处的值与刚才第一次调试过程中的值不一样，因此直接在`0013E890`及`0013E894`两处下硬件写中断。从中断后返回地址可以知道，改变这两处值的是我们忽略的一个函数：

`012B2A83`  `|.  E8 88FEFFF>call CliSec\_1.012B2910`

这个调用的具体过程，是下一篇的内容，呵呵。这里，我们先放过它。接着向下走：

`012B2B2E`  `|.  8B50 04    mov edx,dword ptr ds:[eax+4]`

`012B2B31`  `|.  8955 AC    mov dword ptr ss:[ebp-54],edx`

`012B2B34`  `|.  8B45 AC    mov eax,dword ptr ss:[ebp-54]`

`012B2B37`  `|.  8B48 04    mov ecx,dword ptr ds:[eax+4]`

`012B2B3A`  `|.  8B55 A4    mov edx,dword ptr ss:[ebp-5C]`

`012B2B3D`  `|.  8B4491 FC  mov eax,dword ptr ds:[ecx+edx\*4>`

`012B2B41`  `|.  8945 A8    mov dword ptr ss:[ebp-58],eax`

`012B2B44`  `|.  8B4D F0    mov ecx,dword ptr ss:[ebp-10]`

`012B2B47`  `|.  51         push ecx                        ; /Arg1`

`012B2B48`  `|.  8B8D 7CFFF>mov ecx,dword ptr ss:[ebp-84]   ; |`

`012B2B4E`  `|.  E8 8DF4FFF>call CliSec\_1.012B1FE0          ; \CliSec\_1.012B1FE0`

如果跟进`012B2B4E`处的调用，你会发现这里取得了主程序的基址`0x11000000`。再向下走，就是分配空间，填充解密过的方法体：

`012B2B60`  `|.  8B55 A0    mov edx,dword ptr ss:[ebp-60]`

`012B2B63`  `|.  0355 A8    add edx,dword ptr ss:[ebp-58]`

`012B2B66`  `|.  8B85 7CFFF>mov eax,dword ptr ss:[ebp-84]`

`012B2B6C`  `|.  8950 38    mov dword ptr ds:[eax+38],edx`

`012B2B6F`  `|.  6A 1C      push 1C`

`012B2B71`  `|.  E8 F950000>call CliSec\_1.012B7C6F          ;  new`

`012B2B76`  `|.  83C4 04    add esp,4`

`012B2B79`  `|.  8945 90    mov dword ptr ss:[ebp-70],eax`

`012B2B7C`  `|.  837D 90 00 cmp dword ptr ss:[ebp-70],0`

`012B2B80`  `|.  74 1A      je short CliSec\_1.012B2B9C`

`012B2B82`  `|.  8B8D 7CFFF>mov ecx,dword ptr ss:[ebp-84]`

`012B2B88`  `|.  8B51 38    mov edx,dword ptr ds:[ecx+38]`

`012B2B8B`  `|.  52         push edx`

`012B2B8C`  `|.  8B4D 90    mov ecx,dword ptr ss:[ebp-70]`

`012B2B8F`  `|.  E8 2CF4FFF>call CliSec\_1.012B1FC0`

`012B2B94`  `|.  8985 78FFF>mov dword ptr ss:[ebp-88],eax`

因此，`012B2B8F`也是一处关键代码，可以跟进去看是怎么解密代码的。（真正有没有解密呢，跟进去就知道了![](https://bbs.kanxue.com/view/img/face/12.gif) ）不过在跟进去之前，先看一下椎栈。

![](http://bbs.pediy.com/upload/2006/4/image/44.jpg)

注意栈顶的第一个值`110A477C`，细心的话会发现这个数值指向了哪里：

![](http://bbs.pediy.com/upload/2006/4/image/55.jpg)

Yes，它指向了`.text`节。而所有的IL方法都是存在`.text`节中的。不妨看一下该处的数据：

![](http://bbs.pediy.com/upload/2006/4/image/66.jpg)

经常调试`.net`程序的可以一眼看出，图中高亮的数据，不正是一个方法体吗。接下来不用说了，改变参数后，调用真正的JIT的`compileMethod`：

`012B2CD6`  `|.  8B55 10    mov edx,dword ptr ss:[ebp+10]`

`012B2CD9`  `|.  C742 14 10>mov dword ptr ds:[edx+14],10`

`012B2CE0`  `|>  8B45 1C    mov eax,dword ptr ss:[ebp+1C]`

`012B2CE3`  `|.  50         push eax                        ; /Arg6`

`012B2CE4`  `|.  8B4D 18    mov ecx,dword ptr ss:[ebp+18]   ; |`

`012B2CE7`  `|.  51         push ecx                        ; |Arg5`

`012B2CE8`  `|.  8B55 14    mov edx,dword ptr ss:[ebp+14]   ; |`

`012B2CEB`  `|.  52         push edx                        ; |Arg4`

`012B2CEC`  `|.  8B45 10    mov eax,dword ptr ss:[ebp+10]   ; |`

`012B2CEF`  `|.  50         push eax                        ; |Arg3`

`012B2CF0`  `|.  8B4D 0C    mov ecx,dword ptr ss:[ebp+C]    ; |`

`012B2CF3`  `|.  51         push ecx                        ; |Arg2`

`012B2CF4`  `|.  8B55 08    mov edx,dword ptr ss:[ebp+8]    ; |`

`012B2CF7`  `|.  52         push edx                        ; |Arg1`

`012B2CF8`  `|.  FF15 E4452>call dword ptr ds:[12D45E4]     ; \mscorjit.7906E7F4`

中断在最后`012B2CF8`处，看一下传给真正的JIT的参数：

![](http://bbs.pediy.com/upload/2006/4/image/77.jpg)

要注意`Arg3`，因为它指向了修改后的IL体。跟进去看一下：

```
0013EA6C  B8 65 A5 00 14 2C A5 00 88 47 0A 11 0E 00 00 00
```

IL方法体为`110A4788`，代码大小`0x0E`，这正是刚才我们说的`110A477C`处的方法，指向啊原文件的`.text`节。这里，又有一个想法：这段代码倒底有没有被解密呢？

我们用`UltraEdit`直接打开原文件，搜索字符串`133001000E00000027000011731D0100060A066F0E01000A`，可以找到两处。根据`110A477C`对应的文件偏移，我们可以确定如下：
