
# 谜----同益起名大师 3.36 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-462.htm`](https://www.pediy.com/kssd/pediy07/pediy7-462.htm)

【软件名称】：同益起名大师 V3.36  

【下载页面】：`http://www.skycn.com/soft/109.html`  

【软件简介】：是一个专业的起名测名软件。  

【调试环境】：`WinXP`、`flyODBG`、`regedit`、Windows 自带的计算器  

【作者声明】：只为技术而破解，试图探讨一下此类软件的解密方法。失误之处敬请诸位大侠赐教！

-------------------------------------------------------

## 1. 脱壳

壳为 `ACProtect`，脱壳方法见 `fly` 的文章，感谢 `fly` 大侠提供脱壳：`http://bbs.pediy.com/showthread.php?s=&threadid=13137`

## 2. 寻找切入点

脱壳后的程序中关于注册的关键代码段还是带壳的（这就是 SDK?），要想爆破，还得把这部分代码脱出来。可这个程序又是多点校验，想要把所有的关键段都脱出来，可不是件容易事。因此破解它还是要跟注册码。注册信息保存在下面注册表项下：

`HKEY_LOCAL_MACHINE\SOFTWARE\GoodSoft\GoodName`

其中有一项 `Appid`，程序在发现有人为对注册表改动后，会把 `Appid` 的值改为 `4D`，然后程序就不能注册了，发生这种情况后可以用 `regedit` 手动把这一项改为 `0`，就又可以注册了。设置 OD 忽略所有异常，载入程序后，先下断：

```
bp RegQueryValueExA
```

然后运行程序，中断后看堆栈第三行，不出现 `Appid` 就 `F9` 运行，出现 `Appid` 后，关闭 `RegQueryValueExA` 这个断点，`Alt+F9` 返回，再经过两个 `ret`，来到这里：

```
005735C6  mov edx,UnPacKed.00573AE0           ; ASCII "Appid"
005735CB  mov eax,dword ptr ss:[ebp-14]
005735CE  call UnPacKed.0043D0EC
005735D3  mov dword ptr ss:[ebp-8],eax  <====返回这里
005735D3  mov dword ptr ss:[ebp-8],eax
005735D6  lea ecx,dword ptr ss:[ebp-C]
005735D9  mov edx,UnPacKed.00573AF0           ; ASCII "Serial"
005735DE  mov eax,dword ptr ss:[ebp-14]
005735E1  call UnPacKed.0043D060
005735E6  lea ecx,dword ptr ss:[ebp-10]
005735E9  mov edx,UnPacKed.00573B00           ; ASCII "FName"
005735EE  mov eax,dword ptr ss:[ebp-14]
005735F1  call UnPacKed.0043D060
005735F6  lea edx,dword ptr ss:[ebp-24]
005735F9  mov eax,dword ptr ss:[ebp-10]
005735FC  call UnPacKed.00408B60  <====从执行前后内存中数据可以看出，这个 call 是复制字符串
```

这里就是启动的验证段，但这里没有对注册码的完整的校验，在点击“注册”以后有完整的校验，因此要找到点击“注册”以后的代码。算注册码的时候离不开复制字符串的操作，因此下断：`bp 00408B60`，可是调用得太多，仔细观察了一下，发现其它调用点一般都在 `00500000` 以下，所以把 `00408B60` 的断点加上条件：`[esp]>00500000`，找到了点“注册”以后的代码段：

```
0055A090  lea edx,dword ptr ss:[ebp-20]
0055A093  mov eax,dword ptr ss:[ebp-4]
0055A096  mov eax,dword ptr ds:[eax+32C]
0055A09C  call UnPacKed.004569A8 取序列号
0055A0A1  mov eax,dword ptr ss:[ebp-20]
0055A0A4  lea edx,dword ptr ss:[ebp-1C]
0055A0A7  call UnPacKed.00408B60 复制序列号
0055A0AC  mov eax,dword ptr ss:[ebp-1C]
0055A0AF  call UnPacKed.00404584 查序列号位数
0055A0B4  cmp eax,0A      这里要求序列号大于 10 位
0055A0B7  jge short UnPacKed.0055A0C1

.............

0055A2F2  mov eax,dword ptr ss:[ebp-4]
0055A2F5  call UnPacKed.005565EC  跟进后有解压缩代码，估计是关键代码。
```

跟进 `call UnPacKed.005565EC`，经过下面一段后，就是解压缩啦，压缩方式仍然是 `ACProtect`，施展跟踪 `ACProect` 的本领来跟吧：

```
00556616  call UnPacKed.004767E8  这里不用解压，可在此处下一个断点
0055661B  pushad
```

## 3. 代码粗跟踪

跟踪 `ACProtect`，大家可以各显其才。我提供一种方法供参考，其实这是我刚发现的方法，我在一开始跟踪时也不知道这个方法，我跟踪时用的方法要比这个费劲得多。

这个软件嵌入的解码有个特点，都有一次 `int3` 异常，这使得硬件断点不能直接使用。但也不是绝对地不能用，要在发生 `int3` 异常以后再用才行。对于此例，到达 `00556616` 处，`esp=12f8b8`。设置 OD 不忽略“`int3` 中断”和“同时忽略以下指定的异常或者异常范围”，其余全忽略。`F9` 运行程序，在 `00558058` 处发生异常，这时下：`hr 12f8b4`，`F9` 运行，第 3 次中断时解码完毕，断下后取消硬件断点。其余地方可按此法处理，不再赘述。

对点“注册”以后的代码粗跟踪如下：

```
00559AEC  call UnPacKed.0050A150  根据系统算出申请码

........................

00559B5B  mov edx,dword ptr ss:[ebp-38]  对话框中的申请码
00559B5E  mov eax,dword ptr ss:[ebp-C]  根据系统算出的申请码
00559B61  call UnPacKed.004046D0  两个申请码比较
00559B66  jnz UnPacKed.00559ED7

................

00559C41  lea eax,dword ptr ss:[ebp-70]
00559C44  push eax
00559C45  lea edx,dword ptr ss:[ebp-74]
00559C48  mov eax,dword ptr ss:[ebp-C]
00559C4B  call UnPacKed.00408B60  复制申请码
00559C50  lea eax,dword ptr ss:[ebp-74]
00559C53  mov edx,UnPacKed.00559FD8           ; ASCII "        "
00559C58  call UnPacKed.0040458C  申请码后面加空格
00559C5D  mov eax,dword ptr ss:[ebp-74]
00559C60  mov ecx,8
00559C65  mov edx,1
00559C6A  call UnPacKed.004047E4  取前 8 位
00559C6F  lea eax,dword ptr ss:[ebp-70]
00559C72  push eax
00559C73  lea edx,dword ptr ss:[ebp-7C]
00559C76  mov eax,dword ptr ds:[esi+300]
00559C7C  call UnPacKed.004569A8  从对话框中取姓
00559C81  mov eax,dword ptr ss:[ebp-7C]
00559C84  lea edx,dword ptr ss:[ebp-78]
00559C87  call UnPacKed.00408B60  复制姓
00559C8C  mov edx,dword ptr ss:[ebp-78]
00559C8F  pop eax
00559C90  call UnPacKed.0040458C  把姓连在申请码之后
00559C95  mov eax,dword ptr ss:[ebp-70]
00559C98  lea edx,dword ptr ss:[ebp-6C]
00559C9B  call UnPacKed.004E00B8  加密（每位和 1F 异或）
00559CA0  mov edx,dword ptr ss:[ebp-6C]
00559CA3  mov eax,dword ptr ds:[57F108]
00559CA8  call UnPacKed.00404318  保存加密后申请码的地址
00559CAD  lea edx,dword ptr ss:[ebp-8C]
00559CB3  mov eax,dword ptr ds:[esi+32C]
00559CB9  call UnPacKed.004569A8  取序列号
00559CBE  mov eax,dword ptr ss:[ebp-8C]
00559CC4  lea edx,dword ptr ss:[ebp-88]
00559CCA  call UnPacKed.00408B60  复制序列号
00559CCF  mov eax,dword ptr ss:[ebp-88]
00559CD5  lea edx,dword ptr ss:[ebp-84]
00559CDB  call UnPacKed.00513FFC  序列号调位
00559CE0  mov eax,dword ptr ss:[ebp-84]
00559CE6  lea edx,dword ptr ss:[ebp-80]
00559CE9  call UnPacKed.004E00B8  加密（每位和 1F 异或）
00559CEE  mov edx,dword ptr ss:[ebp-80]
00559CF1  mov eax,dword ptr ds:[57EB9C]  保存地址
00559CF6  call UnPacKed.00404318
00559CFB  lea edx,dword ptr ss:[ebp-C]
00559CFE  mov eax,dword ptr ds:[57F108]
00559D03  mov eax,dword ptr ds:[eax]
00559D05  call UnPacKed.004FBC54  对由申请码和姓字符串变换 f1()得一字符串 A
00559D0A  lea eax,dword ptr ss:[ebp-94]
00559D10  push eax
00559D11  lea edx,dword ptr ss:[ebp-98]
00559D17  mov eax,dword ptr ds:[57EB9C]
00559D1C  mov eax,dword ptr ds:[eax]
00559D1E  call UnPacKed.004E00B8  对序列号的变换结果再变换，即还原出序列号
00559D23  mov eax,dword ptr ss:[ebp-98]
00559D29  mov ecx,5
00559D2E  mov edx,17      从 17H=23 位开始取 5 位
00559D33  call UnPacKed.004047E4
00559D38  mov eax,dword ptr ss:[ebp-94]
00559D3E  lea edx,dword ptr ss:[ebp-90]
00559D44  call UnPacKed.004ED010  对 23~27 位变换 f2()，得到 B
00559D49  mov edx,dword ptr ss:[ebp-90]  B
00559D4F  mov eax,dword ptr ss:[ebp-C]  正确值 A
00559D52  call UnPacKed.004046D0  比较
00559D57  jnz UnPacKed.00559ED7    关键之一
00559D5D  lea eax,dword ptr ss:[ebp-A0]
00559D63  push eax
00559D64  lea edx,dword ptr ss:[ebp-A4]
00559D6A  mov eax,dword ptr ds:[57EB9C]
00559D6F  mov eax,dword ptr ds:[eax]
00559D71  call UnPacKed.004E00B8
00559D76  mov eax,dword ptr ss:[ebp-A4]
00559D7C  mov ecx,5
00559D81  mov edx,17
00559D86  call UnPacKed.004047E4
00559D8B  mov eax,dword ptr ss:[ebp-A0]
00559D91  lea edx,dword ptr ss:[ebp-9C]
00559D97  call UnPacKed.004ED010
00559D9C  mov edx,dword ptr ss:[ebp-9C]
00559DA2  mov eax,dword ptr ss:[ebp-C]
00559DA5  call UnPacKed.004046D0  比较之二，与比较之一相同
00559DAA  jnz UnPacKed.00559E6A
00559DB0  lea eax,dword ptr ss:[ebp-A8]
00559DB6  push eax
00559DB7  lea edx,dword ptr ss:[ebp-AC]
00559DBD  mov eax,dword ptr ds:[57EB9C]
00559DC2  mov eax,dword ptr ds:[eax]
00559DC4  call UnPacKed.004E00B8
00559DC9  mov eax,dword ptr ss:[ebp-AC]
00559DCF  mov ecx,3
00559DD4  mov edx,1D
00559DD9  call UnPacKed.004047E4  取 29~31 位
00559DDE  mov eax,dword ptr ss:[ebp-A8]
00559DE4  call UnPacKed.004E9BE0  对 29~31 位变换
00559DE9  mov ebx,eax
00559DEB  lea eax,dword ptr ss:[ebp-B0]
00559DF1  push eax
00559DF2  lea edx,dword ptr ss:[ebp-B4]
00559DF8  mov eax,dword ptr ds:[57EB9C]
00559DFD  mov eax,dword ptr ds:[eax]
00559DFF  call UnPacKed.004E00B8
00559E04  mov eax,dword ptr ss:[ebp-B4]
00559E0A  mov ecx,1C
00559E0F  mov edx,1
00559E14  call UnPacKed.004047E4  取 1~28 位
00559E19  mov eax,dword ptr ss:[ebp-B0]
00559E1F  call UnPacKed.004F0748  对 1~28 位变换
00559E24  cmp ebx,eax      关键比较之三
00559E26  jnz UnPacKed.00559ED7
00559E2C  mov eax,UnPacKed.00559FF8                ;  ASCII "111"
```
