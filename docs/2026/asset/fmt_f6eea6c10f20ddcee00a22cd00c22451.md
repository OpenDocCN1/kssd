
# ACProtect Professional 1.3C 主程序脱壳

脱壳过程感觉与Unpacking Saga的那个UnpackMe没有太大的不同。最明显的一点是,其中的异常多了很多。大部分是固定模式的`int 3`解码。因为一开始打算全程跟,所以一边跟一边修改去除junk code的IDC script,写OllyScript脚本,并用WinHex把解出的代码贴到`ACProtect_Fixed.exe`,在IDA中用`Load File|Reload the input file`加载后对照。

![](https://www.kanxue.com/paper-1.files/image002.jpg)

## 1. 关于 `int 3` 解码

壳中有大量的`int 3`解码,一边执行一边解。具有相同的调用模式:

![](https://www.kanxue.com/paper-1.files/image004.jpg)

在`int 3`的SHE中(下面的`div 0`异常也是同一个SEH),当exception code为`0x80000003`时,在`Dr0-Dr3`中设置了新的值,即设置了4个硬件执行断点,就在紧临`int 3`下面的位置。这样执行到这4句,会触发4次异常。如果在这里F7过,SHE不会执行。

![](https://www.kanxue.com/paper-1.files/image006.jpg)

当由上面的4个断点引发异常时:

![](https://www.kanxue.com/paper-1.files/image008.jpg)

每次异常,会设置某个寄存器。4次异常处理必须执行,否则寄存器中没有正确值,下面的解码会失败。这样又顺便清除cracker的断点,是个不错的主意J。

![](https://www.kanxue.com/paper-1.files/image010.jpg)

结束异常处理后,开始解码。格式是一样的,用的register不同。这里可以得到起始地址(<-edi)。

![](https://www.kanxue.com/paper-1.files/image012.jpg)

在`jnz`下面的地址F4,然后查看前面用于寻址的寄存器的值,减1就是解码结束地址。

![](https://www.kanxue.com/paper-1.files/image014.jpg)

解完后跟着一个`div 0`异常。只有anti-debug作用,直接跳过即可。

## 2. 避开IAT加密

在第16个`int 3`的解码过程中,隐藏了IAT加密。其实在OllyDbg的状态栏可以看到这附近有不少dll被加载了。

![](https://www.kanxue.com/paper-1.files/image015.jpg)

有4处检查,决定是否特殊处理。在用`GetProcAddress`得到API的地址后,开始检查。是否为特殊的API?

![](https://www.kanxue.com/paper-1.files/image016.jpg)

是否为`MessageBoxA`或`RegisterHotKey`(这个比较少见J)?

是则替换api地址。

![](https://www.kanxue.com/paper-1.files/image018.jpg)

下面检查dll的映射地址是否为`kernel32.dll`或`user32.dll`。

![](https://www.kanxue.com/paper-1.files/image019.jpg)

这里通不过检查,就不做处理,直接把API的地址保存到IAT了。否则:

![](https://www.kanxue.com/paper-1.files/image021.jpg)

写IAT:

![](https://www.kanxue.com/paper-1.files/image023.jpg)

用脚本执行到这一段,patch掉4处的跳转,在处理完全部dll的地方下断,用ImportRec得到完整的IAT,全部有效。

![](https://www.kanxue.com/paper-1.files/image025.jpg)

![](https://www.kanxue.com/paper-1.files/image027.jpg)

`size = 6E3828 C 6E3140 = 6E8`

![](https://www.kanxue.com/paper-1.files/image029.jpg)

注意此时OEP不对。但IAT已经拿到了。先把tree保存起来。

## 3. 寻找OEP

得到IAT后,试试忽略所有异常,到这里已经可以在OllyDbg下执行了。试了一下,关闭其它异常,只留下`int 3`,还要shift-F9 60次才能运行L。这样一步步跟不把人累死。算了,先换省力的办法吧。

干脆,在执行目前的script停下后,忽略所有异常。对Code section下内存访问断点。

![](https://www.kanxue.com/paper-1.files/image031.jpg)

### 1) `406EDC`,返回到壳

![](https://www.kanxue.com/paper-1.files/image033.jpg)

返回到壳代码,所以这是stolen code发出的`call`。

![](https://www.kanxue.com/paper-1.files/image035.jpg)

### 2) `46261C`,返回到壳

![](https://www.kanxue.com/paper-1.files/image037.jpg)

### 3) 读操作断下

![](https://www.kanxue.com/paper-1.files/image039.jpg)

### 4) `462634`,返回到壳

![](https://www.kanxue.com/paper-1.files/image041.jpg)

这里的代码也会引发异常。先remove内存访问断点,到`73CEDD`设断。断下后恢复内存访问断点。

### 5) 读操作断下

![](https://www.kanxue.com/paper-1.files/image043.jpg)

### 6) 又在`462634`,是循环吗?按第4步的办法处理

![](https://www.kanxue.com/paper-1.files/image045.jpg)

![](https://www.kanxue.com/paper-1.files/image047.jpg)

### 7) 到站了J

![](https://www.kanxue.com/paper-1.files/image049.jpg)

这里是false OEP。下面的空间有限。可以看到,stolen codes中含有的`call`:

`406EDC`,`46261C`,`462634`(执行2次)。

在dump窗口中看:

![](https://www.kanxue.com/paper-1.files/image051.jpg)

OEP应该在`4D9DE4`。

## 4. dump

根据脱US UnpackMe的经验,不能在false OEP处dump,此时BSS section中许多数据已经初始化了。最好在第一句(`push ebp`)就dump。可是那个`push ebp`离false OEP很远L。

Packer EP的初始化环境:

![](https://www.kanxue.com/paper-2.files/image002.jpg)

![](https://www.kanxue.com/paper-2.files/image004.jpg)

![](https://www.kanxue.com/paper-2.files/image006.jpg)

先看看发出第1个`call`的壳代码:

![](https://www.kanxue.com/paper-2.files/image008.jpg)

可以看到(跟一下可以证实),在`call`入原程序空间前,最后的异常是`div 0`。用现在的OllyDbg脚本(跟到`737B63`),停下后修改异常拦截选项:

![](https://www.kanxue.com/paper-2.files/image010.jpg)

试试能否拦截异常找到合适的dump位置。当前OllyScript脚本停下时的环境:

![](https://www.kanxue.com/paper-2.files/image012.jpg)

栈中为`pushad`的结果。

第7次`div 0`异常时:

![](https://www.kanxue.com/paper-2.files/image014.jpg)

![](https://www.kanxue.com/paper-2.files/image016.jpg)

注意`ebp`的值已经入栈了。`pushad`的结果,对应寄存器值为:

`esp = 12FFC0`

(原来为`12FFC4`)

`ebp = 12FFC0 = esp` (原来为`12FFF0`)

所以,在这里已经执行过了:

```
Push ebp
Mov ebp,esp
```

看看BSS section:

![](https://www.kanxue.com/paper-2.files/image018.jpg)

![](https://www.kanxue.com/paper-2.files/image020.jpg)

全为0,就在这里dump。如果需要仔细跟stolen code,也可以从这里入手(可以从脚本停下的第6次`div 0`异常处理后开始)。

先用`4D9DE4`为OEP。

![](https://www.kanxue.com/paper-2.files/image022.jpg)

用LoadPE查看节表:

![](https://www.kanxue.com/paper-2.files/image024.jpg)

先把`CODE`的`Vsize`和`Rsize`加大,以避免修复stolen codes时空间不够。

![](https://www.kanxue.com/paper-2.files/image026.jpg)

重新跟到false OEP(`4D9E37`)。重建输入表。

![](https://www.kanxue.com/paper-2.files/image028.jpg)

用了`Add new section`,会不会有问题(好象有篇脱文里提到过这个)? 先这样处理。

用IDA编译`dumped_.exe`,结果不错。

现在剩下stolen code和replaced code。

## 5. 修复stolen code

既然不打算仔细跟,就只有猜了。对执行第1个`call`以前的stolen code进行猜测,应该是安全的。后面的4次`call`需要仔细看看。

### 1) `call 406EDC`

在IDA中看`dumped_.exe`的结果:

![](https://www.kanxue.com/paper-2.files/image030.jpg)

先执行脚本,停下后只勾选`div 0`异常。直到`73A9AF`。经过一番遮遮掩掩的代码:

![](https://www.kanxue.com/paper-2.files/image032.jpg)

在`call`前的寄存器:

![](https://www.kanxue.com/paper-2.files/image034.jpg)

堆栈:

![](https://www.kanxue.com/paper-2.files/image036.jpg)

`call`完后,下面的`pushad`为分界线,标识stolen code的结束。所以到这里的stolen code为(对于不确定的,可以在Packer EP修改寄存器值,到这里核对):

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
```

`call`完后,`pushad`前的环境:

![](https://www.kanxue.com/paper-2.files/image038.jpg)

![](https://www.kanxue.com/paper-2.files/image040.jpg)

### 2) `call 46261C`

![](https://www.kanxue.com/paper-2.files/image042.jpg)

![](https://www.kanxue.com/paper-2.files/image044.jpg)

到这里的stolen code:

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
push edi
mov ecx,dword ptr ds:[503938]
```

从这里开始,不能图省事了。要追出全部的stolen code,必须跟(必须确保上一次`pushad`和下一次`popad`时的环境一致,才不会丢代码)。先试直接拦截`div 0`异常,注意后面是否有`popad`。如果两次的环境不一致,则必须单步跟L。

另外,追stolen code不是一次完成的,所以有些图中寄存器和堆栈中的数据对不上,不必管它。

### 3) `0073BC47`

![](https://www.kanxue.com/paper-2.files/image046.jpg)

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
call 46261C         //这里去掉前面的push edi
mov ecx,dword ptr ds:[503938]
mov eax,dword ptr ds:[503B94]
mov eax,dword ptr ds:[eax]
push eax
push ecx
mov ecx,47FCD8    //多余,前面push ecx,后面还会pop的吧?
mov eax,47FCD8
```

### 4) `0073C558`

![](https://www.kanxue.com/paper-2.files/image048.jpg)

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
push edx
mov edx, 462634      //注意这是下一次call的地址
```

### 5) 第1次`call 462634`

![](https://www.kanxue.com/paper-2.files/image050.jpg)

这个函数有2个参数L。

![](https://www.kanxue.com/paper-2.files/image052.jpg)

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
push eax
push esi
mov eax, 50385C
```

### 6) 第2次`call 462634`

![](https://www.kanxue.com/paper-2.files/image054.jpg)

