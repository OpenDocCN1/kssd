
# [原创]bang加固简单分析

自己的一个demo随手就上传加固了一下，然后开始分析,是免费版的，应该不少人已经分析过了

## dex

dex加固，可以使用`frida-dexdump`可以直接dump下来

![](https://bbs.kanxue.com/upload/tmp/803962_H2HXTMKXVDYHUC5.jpg)

可以看到加载了`SecShell`进行脱壳调用，这个`libSecShell.so`是32位的

## libSecShell.so

`export`列表中看到了`JNI_Onload`，但是是加密的，分析不出来，修改代码的话一定会调用`mprotect`，在`mprotect`处交叉引用，找不到调用，于是猜测可能是`svc`调用，用脚本跑了一下，发现了`mprotect`，脚本是之前论坛上看到的

```
system call : 7d 70

addr : c0783

Func Name : __NR_mprotect

 c0 70 a0 e3 00 00 00 ef
```

![](https://bbs.kanxue.com/upload/tmp/803962_52GEEF2BAFJ5H89.jpg)

在这里交叉引用发现都在`sub_C0C30`里调用

用`frida`去hook这个函数

```
var mprotect_cnt = 0

//frida -U --no-pause -f com.testlinker.ty -l hook.js

function sleep(delay) {

    var start = (new Date()).getTime();

    while ((new Date()).getTime() - start < delay) {

      continue;

    }

}

 

function hook_svc_mprotect() {

    let base_svc_mprotect = Module.findBaseAddress("libSecShell.so");

    if (base_svc_mprotect != null) {

        console.log("base_svc_mprotect : " + base_svc_mprotect)

    }else{

        return ;

    }

    let svc_mprotect = base_svc_mprotect.add(0xC0778);//32位

    Interceptor.attach(svc_mprotect, {

        onEnter: function(args) {

            console.log("==========================================")

           

            console.log("svc_mprotect: start = " + args[0] + " , len = " + args[1] + " , ATTRIBUTES = " + args[2])

            mprotect_cnt += 1

            console.log(hexdump(base_svc_mprotect.add(0x281B4)))

        },

        onLeave: function(){

            console.log("svc_mprotect leave")

            console.log("==========================================")

        }

    })

}

function dis(address, number) {

    for (var i = 0; i < number; i++) {

        var ins = Instruction.parse(address);

        console.log("address:" + address + "--dis:" + ins.toString());

        address = ins.next;

    }

}

//libc->strstr()  从linker里面找到call_function的地址

function hook() {

//call_function("DT_INIT", init_func_, get_realpath());

    var linkermodule

    if (Process.pointerSize == 4) {

        linkermodule = Process.findModuleByName("linker");

    }else if (Process.pointerSize == 8) {

        linkermodule = Process.findModuleByName("linker64");

    }

    // var linkermodule = Process.getModuleByName("linker");

    var call_function_addr = null;

    var symbols = linkermodule.enumerateSymbols();

    for (var i = 0; i < symbols.length; i++) {

        var symbol = symbols[i];

        //LogPrint(linkername + "->" + symbol.name + "---" + symbol.address);

        if (symbol.name.indexOf("__dl__ZL13call_functionPKcPFviPPcS2_ES0_") != -1) {

            call_function_addr = symbol.address;

            //LogPrint("linker->" + symbol.name + "---" + symbol.address)

        }

    }

    Interceptor.attach(call_function_addr, {

        onEnter: function (args) {

            var type = ptr(args[0]).readUtf8String();

            var address = args[1];

            var sopath = ptr(args[2]).readUtf8String();

            console.log("loadso:" + sopath + "--addr:" + address + "--type:" + type);

            if (sopath.indexOf("libSecShell.so") != -1) {

                var libnativemodule = Process.getModuleByName("libSecShell.so");//call_function正在加载目标so，这时就拦截下来

                var base = libnativemodule.base;

                hook_svc_mprotect()

            }

        }

    })

}

function main() {

    hook();

}

setImmediate(main)
```

可以发现经过`mprotect`一次后，对应地址的值发生了变化

```
[Pixel 3::com.example.cryptotest ]-> 

base_svc_mprotect : 0xcfaea000

==========================================

svc_mprotect: start = 0xcfaea000 , len = 0xa1000 , ATTRIBUTES = 0x7

           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF

cfb121b4  9b 66 a6 75 82 ab ba fb 1a 80 e6 75 d7 0e 7f 1b  .f.u.......u....

svc_mprotect leave

==========================================

==========================================

svc_mprotect: start = 0xcfb8b000 , len = 0x1f000 , ATTRIBUTES = 0x3

           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF

cfb121b4  2d e9 f0 4f ad f6 ac 4d df f8 44 4e df f8 44 3e  -..O...M..DN..D>

svc_mprotect leave

==========================================

==========================================

svc_mprotect: start = 0xcfaea000 , len = 0xa1000 , ATTRIBUTES = 0x7

           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF

cfb121b4  2d e9 f0 4f ad f6 ac 4d df f8 44 4e df f8 44 3e  -..O...M..DN..D>

svc_mprotect leave

==========================================
```

用`memdumper64`（github上有，速度挺快） dump出`so`，用`Sofixer`修复`so`文件

打开跳转到`JNI_Onload`（`0x1E5DC`）

发现`ida`没有自动创建函数，按`p`会报错`The function has undefined instruction/data at the specified address`

用`idapython`强制创建函数

```
ida_funcs.add_func(0x281B4,0x2A5CC)
```

随便打开一个函数，发现是这样的

![](https://bbs.kanxue.com/upload/tmp/803962_FBHU4AMV4374UXR.jpg)

```
.data:00085E24 DD F9 97 E4 off_85E24 DCD 0xE497F9DD ; DATA XREF: sub_DF08+8↑r
```

`cat /proc/18395/maps | grep e49`看一下这个地址

![](https://bbs.kanxue.com/upload/tmp/803962_AFFEUSP7JUC6844.jpg)

发现是`libc.so`，把这个`libc.so`拖出来放到`ida`分析

计算一下`0xE497F9DD-0xe494b000 = 0x349dd`

看一下`libc.so`，所以这个函数就是`strcpy`

![](https://bbs.kanxue.com/upload/tmp/803962_6XXZABCN7M34UNP.jpg)

**感觉可以写一个`idapython`脚本去修复一下**

然后就写了一下，先从`libc.so`中提取函数地址和函数名

```
from idautils import *

from idaapi import *

from idc import *

f = open("./func.txt",'w')

for func_addr in Functions(0,0x5B18BC):

    func_name = get_func_name(func_addr)

    print(func_addr , func_name)

    f.write(str(func_addr) + "," + func_name + "\n")

    # f.writelines()

f.close()
```

效果：

![](https://bbs.kanxue.com/upload/tmp/803962_37VQD3RCU6RA3A8.jpg)

然后从`.data`段中找到相应地址，相减得到`libc.so`中地址的偏移，然后对应起来，去修改函数名

```
from idautils import *

from idaapi import *

from idc import *

f = open(r"CryptoTest_32\CryptoTest\lib\func.txt",'r')

func_info = {}

while True:

    info = f.readline().strip('\n')

    if not info:

        break

    addr, func_name = info.split(',')

    # print(addr + func_name)

    func_info[int(addr,10)] = func_name

# print(func_info)

f.close()

textStart = 0xA2984

textEnd = 0xC2000

# textStart = 0xA2DE0

# textEnd = 0xA2E04

libc_dump_base = 0xe494b000

for i in range(textStart,textEnd,4):

    dword_ = get_dword(i)

    if dword_ > libc_dump_base:

        libc_func = dword_ - libc_dump_base

        # print(dword_,libc_func)

        func_name = func_info.get(libc_func)

        if not func_name:

            func_name = func_info.get(libc_func-1)  #thumb

        if not func_name:

            continue

        raw_name_off = get_name(i)

        patch_name_off = func_name + "_ptr_" + raw_name_off

        set_name(i,patch_name_off)

        xrefaddrs = XrefsTo(i, flags=0)

        for xrefaddr in xrefaddrs:

            raw_name = get_func_name(xrefaddr.frm)          #拿到函数原名称

            patch_fun_addr = get_name_ea_simple(raw_name)   #拿到函数地址

            # print(get_func_name(xrefaddr.frm))

            if raw_name and patch_fun_addr:

                break

        if raw_name and patch_fun_addr:

            patch_name = func_name + "_" + raw_name

            print("patch_name : ",patch_name)

            set_name(patch_fun_addr,patch_name)

        print(dword_,func_name)
```

效果如下：

![](https://bbs.kanxue.com/upload/tmp/803962_MFZ9HRR6QTJDE3A.jpg)

这样就容易分析得多，其实不止`libc.so`，还有`libdl.so`等，不过这个函数少，就手动恢复了

## init_array

地址：`0x11720`

![](https://bbs.kanxue.com/upload/tmp/803962_MFQHX2VJMYVAKDJ.jpg)

`sub_13E48`：打开`libc.so`，通过`dlsym`获取了`mprotect`、`mmap`、`munmap`、`fopen`、`fclose`、`fgets`、`fwrite`、`fread`、`sprintf`、`pthread_create`函数指针

接着跟着`frida`的log，程序运行到了`case 2`

流程是`case 2` -> `case 5` -> `case 4`（读`cmdline`） -> `case 1` -> `case 5` -> `case 4`循环读取

这里主要是记录包名的长度，存在`v8`里

![](https://bbs.kanxue.com/upload/attach/202207/803962_UTYR5UWASDK9RQ2.jpg)

最终执行到`case 0`，读包名，然后和`/system/bin/dex2oat`对比，这里我包名和`/system/bin/dex2oat`不匹配，不进入下面的步骤（这个过程看不懂它要干啥）

然后进入到`JNI_Onload`

## JNI_Onload

### 字符串解密

刚看到`JNI_Onload`，发现用了`sub_12B94`函数大量解密字符串，
