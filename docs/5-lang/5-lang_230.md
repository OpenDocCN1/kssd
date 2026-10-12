
# 用 Python 反编译 Python 软件

> 原文：[`www.pediy.com/kssd/pediy11/111428.html`](https://www.pediy.com/kssd/pediy11/111428.html)

【文章标题】: 用 Python 反编译 Python 软件  
【文章作者】: Ptero  
【软件名称】: `****`  
【下载地址】: `****`  
【加壳方式】: `UPX`  
【保护方式】: 序列号，重启验证  
【使用工具】: `7-zip`, `LordPE`, `Python`, `WinHex`  
【操作平台】: `Windows`, `Linux`  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
--------------------------------------------------------------------------------
【版权声明】: 本文原创于看雪软件安全论坛, 转载请注明作者并保持文章的完整, 谢谢!

论坛讨论 `Python` 的文章似乎比较少。此文权当抛砖引玉，不当之处请多指教。

关于 `Py` 的反编译，网上流传的说法是，2.4 版本以后的比较困难。因为针对低版本，有一些开源代码可以实现反编译。到 2.4 以后，原作者要么不更新了，要么开始收费了。只有少数牛人能修改旧版的代码继续反编译下去。

本文提供了一种方法，使得在没有反编译器的情况下，也能分析用 `Python` 写的软件。

下面开始。

试炼软件是用 `upx` 加壳的，`upx -d` 简单脱掉。

再用 `peid` 查看, 发现如下信息：

`Microsoft Visual C++ 7.0 Method2 [ZIP SFX]`。

很少见。再用 `OD` 加载，查看字符串，发现是 `py2exe` 生成的。

这种文件，可以用 `7zip` 当作压缩包打开，里面可以看到一堆编译好的 `pyc` 和 `pyo` 文件。从名称来看，那些应该是 `Python` 自带的库文件，与程序无关。这里从库文件的名称可以看出是 `Python 2.4` 版本。

程序自带了一个 `pyo` 文件，200 多 `K`。网上找到能免费反编译 `Python 2.4` 以后版本的，只有 `decompyler 2.3` 的一个修改版(2.3 开源，后续版本收费了), 还有就是 `UnPyc`。

`decompyler` 的修改版，我没有找到。`UnPyc`，貌似只支持 `Linux` 的。在学校的 `Linux` 服务器上安装上，反编译却得到一堆错误。反汇编(生成 `bytecode` 的助记符，相当于汇编代码)倒是可以。

在反汇编当中查找关键字符串，没找到。我把所有的函数名称看了一遍，也没有可疑的。看来那个 200 多 `K` 的文件，不是关键代码所在。还得另谋出路。

关键代码不在附带的库里面，那还能在哪里呢？只能在程序自身了！一定是 `py2exe` 把那些代码编到 `exe` 里面了。

有 2 种可能：1、编成 `native code`。2、编成 `Python bytecode`，通过 `Python` 虚拟机执行。在 `OD` 中跟踪一下，发现执行到关键代码的时候，已经处在虚拟机的代码之中了。所以排除 1 的可能性。

下面，要找出关键代码的藏身之处。代码会藏在哪呢？`.text` 段？不大可能，因为那里都是 `sfx` 的代码。`.data` 段？也不可能，因为那里的数据只会被 `sfx` 用到。在 `zip` 压缩包里面？没找到可疑文件。那么，只剩下 `.rsrc` 了！

用 `LordPE` 查看资源段(这里不用 `Reshacker`，因为它 `dump` 下来的不正确)，找到 “`Python24.dll`”， “`PZIB.PYD`”，还有就是 “`PYTHONSCRIPT`”。光是看名字就很可疑了。把它 `dump` 下来，在 `WinHex` 当中查找关键字符串，果然找到了。

下面就是要如何反编译这个 `script` 了。

网上没有找到相关资料。于是下载了一份 `py2exe` 源码，找到这里：

```
# We create a list of code objects, and write it as a marshaled
# stream.  The framework code then just exec's these in order.
# First is our common boot script.
boot = self.get\_boot\_script("common")
boot\_code = compile(file(boot, "U").read(),
                    os.path.abspath(boot), "exec")
code\_objects = \[boot\_code\]
if self.bundle\_files < 3:
    code\_objects.append(
        compile("import zipextimporter; zipextimporter.install()",
                "<install zipextimporter>", "exec"))
for var\_name, var\_val in vars.items():
    code\_objects.append(
        compile("%s=%r\\n" % (var\_name, var\_val), var\_name, "exec")
    )
if self.custom\_boot\_script:
    code\_object = compile(file(self.custom\_boot\_script, "U").read() + "\\n",
                        os.path.abspath(self.custom\_boot\_script), "exec")
    code\_objects.append(code\_object)
if script:
    code\_object = compile(open(script, "U").read() + "\\n",
                        os.path.basename(script), "exec")
    code\_objects.append(code\_object)
code\_bytes = marshal.dumps(code\_objects)
if self.distribution.zipfile is None:
    relative\_arcname = ""
si = struct.pack("iiii",
                0x78563412, # a magic value,
                self.optimize,
                self.unbuffered,
                len(code\_bytes),
                ) + relative\_arcname + "\\000"
script\_bytes = si + code\_bytes + '\\000\\000'
self.announce("add script resource, %d bytes" % len(script\_bytes))
if not self.dry\_run:
    add\_resource(ensure\_unicode(exe\_path), script\_bytes, u"PYTHONSCRIPT", 1, True)
```

可以看出，是由几个 `py` 文件编译后，添加到一个 `list` 里面，然后直接 `dump` 下来的，当然前后还加了一堆东西。

用 `WinHex` 修改 `dump` 下来的文件，把添加的东西都去掉，这样就只剩下了 `code\_bytes`。

现在，本文的主人公要华丽地出场了！有请 `Python`！(掌声)

在 `Python` 中输入：

```
\>>>import marshal
\>>>mylist=marshal.load(open("dumpfile", "r"))
```

目的是为了把 `dump` 下来的文件加载到内存当中，成为 `Python` 的一个对象。

注：加载 `dump` 下来的对象，`Python` 版本一定要和 `dump` 时候的版本兼容才行。这个例子中，`dump` 时用的 2.4 ，我用 2.5 `load`，完全可以。用 3.1 `load`，就出错了。

现在可以看看我们的这个对象了：

```
\>>>mylist
\[<code object ? at 0xb75df650, file "D:\\python24\\lib\\site-packages\\py2exe\\boot\_common.py", line 44>, <code object ? at 0xb75df698, file "<install zipextimporter>", line 1>, <code object ? at 0xb75f4ad0, file "\*\*\*\*.py", line 2>\]
```

包含了 3 个 `code object` 对象。第一个是 `py2exe` 初始化用的，第二个是解压 `zip` 用的，第三个就是我们的关键脚本了。

这里简单介绍一下 `py`, `pyc`, `pyo`, `bytecode`, `code object` 之间的关系。`py` 是 `Python` 的源代码文件，纯文本文件。用 `Python` 可以编译成二进制伪代码，也就是 `bytecode`。`code object` 实际上就是这些伪代码。把 `code object` 前面加一个 `header`，写成文件，就是 `pyc` 了，也就是编译过的 `py` 文件。如果在编译的时候加上优化选项，则会生成 `pyo` 文件，也就是优化过的 `py` 文件，本质上和 `pyc` 是一样的。

如果说 `py` 相当于 `java` 文件，那么 `pyc`, `pyo`, `bytecode`, `code object` 就相当于 `class` 文件了。

下面言归正传。

`Python` 有一个很好很强大的库：`dis`，里面有一个很好很强大的同名函数：`dis()`。这个函数就是实现反汇编功能了。它能把 `code object` 生成可读的代码(类似于汇编)。

对这个函数加以简单扩展，可以让其变得更好更强大。(参见 [`blog.csdn.net/balabalamerobert...2/1662025.aspx`](http://blog.csdn.net/balabalamerobert/archive/2007/06/22/1662025.aspx))

```
import dis as pydis
import types

code = None
def read(filename):
    f = open(filename)
    content = f.read()
    global code
    code = compile(content, filename, 'exec')
    f.close()

def find\_code(code, name):
    for item in code.co\_consts:
        if isinstance(item, types.CodeType):
            if item.co\_name == name:
                return item
    return None
    
def dis(code\_name=None):
    if code\_name is None:
        co = code
        pydis.dis(co)
        return co
    names = code\_name.split(".")
    co = code
    for name in names:
        co = find\_code(co, name)
        if not co:
            print '%s is not a valid name' % code\_name
    if co:
        print ("   byte code for %s   " % code\_name).center(60, '\*')
        pydis.dis(co)
```

下面就可以慢慢找出关键模块了。

首先看一下模块包含哪些常量：

```
\>>>mylist\[2\].co\_consts:
(1, None, ('gdi',), ('EnumProcesses',), ('Button',), ('Editbox',), ('Textin',), ('LOWORD', 'HIWORD', 'RGB', 'RECT'), ('Msg',), ('Listview',), ('Listbox',), ('Combobox',), ('Checkbox',), ('Radiobox',), ('Treeview',), ('StaticText',), ('Groupbox',), ('ContextMenu',), ('Menu',), ('SystemCursor',), ('GetVirtualScreenSize',), ('static',), ('Queue',), ('latin\_1', 'gbk', 'utf\_8', 'ascii', 'gb2312', 'gb18030'), ('StringIO',), ('dbapi2',), ('WinExec',), ('CF\_TEXT', 'GHND'), 'Display\_REG\_Dialog', <code object Display\_REG\_Dialog at 0xb7cc9f50, file "\*\*\*\*.py", line 50>, 'Display\_INPUT\_Dialog', <code object Display\_INPUT\_Dialog at 0xb7cd7188, file "\*\*\*\*.py", line 96>, 'Display\_INPUT\_TIME\_Dialog', <code object Display\_INPUT\_TIME\_Dialog at 0xb7cd7380, file "\*\*\*\*.py", line 134>, 'Setup\_Find\_Dialog', <code object Setup\_Find\_Dialog at 0xb7cd7530, file "\*\*\*\*.py", line 168>, 'Get\_yo2\_user\_Dialog', <code object Get\_yo2\_user\_Dialog at 0xb7cd7770, file "\*\*\*\*.py", line 209>, 'Edit\_User\_Dialog', <code object Edit\_User\_Dialog at 0xb7cd79b0, file "\*\*\*\*.py", line 279>, 'Add\_User\_Dialog', <code object Add\_User\_Dialog at 0xb7cd7d10, file "\*\*\*\*.py", line 332>, 'Add\_Cate\_Dialog', <code object Add\_Cate\_Dialog at 0xb7cd7f50, file "\*\*\*\*.py", line 546>, 'Get\_Pic\_Dialog', <code object Get\_Pic\_Dialog at 0xb7cdd260, file "\*\*\*\*.py", line 602>, 'Get\_MPic\_Dialog', <code object Get\_MPic\_Dialog at 0xb7cdd530, file "\*\*\*\*.py", line 934>, 'Select\_ExportType\_Dialog', <code object Select\_ExportType\_Dialog at 0xb7cdd9b0, file "\*\*\*\*.py", line 1276>, 'Setup\_PROXY\_Dialog', <code object Setup\_PROXY\_Dialog at 0xb7cddad0, file "\*\*\*\*.py", line 1412>, 'Update\_Dialog', <code object Update\_Dialog at 0xb7cddba8, file "\*\*\*\*.py", line 1564>, 'MyWindow', <code object MyWindow at 0xb7cef2f0, file "\*\*\*\*.py", line 1698>, '')
```



`Display_REG_Dialog` 这个比较可疑，但是跟进去发现只是保存了注册码就返回了。相关代码省略。

因为软件是在启动时检测注册码的，所以窗口的启动代码也比较可疑。

代码:

```
>>>import sdis
>>>sdis.code = mylist[2]
>>>sdis.dis("MyWindow.__init__")
*****************   byte code for MyWindow.__init__  *****************
(省略部分代码)
1732         439 LOAD_CONST                 5 (0)
             442 STORE_FAST                 8 (conf_user_tag)

1733         445 LOAD_GLOBAL               40 (os)
             448 LOAD_ATTR                 35 (path)
             451 LOAD_ATTR                 41 (isfile)
             454 LOAD_CONST                21 ('****.ini')
             457 CALL_FUNCTION              1
             460 JUMP_IF_FALSE             48 (to 511)
             463 POP_TOP                 

1734         464 SETUP_EXCEPT              28 (to 495)

1735         467 LOAD_GLOBAL               42 (dict4ini)
             470 LOAD_ATTR                 43 (DictIni)
             473 LOAD_CONST                21 ('****.ini')
             476 CALL_FUNCTION              1
             479 LOAD_FAST                  0 (self)
             482 STORE_ATTR                44 (conf_user)

1736         485 LOAD_CONST                22 (1)
             488 STORE_FAST                 8 (conf_user_tag)
             491 POP_BLOCK                 
             492 JUMP_ABSOLUTE            518

1737     >>  495 POP_TOP                 
             496 POP_TOP                 
             497 POP_TOP                 

1738         498 LOAD_CONST                 5 (0)
             501 STORE_FAST                 8 (conf_user_tag)
             504 JUMP_ABSOLUTE            518
             507 END_FINALLY             
             508 JUMP_FORWARD               7 (to 518)
         >>  511 POP_TOP                 

1740         512 LOAD_CONST                 5 (0)
             515 STORE_FAST                 8 (conf_user_tag)

1741     >>  518 LOAD_FAST                  8 (conf_user_tag)
             521 LOAD_CONST                 5 (0)
             524 COMPARE_OP                 2 (==)
             527 JUMP_IF_FALSE             50 (to 580)
             530 POP_TOP                 

1742         531 LOAD_GLOBAL               42 (dict4ini)
             534 LOAD_ATTR                 43 (DictIni)
             537 LOAD_CONST                21 ('****.ini')
             540 CALL_FUNCTION              1
             543 LOAD_FAST                  0 (self)
             546 STORE_ATTR                44 (conf_user)

1743         549 LOAD_CONST                23 ('')
             552 LOAD_FAST                  0 (self)
             555 LOAD_ATTR                 44 (conf_user)
             558 LOAD_ATTR                 45 (config)
             561 STORE_ATTR                46 (regnum)

1744         564 LOAD_FAST                  0 (self)
             567 LOAD_ATTR                 44 (conf_user)
             570 LOAD_ATTR                 47 (save)
             573 CALL_FUNCTION              0
             576 POP_TOP                 
             577 JUMP_FORWARD               1 (to 581)
         >>  580 POP_TOP                 

1745     >>  581 LOAD_FAST                  0 (self)
             584 LOAD_ATTR                 44 (conf_user)
             587 LOAD_ATTR                 48 (has_key)
             590 LOAD_CONST                24 ('config')
             593 CALL_FUNCTION              1
             596 JUMP_IF_TRUE              32 (to 631)
             599 POP_TOP                 

1746         600 LOAD_CONST                23 ('')
             603 LOAD_FAST                  0 (self)
             606 LOAD_ATTR                 44 (conf_user)
             609 LOAD_ATTR                 45 (config)
             612 STORE_ATTR                46 (regnum)

1747         615 LOAD_FAST                  0 (self)
             618 LOAD_ATTR                 44 (conf_user)
             621 LOAD_ATTR                 47 (save)
             624 CALL_FUNCTION              0
             627 POP_TOP                 
             628 JUMP_FORWARD               1 (to 632)
         >>  631 POP_TOP                 

1748     >>  632 LOAD_FAST                  0 (self)
             635 LOAD_ATTR                 44 (conf_user)
             638 LOAD_ATTR                 45 (config)
             641 LOAD_ATTR                 48 (has_key)
             644 LOAD_CONST                25 ('regnum')
             647 CALL_FUNCTION              1
             650 JUMP_IF_TRUE              32 (to 685)
             653 POP_TOP                 

1749         654 LOAD_CONST                23 ('')
             657 LOAD_FAST                  0 (self)
             660 LOAD_ATTR                 44 (conf_user)
             663 LOAD_ATTR                 45 (config)
             666 STORE_ATTR                46 (regnum)

1750         669 LOAD_FAST                  0 (self)
             672 LOAD_ATTR                 44 (conf_user)
             675 LOAD_ATTR                 47 (save)
             678 CALL_FUNCTION              0
             681 POP_TOP                 
             682 JUMP_FORWARD               1 (to 686)
         >>  685 POP_TOP                 

1751     >>  686 LOAD_CONST                23 ('')
             689 LOAD_FAST                  0 (self)
             692 STORE_ATTR                49 (regno)

1752         695 LOAD_CONST                23 ('')
             698 LOAD_FAST                  0 (self)
             701 STORE_ATTR                50 (regno2)

1753         704 LOAD_FAST                  0 (self)
             707 LOAD_ATTR                 51 (get_reg_no_true)
             710 CALL_FUNCTION              0
             713 LOAD_FAST                  0 (self)
             716 STORE_ATTR                52 (reg_true)
```

上面是取注册码，并且调用 `get_reg_no_true()` 函数来验证，然后把结果保存在 `reg_true` 变量里面。再往下看：

代码:

```
2110     >> 4879 LOAD_FAST                  0 (self)
             4882 LOAD_ATTR                 52 (reg_true)
             4885 LOAD_CONST               214 ('YES')
             4888 COMPARE_OP                 3 (!=)
             4891 JUMP_IF_FALSE            29 (to 4923)
             4894 POP_TOP      

2111         4895 LOAD_GLOBAL               155 (Msg)
             4898 LOAD_FAST                  0 (self)
             4901 LOAD_ATTR                 156 (Hwnd)
             4904 LOAD_CONST               228 ('\xb5\xb1\xc7\xb0****\xce\xaa\xce\xb4\xd7\xa2\xb2\xe1\xb0\xe6\xb1\xbe\xa3\xac\xb5\xbc\xb3\xf6\xb9\xa6\xc4\xdc\xbb\xe1\xca\xdc\xb5\xbd\xcf\xde\xd6\xc6\xa3\xac\xc7\xeb\xb5\xbd****.com\xb8\xb6\xb7\xd1\xd7\xa2\xb2\xe1\xa3\xa1')
             4907 LOAD_CONST               229 ('\xcc\xe1\xca\xbe')
             4910 LOAD_CONST               225 ('ok')
             4913 LOAD_CONST               226 ('defbutton1')
             4916 CALL_FUNCTION              5
             4919 POP_TOP                 
             4920 JUMP_FORWARD               1 (to 4924)
         >> 4923 POP_TOP      
```

如果 `reg_true` 不等于 `'YES'`，就要弹出提示注册的对话框了。

关键 `call` 就是 `get_reg_no_true()` 了。只要让其返回 `'YES'` 便可。

这里可以修改 `get_reg_no_true()` 爆破了，但既然已经走了这么远了，索性再走一程，深入关键 `call` 去看个究竟。

代码:



```
\>>>sdis.dis("MyWindow.get\_reg\_no\_true")
\*\*\*\*\*\*\*\*   byte code for MyWindow.get\_reg\_no\_true  \*\*\*\*\*\*\*\*
2117           0 SETUP\_EXCEPT           638 (to 641)

2118           3 SETUP\_EXCEPT           173 (to 179)

2119           6 LOAD\_CONST               1 ('')
             9 STORE\_FAST               9 (mac\_address)

2120          12 LOAD\_CONST               2 ('.')
            15 STORE\_FAST               6 (strComputer)

2121          18 LOAD\_GLOBAL              2 (win32com)
            21 LOAD\_ATTR                3 (client)
            24 LOAD\_ATTR                4 (Dispatch)
            27 LOAD\_CONST               3 ('WbemScripting.SWbemLocator')
            30 CALL\_FUNCTION            1
            33 STORE\_FAST               3 (objWMIService)

2122          36 LOAD\_FAST               3 (objWMIService)
            39 LOAD\_ATTR                6 (ConnectServer)
            42 LOAD\_FAST               6 (strComputer)
            45 LOAD\_CONST               4 ('root\\cimv2')
            48 CALL\_FUNCTION            2
            51 STORE\_FAST               8 (objSWbemServices)

2123          54 LOAD\_FAST               8 (objSWbemServices)
            57 LOAD\_ATTR                8 (ExecQuery)
            60 LOAD\_CONST               5 ('Select \* from Win32\_NetworkAdapter')
            63 CALL\_FUNCTION            1
            66 STORE\_FAST              10 (colItems)

2124          69 SETUP\_LOOP              80 (to 152)
             72 LOAD\_FAST              10 (colItems)
             75 GET\_ITER
        >>   76 FOR\_ITER               72 (to 151)
             79 STORE\_FAST              4 (objItem)

2125          82 LOAD\_FAST               4 (objItem)
             85 LOAD\_ATTR              11 (MACAddress)
             88 LOAD\_CONST              0 (None)
             91 COMPARE\_OP              3 (!=)
             94 JUMP\_IF\_FALSE          50 (to 147)
             97 POP\_TOP

2142          98 LOAD\_CONST               6 ('VEN\_')
            101 LOAD\_FAST               4 (objItem)
            104 LOAD\_ATTR              13 (PNPDeviceID)
            107 COMPARE\_OP              6 (in)
            110 JUMP\_IF\_FALSE          30 (to 143)
            113 POP\_TOP
            114 LOAD\_CONST               7 ('DEV\_')
            117 LOAD\_FAST               4 (objItem)
            120 LOAD\_ATTR              13 (PNPDeviceID)
            123 COMPARE\_OP              6 (in)
            126 JUMP\_IF\_FALSE          14 (to 143)
            129 POP\_TOP

2143         130 LOAD\_FAST               4 (objItem)
            133 LOAD\_ATTR              11 (MACAddress)
            136 STORE\_FAST              9 (mac\_address)

2145         139 BREAK\_LOOP
            140 JUMP\_ABSOLUTE          148
        >>  143 POP\_TOP
            144 JUMP\_ABSOLUTE           76
        >>  147 POP\_TOP
        >>  148 JUMP\_ABSOLUTE           76
        >>  151 POP\_BLOCK

2146     >>  152 LOAD\_FAST               9 (mac\_address)
            155 LOAD\_CONST               1 ('')
            158 COMPARE\_OP              2 (==)
            161 JUMP\_IF\_FALSE          10 (to 174)
            164 POP\_TOP

2147         165 LOAD\_CONST               8 ('00:0A:EB:F5:D4:14')
            168 STORE\_FAST              9 (mac\_address)
            171 JUMP\_FORWARD            1 (to 175)
        >>  174 POP\_TOP
        >>  175 POP\_BLOCK
            176 JUMP\_FORWARD           13 (to 192)

2148     >>  179 POP\_TOP
            180 POP\_TOP
            181 POP\_TOP

2149         182 LOAD\_CONST               8 ('00:0A:EB:F5:D4:14')
            185 STORE\_FAST              9 (mac\_address)
            188 JUMP\_FORWARD            1 (to 192)
            191 END\_FINALLY

2150     >>  192 SETUP\_EXCEPT             68 (to 263)

2151         195 LOAD\_GLOBAL             14 (md5)
            198 LOAD\_ATTR               15 (new)
            201 LOAD\_FAST               9 (mac\_address)
            204 CALL\_FUNCTION            1
            207 LOAD\_ATTR               16 (hexdigest)
            210 CALL\_FUNCTION            0
            213 STORE\_FAST              7 (s)

2152         216 LOAD\_GLOBAL             14 (md5)
            219 LOAD\_ATTR               15 (new)
            222 LOAD\_FAST               7 (s)
            225 LOAD\_CONST              9 (16)
            228 SLICE+1
            229 LOAD\_FAST               7 (s)
            232 LOAD\_CONST              9 (16)
            235 SLICE+2
            236 BINARY\_ADD
            237 CALL\_FUNCTION            1
            240 LOAD\_ATTR               16 (hexdigest)
            243 CALL\_FUNCTION            0
            246 LOAD\_CONST             10 (6)
            249 LOAD\_CONST             11 (-6)
            252 SLICE+3
            253 LOAD\_FAST               0 (self)
            256 STORE\_ATTR             19 (regno)
            259 POP\_BLOCK
            260 JUMP\_FORWARD            16 (to 279)

2153     >>  263 POP\_TOP
            264 POP\_TOP
            265 POP\_TOP

2154         266 LOAD\_CONST             12 ('00e6e95aff213b8e40ff')
            269 LOAD\_FAST               0 (self)
            272 STORE\_ATTR             19 (regno)

2155         275 JUMP\_FORWARD            1 (to 279)
            278 END\_FINALLY

2156     >>  279 SETUP\_EXCEPT           323 (to 605)

2157         282 LOAD\_CONST               1 ('')
            285 STORE\_FAST              2 (tmp\_result)

2158         288 LOAD\_FAST               0 (self)
            291 LOAD\_ATTR              21 (conf\_user)
            294 LOAD\_ATTR              22 (config)
            297 LOAD\_ATTR              23 (regnum)
            300 LOAD\_FAST               0 (self)
            303 STORE\_ATTR             24 (regno2)

2159         306 LOAD\_GLOBAL             14 (md5)
            309 LOAD\_ATTR               15 (new)
            312 LOAD\_FAST               0 (self)
            315 LOAD\_ATTR               19 (regno)
            318 CALL\_FUNCTION            1
            321 LOAD\_ATTR               16 (hexdigest)
            324 CALL\_FUNCTION            0
            327 STORE\_FAST              7 (s)

2160         330 LOAD\_GLOBAL             14 (md5)
            333 LOAD\_ATTR               15 (new)
            336 LOAD\_FAST               7 (s)
            339 LOAD\_CONST             13 (14)
            342 SLICE+1
            343 LOAD\_CONST             14 ('bb2')
            346 BINARY\_ADD
            347 LOAD\_FAST               7 (s)
            350 LOAD\_CONST             13 (14)
            353 SLICE+2
            354 BINARY\_ADD
            355 CALL\_FUNCTION            1
            358 LOAD\_ATTR               16 (hexdigest)
            361 CALL\_FUNCTION            0
            364 STORE\_FAST              1 (md5\_tmp)

2161         367 LOAD\_FAST               1 (md5\_tmp)
            370 LOAD\_CONST             15 (28)
            373 SLICE+1
            374 LOAD\_FAST               1 (md5\_tmp)
            377 LOAD\_CONST              9 (16)
            380 LOAD\_CONST             16 (20)
            383 SLICE+3
            384 BINARY\_ADD
            385 LOAD\_FAST               1 (md5\_tmp)
            388 LOAD\_CONST              18 (0)
            391 LOAD\_CONST              18 (5)
            394 SLICE+3
            395 BINARY\_ADD
            396 LOAD\_FAST               1 (md5\_tmp)
            399 LOAD\_CONST              19 (7)
            402 LOAD\_CONST              20 (9)
            405 SLICE+3
            406 BINARY\_ADD
            407 LOAD\_FAST               1 (md5\_tmp)
            410 LOAD\_CONST              18 (5)
            413 LOAD\_CONST              19 (7)
            416 SLICE+3
            417 BINARY\_ADD
            418 LOAD\_FAST               1 (md5\_tmp)
            421 LOAD\_CONST              20 (9)
            424 LOAD\_CONST              9 (16)
            427 SLICE+3
            428 BINARY\_ADD
            429 LOAD\_FAST               1 (md5\_tmp)
            432 LOAD\_CONST              16 (20)
            435 LOAD\_CONST              15 (28)
            438 SLICE+3
            439 BINARY\_ADD
            440 LOAD\_FAST               1 (md5\_tmp)
            443 LOAD\_CONST              21 (8)
            446 LOAD\_CONST              22 (12)
            449 SLICE+3
            450 BINARY\_ADD
            451 LOAD\_FAST               1 (md5\_tmp)
            454 LOAD\_CONST              23 (17)
            457 LOAD\_CONST              24 (21)
            460 SLICE+3
            461 BINARY\_ADD
            462 STORE\_FAST              5 (reg\_tmp)

2162         465 LOAD\_FAST               0 (self)
            468 LOAD\_ATTR               24 (regno2)
            471 LOAD\_CONST              17 (0)
            474 LOAD\_CONST              25 (4)
            477 SLICE+3
            478 LOAD\_FAST               5 (reg\_tmp)
            481 LOAD\_CONST              17 (0)
            484 LOAD\_CONST              25 (4)
            487 SLICE+3
            488 COMPARE\_OP              2 (==)
            491 JUMP\_IF\_FALSE         106 (to 600)
            494 POP\_TOP

2163         495 LOAD\_FAST               0 (self)
            498 LOAD\_ATTR               24 (regno2)
            501 LOAD\_CONST              25 (4)
            504 LOAD\_CONST              21 (8)
            507 SLICE+3
            508 LOAD\_FAST               5 (reg\_tmp)
            511 LOAD\_CONST              25 (4)
            514 LOAD\_CONST              21 (8)
            517 SLICE+3
            518 COMPARE\_OP              2 (==)
            521 JUMP\_IF\_FALSE          72 (to 596)
            524 POP\_TOP

2164         525 LOAD\_FAST               0 (self)
            528 LOAD\_ATTR               24 (regno2)
            531 LOAD\_CONST              19 (7)
            534 LOAD\_CONST              26 (15)
            537 SLICE+3
            538 LOAD\_FAST               5 (reg\_tmp)
            541 LOAD\_CONST              19 (7)
            544 LOAD\_CONST              26 (15)
            547 SLICE+3
            548 COMPARE\_OP              2 (==)
            551 JUMP\_IF\_FALSE          38 (to 592)
            554 POP\_TOP

2165         555 LOAD\_FAST               0 (self)
            558 LOAD\_ATTR               24 (regno2)
            561 LOAD\_CONST              13 (14)
            564 SLICE+1
            565 LOAD\_FAST               5 (reg\_tmp)
            568 LOAD\_CONST              13 (14)
            571 SLICE+1
            572 COMPARE\_OP              2 (==)
            575 JUMP\_IF\_FALSE          10 (to 588)
            578 POP\_TOP

2166         579 LOAD\_CONST              27 ('ok')
            582 STORE\_FAST              2 (tmp\_result)
            585 JUMP\_ABSOLUTE          593
        >>  588 POP\_TOP
            589 JUMP\_ABSOLUTE          597
        >>  592 POP\_TOP
        >>  593 JUMP\_ABSOLUTE          601
        >>  596 POP\_TOP
        >>  597 JUMP\_FORWARD            1 (to 601)
        >>  600 POP\_TOP
        >>  601 POP\_BLOCK
            602 JUMP\_FORWARD            7 (to 612)

2167     >>  605 POP\_TOP
            606 POP\_TOP
            607 POP\_TOP

2168         608 JUMP\_FORWARD            1 (to 612)
            611 END\_FINALLY

2169     >>  612 LOAD\_FAST               2 (tmp\_result)
            615 LOAD\_CONST              27 ('ok')
            618 COMPARE\_OP              2 (==)
            621 JUMP\_IF\_FALSE           8 (to 632)
            624 POP\_TOP

2170         625 LOAD\_CONST              28 ('YES')
            628 RETURN\_VALUE
            629 JUMP\_FORWARD            5 (to 637)
        >>  632 POP\_TOP

2172         633 LOA
```



代码有点长，看着看着就迷失在代码的丛林里了。这里可以讨个巧，借助免费的反编译引擎来帮忙。

首先获得`get_reg_no_true()`这个函数的`code object`，然后使用`marshal.dump()`保存成文件(从`py2exe`的源代码哪里学来的)。然后用`WinHex`加上`8`个字节的`file header`。前`4`个字节代表`Python`版本号，`2.4`是`6DF20D0A`。后`4`个字节是`timestamp`，随便写就是。

接着来到这里：[`www.depython.net/`](http://www.depython.net/)

这个据说是`team509`做的，它可以免费反编译小于`5KB`的文件。

反编译出的结果：

```
try:
    try:
        mac_address = ''
        strComputer = '.'
        objWMIService = win32com.client.Dispatch('WbemScripting.SWbemLocator')
        objSWbemServices = objWMIService.ConnectServer(strComputer, 'root\\cimv2')
        colItems = objSWbemServices.ExecQuery('Select * from Win32_NetworkAdapter')
        for objItem in colItems:
            if (objItem.MACAddress != None):
                if (('VEN_' in objItem.PNPDeviceID) and ('DEV_' in objItem.PNPDeviceID)):
                    mac_address = objItem.MACAddress
                    break

        if (mac_address == ''):
            mac_address = '00:0A:EB:F5:D4:14'
    except:
        mac_address = '00:0A:EB:F5:D4:14'
    try:
        s = md5.new(mac_address).hexdigest()
        self.regno = md5.new((s[16:] + s[:16])).hexdigest()[6:-6]
    except:
        self.regno = '00e6e95aff213b8e40ff'
    try:
        tmp_result = ''
        self.regno2 = self.conf_user.config.regnum
        s = md5.new(self.regno).hexdigest()
        md5_tmp = md5.new(((s[14:] + 'bb2') + s[:14])).hexdigest()
        reg_tmp = ((((((((md5_tmp[28:] + md5_tmp[16:20]) + md5_tmp[0:5]) + md5_tmp[7:9]) + md5_tmp[5:7]) + md5_tmp[9:16]) + md5_tmp[20:28]) + md5_tmp[8:12]) + md5_tmp[17:21])
        if (self.regno2[0:4] == reg_tmp[0:4]):
            if (self.regno2[4:8] == reg_tmp[4:8]):
                if (self.regno2[7:15] == reg_tmp[7:15]):
                    if (self.regno2[14:] == reg_tmp[14:]):
                        tmp_result = 'ok'
    except:
        pass
    if (tmp_result == 'ok'):
        return 'YES'
    else:
        return 'ERR'
except:
    return 'ERR'
```

算法一目了然！稍微改一下，注册机就可以出炉了。

综上，使用`Python`开发的商业软件，其安全性还值得商榷。抵御攻击的做法是使用第三方库编译成`native code`，使用代码混淆器，或者修改`Python`源代码防止被反汇编。
