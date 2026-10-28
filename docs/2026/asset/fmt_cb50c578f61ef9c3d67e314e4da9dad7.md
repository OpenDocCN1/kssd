
因为调用它的前提是打开服务，而打开服务的前提是创建服务失败(因为已经存在)；

所以，我们要搞清楚，服务已经存在且已经运行后，再次打开服务又做了什么；

![image-20210419150445339](https://bbs.kanxue.com/upload/attach/202104/856790_G9KHJY3G2YGC8PW.jpg)

我们先看看这个字段的意思；

![image-20210419151201092](https://bbs.kanxue.com/upload/attach/202104/856790_76DWY2NP4GUJNP6.jpg)

![image-20210419151225719](https://bbs.kanxue.com/upload/attach/202104/856790_PW5B3TCN99URYUS.jpg)

给大哥们翻译翻译这个惊喜：请求服务停止；

这个逻辑就很清晰了，现在问题是这个服务到底干了嘛；

- `IDA`打开`Lab10-1.sys`；

    分析`dll`程序的时候，常说`DllMain`并不是真正的入口，只是在系统加载动态库的时候会加载这个函数进行必要的初始化后，再**跳转**到真正的程序入口；

    `sys`文件也是同样的道理，在`DriverEntry`的处是一些必要的初始化和检查，然后才**跳转**到真正的驱动入口；

    ![image-20210419151933037](https://bbs.kanxue.com/upload/attach/202104/856790_6PVEXFP3WWCS5QY.jpg)

    在这里，我们可以确认的是`sub_10906`才是真正的驱动程序入口，如果不放心，可以去上面的`call`看一下都干了啥；

- 我们看一下`sub_10906`；

    ![image-20210419152401257](https://bbs.kanxue.com/upload/attach/202104/856790_EH4877KVVN6XERH.jpg)

    这就结束了，就是把一个函数的偏移给了一个内存位置，既无调用也无跳转，看的有点迷糊，不慌，换个视角就相对容易理解了；

    ```c
    sub_10906(...){
        DriverObject->DriverUnload = (*)sub_10486;
        return 0;
    }
    ```

- 所以就是相当于把一个函数指针放在了一个服务里，让服务去执行；

    那就进`sub_10486`去瞅一瞅；

    ![image-20210419152856707](https://bbs.kanxue.com/upload/attach/202104/856790_AFQ9UKDPT4M3DAJ.jpg)

    然后就是一堆函数跟字符串，换个视角；

    ```c
    sub(...){
        RtlCreteRegistryKey(...);
        RtlCreteRegistryKey(...);
        RtlCreteRegistryKey(...);
        RtlCreteRegistryKey(...);
        RtlWriteRegistryValue(...);

        return RtlWriteRegistryValue(...);
    }
    ```

    在此，因为`Rtl`开头的是微软未公开的函数，我使用的MSDN并没有查到，而且使用某个大佬分享的[未公开函数查询网站](https://bbs.kanxue.com/elink@581K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2F1k6r3!0U0N6h3#2W2L8Y4c8W2k6q4)9J5k6h3&6@1K9h3&6@1k6i4u0F1j5h3I4K6i4K6u0W2L8X3g2@1i4K6u0r3)也没有查到；

    最后还是查询了[微软官方文档](https://bbs.kanxue.com/elink@0fbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1j5`.)，了解了一些功能细节；

    ```c
    RtlCreteRegistryKey(...); // 沿着给定的相对路径在注册表中添加一个键值对象。
    RtlWriteRegistryValue(...); // 调用方提供的数据沿着指定的相对路径在给定的值名处写入注册表。
    ```

    - 到这里，我们就需要关注每一次的注册表操作细节；

        分别是给一下的路径进行添加键对象：

        ```
        "\\Registry\\Machine\\SOFTWARE\\Policies\\Microsoft";
        "\\Registry\\Machine\\SOFTWARE\\Policies\\Microsoft\\WindowsFirewall";
        "\\Registry\\Machine\\SOFTWARE\\Policies\\Microsoft\\WindowsFirewall\\DomainProfile";
        "\\Registry\\Machine\\SOFTWARE\\Policies\\Microsoft\\WindowsFirewall\\StandardProfile";
        ```

    - 然后就是两次写入键值；

        在此直接给出代码观：

        ```c
        RtlWriteRegistryValue(
            0,            L"\\Registry\\Machine\\SOFTWARE\\Policies\\Microsoft\\WindowsFirewall\\DomainProfile",
            &ValueName,
            4u, // ; ValueType
            &ValueData,
            4u); // ; ValueLength
        ```

        我就很好奇，这个`ValueType`字段是啥意思；

        找到官网，找到字段描述：

        ![image-20210419155503841](https://bbs.kanxue.com/upload/attach/202104/856790_HZJ786729UDM7RQ.jpg)

        点进去，找到类型；

        ![image-20210419155619622](https://bbs.kanxue.com/upload/attach/202104/856790_JJUZSG3A8VA4FX3.jpg)

        到`VC++`中，输入字段名，右击找到定义；

        ![image-20210419155756283](https://bbs.kanxue.com/upload/attach/202104/856790_B4P3USQAZ5REPYJ.jpg)

        是`4`的有两个，所以只能猜测是描述32位系统了；

        *这是我使用的方式，如果有不对或者更好的，还请大佬赐教*；

到这里就很清晰了，这个函数做了很多关于注册表的操作，因为跟防火墙相关，所以我们可以合理猜测是禁用防火墙；

#### 问题

> 1、这个程序是否直接修改了注册表(使用`procmon`来检查)？

没有用`Procmon`工具，但是经过分析，是直接修改了注册表；

> 2、用户态的程序调用了`ControlService`函数，你是否能够使用`WinDbg`设置一个断点，以此来观察由于`ControlService`的调用导致内核执行了怎样的操作？

没有用`WinDbg`工具，但是经过分析，我们知道，是卸载操作；

> 3、这个程序做了些什么？

创建一个服务加载驱动，然后创建和写入了一些注册表项，用于禁用防火墙；

---

### Lab10-2

> 该实验的文件为`Lab10-02.exe`；

#### Lab10-2分析

1. 查壳；

    ![image-20210419160556465](https://bbs.kanxue.com/upload/attach/202104/856790_4PB957VUCQE5Z2Y.jpg)

    无壳；

2. 查看导入表；

    因为过长，就不全面截图了；

    ![image-20210419161031691](https://bbs.kanxue.com/upload/attach/202104/856790_WV7CXE9MBQMNQGK.jpg)

    首先依旧是服务相关的函数，其次是发现了资源相关的函数，所以我们有理由怀疑是否资源节中隐藏了什么秘密；

    ![image-20210419161203945](https://bbs.kanxue.com/upload/attach/202104/856790_Z5YB9HTQTBZGNNU.jpg)

    貌似加载了什么库，还申请了内存空间，并且获取程序地址；

3. 查看字符串；

    ![image-20210419161345744](https://bbs.kanxue.com/upload/attach/202104/856790_G9MBBGVQBMV25BQ.jpg)

    一个路径，和一个驱动服务名；

4. 过程分析；

    - 进入`IDA`看源代码；

        ![image-20210419161438751](https://bbs.kanxue.com/upload/attach/202104/856790_GNRMFNTDC8WPTWN.jpg)

        程序开始就加载了资源，所以我们还是使用`Resource Hacker`查看一下资源节；

        ![image-20210419161614374](https://bbs.kanxue.com/upload/attach/202104/856790_WFFB3Q4MFA2CDYF.jpg)

        基本可以确定里面隐藏了一个程序；

        保存到桌面，用`IDA`打开；

        ![image-20210419161812767](https://bbs.kanxue.com/upload/attach/202104/856790_CSVWRZHNU3AKTZ9.jpg)

        `IDA`已经自动识别这是一个驱动文件了；

        暂时不要着急分析这个驱动文件，我们先去看看原程序是如何加载这个驱动文件的；

    - 这一段主程序，不算复杂，依旧是使用代码表示结构；

        ```c
        int main(){
            if(FindResourceA(...)){
                hres = LoadResource(...);
                if(CreateFileA(...) != -1){
                    SizeofResource(...);
                    WriteFile(...,hres,...);
                    CloseHandle(...);
                    if(!OpenSCManagerA(...)){
                        printf(...);
                        return 0;
                    }
                    if(!CreateService(...)){
                        printf(..);
                        return 0;
                    }
                    if(!StartServiceA(...)){
                        printf(...);
                    }
                    CloseServiceHandle(...);
                }
            }
            return 0;
        }
        ```

        由此，我们可以了解到，先是从资源中去除，然后将其写入到路径为`C:\\Windows\\System32\\Mlwx486.sys`文件，然后就是依次的创建服务打开服务；

    - 既然如此，我们把重心放在从资源节导出的驱动文件上；

        - 先看一眼导入表；

            ![image-20210419163631154](https://bbs.kanxue.com/upload/attach/202104/856790_6F2NDSVD9XF58NX.jpg)

            都是内核操作；

        - 再看一眼字符串；

            ![image-20210419163729969](https://bbs.kanxue.com/upload/attach/202104/856790_3YPRKKQFP84APUS.jpg)

            暂时没看到什么亮点；

        - 过程分析；

            - 跳转`sub_10706`；

                ![image-20210419163853529](https://bbs.kanxue.com/upload/attach/202104/856790_5W9XT45W9U2AT92.jpg)

            - 这一段貌似跟我们之前做的有些区别，在最后将一个函数指针给某个内存空间前，貌似做了很多的工作，我们先理一下结构；

                ```c
                int sub_10706(...){
                    RtlInitUnicodeString(...);
                    RtlInitUnicodeString(...);
                    v2 = MmGetSystemRoutineAddress(...);
                    v3 = *(PVOID **)MmGetSystemRoutineAddress(...);
                    v4 = 0;
                    do
                    {
                        ++v3;
                        if (*v3 == v2 )
                            break;
                        ++v4;
                    }
                    while ( v4 < 284 );
                    dword_1068C = (int)v2;
                    dword_10690 = (int)v3;
                    *v3 = sub_10486;
                    return 0;
                }
                ```

                在此我们要先搞清楚这里面用到的内核函数是什么，使用了[微软官方文档](https://bbs.kanxue.com/elink@68dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1j5`.)；

                ```c
                RtlInitUnicodeString(...); // 将字符串转换成unicde类型；
                MmGetSystemRoutineAddress(...); // 返回一个指向SystemRoutineName指定的函数的指针;
                ```

                再看反汇编，先是把`N`和`KeServiceDescriptorTable`转换成`unicode`，然后再解析成函数指针；

                **转换成`unicode`好理解，因为在Windows系统内部的底层，都是先将`A`结尾的函数解析成`W`结尾的函数，也就是说，内核层应该都是使用宽字符的**；

                **至于返回函数指针我也是有点懵，看了书后答案才知道，是返回`N`和`KeServiceDescriptorTable`的地址偏移量偏移**；

                到此，对于`do{}while`语句的理解貌似卡克了，但是好在到目前为止，还不是我们要找的重点；

                在此直接借鉴书后答案的原话：*我们看到`DriverEntry`例程以参数`KeServiceDescriptorTable`和·`NtQueryDirectoryFile`调用了`RtlInitUnicodeString`函数，然后调用`MmGetSystemRoutineAddressPatchFunction`的地址覆盖这个项。*

            - 进而来分析`sub_10486`字段；

                还是先整理一下函数逻辑：

