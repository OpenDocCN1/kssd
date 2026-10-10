

# 2008 看雪论坛读书月第二题 KeyGen

> 原文：[`www.pediy.com/kssd/pediy10/69321.html`](https://www.pediy.com/kssd/pediy10/69321.html)

破解过程明天补上。

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || KeyGen.rar  
 |

【文章标题】: 【原创】【看雪读书月】《2008 看雪论坛读书月第二题 crackMe02》KeyGen  
【文章作者】: 沙金  
【软件名称】: 2008 看雪论坛读书月第二题 crackMe02  
【软件大小】: 49.2 KB  
【下载地址】: [`bbs.pediy.com/showthread.php?t=68795`](http://bbs.pediy.com/showthread.php?t=68795)  
【软件作者】: MegaX  
【编写语言】: Microsoft Visual C++ 2008 (.net)  
【使用工具】: OD  
【操作平台】: WINDOWS2003  
【软件介绍】: 2008 看雪论坛读书月第二题  
\--------------------------------------------------------------------------------  
【详细过程】  
  
一、测试篇  
    拿到题以后，首先用 OD 载入，直接报错（我的系统是 winxp）。  
    启动 VM，把题拷入 2003 系统下，运行，OK 了。  
    随便输入用户名和 key，提示重启应用程序，查看没有生成文件，于是在注册表里查找刚才输入的用户名，果然在 HKEY\_CURRENT\_USER\\Software\\MegaX 找到。  
二、反编译篇  
    由于从来没有调试过.net 程序，所以，在 google 一下，才知道.net 可以反编译，于是，开始下载工具。  
    首先是 Reflector 下载了最新版，结果反编译失败。  
    然后使用 ildasm 可以读出部分结构，但很多类名乱码，没有得到可用信息。  
    本想试试 PEBrowse ，在 google 里发现，在使用 PEBrowse 前，要用 ildasm 反编译后，生成带 debug 信息的 exe 才有效果。只能放弃。  
    仔细想想，如果可以这么简单，那就不是看雪的风格了。应该是加了壳，所以一般的反编译工具没有办法。只能动态调试了。  
三、OD 篇  
    用 OD 载入，程序直接自动启动，OD 的 ICE 区一片空白。  
    想到有注册表的操作，于是 bp RegOpenKeyExA 和 bp RegOpenKeyExW ，重新载入，在 RegOpenKeyExW 处断下。可以判断是 UNICODE 码。  
    断了 70 次左右，总算看到了"MegaX" 字符串，记录下当前寄存器变量发现 esi 值可以利用，于是，  
    在 77F57AAB >  81FE 04000080   cmp     esi, 80000004 下 esi == 1DC 的条件断点，再次重启程序，果然断下。（ESI 和不同的系统，值不一样，而且，也不是每次都可以成功断下）  
    接下来的工作非常郁闷，一直没法到“程序领域”，经过 N 次尝试后发现，程序代码在堆空间里，而且，边执行边解压，难怪反编译不成功。  
    而且，更郁闷的是，由于在堆空间里，所以，没办法保存断点，只要跑飞了，就得从头再来。代码里，很多暗桩，稍不注意就飞了，大大增加了分析难度。  
    废话了这么多，现在开始  

代码:

```
00F19A43    8B8F 60010000   mov     ecx, dword ptr \[edi+160\]         
00F19A49    8B97 58010000   mov     edx, dword ptr \[edi+158\]         ; name
00F19A4F    FFB7 5C010000   push    dword ptr \[edi+15C\]              ; key
00F19A55    FF15 90659A00   call    dword ptr \[9A6590\]               ; 比较函数

```

  
    F7 跟进，到  
代码:

```
00F1B7C0    8B8D 0CFFFFFF   mov     ecx, dword ptr \[ebp-F4\]
00F1B7C6    FF15 9C659A00   call    dword ptr \[9A659C\]
00F1B7CC    83F8 05         cmp     eax, 5 

```

    可以确定 key 的格式为 key1-key2-key3-key4  
    key1,key2,key3,key4 的长度都为 5，如果不满足条件，就跳出函数，注册失败。  
    于是，改注册表里的 KEY 键值为 AAAAA-BBBBB-CCCCC-DDDDD 后重新分析。  
  
代码:

```
00F1B90A    50              push    eax
00F1B90B    FFB5 24FFFFFF   push    dword ptr \[ebp-DC\]               
00F1B911    8B95 1CFFFFFF   mov     edx, dword ptr \[ebp-E4\]
00F1B917    8B8D 20FFFFFF   mov     ecx, dword ptr \[ebp-E0\]
00F1B91D    FF15 AC659A00   call    dword ptr \[9A65AC\] 

```

    此函数产生一个新字符串: key1+key3+"MegaX"+name  
  
代码:

```
00F1B923    8BC8            mov     ecx, eax
00F1B925    8B95 18FFFFFF   mov     edx, dword ptr \[ebp-E8\]
00F1B92B    FF15 B0659A00   call    dword ptr \[9A65B0\]               
00F1B931    8985 08FFFFFF   mov     dword ptr \[ebp-F8\], eax
00F1B937    BF 58000000     mov     edi, 58

```

    此函数通过 key3 生成一个 5 位长度的字符串，跟进  
  
代码:

```
00F1CDA1    8BF0            mov     esi, eax
00F1CDA3    8BD3            mov     edx, ebx
00F1CDA5    8BCE            mov     ecx, esi
00F1CDA7    FF15 B87A9A00   call    dword ptr \[9A7AB8\]

```

    生成一个 0x3E 大小的字符串数组，内容如下：  
    第一个为:8x3p5BeabcdfghijklmnoqrstuvwyzACDEFGHIJKLMNOPQRSTUVWXYZ1246790  
    第二个为:x3p5BeabcdfghijklmnoqrstuvwyzACDEFGHIJKLMNOPQRSTUVWXYZ12467908  
    最后一个为:08x3p5BeabcdfghijklmnoqrstuvwyzACDEFGHIJKLMNOPQRSTUVWXYZ124679  
    可以看出，第一个是 a-z,A-Z,0-9 的字符串组合，以 8x3p5Be 打头，后面的字符串相应位置去掉 8x3p5Be 字符。  
    后面的字符串做循环左移。  
    同时，产生了新的字符串 "CrackMe" + key1 + key2 + "MegaX" + name + "MegaX"  
代码:

```
    
00F1D160    8BD0            mov     edx, eax
00F1D162    8BCE            mov     ecx, esi
00F1D164    FF15 BC7A9A00   call    dword ptr \[9A7ABC\]

```

    此函数对"CrackMe" + key1 + key2 + "MegaX" + name + "MegaX"字符串进行 md5 的 hash 运算  
    并按每一位的 asc 码产生一个十进制的数字字符串  
    算法如下：  
代码:

```
    CString StrToMd5Hash(char \*pCharDate,int nOutSize)
    {
        HCRYPTPROV hProv = 0;
        HCRYPTHASH hHash;  
        byte hash\[MAXBYTE\];
        DWORD nLen = strlen(pCharDate), nSize = nOutSize; 
        CryptAcquireContext(&hProv, NULL, NULL, PROV\_RSA\_FULL, 0);
    
        CryptCreateHash(hProv, CALG\_MD5, 0, 0, &hHash);
        CryptHashData(hHash,(const byte \*) pCharDate, nLen, 0);
        CryptGetHashParam(hHash, HP\_HASHVAL, (byte \*)hash, &nSize, 0); 
    
        CString OutBuff = pByteToAscStr(hash);

        CryptDestroyHash(hHash);   
        CryptReleaseContext(hProv, 0);
        return OutBuff;
    }
    CString pByteToAscStr(byte \*pByteDate)
    {
        CString str1;
        CString OutBuff = "";
        for (int i=0;i<16;i++)
        {
            str1.Format("%d",pByteDate\[i\]);
            OutBuff += str1;
        }
        return OutBuff;
    }

```

  
    接下来是本题的核心算法  
    我没以四个变量来表示参加运算的字符串  
    strBuff         <--"CrackMe" + key1 + key2 + "MegaX" + name + "MegaX"  
    strMd5          <-- StrToMd5Hash(strBuff)  
    arrayKeyStr     <-- x3E 大小的字符串数组  
    key3            <-- key 的第三部分  
  
    1.取出 key3 的第 i 位，在 arrayKeyStr\[0\]里查找其相应的位置 放如变量 nIndex  
    2.取出 strMd5 的第 i 位  
    3.在 arrayKeyStr\[j\]里取出 nIndex 位(j 从 1 到 0x3E)与 strMd5 的第 i 位 比较  
    4.找到相同的字符时，取出 arrayKeyStr\[j\]的第一个字符记录下  
    如此循环 5 次  
  
    算法如下：  
代码:

```
    CString str = "CrackMe" + keyArray\[0\] + keyArray\[1\] + "MegaX" + strUser + "MegaX";   //构成 strBuff
    CString strmd5 = StrToMd5Hash((LPSTR)(LPCTSTR)str,16);                               //构成 strMd5
    int j=0;
    CString strout = "";
    for (int i=0;i<5;i++)
    {
        int nIndex = keyStrArray\[0\].Find((LPSTR)(LPCTSTR)keyArray\[2\].Mid(i,1),0);       //keyArray\[2\] 表示 key 的第三部分
        CString strIndex = strmd5.Mid(i,1);
        j=0;
        while (strcmp((LPSTR)(LPCTSTR)keyStrArray\[j\].Mid(nIndex,1),(LPSTR)(LPCTSTR)strIndex) != 0)
        {
            j++;
            if (j>=keyStrArray.GetSize())
            {
                j=0;
            }
        }
        if (j>0)
        {
            strout += keyStrArray\[j-1\].Mid(1,1);
        }
        else
        {
            strout += keyStrArray\[j\].Mid(1,1);
        }
    }
    strout.MakeUpper();    //转成大写

```

    strout 是通过算法产生的新字符串。这时，仔细想算认证过程，发现，此题没有唯一的解，key1,key2,key3 是可以随机生成的。关键是 key4  
    我本以为 strout 就是 key4，结果发现不对。。  
    再跟，终于在  
代码:

```
794F955A    8B80 18010000   mov     eax, dword ptr \[eax+118\]
794F9560    FF70 04         push    dword ptr \[eax+4\]
794F9563    8D4C24 10       lea     ecx, dword ptr \[esp+10\]
794F9567    33D2            xor     edx, edx
794F9569    E8 06AFEAFF     call    793A4474   

```

    发现了原因  
    这里的参数是一个字符串数组  
    arry1 = strout //刚才生成的  
    arry2 = key4   //自己输入的  
    arry3 = "MegaX"  
    arry4 = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"  
  
    运行函数后，会返回 0 和 1 这个返回值决定了注册是否成功，修改返回值为 1，爆破成功。  
    分析函数，发现在比较时产生了一个新的字符串。试着把输入的 key4 改为此值，注册成功！！哈哈，运气太好了。。。  
    分析字符串的生成法，发现很简单。算法如下：  
代码:

```
  CString GetKey4(CString &strKey)
  {
    CString str1 = "";
    CString str2 = "MegaX";
    CString str3 = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
    for (int i=0;i<5;i++)
    {
      char cTemp1 = strKey.GetAt(i);
      char cTemp2 = str2.GetAt(i);
      int nIndex =  (cTemp1 + cTemp2 - 1) % str3.GetLength();
      str1 += str3.Mid(nIndex,1);
        }
    return str1;
  }

```

    asc 码相加，取模得到位置，取出即可。  
  
    分析算法完毕，下面看看他的保护机制。  
    1、程序使用了混淆工具混淆了代码  
    2、对 Reflector、PEBrowse、Anakrino、Assembly View、ProcessDasm、VirtualCode、  
    Dasm、FrogsICE、DriverWorkbench、OllyDbg、flyDBG、OllyICE、CrackMenetexe  
    等工具做了判断，当发现其进程，就退出  
    3、代码采用边执行边解压  
    4、代码有很多花指令  
    5、代码有利用 esp 使其返回到其他地址，造成很多地方不能用 F8 跳过。  
    分析保护机制我太菜了，有很多不对的，请指正，谢谢！  
  
  
    科锐学子：沙金