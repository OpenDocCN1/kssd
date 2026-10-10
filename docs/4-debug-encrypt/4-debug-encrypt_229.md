

# 谜----同益起名大师 3.36 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-462.htm`](https://www.pediy.com/kssd/pediy07/pediy7-462.htm)

【软件名称】：同益起名大师 V3.36   
【下载页面】：http://www.skycn.com/soft/109.html  
【软件简介】：是一个专业的起名测名软件。  
【调试环境】：WinXP、flyODBG、regedit、Windows 自带的计算器  
【作者声明】：只为技术而破解，试图探讨一下此类软件的解密方法。失误之处敬请诸位大侠赐教！              
\-------------------------------------------------------  
【1. 脱壳】  
  
    壳为 ACProtect，脱壳方法见 fly 的文章，感谢 fly 大侠提供脱壳：http://bbs.pediy.com/showthread.php?s=&threadid=13137           
  
【2. 寻找切入点】  
  
    脱壳后的程序中关于注册的关键代码段还是带壳的（这就是 SDK?），要想爆破，还得把这部分代码脱出来。可这个程序又是多点校验，想要把所有的关键段都脱出来，可不是件容易事。因此破解它还是要跟注册码。注册信息保存在下面注册表项下：  
  
HKEY\_LOCAL\_MACHINE\\SOFTWARE\\GoodSoft\\GoodName  
  
    其中有一项"Appid"，程序在发现有人为对注册表改动后，会把"Appid"的值改为"4D"，然后程序就不能注册了，发生这种情况后可以用 regedit 手动把这一项改为 0，就又可以注册了。设置 OD 忽略所有异常，载入程序后，先下断：  
  
bp RegQueryValueExA  
  
    然后运行程序，中断后看堆栈第三行，不出现"Appid"就 F9 运行，出现"Appid"后，关闭 RegQueryValueExA 这个断点，Alt+F9 返回，再经过两个 ret，来到这里：  
  
005735C6  mov edx,UnPacKed.00573AE0           ; ASCII "Appid"  
005735CB  mov eax,dword ptr ss:\[ebp-14\]  
005735CE  call UnPacKed.0043D0EC    
005735D3  mov dword ptr ss:\[ebp-8\],eax  <====返回这里  
005735D3  mov dword ptr ss:\[ebp-8\],eax  
005735D6  lea ecx,dword ptr ss:\[ebp-C\]  
005735D9  mov edx,UnPacKed.00573AF0              ; ASCII "Serial"  
005735DE  mov eax,dword ptr ss:\[ebp-14\]  
005735E1  call UnPacKed.0043D060  
005735E6  lea ecx,dword ptr ss:\[ebp-10\]  
005735E9  mov edx,UnPacKed.00573B00              ; ASCII "FName"  
005735EE  mov eax,dword ptr ss:\[ebp-14\]  
005735F1  call UnPacKed.0043D060  
005735F6  lea edx,dword ptr ss:\[ebp-24\]  
005735F9  mov eax,dword ptr ss:\[ebp-10\]  
005735FC  call UnPacKed.00408B60  <====从执行前后内存中数据可以看出，这个 call 是复制字符串  
  
    这里就是启动的验证段，但这里没有对注册码的完整的校验，在点击“注册”以后有完整的校验，因此要找到点击“注册”以后的代码。算注册码的时候离不开复制字符串的操作，因此下断：bp 00408B60，可是调用得太多，仔细观察了一下，发现其它调用点一般都在 00500000 以下，所以把 00408B60 的断点加上条件：\[esp\]>00500000，找到了点“注册”以后的代码段：  
  
0055A090  lea edx,dword ptr ss:\[ebp-20\]  
0055A093  mov eax,dword ptr ss:\[ebp-4\]  
0055A096  mov eax,dword ptr ds:\[eax+32C\]  
0055A09C  call UnPacKed.004569A8  取序列号  
0055A0A1  mov eax,dword ptr ss:\[ebp-20\]  
0055A0A4  lea edx,dword ptr ss:\[ebp-1C\]  
0055A0A7  call UnPacKed.00408B60  复制序列号  
0055A0AC  mov eax,dword ptr ss:\[ebp-1C\]  
0055A0AF  call UnPacKed.00404584  查序列号位数  
0055A0B4  cmp eax,0A      这里要求序列号大于 10 位  
0055A0B7  jge short UnPacKed.0055A0C1  
.............  
0055A2F2  mov eax,dword ptr ss:\[ebp-4\]  
0055A2F5  call UnPacKed.005565EC  跟进后有解压缩代码，估计是关键代码。  
  
    跟进 call UnPacKed.005565EC，经过下面一段后，就是解压缩啦，压缩方式仍然是 ACProtect，施展跟踪 ACProect 的本领来跟吧：  
  
00556616  call UnPacKed.004767E8  这里不用解压，可在此处下一个断点  
0055661B  pushad  
  
【3. 代码粗跟踪】  
  
    跟踪 ACProtect，大家可以各显其才。我提供一种方法供参考，其实这是我刚发现的方法，我在一开始跟踪时也不知道这个方法，我跟踪时用的方法要比这个费劲得多。  
    这个软件嵌入的解码有个特点，都有一次 int3 异常，这使得硬件断点不能直接使用。但也不是绝对地不能用，要在发生 int3 异常以后再用才行。对于此例，到达 00556616 处，esp=12f8b8。设置 OD 不忽略“int3 中断”和“同时忽略以下指定的异常或者异常范围”，其余全忽略。F9 运行程序，在 00558058 处发生异常，这时下：hr 12f8b4，F9 运行，第 3 次中断时解码完毕，断下后取消硬件断点。其余地方可按此法处理，不再赘述。  
  
    对点“注册”以后的代码粗跟踪如下：  
  
00559AEC  call UnPacKed.0050A150  根据系统算出申请码  
........................  
00559B5B  mov edx,dword ptr ss:\[ebp-38\]  对话框中的申请码  
00559B5E  mov eax,dword ptr ss:\[ebp-C\]  根据系统算出的申请码  
00559B61  call UnPacKed.004046D0  两个申请码比较  
00559B66  jnz UnPacKed.00559ED7  
................  
00559C41  lea eax,dword ptr ss:\[ebp-70\]  
00559C44  push eax  
00559C45  lea edx,dword ptr ss:\[ebp-74\]  
00559C48  mov eax,dword ptr ss:\[ebp-C\]  
00559C4B  call UnPacKed.00408B60  复制申请码  
00559C50  lea eax,dword ptr ss:\[ebp-74\]  
00559C53  mov edx,UnPacKed.00559FD8           ; ASCII "        "  
00559C58  call UnPacKed.0040458C  申请码后面加空格  
00559C5D  mov eax,dword ptr ss:\[ebp-74\]  
00559C60  mov ecx,8  
00559C65  mov edx,1  
00559C6A  call UnPacKed.004047E4  取前 8 位  
00559C6F  lea eax,dword ptr ss:\[ebp-70\]  
00559C72  push eax  
00559C73  lea edx,dword ptr ss:\[ebp-7C\]  
00559C76  mov eax,dword ptr ds:\[esi+300\]  
00559C7C  call UnPacKed.004569A8  从对话框中取姓  
00559C81  mov eax,dword ptr ss:\[ebp-7C\]  
00559C84  lea edx,dword ptr ss:\[ebp-78\]  
00559C87  call UnPacKed.00408B60  复制姓  
00559C8C  mov edx,dword ptr ss:\[ebp-78\]  
00559C8F  pop eax  
00559C90  call UnPacKed.0040458C  把姓连在申请码之后  
00559C95  mov eax,dword ptr ss:\[ebp-70\]  
00559C98  lea edx,dword ptr ss:\[ebp-6C\]  
00559C9B  call UnPacKed.004E00B8  加密（每位和 1F 异或）  
00559CA0  mov edx,dword ptr ss:\[ebp-6C\]  
00559CA3  mov eax,dword ptr ds:\[57F108\]  
00559CA8  call UnPacKed.00404318  保存加密后申请码的地址  
00559CAD  lea edx,dword ptr ss:\[ebp-8C\]  
00559CB3  mov eax,dword ptr ds:\[esi+32C\]  
00559CB9  call UnPacKed.004569A8  取序列号  
00559CBE  mov eax,dword ptr ss:\[ebp-8C\]  
00559CC4  lea edx,dword ptr ss:\[ebp-88\]  
00559CCA  call UnPacKed.00408B60  复制序列号  
00559CCF  mov eax,dword ptr ss:\[ebp-88\]  
00559CD5  lea edx,dword ptr ss:\[ebp-84\]  
00559CDB  call UnPacKed.00513FFC  序列号调位  
00559CE0  mov eax,dword ptr ss:\[ebp-84\]  
00559CE6  lea edx,dword ptr ss:\[ebp-80\]  
00559CE9  call UnPacKed.004E00B8  加密（每位和 1F 异或）  
00559CEE  mov edx,dword ptr ss:\[ebp-80\]  
00559CF1  mov eax,dword ptr ds:\[57EB9C\]  保存地址  
00559CF6  call UnPacKed.00404318  
00559CFB  lea edx,dword ptr ss:\[ebp-C\]  
00559CFE  mov eax,dword ptr ds:\[57F108\]  
00559D03  mov eax,dword ptr ds:\[eax\]  
00559D05  call UnPacKed.004FBC54  对由申请码和姓字符串变换 f1()得一字符串 A  
00559D0A  lea eax,dword ptr ss:\[ebp-94\]  
00559D10  push eax  
00559D11  lea edx,dword ptr ss:\[ebp-98\]  
00559D17  mov eax,dword ptr ds:\[57EB9C\]  
00559D1C  mov eax,dword ptr ds:\[eax\]  
00559D1E  call UnPacKed.004E00B8  对序列号的变换结果再变换，即还原出序列号  
00559D23  mov eax,dword ptr ss:\[ebp-98\]    
00559D29  mov ecx,5  
00559D2E  mov edx,17      从 17H=23 位开始取 5 位  
00559D33  call UnPacKed.004047E4    
00559D38  mov eax,dword ptr ss:\[ebp-94\]  
00559D3E  lea edx,dword ptr ss:\[ebp-90\]  
00559D44  call UnPacKed.004ED010  对 23~27 位变换 f2()，得到 B  
00559D49  mov edx,dword ptr ss:\[ebp-90\]  B  
00559D4F  mov eax,dword ptr ss:\[ebp-C\]  正确值 A  
00559D52  call UnPacKed.004046D0  比较  
00559D57  jnz UnPacKed.00559ED7    关键之一  
00559D5D  lea eax,dword ptr ss:\[ebp-A0\]  
00559D63  push eax  
00559D64  lea edx,dword ptr ss:\[ebp-A4\]  
00559D6A  mov eax,dword ptr ds:\[57EB9C\]  
00559D6F  mov eax,dword ptr ds:\[eax\]  
00559D71  call UnPacKed.004E00B8  
00559D76  mov eax,dword ptr ss:\[ebp-A4\]  
00559D7C  mov ecx,5  
00559D81  mov edx,17  
00559D86  call UnPacKed.004047E4  
00559D8B  mov eax,dword ptr ss:\[ebp-A0\]  
00559D91  lea edx,dword ptr ss:\[ebp-9C\]  
00559D97  call UnPacKed.004ED010  
00559D9C  mov edx,dword ptr ss:\[ebp-9C\]  
00559DA2  mov eax,dword ptr ss:\[ebp-C\]  
00559DA5  call UnPacKed.004046D0  比较之二，与比较之一相同  
00559DAA  jnz UnPacKed.00559E6A  
00559DB0  lea eax,dword ptr ss:\[ebp-A8\]  
00559DB6  push eax  
00559DB7  lea edx,dword ptr ss:\[ebp-AC\]  
00559DBD  mov eax,dword ptr ds:\[57EB9C\]  
00559DC2  mov eax,dword ptr ds:\[eax\]  
00559DC4  call UnPacKed.004E00B8  
00559DC9  mov eax,dword ptr ss:\[ebp-AC\]  
00559DCF  mov ecx,3  
00559DD4  mov edx,1D  
00559DD9  call UnPacKed.004047E4  取 29~31 位  
00559DDE  mov eax,dword ptr ss:\[ebp-A8\]  
00559DE4  call UnPacKed.004E9BE0  对 29~31 位变换  
00559DE9  mov ebx,eax  
00559DEB  lea eax,dword ptr ss:\[ebp-B0\]  
00559DF1  push eax  
00559DF2  lea edx,dword ptr ss:\[ebp-B4\]  
00559DF8  mov eax,dword ptr ds:\[57EB9C\]  
00559DFD  mov eax,dword ptr ds:\[eax\]  
00559DFF  call UnPacKed.004E00B8  
00559E04  mov eax,dword ptr ss:\[ebp-B4\]  
00559E0A  mov ecx,1C  
00559E0F  mov edx,1  
00559E14  call UnPacKed.004047E4  取 1~28 位  
00559E19  mov eax,dword ptr ss:\[ebp-B0\]  
00559E1F  call UnPacKed.004F0748  对 1~28 位变换  
00559E24  cmp ebx,eax      关键比较之三  
00559E26  jnz UnPacKed.00559ED7  
00559E2C  mov eax,UnPacKed.00559FF8                ;  ASCII "111"  
00559E31  call UnPacKed.00506940  关键之四，校验 17~22 位  
00559E36  mov byte ptr ss:\[ebp-1\],al  
00559E39  cmp byte ptr ss:\[ebp-1\],0  
00559E3D  je short UnPacKed.00559E4C  
00559E3F  lea eax,dword ptr ss:\[ebp-C\]  
00559E42  mov edx,UnPacKed.00559FF8                ;  ASCII "111"  
00559E47  call UnPacKed.0040435C  
00559E4C  cmp byte ptr ss:\[ebp-1\],0  
00559E50  je UnPacKed.00559ED7  
00559E56  mov eax,dword ptr ss:\[ebp-C\]  
00559E59  mov edx,UnPacKed.00559FF8                ;  ASCII "111"  
00559E5E  call UnPacKed.004046D0  
00559E63  jnz short UnPacKed.00559E6A  
00559E65  call UnPacKed.00503320  关键之五，校验 1~16 位  
  
    跟踪时发现，根据系统计算申请码时用到了 cpuid 这个 API，可是在我的电脑上多次运行这个 API，返回值中 ebx 的值是在两个值中变的，因此算出的申请码也有两种，程序发现两个申请码不同也会通不过注册，因此跟踪时要注意一下 00559B66 这个地方。我觉得这也算是软件的一个 BUG。  
    对注册码验证前先对注册码调位，调用点如 00559CDB。调位的方法是第 1、4 位互换，第 5、8 位互换，4\*i+1 位和 4\*(i+1)位互换，i=0,1,2,...，末尾不足 4 位时，最后两位互换。以下所讲的除非特别说明的以外，均是调位以后的注册码，程序把注册码分成四段进行校验。  
  
【4. 对 23~27 位的校验】  
  
    其校验过程大致是：由申请码加上姓得到一个字符串，加密（每位和 1F 异或）后，再进行较复杂的运算 f1()，得到 A。注册码的 23~27 位进行另一种变换 f2()，得到 B，A 和 B 进行比较。f1()不必求逆，直接用结果就行。跟进 call UnPacKed.004ED010，看看 f2()是什么：  
  
004F026F  call UnPacKed.0040435C  
004F0274  mov eax,ebx  
004F0276  call UnPacKed.004042C4  
004F027B  mov esi,1  
004F0280  mov eax,edi  
004F0282  call UnPacKed.00404584  查位数  
004F0287  mov dword ptr ss:\[ebp-4\],eax  
004F028A  mov eax,dword ptr ss:\[ebp-4\]  
004F028D  mov ecx,5  
004F0292  cdq  
004F0293  idiv ecx  
004F0295  test eax,eax  
004F0297  jle UnPacKed.004F03AD  
004F029D  mov dword ptr ss:\[ebp-20\],eax  
004F02A0  lea eax,dword ptr ss:\[ebp-24\]  
004F02A3  mov dl,byte ptr ds:\[edi+esi-1\]  取出第一位  
004F02A7  call UnPacKed.004044AC  
004F02AC  mov eax,dword ptr ss:\[ebp-24\]  
004F02AF  mov edx,dword ptr ss:\[ebp-1C\]  
004F02B2  call UnPacKed.004048C8  查表 EBP-1C   0> 004F060C  ASCII "0KMT1EIJ2AB34FGH56PYZ7NRS89CDUVX"  
004F02B7  dec eax  
004F02B8  mov dword ptr ss:\[ebp-8\],eax    保存  
004F02BB  lea eax,dword ptr ss:\[ebp-28\]  
004F02BE  mov dl,byte ptr ds:\[edi+esi\]    取出第二位  
004F02C1  call UnPacKed.004044AC  
004F02C6  mov eax,dword ptr ss:\[ebp-28\]  
004F02C9  mov edx,dword ptr ss:\[ebp-1C\]  
004F02CC  call UnPacKed.004048C8  
004F02D1  dec eax  
004F02D2  mov dword ptr ss:\[ebp-C\],eax  
004F02D5  lea eax,dword ptr ss:\[ebp-2C\]  
004F02D8  mov dl,byte ptr ds:\[edi+esi+1\]  取出第三位  
004F02DC  call UnPacKed.004044AC  
004F02E1  mov eax,dword ptr ss:\[ebp-2C\]  
004F02E4  mov edx,dword ptr ss:\[ebp-1C\]  
004F02E7  call UnPacKed.004048C8    查表  
004F02EC  dec eax  
004F02ED  mov dword ptr ss:\[ebp-10\],eax    保存  
004F02F0  lea eax,dword ptr ss:\[ebp-30\]  
004F02F3  mov dl,byte ptr ds:\[edi+esi+2\]  取出第四位  
004F02F7  call UnPacKed.004044AC  
004F02FC  mov eax,dword ptr ss:\[ebp-30\]  
004F02FF  mov edx,dword ptr ss:\[ebp-1C\]  
004F0302  call UnPacKed.004048C8  
004F0307  dec eax  
004F0308  mov dword ptr ss:\[ebp-14\],eax  
004F030B  lea eax,dword ptr ss:\[ebp-34\]  
004F030E  mov dl,byte ptr ds:\[edi+esi+3\]  取出第五位  
004F0312  call UnPacKed.004044AC  
004F0317  mov eax,dword ptr ss:\[ebp-34\]  
004F031A  mov edx,dword ptr ss:\[ebp-1C\]  
004F031D  call UnPacKed.004048C8  
004F0322  dec eax  
004F0323  mov dword ptr ss:\[ebp-18\],eax  
004F0326  add esi,5  
004F0329  lea eax,dword ptr ss:\[ebp-38\]  
004F032C  mov edx,dword ptr ss:\[ebp-8\]    取出第一位查表得的数  
004F032F  shl edx,3        左移 3 位  
004F0332  and edx,0F8        取高五位  
004F0338  mov ecx,dword ptr ss:\[ebp-C\]    取第二位的数  
004F033B  shr ecx,2        右移 2 位  
004F033E  and ecx,7        取低 3 位  
004F0341  or edx,ecx        两数取或，这就是结果第一字节，应为 52  
004F0343  call UnPacKed.004044AC    保存  
004F0348  mov edx,dword ptr ss:\[ebp-38\]  
004F034B  mov eax,ebx  
004F034D  call UnPacKed.0040458C  
004F0352  lea eax,dword ptr ss:\[ebp-3C\]  
004F0355  mov edx,dword ptr ss:\[ebp-14\]    取第四位的数  
004F0358  shl edx,3        左移 3 位  
004F035B  and edx,0C0        取高 2 位  
004F0361  mov ecx,dword ptr ss:\[ebp-C\]    取第 2 位的数  
004F0364  shl ecx,4        左移 4 位  
004F0367  and ecx,30        取高 2 位  
004F036A  or edx,ecx        取或  
004F036C  mov ecx,dword ptr ss:\[ebp-10\]    取第 3 位的数  
004F036F  and ecx,0F        取低 4 位  
004F0372  or edx,ecx        取或  
004F0374  call UnPacKed.004044AC  
004F0379  mov edx,dword ptr ss:\[ebp-3C\]  
004F037C  mov eax,ebx  
004F037E  call UnPacKed.0040458C  
004F0383  lea eax,dword ptr ss:\[ebp-40\]  
004F0386  mov edx,dword ptr ss:\[ebp-14\]    取第四位的数  
004F0389  shl edx,5        左移 5 位  
004F038C  and edx,0E0        取高 3 位  
004F0392  or edx,dword ptr ss:\[ebp-18\]    与第五位的数取或  
004F0395  call UnPacKed.004044AC    保存  
004F039A  mov edx,dword ptr ss:\[ebp-40\]  
004F039D  mov eax,ebx  
004F039F  call UnPacKed.0040458C    连接 52 35 26  
004F03A4  dec dword ptr ss:\[ebp-20\]  计算结果应为 52 36 46，有时变为 31 4B 4B  
004F03A7  jnz UnPacKed.004F02A0  
  
    总结：f2()具体变换是：首先按位在"0KMT1EIJ2AB34FGH56PYZ7NRS89CDUVX"中查表，得到数 a、b、c、d、e，然后对 a、b、c、d、e 进行移位，重新组合成三个字节即数 B，调位方法：  
  
a4,a3,a2,a1,a0,b4,b3,b2  
d4,d3,b1,b0,c3,c2,c1,c0  
d2,d1,d0,e4,e3,e2,e1,e0  
  
    求逆方法：  
  
1. 执行到 00559D0A，然后下：d \[ebp-c\]，记下数据区显示的正确值 x,y,z；  
2. 对 3 个字节的正确值进行调位，得到 5 个字节 a,b,c,d,e，调位方法为：  
  
0,0,0,x7,x6,x5,x4,x3  
0,0,0,x2,x1,x0,y5,y4  
0,0,0,0 ,y3,y2,y1,y0  
0,0,0,y7,y6,z7,z6,z5  
0,0,0,z4,z3,z2,z1,z0  
  
3. 根据 5 个字节查表"0KMT1EIJ2AB34FGH56PYZ7NRS89CDUVX"，得到 5 位注册码即为 23~27 位。  
  
【5. 对 29~31 位的校验】  
  
    29~31 位相当于前 28 位的校验和，首先求 29~31 位的值，跟进 call UnPacKed.004E9BE0：  
  
004ECE3B  mov eax,dword ptr ss:\[ebp-8\]  
004ECE3E  call UnPacKed.00404584  查位数  
004ECE43  mov esi,eax  
004ECE45  mov al,byte ptr ds:\[edi+esi-1\]  
004ECE49  call UnPacKed.004ECFF0  在此表中查位置"0AH6CD3BEF4TRS2PUV5K1MN78YZ9GIJ"  
004ECE4E  mov dword ptr ss:\[ebp-4\],eax  
004ECE51  cmp esi,1  
004ECE54  jle short UnPacKed.004ECE8E  
004ECE56  mov ebx,esi  
004ECE58  dec ebx  
004ECE59  cmp ebx,1  
004ECE5C  jl short UnPacKed.004ECE8E  
004ECE5E  push 4003  
004ECE63  push F8000000    浮点数 31  
004ECE68  push 0  
004ECE6A  mov eax,esi  
004ECE6C  sub eax,ebx  
004ECE6E  call UnPacKed.0042EE28  计算 31 的 n 次方  
004ECE73  call UnPacKed.00402B80  从浮点栈中弹出计算结果  
004ECE78  push eax  
004ECE79  mov al,byte ptr ds:\[edi+ebx-1\]  
004ECE7D  call UnPacKed.004ECFF0  
004ECE82  pop edx  
004ECE83  imul edx,eax  
004ECE86  add dword ptr ss:\[ebp-4\],edx  
004ECE89  dec ebx  
004ECE8A  test ebx,ebx  
004ECE8C  jnz short UnPacKed.004ECE5E  
  
    然后对 1~28 位求和，跟进 call UnPacKed.004F0748，当然也可以不看这一段算法：  
  
004F3983  call UnPacKed.00404584  
004F3988  mov edx,eax  
004F398A  test dl,dl  
004F398C  jbe short UnPacKed.004F39A0  
004F398E  mov al,1  
004F3990  xor ecx,ecx  
004F3992  mov cl,al  
004F3994  movzx ecx,byte ptr ds:\[edi+ecx-1\]  取 1~28 位中的一位  
004F3999  add esi,ecx        累加  
004F399B  inc eax  
004F399C  dec dl  
004F399E  jnz short UnPacKed.004F3990  
004F39A0  xor eax,eax  
004F39A2  mov al,byte ptr ds:\[edi+2\]  取第 3 位  
004F39A5  mov ecx,0A  
004F39AA  xor edx,edx  
004F39AC  div ecx      除以 10  
004F39AE  mov eax,edx  
004F39B0  test al,al      看余数，如为 0 则置为 10  
004F39B2  jnz short UnPacKed.004F39B6  
004F39B4  mov al,0A  
004F39B6  and eax,0FF  
004F39BB  imul esi      累加和乘以这个余数  
004F39BD  mov esi,eax  
004F39BF  cmp esi,6978      结果与 27000 比较，如大于这个数，则除以这个数取余  
004F39C5  jle short UnPacKed.004F39D3  
004F39C7  mov eax,esi  
004F39C9  mov ecx,6978  
004F39CE  cdq  
004F39CF  idiv ecx  
004F39D1  mov esi,edx  
  
    对 1~28 位各位 ASCII 码之累加和，再乘以第 3 位除以 10 的余数，如乘积大于 27000，则除以 27000 取余；然后把 29~31 位看成是 31 进制数，求 16 进制值，其变换表为："0AH6CD3BEF4TRS2PUV5K1MN78YZ9GIJ"，所得结果和 1~28 位的累加和进行比较。  
  
    求逆方法：  
  
1. 执行到 00559E24，记下 eax 中的值 x，或按照上面的方法计算出这个值；  
2. x 除以 31，用余数查表"0AH6CD3BEF4TRS2PUV5K1MN78YZ9GIJ"，所得字符作为第 31 位；  
3. 第 2 步的商除以 31，用余数查表，作为第 30 位；  
4. 第 3 步的商查表，作为第 29 位。  
  
    由于这一步要先确定前 28 的值，所以这一步可以在把别的码都求出以后再重跟一遍时进行。  
  
【6. 对 17~22 位的校验】  
  
    跟进 call UnPacKed.00506940：  
  
00509DB6  lea ecx,dword ptr ss:\[ebp-60\]  
00509DB9  mov edx,dword ptr ss:\[ebp-20\]  
00509DBC  mov eax,dword ptr ss:\[ebp-1C\]  
00509DBF  call UnPacKed.004F46B4    这一步就求出结果，\[ebp-60\]指向该结果，前 6 位有效  
00509DC4  mov edx,dword ptr ss:\[ebp-60\]  
....................  
00509E32  lea edx,dword ptr ss:\[ebp-70\]  
00509E35  mov eax,dword ptr ds:\[57EB9C\]  
00509E3A  mov eax,dword ptr ds:\[eax\]  
00509E3C  call UnPacKed.004E00B8  还原出注册码  
00509E41  lea eax,dword ptr ss:\[ebp-70\]  
00509E44  mov edx,UnPacKed.0050A114                ; ASCII "                         "  
00509E49  call UnPacKed.0040458C  注册码后面加一串空格  
00509E4E  mov eax,dword ptr ss:\[ebp-70\]  
00509E51  mov ecx,6  
00509E56  mov edx,11  
00509E5B  call UnPacKed.004047E4  从第 17 位开始取 6 位注册码  
00509E60  mov eax,dword ptr ss:\[ebp-6C\]  
00509E63  lea edx,dword ptr ss:\[ebp-68\]  
00509E66  call UnPacKed.004F0630  注册码加密  
00509E6B  mov eax,dword ptr ss:\[ebp-68\]  
00509E6E  push eax  
00509E6F  lea eax,dword ptr ss:\[ebp-78\]  
00509E72  push eax  
00509E73  lea eax,dword ptr ss:\[ebp-7C\]  
00509E76  mov ecx,UnPacKed.0050A138                ; ASCII "2AB75ADD"  
00509E7B  mov edx,dword ptr ss:\[ebp-20\]  
00509E7E  call UnPacKed.004045D0  连接两个字符串，其中一个为申请码来的串  
00509E83  mov eax,dword ptr ss:\[ebp-7C\]  
00509E86  mov ecx,6  
00509E8B  mov edx,1  
00509E90  call UnPacKed.004047E4  取前 6 位，即是正确值  
00509E95  mov eax,dword ptr ss:\[ebp-78\]  
00509E98  lea edx,dword ptr ss:\[ebp-74\]  
00509E9B  call UnPacKed.004F0630  加密，与注册码加密算法相同  
00509EA0  mov edx,dword ptr ss:\[ebp-74\]  
00509EA3  pop eax  
00509EA4  call UnPacKed.004046D0  比较  
00509EA9  jnz short UnPacKed.00509EC5    
  
    正确码和待较验的码进行相同的加密算法后再进行比较，但加密之前可以看到正确码。  
  
    求逆方法：  
  
    执行到 00509DC4，下：d \[ebp-60\]，看到一个字符串，记下前 6 位，即是正确的 17~22 位。  
  
【7. 对 1~16 位的校验】  
  
    跟进 call UnPacKed.00503320：  
  
0050659F  lea edx,dword ptr ss:\[ebp-18\]  
005065A2  mov eax,dword ptr ds:\[57F108\]  
005065A7  mov eax,dword ptr ds:\[eax\]  
005065A9  call UnPacKed.004F51C0  由申请码加姓计算得一个数  
005065AE  lea eax,dword ptr ss:\[ebp-18\]  
005065B1  lea edx,dword ptr ss:\[ebp-8\]  
005065B4  call UnPacKed.004F5234  查表变成字符串  
005065B9  lea eax,dword ptr ss:\[ebp-20\]  
005065BC  push eax  
005065BD  lea edx,dword ptr ss:\[ebp-24\]  
005065C0  mov eax,dword ptr ds:\[57EB9C\]  
005065C5  mov eax,dword ptr ds:\[eax\]  
005065C7  call UnPacKed.004E00B8  还原出注册码  
005065CC  mov eax,dword ptr ss:\[ebp-24\]  
005065CF  mov ecx,10  
005065D4  mov edx,1  
005065D9  call UnPacKed.004047E4  取注册码的前 16 位  
005065DE  mov eax,dword ptr ss:\[ebp-20\]  注册码前 16 位  
005065E1  lea ecx,dword ptr ss:\[ebp-1C\]  
005065E4  mov edx,dword ptr ss:\[ebp-8\]  申请码变来的字符串  
005065E7  call UnPacKed.004F4728  用 DES 加密算法计算得一个 8 字节的数 ★ 只要这个数等于 S2 即可  
005065EC  mov eax,dword ptr ss:\[ebp-1C\]  
005065EF  lea edx,dword ptr ss:\[ebp-18\]  
005065F2  call UnPacKed.004F51C0  计算得一个数 C1  
005065F7  lea eax,dword ptr ss:\[ebp-18\]  
005065FA  push eax  
005065FB  lea edx,dword ptr ss:\[ebp-38\]  
005065FE  mov eax,dword ptr ds:\[57F108\]  
00506603  mov eax,dword ptr ds:\[eax\]  加密过的申请码加姓  
00506605  call UnPacKed.004F893C  计算得一个字符串 S2  
0050660A  mov eax,dword ptr ss:\[ebp-38\]  
0050660D  lea edx,dword ptr ss:\[ebp-34\]  
00506610  call UnPacKed.004F51C0  计算得一个数 C2  
00506615  lea eax,dword ptr ss:\[ebp-34\]  
00506618  pop edx  
00506619  call UnPacKed.004F52B8  关键比较，大数比较  
0050661E  mov ebx,eax      标志存在 bl 里，后面有五次检测此标志，把这个标志改为 1，后面即可畅通无阻  
00506620  test bl,bl      第一次检测标志  
00506622  jnz short UnPacKed.00506638  
00506624  xor eax,eax  
00506626  call UnPacKed.00518794  清除注册信息  
  
    跟进 call UnPacKed.004F4728：  
  
004F475A  lea edx,dword ptr ss:\[ebp-C\]  
004F475D  mov eax,dword ptr ss:\[ebp-4\]  
004F4760  call UnPacKed.004E6784  把注册码每两位一组查表，看成 31 进制数求值。表为："0AH6CD3BEF4TRS2PUV5K1MN78YZ9GIJ"  
004F4765  mov ecx,ebx  
004F4767  mov edx,dword ptr ss:\[ebp-8\]  申请码变来的字符串  
004F476A  mov eax,dword ptr ss:\[ebp-C\]  注册码变来的数  
004F476D  call UnPacKed.004F4438  DES 计算  
004F4772  xor eax,eax  
  
    跟进 call UnPacKed.004F4438  
  
004F4498  xor ebx,ebx  
004F449A  lea eax,dword ptr ss:\[ebp-24\]  
004F449D  /mov edx,dword ptr ss:\[ebp-8\]  
004F44A0  |mov dl,byte ptr ds:\[edx+ebx\]  
004F44A3  |mov byte ptr ds:\[eax\],dl  
004F44A5  |inc ebx  
004F44A6  |inc eax  
004F44A7  |cmp ebx,8  
004F44AA  \\jnz short UnPacKed.004F449D  取申请码来的串的前 8 个字符，作为 64 位的密钥  
004F44AC  push 0F  
004F44AE  mov ecx,UnPacKed.00580E1C  子密钥要存放的地址  
004F44B3  lea eax,dword ptr ss:\[ebp-24\]  
004F44B6  mov edx,7  
004F44BB  call UnPacKed.004F3E24  设置子密钥  
004F44C0  lea eax,dword ptr ss:\[ebp-28\]  
004F44C3  call UnPacKed.004042C4  
004F44C8  mov eax,dword ptr ss:\[ebp-4\]  
004F44CB  call UnPacKed.00404584  
004F44D0  test eax,eax  
004F44D2  jns short UnPacKed.004F44D7  
004F44D4  add eax,7  
004F44D7  sar eax,3  
004F44DA  dec eax  
004F44DB  test eax,eax  
004F44DD  jl short UnPacKed.004F455D  
004F44DF  inc eax  
004F44E0  mov dword ptr ss:\[ebp-30\],eax  
004F44E3  mov dword ptr ss:\[ebp-2C\],0  
004F44EA  /xor ebx,ebx  
004F44EC  |lea eax,dword ptr ss:\[ebp-14\]  
004F44EF  |/mov edx,dword ptr ss:\[ebp-2C\]  
004F44F2  ||shl edx,3  
004F44F5  ||add edx,ebx  
004F44F7  ||mov ecx,dword ptr ss:\[ebp-4\]  
004F44FA  ||mov dl,byte ptr ds:\[ecx+edx\]  
004F44FD  ||mov byte ptr ds:\[eax\],dl  
004F44FF  ||inc ebx  
004F4500  ||inc eax  
004F4501  ||cmp ebx,8  
004F4504  |\\jnz short UnPacKed.004F44EF  取注册码变来的数  分组  
004F4506  |lea eax,dword ptr ss:\[ebp-1C\]  
004F4509  |push eax                         ; /Arg2  
004F450A  |push 7                           ; |Arg1 = 00000007  
004F450C  |lea edx,dword ptr ss:\[ebp-14\]    ; |  
004F450F  |mov ecx,7                        ; |  
004F4514  |mov al,1                         ; |  
004F4516  |call UnPacKed.004F407C           ; \\UnPacKed.004F407C  DES 运算  
004F451B  |mov ebx,8  
004F4520  |lea esi,dword ptr ss:\[ebp-1C\]  
004F4523  |/lea eax,dword ptr ss:\[ebp-34\]  
004F4526  ||mov dl,byte ptr ds:\[esi\]  
004F4528  ||call UnPacKed.004044AC  
004F452D  ||mov edx,dword ptr ss:\[ebp-34\]  
004F4530  ||lea eax,dword ptr ss:\[ebp-28\]  
004F4533  ||call UnPacKed.0040458C  
004F4538  ||inc esi  
004F4539  ||dec ebx  
004F453A  |\\jnz short UnPacKed.004F4523  复制运算结果  
  
    跟进 call UnPacKed.004F407C：  
  
004F407C  push ebp   
004F407D  mov ebp,esp  
004F407F  add esp,-18  
004F4082  push ebx  
004F4083  push esi  
004F4084  push edi  
004F4085  mov ebx,ecx  
004F4087  test ebx,ebx  
004F4089  js short UnPacKed.004F4095  
004F408B  shr ebx,2  
004F408E  /mov esi,dword ptr ds:\[edx+ebx\*4\]  
004F4091  |dec ebx  
004F4092  |push esi  
004F4093  \\jns short UnPacKed.004F408E  
004F4095  mov edx,esp  
004F4097  mov dword ptr ss:\[ebp-4\],edx  
004F409A  mov ebx,eax  
004F409C  mov dword ptr ss:\[ebp-8\],8  
004F40A3  mov eax,dword ptr ss:\[ebp-4\]  
004F40A6  mov ecx,dword ptr ss:\[ebp+C\]  
004F40A9  /mov dl,byte ptr ds:\[eax\]  
004F40AB  |mov byte ptr ds:\[ecx\],dl  
004F40AD  |inc ecx  
004F40AE  |inc eax  
004F40AF  |dec dword ptr ss:\[ebp-8\]  
004F40B2  \\jnz short UnPacKed.004F40A9  
004F40B4  mov eax,dword ptr ss:\[ebp+C\]  
004F40B7  mov edx,dword ptr ss:\[ebp+8\]  
004F40BA  call UnPacKed.004F3A40  DES 加密过程的第一步，IP 置换  
004F40BF  test bl,bl  
004F40C1  jnz UnPacKed.004F417C    跳，因为在调用 004F407C 前设定了 al=1,又有 bl=al，所以这里肯定跳  
..............  
004F417C  cmp bl,1      跳到这儿  
004F417F  jnz UnPacKed.004F4237  
004F4185  mov dword ptr ss:\[ebp-8\],-10  循环次数  
004F418C  mov ebx,UnPacKed.00580E76  第一个子密钥地址。一个子密钥占 6 字节，最后一个子密钥地址为 00580E1C，做逆运算时改成它  
004F4191  /mov eax,4  
004F4196  |mov edx,dword ptr ss:\[ebp+C\]  
004F4199  |lea esi,dword ptr ss:\[ebp-C\]  这里是对消息进行分组,把 64 位的消息分为 L0 和 R0  
004F419C  |/mov cl,byte ptr ds:\[edx\]    
004F419E  ||mov byte ptr ds:\[esi\],cl    
004F41A0  ||inc esi  
004F41A1  ||inc edx  
004F41A2  ||dec eax  
004F41A3  |\\jnz short UnPacKed.004F419C  取出 4 个待加密的数据，即 L0  
004F41A5  |mov eax,4  
004F41AA  |mov edx,dword ptr ss:\[ebp+C\]  
004F41AD  |add edx,4  
004F41B0  |/mov cl,byte ptr ds:\[edx\]  R0  
004F41B2  ||mov byte ptr ds:\[edx-4\],cl  
004F41B5  ||inc edx  
004F41B6  ||dec eax  
004F41B7  |\\jnz short UnPacKed.004F41B0  这个循环是作 L1=R0  
004F41B9  |push 5                                       ; /Arg3 = 00000005  
004F41BB  |lea eax,dword ptr ss:\[ebp-10\]                ; |  
004F41BE  |push eax                                     ; |Arg2  
004F41BF  |push 3                                       ; |Arg1 = 00000003  
004F41C1  |mov ecx,ebx                                  ; |  
004F41C3  |mov eax,dword ptr ss:\[ebp+C\]                 ; |  
004F41C6  |mov edx,dword ptr ss:\[ebp+8\]                 ; |  
004F41C9  |call UnPacKed.004F3F54                       ; \\UnPacKed.004F3F54  DES 中的 F 函数  
004F41CE  |mov eax,4  
004F41D3  |lea edx,dword ptr ss:\[ebp-C\]    L\[i-1\]  
004F41D6  |lea esi,dword ptr ss:\[ebp-10\]  F 函数的结果  
004F41D9  |mov ecx,dword ptr ss:\[ebp+C\]  
004F41DC  |add ecx,4  
004F41DF  |mov dword ptr ss:\[ebp-18\],ecx  
004F41E2  |/mov cl,byte ptr ds:\[edx\]  R\[i\]=L\[i-1\]^F(R\[i-1\],K\[15-i\])  
004F41E4  ||xor cl,byte ptr ds:\[esi\]  
004F41E6  ||mov edi,dword ptr ss:\[ebp-18\]  
004F41E9  ||mov byte ptr ds:\[edi\],cl  R\[i\]  
004F41EB  ||inc dword ptr ss:\[ebp-18\]  
004F41EE  ||inc esi  
004F41EF  ||inc edx  
004F41F0  ||dec eax  
004F41F1  |\\jnz short UnPacKed.004F41E2  
004F41F3  |sub ebx,6      子密钥地址，作逆运算时改为 add ebx,6  
004F41F6  |inc dword ptr ss:\[ebp-8\]  
004F41F9  \\jnz short UnPacKed.004F4191  
004F41FB  mov eax,4  
004F4200  mov edx,dword ptr ss:\[ebp+C\]  
004F4203  add edx,4  
004F4206  lea ebx,dword ptr ss:\[ebp-C\]  
004F4209  /mov cl,byte ptr ds:\[edx\]  R\[15\]  
004F420B  |mov byte ptr ds:\[ebx\],cl  
004F420D  |inc ebx  
004F420E  |inc edx  
004F420F  |dec eax  
004F4210  \\jnz short UnPacKed.004F4209  
004F4212  mov eax,4  
004F4217  mov ebx,dword ptr ss:\[ebp+C\]  
004F421A  /mov dl,byte ptr ds:\[ebx\]  L\[15\]  
004F421C  |mov byte ptr ds:\[ebx+4\],dl  
004F421F  |inc ebx  
004F4220  |dec eax  
004F4221  \\jnz short UnPacKed.004F421A  
004F4223  mov eax,4  
004F4228  lea ebx,dword ptr ss:\[ebp-C\]  
004F422B  mov edx,dword ptr ss:\[ebp+C\]  
004F422E  /mov cl,byte ptr ds:\[ebx\]  
004F4230  |mov byte ptr ds:\[edx\],cl  
004F4232  |inc edx  
004F4233  |inc ebx  
004F4234  |dec eax  
004F4235  \\jnz short UnPacKed.004F422E  
004F4237  mov eax,dword ptr ss:\[ebp+C\]  
004F423A  mov edx,dword ptr ss:\[ebp+8\]  
004F423D  call UnPacKed.004F3AC4  DES 加密过程的第三步，IP 逆置换  
004F4242  mov edi,dword ptr ss:\[ebp-24\]  
004F4245  mov esi,dword ptr ss:\[ebp-20\]  
004F4248  mov ebx,dword ptr ss:\[ebp-1C\]  
004F424B  mov esp,ebp  
004F424D  pop ebp  
004F424E  retn 8  
  
    看“看雪精华 5”中 风雨无阻 的“公路坐标计算系统  1.0”，（这篇文章写得很好，看了这一篇你就会明白 DES 加密的原理，如果没这篇文章，我的破解还真难完成呢，不过我们这里不需要完全明白 DES），得知 DES 的加密过程主要分为三步：  
  
1. 待加密数据进行 IP 置换；  
2. 进行 16 轮变换，每一轮一个子密钥；  
3. 进行 IP 逆置换。  
  
    解密过程和加密过程基本一样，不同的是把 16 子密钥倒转顺序，这一点最重要! 这也是我破解此软件的核心。再参考一下“看雪精华 6”中 cnbragon 的“XPSecurity2005c 注册算法分析---标准 DES+变形 MD5”，分析出了 DES 运算的流程（竟然除了地址不同外，代码均一样!?）。因此在我们知道了其子密钥的地址以后，就有了下面的求逆方法，让起名大师帮我们解密。  
  
    求逆方法：  
  
1. 首先运行到 0050660D，下：d eax，记下要做 DES 逆运算的 8 字节数据；  
2. 设断于 004F476D，重新运行，中断后下：d edx，更改数据为第 1 步的数据；  
3. 更改 004F418C、004F41F3 处代码：  
004F418C  mov ebx,UnPacKed.00580E1C  
004F41F3  add ebx,6  
4. 设断于 004F4772，F9 运行，断下后下：d \[ebx\]，记下其注册码应该变成的数据；  
5. 作 004F4760  call UnPacKed.004E6784 的逆运算，即从前面的数据中取一个字节，除以 31，用商在表"0AH6CD3BEF4TRS2PUV5K1MN78YZ9GIJ"中查出一个字符，用余数再查出一个字符，连续写下来，就得到了正确的前 16 位注册码。  
  
【8. 总结】  
  
    按以上方法求注册码，可其中没有涉及第 25 位注册码，随便取一个值，提示注册成功。可是用一下“个人起名”功能以后，软件又会提示“软件尚未注册”，我没弄清还在什么地方有什么校验，看来这要请高人来指点啦:) 

> 引用:
> 
> * * *
> 
> *最初由 moon 发布*  
> **okdodo：你好，我知道你对此软件也有研究，可能你已成功破解了 it，我很想得到你的帮助。  
>   
> 你说的“暗桩”，“姓氏笔划数”是不是和第 25 位码有关呀？ **
> 
> * * *

  
  
正确 ![](img/0cf410da109ac47387dd713fcee2cd79.jpg)  与姓氏笔画和公司名第二字笔画数有关（以同益的笔画数为准）  
  
key\[24\]=密码表 1 的第\[(bh1+bh2)%31\]号位字符;  
  
如果第一次没有越过这个暗桩 我就卸载并重新安装  
否则即使注册码正确也会有问题（没仔细看哪儿有记录） ～ 

【8. 第 28 位的校验】  
  
    按以上方法求注册码，可其中没有涉及第 28 位（调位前是第 25 位）注册码，随便取一个值，提示注册成功。可是用一下“个人起名”功能以后，软件又会提示“软件尚未注册”。设断于 004E00B8，点“开始分析”，可以跟到：  
  
0054F2CE  lea eax,dword ptr ss:\[ebp-4C\]  
0054F2D1  push eax  
0054F2D2  lea edx,dword ptr ss:\[ebp-50\]  
0054F2D5  mov eax,dword ptr ds:\[57EB9C\]  
0054F2DA  mov eax,dword ptr ds:\[eax\]  
0054F2DC  call UnPacKed.004E00B8  还原出注册码  
0054F2E1  mov eax,dword ptr ss:\[ebp-50\]  
0054F2E4  mov ecx,1  
0054F2E9  mov edx,1C  
0054F2EE  call UnPacKed.004047E4  取第 28 位  
0054F2F3  mov eax,dword ptr ss:\[ebp-4C\]  
0054F2F6  call UnPacKed.004E9BE0  查表"0AH6CD3BEF4TRS2PUV5K1MN78YZ9GIJ"求值  
0054F2FB  mov edi,eax      暂存于 edi  
0054F2FD  lea eax,dword ptr ss:\[ebp-54\]  
0054F300  push eax  
0054F301  lea eax,dword ptr ss:\[ebp-58\]  
0054F304  push eax  
0054F305  mov ecx,2  
0054F30A  mov edx,1  
0054F30F  mov eax,dword ptr ss:\[ebp-8\]  
0054F312  call UnPacKed.004047E4  取姓第一个汉字  
0054F317  mov edx,dword ptr ss:\[ebp-58\]  
0054F31A  mov eax,dword ptr ds:\[57EC3C\]  
0054F31F  mov eax,dword ptr ds:\[eax\]  
0054F321  mov eax,dword ptr ds:\[eax+3BC\]  
0054F327  mov ecx,UnPacKed.0054F908         ; ASCII "113"  
0054F32C  call UnPacKed.005200D0  算汉字笔划数，得到的是字符串  
0054F331  mov eax,dword ptr ss:\[ebp-54\]  
0054F334  call UnPacKed.00409034  变成数字  
0054F339  cmp edi,eax      比较  
0054F33B  je short UnPacKed.0054F398  
  
    原来是取姓氏笔划数查表得一个字符作为第 28 位。至此，同益起名大师 3，36 注册算法分析完毕。第 28 位的分析与 okdodo 大侠略有不同，可能是由于我没有输入公司名的原因吧。我在我的机器上已测试通过。  
    这个校验很隐蔽，不到最关键的时候不校验，而且一次通不过这个校验就会在系统里留下标记（具体在哪儿不清楚），要重启一下，下次才有可能通过。总的来说，这个软件保护比较强，我就是 her 越强，我越想 fuck her，结果就导致了今天的局面，呵呵 

> 引用:
> 
> * * *
> 
> *最初由 okdodo 发布*  
> **  
>   
> 如果第一次没有越过这个暗桩 我就卸载并重新安装  
> 否则即使注册码正确也会有问题（没仔细看哪儿有记录） ～  
>  **
> 
> * * *

  
今天才看到这篇妙文，可惜太迟了，许多东西我都忘记了，当时没有做记录，只是随手看了一下这个软件的注册方法，下面是凭记忆写的，可能有不对的地方。  
这个软件逆算注册码是非常困难的，非常佩服 moon 的技术和毅力，这不是一般人可以做到的，这个软件爆破则不是很困难，脱壳后花几小时即可做到，但软件有些 bug 如主 Form 在建立时初始化了一组 TString（建立），在一个子过程中使用了相同的名字和地址重新进行初始化，也有限制，比如屏幕拷贝，还有注册一次只能用注册的姓起名，另外运行软件总是出现同益字样等。如果要将这些都改过来，我不知别人需要多少时间，我花了大概十几个小时（分几天）才完成。  
在数据库中，最简单的办法是安装完后，不要运行，将数据库 goodname.dll 复制一份，注册失败后将这个数据库拷贝回来覆盖即可。  
另外你也可以打开这个数据库，是 Sybase 的数据库，密码在资源表中(是一串乱码，不好记，我忘了，好像有~和^，因为非常特殊，才有点映象)，打开这个数据库，其中有几张表是和注册有关的。  
第一张表  
slidk  
这张表不同的版本略有不同，对于 336 板有三个字段：  
ID   SC   JS  
77 木水木  
其中 ID 在正确注册后填的是你的注册申请码，如果注册失败，则会填一个和注册表中相应字段(AppID)相同的值(具体数值忘记了)。SC 字段是五行的三个字，如“木金木”，如果注册失败也会填入一个五行的字符串(同样也记不住了)。JS 字段没有使用。  
如果想重新注册，只要将 ID 值改为没有注册前的数值即可(同样也记不清了，好像是 77)，而五行字符串好像是“木金木”，或“木水木”  
第二张表  
slxsk  
这张表有两个字段，如果你正确的注册了，这张表体现了你可以使用几个姓：  
ID633   SC  
  8    木金土   这个对应同益的“同”  
 12    火木水   这个对应同益的“益”  
 16    木水木   这个是你的第一个姓，我这里的五行字是瞎填的具体如何得到这个东西我下面再将。  
 20    土木水   这个是对应你的公司后缀或你能够使用的第二个姓。  
  
这个字段的值是通过一句 SQL 语句得到的，语句如下：  
update slidk set sc=(select wtrdg from slsck where id=(select id+1 from slsck where wtrdg=(select sc from slidk)))  
如果注册失败只要将 ID633 大于 12 的项删除即可。  
  
第三张表  
slsck  
这张有四个字段，只是一张实用的表结构如下：  
WTRDG  
木木木  
JS  
性情温厚平静，一般与同事和朋友的关系好，并具有努力奋斗不屈不挠的精神，若先天条件不忌木者，这是良好配置。若是女性，易陷孤独。  
JX   ID  
吉    1  
上面那张表生成就是查询这张表。  
第四张表  
slzik  
这是一张非常大的数据表，整个一个汉字字典：  
WD FWD ZPY  ........ WZ  BH .....  
张  張  zhang ........  1   11 .....  
这里注册使用到三个字段： WD WZ BH  
这 BH 意思是该字繁体字的笔画数，WZ 是一个非常奇怪的东西，一般为 0 或 1。当你注册了你的姓后，比如张，WZ 就会填入一个大于 3 的数（如何计算，我忘了，非常简单的计算，没有使用什么加密的方法，只是加加减减），如果你注册多于一个姓或注册公司后缀，则所对应的 WZ 都会填入一个大于 3 的数。  
moon 【8. 第 28 位的校验】中  
0054F32C  call UnPacKed.005200D0  算汉字笔划数，得到的是字符串  
005200D0 这个子过程其实就是在 slizk 库中将所有 WZ 大于 3 的笔画数累加起来得到的，用的是如下一句 SQL：  
select sum(bh) from slzik where wz>3  
如果注册错误，这个可以不用管，同益自己会删除的。  
读了 moon 的文章，翻了一下同益代码的相关部分，估计用汇编语言实现，大概需要 5000～7000 句，自己写的部分不多，主要是将相应的加密算法拷贝过来，不过源程序是 Delphi 写的，Delphi 有自己的一套异常处理和字符串管理办法，你不可能将这部分都拷贝过来，所以用汇编实现，这些拷贝过来的代码需要修改，这个工作量就很大了，而且没有什么技术，只是一个有点脑子的搬运工。最好还是用 Delphi 写，将这些算法逆向。可惜我最熟悉的就是汇编，我用 Delphi 只是瞎混混，离正常开发还有一段距离。所以有时间我会用汇编试试。 

 4dd466d0e1e0cc123afbe504da2462ca