
发现该函数会查询注册表是否有`DirectoryServiceExtPt`值我的域控是2016的看了一下没有这个值

![](https://bbs.kanxue.com/upload/attach/202109/931257_ZGQR986WWNWKFBK.jpg)

接下来会继续尝试加载该文件由于没有所以走下面的else返回`0xc00000BB`

![](https://bbs.kanxue.com/upload/attach/202109/931257_VWDKPHWRARYYN59.jpg)

通过微软官方文档查询报错值发现是不支持该请求类型的错误`STATUS_NOT_SUPPORTED`，keberos认证流程中客户端会在`AS-REQ`中声明自己支持的所有加密类型`etype`，域控会并检查客户端是否支持`AES`如果客户端支持则域控会在`AS-REP`的`PA-ETYPE-INFO2`中响应对应的加密方式。但如果客户端支持的加密类型中有`AES`，但`DC`并没有回应对应的`AES`加密类型，并且报错类型是`STATUS_NOT_SUPPORTED`很大可能受到了`skeleton`攻击

![](https://bbs.kanxue.com/upload/attach/202109/931257_VT4377ZQ5TS66JP.jpg)

我这里测了一下`skeleton`攻击并抓取了流量包，客户端发送的`AS-REQ`请求存在`AES`加密

![](https://bbs.kanxue.com/upload/attach/202109/931257_Y7KFUEDYCJJ42QF.jpg)

但是域控的响应中却没有`AES`加密同时确实是报错`STATUS_NOT_SUPPORTED`

![](https://bbs.kanxue.com/upload/attach/202109/931257_9TEZ44387268FPY.jpg)

我们继续看下`kdcsvc.dll`的流程在域控通过`kdcgetuserskey`函数接收到用户的密码后不单单是会走`SamIRetrieveMultiplePrimaryCredentials`函数同时会将密码传递给`kerbhashpassword`函数最终调用`cryptdll.dll`的导出函数`CDLocateCSystem`选择对应的加密系统对用户传入的凭据进行处理

![](https://bbs.kanxue.com/upload/attach/202109/931257_BPV2JTQAYCWBMD9.jpg)

![图片]()

接下来就可以看到`mimikatz`继续利用`kull_m_process_getVeryBasicModuleInformationsForName`函数也是通过`pebldr`从`lsass`进程中获取`cryptdll.dll`的模块信息，同时获取了该`dll`的句柄保存在`localaddr`中

![](https://bbs.kanxue.com/upload/attach/202109/931257_QRJJQZMFYUZ9W4N.jpg)

然后通过`CDLocateCSystem`函数找到`RC4`类型的加密系统存储在`pCrypt`结构体中，可以看到该加密类型的一系列函数存储在结构体中

![](https://bbs.kanxue.com/upload/attach/202109/931257_GT4QVDNSSAD4YX6.jpg)

接下来给最开始定义的`extensions`赋值，将`cryptdll.dll!rc4HmacInitialize`，`cryptdll.dll!rc4HmacDecrypt`函数地址传入

![](https://bbs.kanxue.com/upload/attach/202109/931257_8BZHRZSNJERK5KN.jpg)

接下来`kull_m_remotelib_CreateRemoteCodeWitthPatternReplace`函数中传入了几个函数名其实是传入函数的地址
![](https://bbs.kanxue.com/upload/attach/202109/931257_WBGFJQPG3HNRCMJ.jpg)

![](https://bbs.kanxue.com/upload/attach/202109/931257_5UJGS9U79C9WZBE.jpg)

第一个参数是`lsass`进程的内存属性，第二个是`rc4_init`函数的地址，第三个参数`rc4_end`函数地址-`rc4_init`函数地址代表的是`rc4_init`到`rc4_end`之间的内存内容其实这块空间包含了`init`函数`decrypt`两块函数内容，第四个参数是存储`extensions`的结构体指针，第五个参数是`lsass`进程的指针

![](https://bbs.kanxue.com/upload/attach/202109/931257_R5QJKUGGCE7Z2S4.jpg)

跟进后首先通过`kull_m_remotelib_GetProcAddressMultipleModules`函数这个函数主要也是调用`kull_m_process_getVeryBasicModuleInformations`函数通过`PEBldr`获取模块信息，给`RemoteExt`结构体也就是`extensions`结构体数组填充对应的值

![](https://bbs.kanxue.com/upload/attach/202109/931257_G6KV3ABZUANTMNU.jpg)

可以看到被填充后的数据如下

![](https://bbs.kanxue.com/upload/attach/202109/931257_6KBJHSVCEJHGA5X.jpg)

这里是新创了个空间`aLocalAddr`大小跟`rc4_end-rc4_init`函数地址相距的大小一致,并把`rc4_end-rc4_init`函数地址内存的内容拷贝到新空间中，并将`extensions[j].ToReplace`的值在该空间中查找其对应的位置，后续用`extensions[j].Point`的值替换该值

![](https://bbs.kanxue.com/upload/attach/202109/931257_HT8V74Y8CBYR8RN.jpg)

我们可以看到`rc4_init`函数到`rc4_end`函数内存空间中还有`rc4_init_decrypt`函数其中`rc4_init`函数与`rc4_decrypt`函数中有很多函数不过名字为`0x4a4a`这种一串奇怪的数字，同时我们前面被填充好的`extensions`结构体数组也存在这些奇怪的数字并且存在正常的模块名称及其对应的函数名以及函数对应的地址，就是通过内存搜索这些特殊字符然后进行对应的替换。

![](https://bbs.kanxue.com/upload/attach/202109/931257_G9E9H6WHJAK636N.jpg)

![](https://bbs.kanxue.com/upload/attach/202109/931257_KVVDT7RUQCMY5SC.jpg)

替换后这些函数内容应该是这样的

![](https://bbs.kanxue.com/upload/attach/202109/931257_8CQPT2VXMXXFGKC.jpg)![图片]()

该空间内存内容替换为正常的函数内容后使用`kull_m_memory_alloc`函数在`lsass`进程中开辟了一块可读可写可执行的内存空间并且大小也是函数`rc4_int`到函数`rc4_end`内存空间的大小并通过`kull_m_memory_copy`函数将该内容拷贝`lsass`进程中拷贝到新开辟的空间中

![](https://bbs.kanxue.com/upload/attach/202109/931257_56J5224Z9RE7E4K.jpg)

可以看到下图左边是`lsass`进程中新开辟的内存空间的内容与我们之前函数空间的内容一致。新开辟的地址是`000001f3d4fb0000`

![](https://bbs.kanxue.com/upload/attach/202109/931257_463864276V9QHDZ.jpg)

所以函数`kull_m_remotelib_CreateRemoteCodeWitthPatternReplace`主要的目的就是将`rc4_int,rc4_decrypt`函数修复后将代码注入到`lsass`空间中，后续通过`kull_M_memory_copy`函数将原地址`7ffa3cabf628`指向的是`cryptdll!rc4HmacInitialize`函数的指针替换为了`0xc79bcdfd88`地址中存储的`8`字节的值也就是`01f3d4fb0000`,这块地址指向的我们自己定义的`kuhl_misc_skeleton_rc4_init`函数的地址也就是`hook`了原始的`rc4HmacInitialize`函数

![](https://bbs.kanxue.com/upload/attach/202109/931257_3TTW65DDTAQSD98.jpg)

从下图可以看到该地址处替换前地址是`00007ffa3cab84c0`，右侧为替换后

![](https://bbs.kanxue.com/upload/attach/202109/931257_ZNP242EG8FVKYQS.jpg)![](https://bbs.kanxue.com/upload/attach/202109/931257_FWNX5YS3JEQFEPE.jpg)

原始的`RC4_init`函数被`hook`后，后续又开始计算自己定义的`rc4_decrypt`函数相较于`rc4_init`函数的偏移，然后`ptrValue`加上偏移就是指向我们自定义`rc4_decrypt`函数的地址指针,`aLsass.address`也通过偏移计算得到`lsass`进程中原始的`rc4_decrypt`函数的地址的指针，然后通过`kull_m_memory_copy`函数将我们的`rc4_decrypt`函数替换掉原始的

![](https://bbs.kanxue.com/upload/attach/202109/931257_QFP3CT85TAQZGSF.jpg)

可以看到又是调用`WriteProcessMemory`函数将原地址`7ffa3cabf638`指向的是`cryptdll!rc4HmacDecrypt`函数的指针替换为了`0xc79bcdfc20`地址中存储的`8`字节的值也就是`01f3d4fb016c`,这块地址指向的我们自己定义的`kuhl_misc_skeleton_rc4_init_decrypt`函数的地址也就是`hook`了原始的`rc4HmacDecrypt`函数

![](https://bbs.kanxue.com/upload/attach/202109/931257_W3EAX55BWMYG8YJ.jpg)

从下图我们可以看到`hook`前的情况`7ffa3cabf638`存的是原始的`rc4HmacDecrypt`函数

![](https://bbs.kanxue.com/upload/attach/202109/931257_4UZ7AYM6TEKZJ8F.jpg)

到这里`skeleton`模块函数已经全部分析完，接下来我们来分析一下我们自定义的`rc4`初始化函数，在`rc4init`函数中首先有个`kiwiKey`数组我们的认证都是传输的`hash`这个数组正好也是`32`位很有可能就是`skeleton`的默认密码`mimikatz`的`hash` 通过计算`mimikatz`的`hash`发现这里确实就是存的`skeleton`的密码`hash`

![](https://bbs.kanxue.com/upload/attach/202109/931257_BYBUTG527U2XRX5.jpg)![](https://bbs.kanxue.com/upload/attach/202109/931257_WZXDDRPH76ZHK7N.jpg)

通过`LocalAlloc`分配了内存指向`pContext`,将`Key`(用户的输入的原始密码`hash`的地址)传入`rc4HmacInitialize`函数中生成原始密码的秘钥流完成后续的`rc4`加密解密的操作，将生成的秘钥流拷贝到开始分配的空间中大小为`16`字节，接着将`kiwiKey`(`mimikatz`字符串的`hash`)传入`rc4HmacInitialize`函数中生成`mimikatz`的秘钥流，接着又继续拷贝到开始分配的空间中大小为`16`字节，最后将原始的密码`hash`地址`Key`也拷贝到`pContext`空间中

![](https://bbs.kanxue.com/upload/attach/202109/931257_AWRVCYGW9KDYX9P.jpg)

我们给`rc4HmacInitialize`函数下断点域控输入`net use \\win2016\c$/user:"test.com\administrator" "mimikatz"`，断下来后我们执行完`*(LPCVOID *) ((PBYTE) *pContext + 32)= Key`这段代码就可以看到`pContext`存储的内容了也就是上图反汇编中`rax`指向的地址，我们查看`dq rax` 第一排是原始密码生成的秘钥流，第二排是`mimikatz`生成的秘钥流，第三排前`8`字节是我们输入的原始密码`hash`的地址查看该地址的内容发现就是`mimikatz`的`hash`，这里由于我们的原始密码输入的就是`mimikatz`所以第一排第二排的秘钥流一致。

![](https://bbs.kanxue.com/upload/attach/202109/931257_6KUDZKB5PWZ72XA.jpg)

由于`rc4`的对称算法加密与解密是一致的都是用上面的秘钥流，解密函数中传入了开始`init`函数中的`pContext`以及被加密的数据，首先函数里面也是先分配了一段空间`buffer`，将待解密的数据拷贝到`buffer`中，将`buffer`以及`pContext`传入`rc4HmacDecrypt`函数中进行解密，这里是利用第一次的秘钥流解密如果解密成功的话就返回了,如果解密失败会再次调用`rc4HmacDecrypt`函数使用第二次我们设置的`mimikatz`生成的秘钥流解密如果成功就会改变`pContext`存在的原始`Key`改成我们设置的`kiwikey`的值，这样的方式就保证了我们无论是输入正确的密码还是我们设置的`kiwikey`来走`rc4`初始化加密解密流程都是可以通过的。

![](https://bbs.kanxue.com/upload/attach/202109/931257_2AMN5QSB65N82J6.jpg)![](https://bbs.kanxue.com/upload/attach/202109/931257_MSD3PF9FY5MJK9U.jpg)

自定义`kiwikey:zhuzhuxiaoba`重新编译后认证成功，不过当前的`mimikatz`的`skeleton`万能密码只支持`kerberos`流程，其实在`ntlm`认证流程中也是可以实现万能钥匙的早期的万能钥匙的恶意样本就是这样操作的。通过在`MSV1_0.dll`中定位函数`MsvpSamValidate`，`MsvpSamValidate`函数中有一个对`MsvpPasswordValidate`函数的调用，通过`hook`该函数，同样的可以实现万能密码的功能。`github`上有个类似的利用笔者在`win10`系统上测了一下确实可以利用，不过在`server`上未能复现

![](https://bbs.kanxue.com/upload/attach/202109/931257_9GFD4EZQQUQZDG6.jpg)

## 3.防御方案

由于该攻击是对`lsass`进程进行了注入， 从`Windows 8.1`（和`Server 2012 R2`）开始，`Microsoft`引入了一项称为`LSA`保护的功能。此功能基于`PPL`技术，它是一种纵深防御的安全功能，旨在“防止非管理员非`PPL`进程通过打开进程之类的函数串改`PPL`进程的代码和数据”。防止对进程`lsass.exe` 的代码注入，这样一来就无法使用 `mimikatz` 对 `lsass.exe` 进行注入，相关操作也会失败。在注册表`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa`

添加`RunAsPPL=dword:00000001`，重启就可以开启了`PPL`保护了

![](https://bbs.kanxue.com/upload/attach/202109/931257_AADQ9UFWY68JJXE.jpg)

开启后`mimikatz`注入`lsass`进程的行为将会失败，虽然`mimikatz`后面又增加了一个`mimidrv.sys`驱动来绕过`PPL`保护，但是加载驱动这一行为已经有明显的日志可以供我们进行检测。

`windows`日志`4697`记录了`mimidrv.sys`驱动的安装

![](https://bbs.kanxue.com/upload/attach/202109/931257_N8TP2SAG2FFA5PH.jpg)

`sysmon`日志`13`可以很明显看到`mimidrv`服务以及对应的驱动程序

![](https://bbs.kanxue.com/upload/attach/202109/931257_8DG7DPKBGS3983H.jpg)

我们可以使用`zBang` 工具扫描当前的域控环境是否已经被注入了万能密码。

`(f0dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0P5h3u0W2M7X3q4J5K9#2)9J5c8Y4A6n7j5h3&6Y4i4@1g2r3i4@1u0o6i4K6R3&6i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6W2n7i4@1p5H3i4@1f1@1i4@1t1^5i4@1u0m8f1$3E0W2L8r3g2@1L8$3^5`. Key 是注入恶意代码到 `lsass.exe` 进程的，所以它只存在于内存中，如果域控制器重启，注入的 Skeleton Key 将会失效。

![](https://bbs.kanxue.com/upload/attach/202109/931257_WPES528N6BRC86R.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
