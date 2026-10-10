

# 2008 看雪论坛读书月第二题的注册机

> 原文：[`www.pediy.com/kssd/pediy10/68954.html`](https://www.pediy.com/kssd/pediy10/68954.html)

请查看附件![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || pediykg2.rar  
 |

输入用户名和密码点 regist 按钮后，用户名和注册码将分别被保存到 HKEY\_CURRENT\_USER\\Software\\MegaX 键下的 name 和 key 中  
  
重新运行 CrackMe.net.exe 后，开始读入上述注册表键值并校验用户名和注册码  
  
首先注册码由 4 组字符串组成，每组 5 个字符，中间由"-"号分开（这里我们分别以 AAAAA, BBBBB, CCCCC, DDDDD 表示四组注册码，USERNAME 表示用户名）  
其次由固定字符串"CrackMe"，"MegaX"，第一，第二组注册码以及用户名合成一个字符串  
CrackMeAAAAABBBBBMegaXUSERNAMEMegaX，并计算这个字符串的 md5  
然后将计算出后的 md5 每个字节都转换为 10 进制数并合成一个字符串，然后取最前面的 5 个字符  
然后在一个固定字符串“8x3p5BeabcdfghijklmnoqrstuvwyzACDEFGHIJKLMNOPQRSTUVWXYZ1246790”中依次查找上一步得到的字符串的每个字符，并将返回的偏移地址减去第三组注册码 CCCCC 中相对应的每个字节在该固定字符串中的偏移地址，如果为负数则加上该固定字符串的长度，最后在该固定字符串中查找上面计算出来的偏移地址的字符，并合成最后一组注册码 

忘记提到了，这道题还有一个特点是如果注册格式不正确则根本就不会继续校验下去，因此在注意到注册码格式之前很容易就跟飞了 

这里先说一声对不起哈，我的文字表达能力不太好，如果有文笔不通之处还请多多包涵了  
     
【文章标题】: 《2008 看雪论坛读书月第二题》分析随笔  
【文章作者】: mstwugui  
【软件名称】: 2008 看雪论坛读书月第二题 CrackMe02  
【文件 MD5】: 62c5e33505563dbd8a597dbaf5331f8e  
【下载地址】: [`bbs.pediy.com/showthread.php?t=68795`](http://bbs.pediy.com/showthread.php?t=68795)  
【加壳方式】: 不知道具体是什么壳，但很明显被混淆保护了  
【开发环境】: Microsoft Visual Studio .NET 2008  
【使用工具】: OllyDbg, regshot, ApiMonitor  
【操作平台】: XP SP3 + .NET Framework 2.0 SP1  
【软件介绍】: 2008 看雪论坛读书月第二题  
     
刚看到第二题时有些意外也有些开心，正好没有分析过.NET 程序这下有机会熟悉熟悉练习一下了，：）于是立刻下载下来运行一下  
![](img/1a1e43f2a840bfca6baf0d0ef43182b2.jpg)  
  
倒。。。电脑上安装好的.NET 1.1 看来没用，再下个 2.0 SP1 吧  
![](img/55c4c864760c67ce93411ca0d4c105a6.jpg)  
  
咣当。。。万事开头难，坚持。。。坚持。。。看了看论坛，似乎其他朋友也遇到了这样那样的问题，但也有成功运行的，看来是环境问题。  
     
重启电脑。。。开机后立刻运行 CrackMe.net.exe，哈，运气不错，正常启动了  
     
可是。。。接下来我该从哪开始呢？  
     
  。。。  
     
嗯，还是先打开 google 搜索了一下.NET 的反编译吧  
     
Reflector…Dis#...Salamander…  
     
看雪的题目果然不一般，想想也对，这道题肯定是混淆保护过的，如果从网上随便下个工具就能反编译出源代码那还玩个什么劲啊，算了吧，还是不浪费时间偷懒找反编译了，老老实实自己动手一定也可以丰衣足食，：）  
     
于是习惯性的 IDA 了一眼。。。倒。。。。一堆问号不知所云。。。再换 OD 看看。。。哈，终于看到些认识的蝌蚪文了  
     
jmp     dword ptr \[<&mscoree.\_CorExeMain>\]  
     
很明显如果跟进去也只是系统调用里瞎转悠，既然不熟悉.NET 也不知道该再哪下断点，还是暂且搁置一下先  
     
再来运行一下 CrackMe.net.exe，随手输入 usernmae 和 key 然后点 regist  
     
再倒。。。又出错退出了，可是我也没有多干啥呀，和刚才刚启动好的时候也就多了几个进程，难道是内存不够？嗯，还是先把那几个新启动的进程都给关了再试试。汗。。。又正常启动了，可能是保护机制有 BUG 吧  
  
  
![](img/9e8989563e4c3d3135e8d95ab61f978a.jpg)  
  
  
看来是程序启动时校验 username 和 key,  这种情况下通常都是将注册信息保存在注册表或是文件系统中。嗯，用 regshot 对文件系统和注册表做一个 snapshot 再 regist 然后比较前后区别。。。果然不出所料，注册信息存放在注册表中  
     
  HKEY\_CURRENT\_USER\\Software\\MegaX\\name: "masterwugui"  
  HKEY\_CURRENT\_USER\\Software\\MegaX\\key: "testkey"  
     
嗯，回 OD 以前还需要确认一下具体是哪个 API 读取的注册信息，打开 ApiMonitor 设置监视 RegQueryValueA, RegQueryValueW, RegQueryValueExA, RegQueryValueExW，然后重新启动 CrackMe.net.exe，哈，注册表访问还不少，在接近最尾部的一处调用看到了是通过 RegQueryValueExW 读取的注册信息。  
     
是时候回到 OD 了，开始调试 CrackMe.net.exe，停在入口点后在 ADVAPI32.RegQueryValueExW 设一个断点，并设置条件为：  
     
  (((\[\[esp+8\]\] == 0x0061006E) || (\[\[esp+8\]\] == 0x0065006B)) && (\[esp+0x14\] != 0))  
     
为什么要设置这个条件？呵呵，RegQueryValueExW 可不会只被调用一次，我们只需要在读取 name 和 key 的时候中断  
     
按下 F9。。。开跑。。。停下来了，先看一眼\[esp+8\]，嗯，指向的是 name，在\[esp+0x14\]设一个单字节硬件访问断点 A，继续跑。。。  
     
先是停在了 ntdll.memmove 里面，向缓冲写入刚刚从注册表里读取出来的用户名，继续跑。。。  
     
接下来停在 mscorwks 里面，看代码很明显是计算字符串长度，继续跑。。。  
     
  mov                  si, word ptr \[eax\]  
  inc                    eax  
  inc                    eax  
  cmp                  si, bx  
  jnz                    $-8  
     
再接下来又停在了 msvcr80.memcpy 里面，对目标地址再设一个单字节硬件访问断点 B，继续跑。。。  
     
又停在 mscorwks，不过这次是清空断点 A 的字符串，看来断点 A 指向的是个临时缓冲，删除了断点 A，继续跑。。。  
     
  add                  word ptr \[edi+eax\], 0  
     
哈，回到了 RegQueryValueExW，赶快看看\[esp+8\]，嗯，现在指向的是 key 了，重复上面的步骤直到设置单字节硬件访问断点 C  
     
好了，现在有了两个单字节硬件访问断点 B 和 C，分别指向 name 和 key。有了这些基本信息后我们可以先 F9 简单看一下流程  
     
连续在 mscorlib 里面同一处停下来两次，一次是因为 name，一次是 key，但没有什么特别的，系统调用，跳过。。。  
     
接下来还是停在了 mscorlib, 只不过这次似乎有些文章，请注意下图中加红框的部分，此时 AX 中存放了 key 指向的字符，而 EDX 指向的是一个字符’-’  
![](img/7138f06029d00dc9e26075c8cba399fd.jpg)  
  
   看来 key 中必须有字符’-’，而根据蓝色的这一句看来，应该需要两个或更多的’-’，不过我们暂时先忽略这个问题，继续 F9。。。哈，没有再遇到其他的断点直接跑出来了  
     
  很明显我们要想继续跟踪下去必须先修改 key，如果 key 中没有’-‘字符就会直接注册校验失败。先把 key 改为 1234-5678，然后再重新调试。  
     
  嗯，现在又走近了一步，停在了 mscorlib 中，很明显这次是把 key 中‘-’字符之前的子字符串复制到目标地址，因此在目标地址下个单字节硬件访问断点 D，继续 F9。。。  
  
![](img/ce94d4692fee962cbec7154124a74aa4.jpg)  
   什么？又是畅通无阻直接跑出来了？看来 key 的格式还有文章，先重新调试回到上一步。但这次我们不要 F9 直接跑出来，重复用 Ctrl+F9 执行到返回查看一下究竟有什么文章。直到返回到这一层  
    
![](img/dbd290c5af73d94f035adf565c9129cd.jpg)  
  
 在这里继续单步 F8 连续几个跳转之后，直到如下代码  
  
![](img/4facf5fc1dfcfa29aadae3911cc80086.jpg)  
  
   这个跳转如果没有进入很快就会从这个函数中返回，所以这里肯定也是一道关卡。原来\[eax+4\]中存放的是 key 分割以后的数量，因此我们需要四组由’-‘分开的 key。现在将 key 改为 1234-5678-ABCD-EFGH 后，重新调试  
     
  在经过上一步数量校验之后，我们继续单步直到如下代码：  
    
![](img/f0ef026ad4351523ca75af403c34ea10.jpg)  
  
   此时，ecx 指向的是分割后的字符串对象（注意这里指向的是个对象而不是字符串本身），而\[B1659C\]的作用是取该字符串的长度，因此 key 不仅仅需要 3 个’-‘分割，而且每组分割后的字符串长度必须为 5。将 key 修改为 01234-56789-ABCDE-FGHIJ，重新调试  
     
  先 F9 连续跑一次试试，哈，看来没有更多的格式校验了，我们终于重新停在了指向 name 的断点 B 上。确定好格式没有问题之后，是时候该改变跟踪策略了。  
     
  因为上一步校验每组密码字符串长度所处的代码在临时分配空间，所以无法直接在此处设置断点重新启动后自动中断下来。而且每次因为太多硬件断点停留也烦人，所以可以先删除所有其他的断点只保留指向 name 的断点 B。当断点 B 在 RegQueryValueExW 取得 name 以后，下一次中断时这个分组字符串长度校验所处的内存空间已经正常分配，这时候就可以在 cmp eax, 5 这条指令上下一个断点，然后 F9 直接跑到这里。连续四次校验完四组密钥之后，一路 F8（此时还会路过 4 次分割 key，但我们暂时忽略），直到看到如下代码  
    
![](img/0408f21db4b083b6b3b3b6810610909d.jpg)  
  
   此时压入堆栈的 eax 和\[ebp-dc\]分别指向两个字符串对象”MegaX”和用户名”masterwugui”，而 ecx 和 edx 分别指向密钥中的第一”01234”和第二组字符串”56789”。  
     
  现在在下一个调用”call dword ptr \[B165B0\]”下个断点 E，继续 F9。。。  
     
  我们会因为指向 name 的断点 B 中断两次，直接跳过，直到回到刚刚添加的断点 E，这时候查看一下 ecx 和 edx 的值，原来前面那个调用是把那四个字符串合成起来，ecx 因此指向了一个字符串对象”0123456789MegaXmasterwugui”，而 edx 指向的是第三组密钥字符串”ABCDE”  
     
  呵呵，黎明越来越近了，很明显下一个调用”call dword ptr \[B165B0\]”非常关键，立刻 F7 单步进入。这里会连续两次动态解析加载代码，直接跳过  
    
![](img/76ba68ce492e19dfa3d3be6c1f8f98b2.jpg)  
  
   在第二次从上述代码返回后，程序开始对之前传入的两个字符串对象”0123456789MegaXmasterwugui”以及”ABCDE”进行一连串的加密动作，这里我们暂时先忽略，继续运行到下图中红框处  
    
![](img/887f4abc9c6b5b572199a47509c45d0b.jpg)  
  
   此时 eax 指向一个字符串对象”uVuyv”，也就是上面两个字符串计算出来的结果。因此对该地址下一个单字节硬件访问断点，F9 继续跑  
     
  第一次停留没有什么特别之处，直接跳过，但第二次停留时。。。  
    
![](img/6c8f3fb0a4ae1c35d491dbe66abbcafa.jpg)  
  
   字符串”uVuyv”被转换成了大写并在被保存到\[edi+eax\]中，继续在\[edi+eax\]下一个单字节硬件访问断点 F，继续 F9。。。  
    
![](img/21c98729eb151f9d8ce56114dd1e96d5.jpg)  
  
   看来是对上面计算出来的字符串”UVUYV”再次加密，此时红框里 edx 指向我们输入的第四组密钥字符串，而 ecx 指向的则是正确的第四组密钥字符串，终于得到了第一组有效的 username 和 key  
     
  Username: masterwugui  
  Key: 01234-56789-ABCDE-RGHF3  
     
  庆祝一下？别急别急，还有两个问题没有解决”RGHF3”和”UVUYV”分别是怎么计算出来的  
     
  第一个问题比较容易，分析一下最后这一步很容易就得出了  
    

代码:

```
    CString CalcFinalKey(CString sKey)
  {
          DWORD           i;
          CString         sRet = "";
          CString         sMegaX = "MegaX";
          CString         sFinalKeys = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
          BYTE            c;
  
          for (i=0; i<5; i++)
          {
                  c = sKey.GetAt(i);
                  c += sMegaX.GetAt(i);
                  c += 0x23;
  
                  sRet += sFinalKeys.GetAt(c%0x24);
          }
  
          return sRet;
  }
  

```

问题是”uVuyv”是从哪里来的。。。现在我们如果回到前面去慢慢单步分析多半会头晕眼花，还是对”uVuyv”设置单字节硬件写入断点反着追吧  
  
接下来并没有多少难点也就不多费口舌了，在连续的设置单字节写入断点反追之后，发现”uVuyv”来自与一个字符串"1129720123519959835898227196100347012562" 的计算，而这个字符串则是由 16 个字节分别转为十进字数后合成的。  
  
又多了一个问题，这 16 个字节哪来的？同上面一样，继续反追之后发现最后对这 16 个字节进行写操作是 ADVAPI32.CryptGetHashParam，看到这里是不是突然明白了？立刻在 ADVAPI32.CryptCreateHash 下一个断点重新调试，果然不出所料，在最后写入那关键的 16 个字节之前，CryptCreateHash 用的参数是 CALC\_MD5  
  
现在大家应该都明白了，原来这 16 个字节就是之前用”0123456789MegaXmasterwugui”和第三组密钥字符串”ABCDE”合成的字符串"CrackMe0123456789MegaXmasterwuguiMegaX"计算出来的 MD5，好了，现在可以完成最后的步骤，写注册机了。  
  
代码:

```
    char GetRandByte()
  {
          char c = rand()%36;
  
          c += (c<10)?0x30:0x37;
  
          return c;
  }
  
  CString GenerateRandKey()
  {
          CString         sRandKey = "";
  
          for (int i=0;i<5;i++)
                  sRandKey += GetRandByte();
  
          return sRandKey;
  }
  
  CString CalcFinalKey(CString sKey)
  {
          DWORD           i;
          CString         sRet = "";
          CString         sMegaX = "MegaX";
          CString         sFinalKeys = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
          BYTE            c;
  
          for (i=0; i<5; i++)
          {
                  c = sKey.GetAt(i);
                  c += sMegaX.GetAt(i);
                  c += 0x23;
  
                  sRet += sFinalKeys.GetAt(c%0x24);
          }
  
          return sRet;
  }
  
  
  CString CalcKey(CString sName, CString sPassword1, CString sPassword2, CString sPassword3)
  {
          CString         sTemp = "CrackMe" + sPassword1 + sPassword2 + "MegaX" + sName + "MegaX";
          CString         sTemp1 = "";
          CString         sRet = "";
          CString         sKeys = "8x3p5BeabcdfghijklmnoqrstuvwyzACDEFGHIJKLMNOPQRSTUVWXYZ1246790";
  
          HCRYPTPROV      hProv;
          HCRYPTHASH      hHash;
  
          if (CryptAcquireContext(&hProv, NULL, NULL, PROV\_RSA\_FULL, 0xF0000000))
          {
                  if (CryptCreateHash(hProv, CALG\_MD5, NULL, 0, &hHash))
                  {
                          if (CryptHashData(hHash, (const BYTE\*)sTemp.GetBuffer(), sTemp.GetLength(), 0))
                          {
                                  DWORD   size = 16;
                                  BYTE    code\[16\];
                                  int             offset;
  
                                  if (CryptGetHashParam(hHash, HP\_HASHVAL, (BYTE\*)code, &size, 0))
                                  {
                                          DWORD   i, j;
  
                                          for (i=0; i<size; i++)
                                          {
                                                  sTemp.Format("%d", code\[i\]);
                                                  sTemp1 += sTemp;
                                          }
  
                                          for (i=0; i< 5; i++)
                                          {
                                                  j = sKeys.Find(sPassword3.GetAt(i));
                                                  offset = sKeys.Find(sTemp1.GetAt(i)) - j;
                                                  if (offset < 0)
                                                          offset += sKeys.GetLength();
                                                  sRet += sKeys.GetAt(offset);
                                          }
                                  }
                          }
                          CryptDestroyHash(hHash);
                  }
                  CryptReleaseContext(hProv,0);
          }
  
          if (sRet.GetLength() == 0)
          {
                  MessageBox(0, "Failed to invoke cryptographic api!", "ERROR", MB\_ICONSTOP);
                  ExitProcess(0);
          }
  
          sRet.MakeUpper();
  
          return CalcFinalKey(sRet);
  }
  
  void CpediykgDlg::OnEnChangeEdit1()
  {
          // TODO:  If this is a RICHEDIT control, the control will not
          // send this notification unless you override the CDialog::OnInitDialog()
          // function and call CRichEditCtrl().SetEventMask()
          // with the ENM\_CHANGE flag ORed into the mask.
  
          // TODO:  Add your control notification handler code here
          OnBnClickedGenerate();
  }
  
  void CpediykgDlg::OnBnClickedGenerate()
  {
          // TODO: Add your control notification handler code here
          CString         sName;
          int                     nNameLength;
          CString         sPassword, sPassword1, sPassword2, sPassword3, sPassword4;
  
  
          m\_edtPassword.SetWindowText("");
          m\_edtName.GetWindowText(sName);
          nNameLength = sName.GetLength();
  
          if (nNameLength == 0)
                  return;
  
          srand(GetTickCount());
          sPassword1 = GenerateRandKey();
          sPassword2 = GenerateRandKey();
          sPassword3 = GenerateRandKey();
          sPassword4 = CalcKey(sName, sPassword1, sPassword2, sPassword3);
  
          sPassword = sPassword1;
          sPassword += "-";
          sPassword += sPassword2;
          sPassword += "-";
          sPassword += sPassword3;
          sPassword += "-";
          sPassword += sPassword4;
          m\_edtPassword.SetWindowText(sPassword);
  }
   
  

```

注意，密钥中的前三组字符串其实也可以有小写字符的，但我写的注册机只用了大写字符和数字，纯粹是为了让密钥整齐一些看起来比较顺眼![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")  
  
最后再唠叨两句吧。技术日新月异，而且不会像赛跑有个理论极限，保不准明天又会出现一门新学问来，所以有时间的时候还是多练练手，有新东西的时候才会比较容易有感觉入手，而培养这个感觉基本上没有什么捷径，大多都是日积月累的经验堆出来的。  
  
以这一题为例，第一步是需要明确入手点，然后以合适的方式切入，中间的过程虽然有些磕磕碰碰，但坚持下去并不断的总结经验，最后总会看到光明的。  
  
破解注册算法其实只是逆向工程很小的一部分，而且最好不要太过依赖于或是期望某个强大的自动化工具或脚本，在逆向的过程中可以学习到很多技巧或机制，这些才是最值得我们学习的。  
  
熟练的逆向=经验+毅力+信心+灵感+。。。呵呵，当然还需要一部分运气  
  
好了，写到这里也该告个段落了，谢谢大家花这么多时间看到这里，也感谢一下 MegaX 和看雪论坛提供了这个练习机会 

这一题应该还有很多不同的解法，我的办法肯定不是最正统的，或许可以先脱壳再常规反编译.NET 程序，但因为不熟悉.NET 环境所以一时我也只能这样了，希望熟悉的朋友不吝指教