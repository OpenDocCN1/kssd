
```
Push ebp
Mov ebp,esp
Push edx
Push edi
Push ebx
Push eax
Mov eax,4D9BCC
call 406EDC
mov eax,dword ptr ds:[503B94]
mov eax,dword ptr ds:[eax]
call 46261C
mov ecx,dword ptr ds:[503938]
mov eax,dword ptr ds:[503B94]
mov eax,dword ptr ds:[eax]
mov edx,dword ptr ds:[47FCD8]
call 462634
mov ecx,dword ptr ds:[50385C]
mov eax,dword ptr ds:[503B94]
mov eax,dword ptr ds:[eax]
mov edx,dword ptr ds:[47F948]
call 462634
mov eax,dword ptr ds:[503B94]
mov eax,dword ptr ds:[eax]
```

这是最后一次后面存在`popad`的`div 0`异常。看看是如何回到原程序的。

在`73D872`忽略所有异常，对code section下内存访问断点。中间在kernel32中有一次内存访问异常。

![](https://www.kanxue.com/paper-2.files/image056.jpg)

最后一次`div ebx`在`73DBE1`。单步跟到这里：

![](https://www.kanxue.com/paper-2.files/image058.jpg)

`73DE38`破坏前面代码。

![](https://www.kanxue.com/paper-2.files/image060.jpg)

至此修复所有stolen codes，将前面的6部分和fasle OEP的2个`call`合在一起。

```
Push ebp
Mov ebp,esp
Push edx
Push edi
Push ebx
Push eax
Mov eax,4D9BCC
call 406EDC
mov eax,dword ptr ds:[503B94]
mov eax,dword ptr ds:[eax]
call 46261C
mov ecx,dword ptr ds:[503938]
mov eax,dword ptr ds:[503B94]
mov eax,dword ptr ds:[eax]
mov edx,dword ptr ds:[47FCD8]
call 462634
mov ecx,dword ptr ds:[50385C]
mov eax,dword ptr ds:[503B94]
mov eax,dword ptr ds:[eax]
mov edx,dword ptr ds:[47F948]
call 462634
mov eax,dword ptr ds:[503B94]
mov eax,dword ptr ds:[eax]
call 4626B4
call 4049D8
lea eax, [eax+0]
```

## 6. 修复IAT

运行程序，crashedL。直接用修复完stolen code的`dumped_.exe`看看。从EP的第1个`call`进去就有问题。

![](https://www.kanxue.com/paper-3.files/image001.jpg)

![](https://www.kanxue.com/paper-3.files/image003.jpg)

![](https://www.kanxue.com/paper-3.files/image004.jpg)

在`OllyDbg`中可以看到：

![](https://www.kanxue.com/paper-3.files/image006.jpg)

有部分IAT在壳中。这部分代码前面是跟到了的（在第6次`int 3`以后），原来认为这只是loader自己需要的API。实际上正常的程序代码也使用了这个IAT。

如果用`ImportRec`把这部分也取出来，属于不同dll的`api`混排了，`ImportRec`不能处理。

这些地址是在`736643`处理的。

![](https://www.kanxue.com/paper-3.files/image008.jpg)

而且有一个不能识别（这个实际上是`Hooked MessageBoxA`）。

![](https://www.kanxue.com/paper-3.files/image010.jpg)

估计这是加壳时做的手脚。当发现主程序使用了与壳代码同样的API时，修改了对应的调用代码，使其调用到壳中的IAT。这样可以加强与壳的联系。反过来看，这里大部分API（除了只有壳使用的），在前面避开IAT加密处理后，都已经import了。

在`ACProtect_Fixed`中已经有解好的`api`名。

![](https://www.kanxue.com/paper-3.files/image011.jpg)

紧临的函数指针：

![](https://www.kanxue.com/paper-3.files/image012.jpg)

注意函数名及指针与`7339A9`附近的代码并不完全对应。有的函数名或指针在别处。

可以在程序入口自己调用`LoadLibrary`，`GetProcAddress`进行处理，填入相应的地址。或者把`Loader`的代码搬过来。也可以修改主程序的代码，使其调用前面得到的干净的IAT而不是壳中的IAT（这样要处理的地方太多，很麻烦）。

注意`ACProtected_Fixed`中，import了几个基本的API：

![](https://www.kanxue.com/paper-3.files/image014.jpg)

在加载API时壳代码会使用这些函数。这几个地址直接填入`007300D8`开始的IAT中了。修改`dumped_.exe`的EP，从壳代码开始（可以看到dump出来的代码，这部分已经解开了），

![](https://www.kanxue.com/paper-3.files/image016.jpg)

![](https://www.kanxue.com/paper-3.files/image018.jpg)

注意`Loader`代码中用的几个API（`GetModuleHandleA`，`GetProcAddress`）改为使用避开IAT加密后得到的主程序的IAT。

![](https://www.kanxue.com/paper-3.files/image019.jpg)

原来在壳代码import的4个API，从主程序的IAT直接取。结束处理后跳回前面的OEP `409DE4`。用`pushad`，`pushad`保存寄存器值。

![](https://www.kanxue.com/paper-3.files/image021.jpg)

注意最后一项`7301CC`实际是`MessageBoxA`，被`ACProtect`用做`SDK`的接口。不要填入`MessageBox`的真正地址。

下面的`73013C`在跟原程序的过程中始终为0，可能是注册版才有？

![](https://www.kanxue.com/paper-3.files/image023.jpg)

处理完后，位于壳空间的IAT已经赋上了正确值。

## 7. 修复Replaced code (1)

用前面的`OllyScript`脚本，停下后对Code section设内存访问断点。忽略所有异常。断下后，在抽取代码的共同入口`722416`设断，运行。第1次调用在`004047E5`。

![](https://www.kanxue.com/paper-3.files/image025.jpg)

里面还有`SMC`，解出后贴到`ACProtect_Fixed`中对照跟踪。这部分的处理与低版本基本相同。

1) 查返回地址表

![](https://www.kanxue.com/paper-3.files/image026.jpg)

`7225CC`后`ebx`指向返回地址表，里面是`RVA`：

![](https://www.kanxue.com/paper-3.files/image028.jpg)

找到匹配地址后，`ecx`等于表项的`offset`，第1项为`0x2A`。

2) 查`opcode`表

用1中的到的`offset`查`opcode`数据表（`007262ED`）：

![](https://www.kanxue.com/paper-3.files/image030.jpg)

密文`opcode`表：

![](https://www.kanxue.com/paper-3.files/image032.jpg)

`counter`表（记录每个地址的调用次数，超出`0x20`次将使用新的地址解码）：

![](https://www.kanxue.com/paper-3.files/image034.jpg)

第1次解出的变形码：

![](https://www.kanxue.com/paper-3.files/image036.jpg)

3) 调整`stack`以便正确调用变形码及返回

![](https://www.kanxue.com/paper-3.files/image038.jpg)

4) 破坏解出的代码，`ret`到变形码

![](https://www.kanxue.com/paper-3.files/image040.jpg)

5) 执行变形码，返回到原程序

![](https://www.kanxue.com/paper-3.files/image042.jpg)

6) `Patch`

先把解出的代码binary copy到`dumped_.exe`（直接copy `722416` - `7226B0`的代码即可）。

关闭`722579`的解码（解出`7225C1` - `7226B0`）：

![](https://www.kanxue.com/paper-3.files/image044.jpg)

关闭`7226A8`处对前面代码的破坏：

![](https://www.kanxue.com/paper-3.files/image045.jpg)

`patch` `72260C`：`jnz`->`jmp`，无论执行多少次都使用同一解码地址

![](https://www.kanxue.com/paper-3.files/image046.jpg)

copy正确的`ImageBase`值，这里为0，改`72FC1F`为`400000`

![](https://www.kanxue.com/paper-3.files/image048.jpg)

![](https://www.kanxue.com/paper-3.files/image050.jpg)

copy正确的10 bytes `key`（第1 byte为0）

![](https://www.kanxue.com/paper-3.files/image052.jpg)

执行，仍然crashedL。

## 8. 修复Replaced code (2)

还有`replaced code`，在这里出异常：

![](https://www.kanxue.com/paper-3.files/image054.jpg)

->

![](https://www.kanxue.com/paper-3.files/image056.jpg)

将解出的代码贴到`ACProtect_Fixed.exe`。可以看到，这些实际上是变形的`call`代码。第1次执行到这里，在`buffer`中解出的变形码为：

![](https://www.kanxue.com/paper-3.files/image058.jpg)

![](https://www.kanxue.com/paper-3.files/image060.jpg)

`XOR`的结果：

![](https://www.kanxue.com/paper-3.files/image062.jpg)

目前的`dumped_.exe`，地址`72ED83`的值为0（这些值是`loader`写入的）。

`406DF8`的变形码原来是`call GetKeyboardType`。原程序的call API全部被抽掉了。壳代码的动作与前面相似，用返回地址查表，获取相应的指针，生成`jmp dword ptr ds:[xxxxxxxx]`指令，该地址则指向类似`72124C`的变形码，调用正确的API。

变形`call` API的返回`RVA`地址表：

开始：

![](https://www.kanxue.com/paper-3.files/image064.jpg)

结束：

![](https://www.kanxue.com/paper-3.files/image066.jpg)

寻址变形码数组下标表（每项1字节），用于查变形码指针数组：

![](https://www.kanxue.com/paper-3.files/image068.jpg)

变形码地址表（指向变形码的指针）：

![](https://www.kanxue.com/paper-3.files/image070.jpg)

开始打算写个`inline-patch`：

用同样的查表动作，把对应的变形码copy出来，得到对应的API地址，与跳过加密而得到的干净IAT对比。查到匹配值后，修改对应的`opcode`，使其直接call到IAT中的地址。

用`OllyScript`脚本跳过IAT加密，得不到变形码（此时从变形码地址表中得到的就是API的真正地址，有46项指针无效，为`0xCCCCCCCC`）。

另一个问题却是难以解决的，`replaced code`只有5字节：

![](https://www.kanxue.com/paper-3.files/image072.jpg)

这里的`call`是`0xE8`，调用壳中的绝对地址。`InlinePatch`写到一定程度才发现，如果要修复代码，使其调用到IAT，需要相对地址调用6 `bytesL`。真是个低级错误。

现在`patch`的结果：

![](https://www.kanxue.com/paper-3.files/image074.jpg)

真正需要的是：

![](https://www.kanxue.com/paper-3.files/image075.jpg)

这样只有保留变形码。把壳中对应的代码copy过来，OEP前生成正确的变形码。而且脱壳后的程序不能直接看到API名字，很不舒服。

只好把壳的相应代码搬过来。再次修改`dumped_.exe`入口处代码，在把`loader`空间中的IAT填好后，跳到处理变形码的位置：

![](https://www.kanxue.com/paper-3.files/image077.jpg)

`loader`在处理IAT时需要调用几个API，及判断dll的映射地址、API地址等，先保存需要的数据（我们有干净的`IATJ`）：

![](https://www.kanxue.com/paper-3.files/image078.jpg)

由于在前面避开了IAT加密，生成变形码需要的数据已经被正确的API地址覆盖了。用`LoadPE`把`ACProtect`的`idata section`存到文件，然后加到`dumped_.exe`。

![](https://www.kanxue.com/paper-3.files/image080.jpg)

把这个`section`的密文数据copy到`dumped_.exe`的`idata section`，覆盖掉干净的IAT，我们已经不需要它了。现在只要伪造好现场J。

![](https://www.kanxue.com/paper-3.files/image082.jpg)

往下执行`loader`的IAT处理代码，做几处小小的修改，使其使用刚才保存的API地址等数据。

![](https://www.kanxue.com/paper-3.files/image083.jpg)

IAT及变形码处理结束后回到OEP。

![](https://www.kanxue.com/paper-3.files/image085.jpg)

执行。又挂了L。这次是内存访问异常。跟一跟可以知道，是在`Hooked MessageBoxA`中。这里面的代码还没有仔细看，有几个`switch-case`分枝。第1次`eax`为5。

![](https://www.kanxue.com/paper-3.files/image086.jpg)

进去后有几个查表动作：

![](https://www.kanxue.com/paper-3.files/image088.jpg)

用调用`Hooked MessageBoxA`的返回地址查表。这张表在`721F25`处，`dumped_.exe`中有，共21项。

注意查表时不是找相等值，而是找大于返回地址值且最接近的值。

![](https://www.kanxue.com/paper-3.files/image089.jpg)

继续->

![](https://www.kanxue.com/paper-3.files/image090.jpg)

这里出现了另外2张表。`7220B5`的表中数据为`size`。`Dumped_.exe`中有：

![](https://www.kanxue.com/paper-3.files/image092.jpg)

问题出在第3张表：

![](https://www.kanxue.com/paper-3.files/image094.jpg)

`dump`出的数据为0。这段代码要把主程序中的一段数据copy到这张表中数据所指的地址。在`loader`中执行时，这里填入了指向动态分配内存的指针。

![](https://www.kanxue.com/paper-3.files/image096.jpg)

显然不能直接复制这些值。有个简单的办法可以骗过`loader`。从那张`size`表中可以看到，最大的数据`FD5D`。用`LoadPE`再次增加1个`section`，`size`为`FFFF`即可。

![](https://www.kanxue.com/paper-3.files/image098.jpg)

修改`dumped_.exe`，设置21项数据，使其全部指向该地址。
