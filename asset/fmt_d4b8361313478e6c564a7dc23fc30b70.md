
# 单词大爆炸 2.0

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-370.htm`](https://www.pediy.com/kssd/pediy08/pediy8-370.htm)

`单词大爆炸 2.0` 采用 `sapi4` 的 TTS，不能调用 `NeoSpeech TTS`，因此做个补钉，提高发音质量。

这是 `Delphi` 编写的

思路：先编写一个调用 `sapi5` 的 DLL，主程序中传入要读的单词或句子，由 DLL 调用 `sapi5` 发音。由于原来也为奇迹智能英语增加 TTS 接口，奇迹智能是 `VB` 编写的，单词为 `UNICODE`，而单词大爆炸的单词为 `ascii`。因此在 DLL 文件中做了两个接口，一个是 `unicode` 用的 `speak`, 一个是 `ascii` 用的 `speaka`。

然后，屏蔽原软件中的 TTS 发音，加入自己的 DLL 调用。

首先要确定程序可 TTS 调用的代码在那里，用 `dreaman` 的 `OllyHTML` 插件中的“获取特定操作相关的过程列表”脚本, 对过程进行跟踪：即比较按下发音按钮前后过程的差别，就可以大致确定 TTS 调用代码位置，然后，一步一步跟踪，就可以找到了。

```
0045B792   .  8B55 E8       mov     edx, [ebp-18]  
0045B795   .  8BC3          mov     eax, ebx  
0045B797   .  E8 B4CDFAFF   call    00408550          跟踪到这里，执行后，eax 返回单词地址，这个可是有用的啊！      
0045B79C     8B7D F0       mov     edi, [ebp-10]    在这里，将程序跳转到自己的 DLL 调用位置：改为 jmp     005682AD  
0045B7A1     83C7 06       add     edi, 6                        
0045B7A2   .  85FF          test    edi, edi
```

继续跟踪 TTS 调用代码

```
0045B836   .  8B45 D8       mov     eax, [ebp-28]  
0045B839   .  E8 7686FAFF   call    00403EB4          跟踪到这个，执行后，eax 返回单词地址  
0045B83E              mov     [ebp-C],eax      单词地址放入[ebp-C]，用于下面代码中进行 TTS 发音, 改为 nop 后，TTS 就不发音了  
0045B841   .  33C0          xor     eax, eax  
....  
0045B87F   .  50            push    eax  
0045B880   .  8B00          mov     eax, [eax]  
0045B882     FF50 1C       call    [eax+1C]          调用 msttssyn.dll 进行发声，由于已将 0045B83E mov [ebp-C],eax 改为 nop，无单词传入，便不发声了  
```

下面是加入自己的 DLL 调用

```
00568290   .  74 74 6...   ascii   "ttd.dll"          自己的 DLL 文件名  
00568298     00           db      00  
00568299     00           db      00  
0056829A     00           db      00  
0056829B   .  73 70 ...    ascii   "speaka"            发声函数名  
005682A6     00           db      00                   用于存放 speaka 函数的地址  
005682AD     53           push    ebx                从 0045B79C 跳来，保存现场  
005682AE     50           push    eax                EAX 中的单词的址，保存现场  
005682AF     8B1D A6825600mov     ebx, [5682A6]     取 speaka 地址  
005682B5     90           nop  
005682B6     90           nop  
005682B7     83FB 00      cmp     ebx, 0              speaka 地址是否已有了？  
005682BA     75 2A       jnz     short 005682E6     已有，直接调用  
005682BC   .  68 90825600  push    00568290           speaka 地址为空，则要调用 LoadLibraryA，传送 DLL 文件名地址：ttd.dll  
005682C1   .  E8 26E2E9FF  call    <jmp.&kernel32.LoadLibraryA>      LoadLibraryA  
005682C6   .  83F8 00      cmp     eax, 0  
005682C9   .  74 22        je      short 005682ED     DLL 是否调用成功？  
005682CB   .  68 9B825600  push    0056829B           成功，传函数名地址 speaka  
005682D0   .  50           push    eax                传 DLL  hModule  
005682D1     E8 8EE1E9FF  call    <jmp.&kernel32.GetProcAddress>  取 speaka  
005682D6     A3 A6825600  mov     [5682A6], eax      将地址保存  
005682DB     83F8 00      cmp     eax, 0             
  
005682DE     74 0D       je      short 005682ED     speaka 是否有效  
005682E0     8B1D A6825600mov     ebx, [5682A6]      有效，将地址放在 ebx  
005682E6     58           pop     eax                弹出单词地址  
005682E7     50           push    eax                传单词地址  
005682E8     FFD3        call    ebx                调用 speaka 进行发音  
005682EA     EB 02       jmp     short 005682EE     
005682EC     90           nop  
005682ED     58           pop     eax  
005682EE     5B           pop     ebx  
005682EF     8B7D F0      mov     edi, [ebp-10]      恢复 0045B79C 0045B7A1 处修改为 JMP 005682AD 的代码  
005682F2     83C7 06      add     edi, 6  
005682F5    ^ E9 A834EFFF  jmp     0045B7A2           跳回原程序处  
```

修改完代码后，将代码保存时，却发生“在可执行文件中无法定位数据”的问题，这个问题也必须解决才行。

## 一、打开 PE Explorer，载入调试程序

## 二、修改 CODE 段的 RAW SIZE

1. 切换到 `Section headers`
2. 选中 `CODE` 段
3. 按下 `Section Editor`
4. 解锁
5. 将 `Size of Raw Data` 修改大一些，如将 `167200` 改为 `167400`，这样修改后文件空间就增大了。
6. 将 `CODE` 段设置为可写。因为程序中，要将取到的 `speaka` 的地址放在 `CODE` 段中的 `[5682A6]` 中，因此要让 `CODE` 段可写才行。当然也可以增加一个 `data` 段，用来存放地址，就可以不用修改这个属性了。

保存！

现在就可以在 `OD` 中增加代码并保存了。但注意的是，这样修改后，在最后的空白处还是不能保存的，但前面可保存空间已足够了

`4dd466d0e1e0cc123afbe504da2462ca`
