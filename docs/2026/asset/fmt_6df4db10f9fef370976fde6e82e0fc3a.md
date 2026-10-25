
```diff
diff --git a/qiling/loader/pe.py b/qiling/loader/pe.py

index b6e46c8e..3375bbf2 100644

--- a/qiling/loader/pe.py

+++ b/qiling/loader/pe.py

@@ -87,15 +87,20 @@ class Process():

                 self.ql.nprint("[+] Cached %s" % path)



         dll_base = self.dll_last_address

+

         dll_len = self.ql.os.heap._align(len(bytes(data)), 0x1000)

         self.dll_size += dll_len

         self.ql.mem.map(dll_base, dll_len, info=dll_name)

         self.ql.mem.write(dll_base, bytes(data))

         self.dll_last_address += dll_len



+

         # add dll to ldr data

         self.add_ldr_data_table_entry(dll_name)



+        # add DLL to coverage images

+        self.images.append(self.QlImage(dll_base, dll_len, path))

+

         self.ql.nprint("[+] Done with loading %s" % path)

         return dll_base



@@ -328,6 +333,8 @@ class QlLoaderPE(QlLoader, Process):

             self.pe_image_address = self.pe_image_address = self.pe.OPTIONAL_HEADER.ImageBase

             self.pe_image_address_size = self.pe_image_address_size = self.pe.OPTIONAL_HEADER.SizeOfImage



+            self.images.append(self.QlImage(self.pe_image_address, self.pe_image_address + self.pe_image_address_size, self.path))

```
这部分代码可以为任意`loader`加载起来的PE内存镜像提供代码覆盖率信息，也包括DLL依赖，当追踪开启时，可通过`block_callback`这一函数回调打印出每个基本块的执行日志。

```python
    def block_callback(ql, address, size, self):
        for mod_id, mod in enumerate(ql.loader.images):
            if mod.base <= address <= mod.end:
                ent = bb_entry(address - mod.base, size, mod_id)
                self.basic_blocks.append(ent)
                break
```

我把这篇博客搁置了一段时间，后来发现`xwings`在6月份已经把这一功能添加到PE中了，所以我提交了一个`PR`以支持获取DLL的代码覆盖率。

## 接触 Dragon Dance

在`IDA Pro`和`Binary Ninja`中的可视化代码覆盖率功能，其事实标准是一个叫作`Lighthouse`的插件。如果你使用上述两种反汇编器，我强烈推荐你使用它。在我的业余时间，我一直在尝试使用`Ghidra`，它有一个很棒的反汇编器和反编译器，并且它还提供了大量`IDA Pro`缺少的功能。一旦你习惯了它的用法，你会发现它是一个非常棒的自由开源的逆向工具。我偶然间发现了一个叫做`Dragon Dance`的插件，它能在`Ghidra`中提供和`Lighthouse`相似的代码覆盖率可视化功能，更重要的是，它支持本文中提到的`DRCOV`追踪格式。它支持大量的内置引用，能够让你做到诸如在高亮语法中进行`diff`追踪这样的功能。下面是从`README`中摘录的一段文字，介绍如何用这些函数来实现脚本接口。

![](https://bbs.kanxue.com/upload/attach/202009/706972_29AGYDBBDCTE4CK.jpg)

除此之外，`Dragon Dance`支持还有修复功能，它本质上是将二进制文件中的代码覆盖率完整性与`Ghidra`反汇编出的指令进行比较。如果这两者不同，插件会提示用户修复`Ghidra`缺失的部分。下面也是从`README`中摘录的一段介绍：

![](https://bbs.kanxue.com/upload/attach/202009/706972_KFGG9TPCMAPMB7D.jpg)

## 组合所有功能

现在我们已经能够追踪PE覆盖率，而且能够在逆向工程软件中将它可视化。接下来看一个例子，`Qiling`附带了一个`WannaCry`的二进制文化，我们用它来进行举例。首先，要想安装它，我需要依照`Dragon Dance`中的编译步骤将它编译出来，编译步骤非常简单。编译并安装之后，就可以在`Ghidra`中使用它了。我运行`qltool`并启用追踪功能来对`WannaCry`进行分析。

```
# python qltool run --disasm -c wannacry-trace.cov -f examples/rootfs/x86_windows/bin/wannacry.bin --rootfs examples/rootfs/x86_windows/ 

[+] Initiate stack address at 0xfffdd000 

[+] Loading examples/rootfs/x86_windows/bin/wannacry.bin to 0x400000

[+] PE entry point at 0x409a16

[+] TEB addr is 0x6000

[+] PEB addr is 0x6044

[+] Loading examples/rootfs/x86_windows/Windows/SysWOW64/ntdll.dll to 0x10000000

[+] Done with loading examples/rootfs/x86_windows/Windows/SysWOW64/ntdll.dll

[+] Loading examples/rootfs/x86_windows/Windows/SysWOW64/kernel32.dll to 0x10141000

[+] Done with loading examples/rootfs/x86_windows/Windows/SysWOW64/kernel32.dll

[+] Loading examples/rootfs/x86_windows/Windows/SysWOW64/advapi32.dll to 0x10215000

[+] Done with loading examples/rootfs/x86_windows/Windows/SysWOW64/advapi32.dll

[+] Loading examples/rootfs/x86_windows/Windows/SysWOW64/ws2_32.dll to 0x102b6000

[+] Done with loading examples/rootfs/x86_windows/Windows/SysWOW64/ws2_32.dll

[+] Loading examples/rootfs/x86_windows/Windows/SysWOW64/msvcp60.dll to 0x102eb000

[+] Done with loading examples/rootfs/x86_windows/Windows/SysWOW64/msvcp60.dll

[+] Loading examples/rootfs/x86_windows/Windows/SysWOW64/iphlpapi.dll to 0x10351000

[+] Done with loading examples/rootfs/x86_windows/Windows/SysWOW64/iphlpapi.dll

[+] Loading examples/rootfs/x86_windows/Windows/SysWOW64/wininet.dll to 0x1036d000

[+] Done with loading examples/rootfs/x86_windows/Windows/SysWOW64/wininet.dll

[+] Loading examples/rootfs/x86_windows/Windows/SysWOW64/msvcrt.dll to 0x10462000

[+] Done with loading examples/rootfs/x86_windows/Windows/SysWOW64/msvcrt.dll

-snip-

[+] 0x408171       50                            push eax

[+] 0x408172       50                            push eax

[+] 0x408173       50                            push eax

[+] 0x408174       6a 01                         push 1

[+] 0x408176       50                            push eax

[+] 0x408177       88 44 24 6b                   mov byte ptr [esp + 0x6b], al

[+] 0x40817b       ff 15 34 a1 40 00             call dword ptr [0x40a134]

[+] 0x1039c18e     8b ff                         mov edi, edi

InternetOpenA(lpszAgent = 0x0, dwAccessType = 0x1, lpszProxy = 0x0, lpszProxyBypass = 0x0, dwFlags = 0x0)

[+] 0x408181       6a 00                         push 0

[+] 0x408183       68 00 00 00 84                push 0x84000000

[+] 0x408188       6a 00                         push 0

[+] 0x40818a       8d 4c 24 14                   lea ecx, [esp + 0x14]

[+] 0x40818e       8b f0                         mov esi, eax

[+] 0x408190       6a 00                         push 0

[+] 0x408192       51                            push ecx

[+] 0x408193       56                            push esi

[+] 0x408194       ff 15 38 a1 40 00             call dword ptr [0x40a138]

[+] 0x103b00f1     8b ff                         mov edi, edi

InternetOpenUrlA(hInternet = 0x0, lpszUrl = "e2dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2#2M7h3g2J5k6Y4y4G2k6s2l9&6K9h3k6B7j5i4m8G2M7$3c8X3K9X3S2Y4L8%4y4#2M7X3W2B7k6X3q4W2N6%4u0%4k6i4u0Y4N6$3g2S2i4K6u0W2j5$3!0E0", lpszHeaders = 0x0, dwHeadersLength = 0x0, dwFlags = 0x84000000, dwContext = 0x0)

[+] 0x40819a       8b f8                         mov edi, eax

[+] 0x40819c       56                            push esi

[+] 0x40819d       8b 35 3c a1 40 00             mov esi, dword ptr [0x40a13c]

[+] 0x4081a3       85 ff                         test edi, edi

[+] 0x4081a5       90                            nop 

[+] 0x4081a6       90                            nop 

[+] 0x4081a7       ff d6                         call esi

[+] 0x10387b49     8b ff                         mov edi, edi

InternetCloseHandle(hInternet = 0x0) = 0x1

-snip-
```

传递给`qltool`的几个选项功能如下：

- `--disasm` 对所有执行的指令进行反汇编
- `-c` 显示 `DRCONV` 输出的 `trace` 文件路径
