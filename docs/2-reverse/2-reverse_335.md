

# 详解定位特征码方法附带源码

> 原文：[`www.pediy.com/kssd/pediy11/121198.html`](https://www.pediy.com/kssd/pediy11/121198.html)

原来的 ID(keellisa) 密码丢了，现在无法找回了。下面的东西是非常基础的东西，给新手一个参考吧。  
      
    1、定位特征码的原因      
      无论破解还是做其他与逆向相关的东西，可能都需要定位特征码，使用绝对地址记住函数地址或者全局变量时，如果对应程序更新后，这个地址就无效了。此时可以通过特征码的方式定位它们，从  
  
而减少地址无效的概率。  
  
    2、定位特征码的方法  
    定位特征码的关键是找到的特征码随目标程序的更新而不改变。但是前提是特征码在对应 DLL 或者 EXE 文件中是唯一的。  
  
    1）如果特征码的代码带有特殊的常量，并且这个常量不是目标程序的内存地址时，优先选取其为特征码。一般常量在程序中很少会改变。  
  
     例如如下为某函数的反汇编代码，获取这个函数的地址，选取"83 FE 2E 74 38"为特征码是比较合适的。-5 地址处就为函数起始地址。   
  
  $-5      > .  56            PUSH ESI                            ;函数地址       
  $-4      > .  8B7424 08     MOV ESI,DWORD PTR SS:\[ESP+8\]  
  $ ==>    > .  83FE 2E       CMP ESI,2E        ;2e 为特殊常量  
  $+3      > .  74 38         JE 0BCCCCA2        ;短跳转  
  $+5      > .  83FE 2D       CMP ESI,2D          
  $+8      > .  74 33         JE 0BCCCCA2  
  $+A      > .  83FE 33       CMP ESI,33  
  
    2）如果特征码带有短跳转的代码，优先选取。因为只要你选取的特战码以及从选取的特征码地址到跳转的地址之间随目标程序更新变化的概率较少。如上面的短跳转指令  
  
    3）如果特征码带有结构体或者类变量的对应偏移的代码，可以选取。一般只要结构体或者类变量不增加成员，就不会改变。  
  
    例如如下代码，要获取全局变量 "106d83b8"的地址，选取"88 46 40 74 45"  为特征码是比较合适的。+0x22 偏移处就为目标全局变量地址。  
      
  $ ==>    >  8846 40         MOV BYTE PTR DS:\[ESI+40\],AL          ;结构体成员变量  
  $+3      >  74 45           JE 10572F00        ;短跳转指令  
  $+5      >  8B11            MOV EDX,DWORD PTR DS:\[ECX\]  
  $+7      >  8B42 30         MOV EAX,DWORD PTR DS:\[EDX+30\]  
  $+A      >  53              PUSH EBX  
  $+B      >  8B1D AC406E10   MOV EBX,DWORD PTR DS:\[106E40AC\]  
  $+11     >  FFD0            CALL EAX  
  $+13     >  83BB E4010000 0>CMP DWORD PTR DS:\[EBX+1E4\],0  
  $+1A     >  894424 18       MOV DWORD PTR SS:\[ESP+18\],EAX  
  $+1E     >  74 29           JE 10572EFF  
  $+20     >  8B0D B8836D10   MOV ECX,DWORD PTR DS:\[106D83B8\]    ;目标全局变量  
    
    4）如果特征码位置，没有以上代码，可以采用特殊指令，不带有绝对地址的代码为特征码。  
  
  下面是某个程序的消息处理函数的代码，可以采用"53 55 8B 6C 24 2C 56 57 FF D2"为特征码。  
  $-19     >/$  83EC 20       SUB ESP,20                                 
  $-16     >|.  A1 6881E40B   MOV EAX,DWORD PTR DS:\[BE48168\]  
  $-11     >|.  33C4          XOR EAX,ESP  
  $-F      >|.  894424 1C     MOV DWORD PTR SS:\[ESP+1C\],EAX  
  $-B      >|.  8B0D 6007EA0B MOV ECX,DWORD PTR DS:\[BEA0760\]  
  $-5      >|.  8B01          MOV EAX,DWORD PTR DS:\[ECX\]  
  $-3      >|.  8B50 0C       MOV EDX,DWORD PTR DS:\[EAX+C\]  
  $ ==>    >|.  53            PUSH EBX        ;特征码  
  $+1      >|.  55            PUSH EBP  
  $+2      >|.  8B6C24 2C     MOV EBP,DWORD PTR SS:\[ESP+2C\]  
  $+6      >|.  56            PUSH ESI  
  $+7      >|.  57            PUSH EDI  
  $+8      >|.  FFD2          CALL EDX  
  
    3、定位特征码注意事项  
    1） 特征码中不能带有绝对地址。  
   2）特征码必须在对应模块中是唯一的，否则搜索到的特征码地址可能是错误的。  
  
    4、搜索特征码  
     下面是我之前写的搜索特征码的代码，可以参考。  
  
// ==========================================================  
// 函数名称：SearchDataFromProcess  
// 函数用途：从指定模块中搜索指定字节集的数据  
// 输入参数：BYTE\* pSearch      要搜索的字节集  
//     int size        要搜索字节集的大小  
//                dllName                                                 要搜索的模块名  
// 返    回：搜索到的进程地址  
// ==========================================================  
int SearchDataFromProcessByDllName(BYTE\* pSearch, int size, char\* dllName)  
{  
  int i,j;  
  DWORD OldProtect;  
  BYTE\* pOrg;  
  BYTE\* pPare;  
  MODULEINFO mMoudleInfo;  
  HMODULE  hMoudle;  
  
  
  //获取模块地址  
  hMoudle = GetModuleHandle(dllName);  
  if(NULL == hMoudle)  
  {  
    hMoudle = LoadLibraryA(dllName);  
    if(NULL == hMoudle)  
    {  
      return 0;  
    }  
  }  
  
  pOrg = (BYTE\*)hMoudle;  
    
  
  //更改模块保护属性  
  VirtualProtectEx(GetCurrentProcess(), hMoudle,1,PAGE\_EXECUTE\_READWRITE,&OldProtect);  
    
  //得到模块大小  
  GetModuleInformation(GetCurrentProcess(), hMoudle,&mMoudleInfo,sizeof(mMoudleInfo));  
  
  //查找指定字节集  
  for(i = 0; i <(int) mMoudleInfo.SizeOfImage; i++)  
  {  
    pPare =pOrg + i;  
  
    for(j = 0; j < size; j++)  
    {  
      if(pPare\[j\] !=   pSearch\[j\])  
      {  
        break;  
      }  
    }  
    //如果找到则返回找到的首地址  
    if(j == size)  
    {  
      VirtualProtectEx(GetCurrentProcess(), hMoudle,1,OldProtect,NULL);  
  
      return (int)(pPare);  
    }  
  }  
    
  //直接退出  
  VirtualProtectEx(GetCurrentProcess(), hMoudle,1,OldProtect,NULL);    
  return 0;  
}  
  
// ==========================================================  
// 函数名称：GetFunAddr  
// 函数用途：得到某函数地址  
// 输入参数：NONE  
// 返    回：NONE  
// ==========================================================  
/\*  
$-5      > .  56            PUSH ESI                                   
$-4      > .  8B7424 08     MOV ESI,DWORD PTR SS:\[ESP+8\]  
$ ==>    > .  83FE 2E       CMP ESI,2E  
$+3      > .  74 38         JE SHORT xx.0BCCCCA2  
$+5      > .  83FE 2D       CMP ESI,2D  
$+8      > .  74 33         JE SHORT xx.0BCCCCA2  
$+A      > .  83FE 33       CMP ESI,33  
83 FE 2E 74 38  
\-5  
\*/  
bool GetFunAddr(void)  
{  
  int Addr;  
  BYTE Data\[\]={0x83 ,0xFE ,0x2E ,0x74 ,0x38};   
  
  //搜索特征码  
  Addr = SearchDataFromProcessByDllName(Data, sizeof(Data), "xx.dll");        
    
  //判断是否查找到特征码  
  if( 0 == Addr)  
  {  
    return false;  
  }  
    
  //取特征码  
  g\_Addr\_Function = Addr - 0x05;  
    
  return true;  
} 

我的 keellisa ID 何时能找回啊![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")  
  
  
     补充一下：对于定位函数地址的小技巧：  
  
   1）如果函数本身内部没有合适的代码作为特征码，可以在调用它的函数中定位到调用目标函数的代码，然后再通过计算获得目标函数的地址；  
  
   下面是个例子：  
// ==========================================================  
// 函数名称：GetFunAddr  
// 函数用途：  
// 输入参数：NONE  
// 返    回：NONE  
// ==========================================================  
/\*  
$-1A     >|.  A1 748D3D0F                 MOV EAX,DWORD PTR DS:\[F3D8D74\]  
$-15     >|.  8B78 34                     MOV EDI,DWORD PTR DS:\[EAX+34\]  
$-12     >|.  8B81 B8010000               MOV EAX,DWORD PTR DS:\[ECX+1B8\]  
$-C      >|.  85C0                        TEST EAX,EAX  
$-A      >|.  897C24 1C                   MOV DWORD PTR SS:\[ESP+1C\],EDI  
$-6      >|.  0F8C 94030000               JL 0F39CE58  
$ ==>    >|.  83F8 06                     CMP EAX,6  
$+3      >|.  0F8D 8B030000               JGE 0F39CE58  
$+9      >|.  8B1485 B0723C0F             MOV EDX,DWORD PTR DS:\[EAX\*4+F3C72B0\]  
$+10     >|.  81C1 90010000               ADD ECX,190  
$+16     >|.  8D5C24 14                   LEA EBX,DWORD PTR SS:\[ESP+14\]  
$+1A     >|.  895424 14                   MOV DWORD PTR SS:\[ESP+14\],EDX  
$+1E     >|.  E8 190C0000                 CALL 0F39D700                   ;  目标函数地址  
83 F8 06 0F 8D 8B 03 00 00  
+1e   
+1f  
(函数地址)0F39D700 = addr + 1e + \[addr + 1f\] + 5  
\*/  
bool GetFunAddr(void)  
{  
  int Addr;  
  int\*pAddr;  
  
  BYTE Data\[\]={0x83 ,0xF8 ,0x06 ,0x0F ,0x8D ,0x8B ,0x03 ,0x00 ,0x00};   
  
  //搜索特征码  
  Addr = SearchDataFromProcessByDllName(Data, sizeof(Data), "AchieveSystem.dll");        
    
  //判断是否查找到特征码  
  if( 0 == Addr)  
  {  
    return false;  
  }  
  //取特征码  
  pAddr = (int\*)(Addr - 0x19);  
  G\_Addr\_Fun = Addr + 0x1e + \*((int\*)(Addr + 0x1f)) + 5;  
  
  return true;  
  
}  
  
   2）如果目标函数为虚函数，可以通过获得对应类全局变量，然后获得类虚函数表，通过虚函数在虚函数表的偏移获得函数地址。；