
```c
lkd> !process 0 1 cpp_test_ano.exe
PROCESS ffffac0f2b92f080
...
PROCESS ffffac0f1ccb6080(第二个是刚刚创建的进程)
...
lkd> dt nt!_EPROCESS sectionobject imagefilepointer ffffac0f1ccb6080
+0x3c0 SectionObject : 0xffff8184`7b06ecf0 Void
+0x448 ImageFilePointer : 0xffffac0f`159a2180 _FILE_OBJECT
lkd> dx -id 0,0,ffffac0f0ede7080 -r1 ((ntkrnlmp!_FILE_OBJECT *)0xffffac0f159a2180)
((ntkrnlmp!_FILE_OBJECT *)0xffffac0f159a2180) : 0xffffac0f159a2180 [Type: _FILE_OBJECT *]
[+0x028] SectionObjectPointer : 0xffffac0f066a6358 [Type: _SECTION_OBJECT_POINTERS *]
lkd> dx -r1 ((ntkrnlmp!_SECTION_OBJECT_POINTERS *)0xffffac0f066a6358)
((ntkrnlmp!_SECTION_OBJECT_POINTERS *)0xffffac0f066a6358) : 0xffffac0f066a6358 [Type: _SECTION_OBJECT_POINTERS *]
[+0x000] DataSectionObject : 0xffffac0f0727b1d0 [Type: void *]
[+0x008] SharedCacheMap : 0x0 [Type: void *]
[+0x010] ImageSectionObject : 0xffffac0f0ebd3720 [Type: void *]
```

刚创建的进程对应的`ImageSectionObject`和之前在`ProcessHerpaderping`进程看到的结果一样，代表刚创建的进程和`ProcessHerpaderping`启动的`cpp_test_ano.exe`进程共享了image类对应的内存，共享了对应的CFF Explorer程序代码。

> 关于上面的解读，这涉及到windows操作系统对section的管理，比如文件的映射细节和文件的缓存管理，有兴趣的读者可以参考 `Windows Internals 7th part1` 第五章内存管理的section小节和`Windows Internals 7th part2`第十一章缓存管理器部分。

> - 该方法的源码可参考这个<a href="elink@d7fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7P5s2W2Q4x3X3c8K6i4K6u0r3K9r3g2J5M7r3q4V1k6i4u0H3K9h3&6Y4">仓库</a>。

> - 该方法的作者用windbg进一步分析了共享的更多细节，可参考这个<a href="elink@876K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7P5s2W2Q4x3X3c8K6i4K6u0r3K9r3g2J5M7r3q4V1k6i4u0H3K9h3&6Y4i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2K9h3&6Q4x3V1k6J5k6i4y4Q4x3V1k6p5K9i4k6A6L8X3N6p5k6h3g2H3k6i4u0Q4x3X3g2E0k6l9`.`.">文档</a>。

## Conclusion

虽然进程注入在不断更新，不过安全厂商也在与时俱进，目前很多安全厂商都有这些方法的监控了（比如上面提到的分析文章，大部分是安全厂商写的）。从`Process Doppelganging`开始，我们能看到新的方法都是源于操作系统的不足，并慢慢衍生，可能以后的进程注入会越来越底层，越来越复杂。

本文涉及的细节很多，不能面面俱到，推荐读者看本文各个小节的推荐文章，最后再来看本文，相信读者会有更多的收获。

目录

- [高级进程注入总结](https://bbs.kanxue.com/#高级进程注入总结)

- [进程注入简介](https://bbs.kanxue.com/#进程注入简介)

- Module Stomping

- Process Hollowing

- Process Doppelganging

- Transacted Hollowing

- Process Ghosting

- Ghostly Hollowing

- Process Herpaderping

- [Conclusion](https://bbs.kanxue.com/#conclusion)

# 高级进程注入总结

前几天看玄武的“每日安全”板块，发现了一篇关于[进程注入](https://bbs.kanxue.com/elink@e73K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9k6h3c8f1k6h3q4E0e0%4m8W2M7X3q4@1K9h3!0F1M7#2)9J5c8V1q4V1N6X3q4F1j5$3g2V1i4K6u0V1f1s2u0G2j5$3g2K6M7#2)9J5k6p5W2F1K9X3g2U0N6r3W2G2L8W2)9J5k6q4N6G2M7X3E0K6K9r3!0H3)的文章，之前了解过一些相关的技术，不过是零散的。看到文章总结的挺详细的，且各种方法的异同细节容易忘，于是就想着总结一下各种`进程注入`的异同和优缺点，因此有了本文。

> - 关于常见的进程注入方法，请参考[这个](https://bbs.kanxue.com/elink@10fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3e0r3W2@1N6r3I4W2d9r3q4F1L8W2)9J5c8Y4m8Q4x3V1j5$3x3K6x3$3z5e0f1H3i4K6u0W2K9s2c8E0L8l9`.`.)和[这个](https://bbs.kanxue.com/elink@98dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8r3q4K6N6r3W2U0i4K6u0W2j5$3!0Q4x3V1k6U0L8W2)9J5c8X3u0D9L8$3N6Q4x3V1k6@1k6h3&6Q4x3X3c8H3M7X3!0U0k6i4y4K6i4K6u0V1K9h3&6B7k6h3y4@1K9h3!0F1i4K6u0V1N6r3g2U0K9r3&6A6M7i4g2W2M7#2)9J5k6s2c8W2j5$3S2F1K9h3y4S2L8q4)9J5k6s2y4#2M7Y4k6W2P5g2)9J5k6r3y4G2L8h3#2G2L8W2)9J5k6r3q4F1k6q4)9J5k6s2c8J5k6h3&6V1K9h3&6Y4i4K6u0V1M7s2u0G2j5$3g2K6M7H3`.`.)，本文将重点描述值得留意的方法（比如Module Stomping），和一些思路新颖的方法。

> - 以下描述的方法在[进程注入](https://bbs.kanxue.com/elink@c38K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9k6h3c8f1k6h3q4E0e0%4m8W2M7X3q4@1K9h3!0F1M7#2)9J5c8V1q4V1N6X3q4F1j5$3g2V1i4K6u0V1f1s2u0G2j5$3g2K6M7#2)9J5k6p5W2F1K9X3g2U0N6r3W2G2L8W2)9J5k6q4N6G2M7X3E0K6K9r3!0H3)这个仓库都能找到。

> - 本文在[我的博客](https://bbs.kanxue.com/elink@8beK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3&6W2j5$3!0&6i4K6u0W2j5$3&6Q4x3V1k6f1k6h3y4Z5L8X3!0D9L8$3N6&6i4K6u0r3f1X3g2$3k6i4u0K6k6g2)9J5c8W2m8J5L8$3y4W2M7%4y4Q4x3X3c8u0L8X3A6W2j5%4c8A6L8$3&6Q4x3V1j5`.)同时发布。

## 进程注入简介

进程注入就是将代码注入到另一个进程中，shellcode注入和DLL注入都属于进程注入。

进程注入的作用是隐藏自身或者利用另一个进程的权限做一些不好的事情^\_^，比如病毒等恶意程序注入到一个正常的进程，以此隐藏自己。

进程注入的方法非常之多，很多与DLL注入有关，比如注册表（Image File Execution Options）、DLL劫持、输入法、COM、LSP劫持（LayerService Provider，与winsock有关）...

除了DLL注入，还有shellcode注入，因为shellcode更小，所以shellcode的使用也更加多样。

2017年，在黑客大会上Eugene Kogan和Tal Liberman又分享了更加隐蔽和特别的方法，比如Process Doppelganging。那么接下来就开始进程注入方法的介绍。

## Module Stomping

该方法通过在目标进程中加载一个合法DLL，然后将shellcode或恶意DLL覆写到这个合法DLL的地址空间里。

伪代码如下（省略错误检查）：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21 | `HANDLE hTargetHandle` `=` `OpenProcess(PROCESS_QUERY_INFORMATION | PROCESS_VM_READ | PROCESS_ALL_ACCESS | PROCESS_SUSPEND_RESUME, FALSE, targetPid);`  `/``/``...`    `/``/` `Use OpenProcess` `+` `VirtualAllocEx` `+` `CreateRemoteThread(the traditional dll injection method)`  `injectLoadLibrary(hTargetHandle, legitimateDll);`  `/``/``...`    `/``/` `Copy every malware dll``'s section into legitimate dll'``s corresponding sections within the target process address space.`  `for``(section : malwareDll){`  `WriteProcessMemory(hTargetHandle, legitimateDllSection, section, section_size...);`    `if` `(has executable section)`  `BypassCFG();`  `}`    `RebuildImportTable();`  `RebuildRelocationTable();`    `callTLSCallback();`    `CreateRemoteThread(malwareDllEntryPoint);` |

通过该方法可以将恶意代码隐藏，使得Defender扫描目标进程空间时不会发现恶意代码的存在（因为恶意代码潜伏在合法的模块里）。

该方法有两个值得关注的点：

- 先在目标进程加载一个合法DLL，以此隐藏恶意代码。

- 它用了`SetProcessValidCallTargets`，该方法可以bypass开启CFG的程序。

> 与CFG(control flow guard)相关的知识可参考`MSDN`或者`System Internals 7th part1` security chapter。

不过该方法还是有明显的不足，比如会调用显眼的`VirtualAllocEx`、`WriteProcessMemory`、`VirtualProtectEx`和`CreateRemoteThread`，通常的EDR都能检测出可能的恶意行为。

> 该方法的详细细节可参考这篇[文章](https://bbs.kanxue.com/elink@a98K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2k6W2)9J5k6s2y4W2j5%4g2J5k6g2)9J5k6h3y4G2L8g2)9J5c8X3S2A6k6r3W2F1k6#2)9J5k6r3#2S2L8r3W2U0K9h3!0#2M7#2)9J5k6r3y4G2k6r3g2Q4x3X3c8%4K9i4c8Z5i4K6u0V1L8h3!0V1N6h3I4W2i4K6u0V1M7%4c8G2L8i4m8A6L8X3N6Q4x3V1j5`.)。

## Process Hollowing

该方法也比较经典，并被广泛使用，其基本流程如下：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8 | `CreateProcess(“svchost.exe” , …, CREATE_SUSPENDED, …);`  `NtUnmapViewOfSection(…)` `and` `VirtualAllocEx(…);`  `For each section:`  `WriteProcessMemory(..., EVIL_EXE, …);`  `Relocate Image``*``;`  `Set` `base address` `in` `PEB``*``;`  `SetThreadContext(…);`  `ResumeThread(…);` |

该方法和`Module Stomping`很像，都是替换掉了一个模块，不过有以下三个不同：

- `Process Hollowing`替换的模块是目标EXE程序

- 该方法使用`NtUnmapViewOfSection`将EXE程序解除映射

- 用`ResumeThread`来启动恶意程序的入口点

> 该方法的详细实现可参考[这个](https://bbs.kanxue.com/elink@2edK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6M7X3g2V1i4K6u0W2N6r3g2S2L8g2)9J5c8X3!0X3k6X3g2F1M7$3W2$3k6g2)9J5k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0r3j5$3!0V1k6g2)9J5k6r3W2F1K9X3g2U0N6r3W2G2L8W2)9J5k6s2m8J5L8$3y4W2M7%4y4Q4x3X3c8A6L8X3A6W2j5%4c8A6L8$3&6Q4x3V1k6H3M7X3!0U0k6i4y4K6i4K6u0V1K9r3!0D9L8r3!0%4K9h3&6Y4i4K6u0V1j5h3&6V1i4K6u0V1M7r3g2Q4x3X3c8A6L8h3q4Y4k6g2)9J5k6s2u0W2L8r3!0U0j5i4c8A6L8$3&6K6i4K6t1K6j5h3I4D9L8$3y4S2N6r3W2F1k6#2)9J5k6r3#2W2L8h3!0J5P5g2)9J5k6r3W2F1i4K6u0V1k6r3g2K6N6r3W2F1j5i4c8A6L8$3&6Q4x3X3c8A6L8h3q4Y4k6b7`.`.)。

## Process Doppelganging

接下来描述的几个进程注入方法使用了不同的注入思路，它们之间只有细微的差别，最后会总结它们的异同点。

该方法是2017年在黑客大会介绍的，演讲者首先说明了`Process Hollowing`涉及的敏感操作和一些不足：

- Unmap目标进程的EXE模块（非常可疑），现代的安全检查一般都有Unmap检查。

- 如果不Unmap，而是直接覆写程序，那么覆写地址的页属性就不是共享的，也很可疑。

  > 页属性共享涉及到操作系统的共享内存机制，这里简单描述一下：
  >
  > 为了节省内存，当一个模块加载进内存时，比如`ntdll.dll`，操作系统会看该模块是否已被其他进程加载过了（即已经在物理内存中了），如果在，那么操作系统只需要简单地将该物理区域映射到需要加载模块的进程中。因此，这些模块对应的内存是共享的，这些模块内存对应的进程pte(page table entry)会指向一个叫`prototypePTE`的结构，该结构会指向这些共享的物理内存。
  >
  > 不过，如果我们向模块，比如`ntdll.dll`，的数据段写入数据，那么这时操作系统就会对`ntdll.dll`的数据段分配一段物理内存，然后当前进程对应数据段的pte就不会再指向`prototypePTE`，而是指向操作系统分配的物理内存。
  >
  > 关于内存共享机制的详细描述，读者看参考Windows Internals 7th，第五章的Page fault handling部分。

- 直接覆写程序不够好，那就Unmap后再Remap：
  - 如果remap时的类型不是IMAGE，通过检查节的类型可判定是否可疑
