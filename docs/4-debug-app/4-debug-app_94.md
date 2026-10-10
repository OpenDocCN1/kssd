

# 破解 360 密盘的加密之谜

> 原文：[`www.pediy.com/kssd/pediy12/133585.html`](https://www.pediy.com/kssd/pediy12/133585.html)

360 密盘是 360 公司最新出的一款保护用户资料不外泄的加密工具，在黑客防线 2010 年第 12 期的文章上我提到绕过文件透明加密机制的方法，我提到过 360 密盘加密机制，它的就是利用一个文件虚拟成磁盘，也就是 FileDisk 的原理，了解到这一步大家就可以明白 360 密盘的工作原理，通过网络验证来打开加密盘符，大家再仔细想一想，是不是验证成功，就马上解密出原先加过密的镜像文件（也就是虚拟磁盘 360 密盘 X），解密的结果如下图 1：  
 ![](img/4ab21c120454de9a72c919d567f41c70.jpg)  
     由于 360 密盘是内嵌入 360 安全卫士的，所以需要安装 360 安全卫士才会安装 360 密盘，其实可以将 360 密盘直接剥离出 360 安全卫士。在 360 安装目录下建一个 360safe 文件夹，这个其实就是安装 360 安全卫士产生的目录，里面建一个文件夹名为 mipan 的目录，还需要一个 360Common.dll 的公用通信的动态链接库文件，如下图 2 所示：  
![](img/3bfbb4707d1c679d0ea21dbc1ad644a6.jpg)  
     用加载驱动的工具加载 360mipan.sys 驱动，然后就可以使用 360 密盘了。在磁盘分区下会生成文件目录 360mipan，里面就保存着密盘(其实它是以文件的形式保存的，对操作文件模拟成对磁盘的操作，对于用户来说是透明的)。第一次登陆时，需要网络验证，登陆后在 mipan 目录下会生成三个文件：mipan.ini、autologin.data、savepass.dat，mipan.ini 是配置文件里面有几个参数，一个是自动登陆的参数 AutoLogin，如果为 1，下次就直接与 autologin.data 中保存的账号和密码进行验证，不需要再进行网络验证，如果将 AutoLogin 设置为 0，并且删除 autologin.data，将需要再次进行网络验证。如果从 autologin.data 中已加密的数据逆推出原始帐号，这肯定不行，万一不是对称加密或者是单向散列算法呢？进行网络截包分析，因为网络验证，最后要需要进行网络应答的，但是这也会存在问题，因为你伪装网络服务器，回应登录成功的包后，解密加过密的密盘文件，仍然需要正确的原始帐号和密码，或者通过原始的帐号和密码对称加过密后的密文。所以上面这些想法我觉得都不太好，最好的方法就是对使用对称加密的密盘进行分析，看是否能找到一点“蛛丝马迹”。我在测试中发现如果加密一个 1G 的盘，其生成的文件要大于 1G，正好相差 1M（1024\*1024 字节），我使用 WinHex 软件打开密盘文件（该文件保存在密盘目录 360mipan 下扩展名为 360sv 的文件,该文件与 360 卫士挂钩，必须通过登录 360 密盘软件才能删除该文件），然后偏移 1M 的文件位置来查看该\*.360sv 文件，360 密盘的文件格式是 NTFS,我们可以结合 NTFS 磁盘格式来分析，效果如下图 3：  
![](img/80e11d9514ca2d4af4a96a9ebd81c2b9.jpg)  
     很显然这些数据肯定是加过密的,因为我们知道磁盘分区的第一个扇区是磁盘启动扇区 ,紧接着是 MFT,由读取 MFT 来定位磁盘上的文件, BOOTSECTOR 是在最开始的第一个扇区，这里我不做过多分析，有机会在跟大家学习，关键看第二个扇区，这个扇区属于 NTLDR 区域(含有 15 个扇区)，总共这 16 个扇区（包括前面的 BOOTSECTOR）我们称之为 NTFS 的引导模块。从第二个扇区 0x21 开始连续有 48 个字节是 0，从上面的截图我们可以看到也就是 0x220-0x240,这 3 行的数据是一样的，大家仔细看看是不是，”真的是呀,我肯定一定以及确定”，开个玩笑，继续言归正传，假设这三行里就是密钥呢？为什么呢，因为如果是 xor 异或算法，A⁰=A,B⁰=B,只要任何数跟 0 进行异或都是它本身，不会变得，所以我们可以认为 0x220 这一行数就是密钥，如下：  
（E3 01 D9 D1 25 8C B1 27 66 F4 DA 6E 7A 62 58 CF），我们拿这 16 个字节循环来与第一个扇区异或，可以看到熟悉的 BOOTSECTOR 里的一些不变信息比如开头 3 个字节的跳转指令（EB 52 90），后面接着 8 个字节（文件系统格式 NTFS+4 个空格），还有出错信息、引导区结束标志（55 AA）,通过这些分析，我们可以知道 360 密盘就是采取异或的方式加密，并且密钥就在文件中，有了这些信息，我们可以写出解密程序，不久前网上已经有人爆出了破解 360 密盘的程序，但是我还是给大家提供了源代码，因为我无聊,所以把它给逆了，并且也仿写了一个跟它一模一样程序，核心代码如下：  
BOOLEAN decode360mipan(LPCSTR sourcefile,LPCSTR destfile)  
{  
   HMODULE hModule;  
   BOOLEAN result=FALSE;  
   DWORD dwFileAttributes;  
   HANDLE sourcehandle;  
   HANDLE desthandle;  
   LARGE\_INTEGER fileSize={0};  
   LARGE\_INTEGER filepos={0};  
   LONGLONG  MinSize=0x100000;   
   HANDLE maphandle;  
   ULONG  tmpsize;  
   DWORD dwNumberOfBytesToMap = 0x1000000;  
   PVOID mapAddress = NULL;  
   PDWORD  contrastpos1,contrastpos2;   
   int num;  
   BOOL different;  
   PCHAR MiPanKeyOffset;  
   ULONG i;  
   BYTE Viscera;  
   DWORD NumberOfBytesWritten;  
   LARGE\_INTEGER remainbytes={0};  
   LARGE\_INTEGER useedbytes={0};  
   float percent;  
  
   dwFileAttributes = GetFileAttributesA(sourcefile);  
   SetFileAttributesA(sourcefile, 0x80u);  
   sourcehandle = CreateFileA(sourcefile,GENERIC\_READ,FILE\_SHARE\_READ, NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,NULL);  
   if (sourcehandle == INVALID\_HANDLE\_VALUE)  
   {  
      SetFileAttributesA(sourcefile, dwFileAttributes);  
   }  
   else  
   {  
       hModule = LoadLibraryA("kernel32.dll");  
     if (!hModule)  
     {  
        printf("LoadDll kernel32.dll failed!\\n ");  
      FreeLibrary(hModule);  
      return FALSE;  
     }  
    DWORD \_GetFileSizeEx=(DWORD)GetProcAddress(hModule,"GetFileSizeEx");  
  
    \_asm{  
            push esi  
      lea esi,fileSize  
      push esi  
      push sourcehandle  
      call \_GetFileSizeEx    
            pop esi  
    }  
  
   //至少文件大小要大于 1M  
   if (fileSize.QuadPart > MinSize)  
   {  
      maphandle = CreateFileMapping(sourcehandle,NULL,PAGE\_READONLY,0,0,NULL);  
          
    if (maphandle)  
    {  
        //解密的文件句柄  
      desthandle  = CreateFileA(destfile,GENERIC\_WRITE,0,NULL,CREATE\_ALWAYS,FILE\_ATTRIBUTE\_NORMAL,NULL);  
      if (desthandle != INVALID\_HANDLE\_VALUE)  
      {  
               //1M 大小  
        tmpsize =  1048576;   
          //从文件起始偏移的位置开始读取文件  
        filepos.QuadPart = 1048576i64;  
  
           while(1)  
           {  
              if (filepos.HighPart >fileSize.HighPart)  
                 break;  
            if (filepos.HighPart >=fileSize.HighPart && tmpsize >= fileSize.LowPart)  
               break;  
           //从已加密的 360 密盘文件中每次读取 16M 内容来解密  
mapAddress=  MapViewOfFile(maphandle,FILE\_MAP\_READ,filepos.HighPart,filepos.LowPart,dwNumberOfBytesToMap);  
           if (mapAddress)  
           {  
              //得到 360 密盘的密钥，起始很简单，xor 算法中任何数跟 0 异或得到的是自己，这就是获取 360 密钥原因，主要是 ntfs 稀疏文件存在的问题，这不是密码的问题，而是设计上的不足  
          if (filepos.LowPart == 1048576 && !filepos.HighPart)  
              {  
                 memcpy((void\*)&MiPanKey,(void \*)((PCHAR)mapAddress+0x220),16);     
             num = 4;  
                       contrastpos1 = (DWORD\*)((PCHAR)mapAddress+0x230);  
             //连续 3 个的 16 字节都是 360 密盘密钥，都相等，   
             different =TRUE;  
                       MiPanKeyOffset =(PCHAR)&MiPanKey;  
                       do   
                       {  
                         if (!num)  
                         break;  
  
  different=(\*(DWORD\*)contrastpos1 == \*(DWORD\*)MiPanKeyOffset) ? TRUE : FALSE;  
               num--;    
             MiPanKeyOffset=(PCHAR)MiPanKeyOffset +4;  
                         contrastpos1++;  
             } while(different);  
            
             if (!different)  
              break;  
               
             //再比较偏移 0x20 的 16 个字节  
             num = 4;     
                       contrastpos2 = (DWORD\*)((PCHAR)mapAddress+0x240);   
                       different =TRUE;  
                       MiPanKeyOffset =(PCHAR)&MiPanKey;  
                       do   
                       {  
                         if (!num)  
                         break;  
  
different=(\*(DWORD\*)contrastpos2 == \*(DWORD\*)MiPanKeyOffset) ? TRUE : FALSE;  
               num--;    
             MiPanKeyOffset=(PCHAR)MiPanKeyOffset +4;  
                         contrastpos2++;  
             } while(different);  
  
             if (!different)  
             break;  
          }  
          //验证得到密钥后，就将前面映射的内容解密后写到解密文件中  
          i=0;  
          while (i<dwNumberOfBytesToMap)  
          {  
            Viscera = \*((PCHAR)mapAddress+i);  
            Viscera = (BYTE)(Viscera^MiPanKey\[i&0xF\]);  
            Outbuffer\[i\]=Viscera;  
            i++;  
          }       
  WriteFile(desthandle,(PVOID)&Outbuffer,dwNumberOfBytesToMap,&NumberOfBytesWritten,NULL);  
         UnmapViewOfFile(mapAddress);  
         }  
          tmpsize+=dwNumberOfBytesToMap;  
          filepos.QuadPart = tmpsize;   
            
        remainbytes.QuadPart =fileSize.QuadPart - 1048576i64;  
                useedbytes.QuadPart +=dwNumberOfBytesToMap;  
                percent = (float)(useedbytes.QuadPart) / (float)(remainbytes.QuadPart);  
                  
      
        printf("Process...%d%%\\r",(int)(percent\*100));  
         }  
  
         if (tmpsize == fileSize.LowPart)  
         {  
           printf("\\n");  
         result = TRUE;  
         }  
        CloseHandle(desthandle);  
      }  
            CloseHandle(maphandle);  
    }   
     CloseHandle(sourcehandle);  
   }  
       SetFileAttributesA(sourcefile, dwFileAttributes);  
   }   
return result;  
}  
    代码我就不多解释了，自己看注视，我认为已经写的很详细了，大家要养成好的代码风格，以后自己再看的时候也会方便些，其实我是在说自己。下面就演示一下解密密盘，不好意思，这里我偷懒一下，我写的是控制台程序，不过原来的也是个控制台程序，示意图 4 如下：  
![](img/628dc951f5d88fbcd6b079c563482cdb.jpg)  
    使用 WinImage 软件将 hacker.img 镜像文件打开，示意图 5 如下啊：  
![](img/b6092b1145b70bdace18286183618c31.jpg)  
    
     文章到这里就结束了，但是我们可以看到造成破解密码的问题并不是 360 密盘采取的异或算法出现问题，而是由于 NTFS 文件格式所引发的问题，最新的 360 密盘应该已经解决这个问题，除非有些用户还没有来得及升级，其实解决的办法很简单，如果遇到连续 16 个字节的 0 时，不进行加密，跳过即可，或者用前面加密数据做为后面加密的密钥，但是这样做并不代表就一定安全，完全可以编写字典数据来进行异或测试、甚至对比分析已知密文、已知明文来进行解密，当然这样难度更大。实际上加密算法越简单，那么加密速度也就越快，对于用户来说，他当然希望越快越好，因为加密过程对于用户来说是透明的，但是要保证加密数据的安全，就应该使用更复杂的加密算法，但是复杂的加密算法则需要大量的时间，因为加密步骤复杂，相应解密步骤也就复杂，对于对称加密算法而言，这是无法调和的矛盾。不管再复杂的加密算法，除非其本身不存在漏洞，不然就有可能被攻破。但是我们知道，世界上没有绝对安全的加密算法，只要有足够长的时间、足够多的集群服务器进行网格计算破解，那么任何密钥都有可能被攻破，题外话。相信大家已经能从中明白点什么了。不足之处，敬请黑防读者们不吝赐教！  
  
本文发表在黑客防线 2011 年第 4 期  
  
附上源程序代码

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || Decode360mipan.rar  
 |