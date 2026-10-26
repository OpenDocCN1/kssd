
```
Util!HttpUtil::OnDebug+0x13f8:

00007fff`0aa7a2c8 cc              int     3

0:000> kv

 # Child-SP          RetAddr           : Args to Child                                                           : Call Site

00 00000042`558fed10 00007fff`0aa78d2f : 00000219`596f44a0 00000042`558ff339 00000042`558ff2f4 00000042`558ff2f4 : Util!HttpUtil::OnDebug+0x13f8

01 00000042`558fee30 00007fff`0ab05616 : 00000042`558fefe0 00000219`596f44a0 00007fff`0aa60000 00000219`59660000 : Util!mba::util::UrlsManager::ParseXml+0x63f

02 00000042`558fee70 00007fff`0ab04879 : 00000000`00000006 00000000`00000006 00000042`558ff2f4 00000042`558ff339 : Util!CWMI::GetOutPutUIntByString+0x3bb6

03 00000042`558ff1b0 00007fff`0ab04af2 : 00000000`00000000 00000000`00000000 00000042`558ff3c0 00000000`00000000 : Util!CWMI::GetOutPutUIntByString+0x2e19

04 00000042`558ff2d0 00007fff`0aaf53df : 00000000`00000000 7fffffff`ffffffff 00000000`00000000 00000000`00000000 : Util!CWMI::GetOutPutUIntByString+0x3092

05 00000042`558ff3a0 00007ff6`fa838ff3 : 7fffffff`ffffffff 00000000`00000000 7fffffff`ffffffff 00000000`00000000 : Util!SMBIOSHelper::IsSupportDevice+0x1f

06 00000042`558ff400 00007ff6`fa852223 : 00000000`00000000 00000000`0000000a 00000000`00000000 00000000`00000000 : PCManager!SMBIOSHelper::operator=+0xa93

07 00000042`558ffc20 00007fff`4f6f7bd4 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : PCManager!Compression::Compression+0x3b93

08 00000042`558ffc60 00007fff`5150ced1 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : KERNEL32!BaseThreadInitThunk+0x14

09 00000042`558ffc90 00000000`00000000 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : ntdll!RtlUserThreadStart+0x21



0:000> g

ntdll!NtTerminateProcess+0x14:

00007fff`5153c644 c3              ret

```

咋还出现int3了呢，难道这里就是检查是不是华为电脑的位置。

使用IDA开始粗略分析，进入int3的位置。一处异常，看不出什么东西。

![7.png](https://bbs.kanxue.com/upload/attach/202001/394321_GM66NJSADZ9Y3CH.jpg)

往上回溯两层，看到很重要的信息`SystemEnvironment::GetSupportMachineList`，确实很像是检查的地方。

![8.png](https://bbs.kanxue.com/upload/attach/202001/394321_4PSKEVCFH5YX788.jpg)

对`sub_180018D00`下断，查看参数。发现第三个参数指向一个xml配置文件路径，看名字很明显是表示支持的机器类型列表。

```
0:000> db 000001f0`587329b0 

000001f0`587329c0  5c bb aa ce aa 5c 50 43-4d 61 6e 61 67 65 72 5f  \....\PCManager_

000001f0`587329d0  53 65 74 75 70 5f 39 2e-31 2e 36 2e 33 33 5c 48  Setup_9.1.6.33\H

000001f0`587329e0  75 61 77 65 69 5c 50 43-4d 61 6e 61 67 65 72 5c  uawei\PCManager\

000001f0`587329f0  5c 63 6f 6e 66 69 67 5c-4d 61 63 68 69 6e 65 54  \config\MachineT

000001f0`58732a00  79 70 65 4c 69 73 74 2e-78 6d 6c 00 ee fe ee ab  ypeList.xml.....

```

打开一看，加密了。

![9.png](https://bbs.kanxue.com/upload/attach/202001/394321_HDQXJUEB2JR5JF3.jpg)

那看看附近代码，应该是先解密，可以把数据dump出来。

通过IDA翻看了一下后面的函数，看到另一个特征`rapidxml::parse_error::`vftable`，这是用了rapidxml开源库来解析xml文档。

把rapidxml的源码下回来，通过对比（一些log字符特征）来确认函数的功能。

![10.png](https://bbs.kanxue.com/upload/attach/202001/394321_HTC5W622T3MKVQ6.jpg)

基本弄清楚xml解密到解析的过程，对`sub_180014EA0`下断，查看参数。果然拿到解密后的xml文件。

```
0:000> dq 000000315e6feb48

00000031`5e6feb48  000001ec`051970b0 00000000`00000000

0:000> db 000001ec`051970b0

000001ec`051970b0  3c 3f 78 6d 6c 20 76 65-72 73 69 6f 6e 3d 22 31  <?xml version="1

000001ec`051970c0  2e 30 22 20 65 6e 63 6f-64 69 6e 67 3d 22 55 54  .0" encoding="UT

000001ec`051970d0  46 2d 38 22 3f 3e 20 0d-0a 3c 6d 61 63 68 69 6e  F-8"?> ..<machin

000001ec`051970e0  65 54 79 70 65 6c 69 73-74 3e 20 0d 0a 09 3c 6d  eTypelist> ...<m

```

xml数据长这样。

![11.png](https://bbs.kanxue.com/upload/attach/202001/394321_98KTNKZAEYYFKDX.jpg)

看到这里，其实已经有一个思路了，就是把自己电脑的品牌加入xml文件，重新加密回去，就能够就能够使用华为电脑管家了。

不过分析解密、拿品牌信息、加密等挺麻烦的，再看看还有方法吗？

比如直接patch比较品牌的函数，绕过检查。

继续分析，一堆xml的node的解析比较，因为华为在编译rapidxml时基本都是内联函数，导致跟源码结构并不是很一致，增加了分析难度。

经过一段数据的比较，还没有找到绕过的点。

我停了下来，看了看调用栈。

```
0:000> kv

 # Child-SP          RetAddr           : Args to Child                                                           : Call Site

00 00000031`5e6fea20 00007fff`11184879 : 00000000`00000006 00000000`00000006 00000031`5e6feea4 00000031`5e6feee9 : Util!CWMI::GetOutPutUIntByString+0x4075

01 00000031`5e6fed60 00007fff`11184af2 : 00000000`00000000 00000000`00000000 00000031`5e6fef70 00000000`00000000 : Util!CWMI::GetOutPutUIntByString+0x2e19

02 00000031`5e6fee80 00007fff`111753df : 00000000`00000000 7fffffff`ffffffff 00000000`00000000 00000000`00000000 : Util!CWMI::GetOutPutUIntByString+0x3092

03 00000031`5e6fef50 00007ff7`70a18ff3 : 7fffffff`ffffffff 00000000`00000000 7fffffff`ffffffff 00000000`00000000 : Util!SMBIOSHelper::IsSupportDevice+0x1f

04 00000031`5e6fefb0 00007ff7`70a32223 : 00000000`00000000 00000000`0000000a 00000000`00000000 00000000`00000000 : PCManager!SMBIOSHelper::operator=+0xa93

05 00000031`5e6ff7d0 00007fff`4f6f7bd4 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : PCManager!Compression::Compression+0x3b93

06 00000031`5e6ff810 00007fff`5150ced1 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : KERNEL32!BaseThreadInitThunk+0x14

07 00000031`5e6ff840 00000000`00000000 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : ntdll!RtlUserThreadStart+0x21

```

才看到被我忽略的一个很重要的信息`Util!SMBIOSHelper::IsSupportDevice`，这个函数名太明显了。

只需要修改这个返回值，基本应该就绕过检查了。

```
BOOL8 __fastcall SMBIOSHelper::IsSupportDevice(SMBIOSHelper *this)

{

 return 1; //patch

}

```

是的，我好像又掉坑里，浪费了挺多时间。

### 4

但是前面为什么会出现异常呢，导致我掉进了一个确实是对的但又有点坑的位置呢。

想到xml的路径，此时我终于反应过来哪个异常可能是什么了。

因为我把软件放在了中文目录中，而软件对中文处理貌似有问题。（我没有深究了）

```
D:\华为\PCManager_Setup_9.1.6.33

```

果然，去掉中文目录后，再没有上面看到的int3。

所以，这又是个坑。C

本来正常的分析逻辑是这样的。

```
0:000> g

ntdll!NtTerminateProcess+0x14:

00007fff`5153c644 c3              ret

0:000> kv

 # Child-SP          RetAddr           : Args to Child                                                           : Call Site

00 00000042`558ffb38 00007fff`5150a9b8 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : ntdll!NtTerminateProcess+0x14

01 00000042`558ffb40 00007fff`4f6fcd8a : 00000000`00000000 00000000`00000000 00007fff`4ecb9b68 00000000`00000000 : ntdll!RtlExitUserProcess+0xb8

02 00000042`558ffb70 00007fff`4ec1ae38 : 00000000`00000000 00000000`00000000 00000042`558ffbf8 00007fff`4ecebc20 : KERNEL32!ExitProcessImplementation+0xa

03 00000042`558ffba0 00007fff`4ec186ef : 00000000`00000000 00000000`00000000 00000000`00000000 00000042`558ffbf0 : ucrtbase!exit_or_terminate_process+0x44

04 00000042`558ffbd0 00007ff6`fa852235 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000001 : ucrtbase!common_exit+0x6f

05 00000042`558ffc20 00007fff`4f6f7bd4 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : PCManager!Compression::Compression+0x3ba5

06 00000042`558ffc60 00007fff`5150ced1 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : KERNEL32!BaseThreadInitThunk+0x14

07 00000042`558ffc90 00000000`00000000 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : ntdll!RtlUserThreadStart+0x21

```

看到exit进程，回溯到调用位置，看看做了什么判断。

```
.text:000000014003221E                 call    WinMain

.text:0000000140032223                 mov     ebx, eax

.text:0000000140032225                 call    __scrt_is_managed_app

.text:000000014003222A                 test    al, al

.text:000000014003222C                 jnz     short loc_140032235

.text:000000014003222E                 mov     ecx, ebx        ; Code

.text:0000000140032230                 call    exit

.text:0000000140032235 ; ---------------------------------------------------------------------------

.text:0000000140032235

.text:0000000140032235 loc_140032235:                          ; CODE XREF: __scrt_common_main_seh(void)+120↑j

.text:0000000140032235                 test    dil, dil

```

是在WinMain中正常退出的，所以判断就是在WinMain中。在IDA翻看一下，很容就找到了重要提示。

![12.png](https://bbs.kanxue.com/upload/attach/202001/394321_KKJNHYKPZHTEAEJ.jpg)

在往上看，正主来了。
