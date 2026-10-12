
# 编写 PPC 程序的 loader--Intumical1.0126

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-445.htm`](https://www.pediy.com/kssd/pediy07/pediy7-445.htm)

目标程序：  IntumiCal V 1。0126  

下载页面：  `http://www.intumi.com/downloads/IntumiCal_10_126.zip`  

软件语言：  英文  

软件类别：  国外软件 / 共享版 / 系统其它  

应用平台：  PocketPC  

软件介绍：  IntumiCal is a replacement for the built-in Pocket PC calendar.  

We found Pocket Outlook's user interface too clumsy, and other alternatives try to be everything to everybody. There was just nothing as simple and elegant as the Palm calendar available for the Pocket PC, so we sat down to create a no-nonsense, easy to use, and nice looking calendar.  

IntumiCal's powerful, yet easily accessible user interface has been deliberately and carefully designed to make day-to-day calendaring tasks a breeze. And of course, IntumiCal is fully compatible with ActiveSync and Outlook.  

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教  

【调试环境】：WinXP、IDA pro 4.8  Microsoft eMbedded Visual C++ 4.0、Microsoft eMbedded Visual C++ 4.0 Service Pack 4 、 Pocket PC 2003 SDK、Windows Mobile 2003 Second Edition 模拟器包 for Pocket PC  

             还有 Visual Studio 2005（不是必须）、010editor 、Charmed。  

—————————————————————————————————  

## 【分析过程】：

### 1、搭建调试环境与基本介绍

`http://www.csdn.net/subject/WMDevTools/`  

上面写得很清楚，就按照 Windows Mobile 2003 Second Edition 的开发环境搭建就行了。  

如果有条件就直接装 vs2005 调试起来更方便.  

顺便到网上找一下 arm 的汇编指令解释。  

2、沐浴  

3、斋戒  

4、祭天  

5、做法、请出 IDA pro 反编译一下.  

6、诵经。  

7、IDA 分析完毕后查看字符串信息
```
aMicrosoftBaseC unicode 0, <Microsoft Base Cryptographic Provider v1.0>,0
.data:00057A00                                         ; DATA XREF: .text:szProvidero
```
应该差不多就是与注册算法相关的信息了。  

往上溯
```
.text:00030698 ; LPCWSTR szProvider
.text:00030698 szProvider      DCD aMicrosoftBaseC     ; DATA XREF: sub_30570+14r
.text:00030698                                         ; "Microsoft Base Cryptographic Provider v"...
```
继续往上溯
```
.text:00030570 sub_30570                               ; CODE XREF: sub_30FB0+64p
.text:00030570

.text:00030570 pbData          = -0x2C
.text:00030570 pdwDataLen      = -0x28
.text:00030570 dwBufLen        = -0x24
.text:00030570 var_20          = -0x20
.text:00030570 hProv           = -0x1C

.text:00030570                 STMFD   SP!, {R4-R8,LR}
.text:00030574                 SUB     SP, SP, #0x14   ; pbData
.text:00030578                 MOV     R4, R0
.text:0003057C                 MOV     R5, R1
.text:00030580                 MOV     R7, R2
.text:00030584                 LDR     R2, =aMicrosoftBaseC ; szProvider //指向刚才那个字符串
.text:00030588                 MOV     R0, #0xF0000000
.text:0003058C                 STR     R0, [SP,#0x2C+pbData]
.text:00030590                 MOV     R8, #0
.text:00030594                 MOV     R3, #1          ; dwProvType
.text:00030598                 STR     R8, [SP,#0x2C+hProv]
.text:0003059C                 MOV     R1, #0          ; szContainer
.text:000305A0                 ADD     R0, SP, #0x2C+hProv ; phProv
.text:000305A4                 BL      CryptAcquireContextW
```
继续往上溯  

这个就是算法 call
```
.text:00030FB0 sub_30FB0                               ; CODE XREF: sub_311F4+5Cp
.text:00030FB0                                         ; sub_312E0+24p
.text:00030FB0

.text:00030FB0 wcstr           = -0x124
.text:00030FB0 var_120         = -0x120
.text:00030FB0 var_11C         = -0x11C
.text:00030FB0 var_118         = -0x118
.text:00030FB0 var_114         = -0x114
.text:00030FB0 mbstr           = -0x10C

.text:00030FB0                 STMFD   SP!, {R4,R5,LR}
.text:00030FB4                 SUB     SP, SP, #0x118
.text:00030FB8                 MOV     R5, R0
.text:00030FBC                 LDR     R1, =aIntumical20051
.text:00030FC0                 ADD     R0, SP, #0x124+wcstr
.text:00030FC4                 BL      CString::CString(ushort const *)
.text:00030FC8                 MOV     R2, #0x100      ; size_t
.text:00030FCC                 MOV     R1, #0          ; int
.text:00030FD0                 ADD     R0, SP, #0x124+mbstr ; void *
.text:00030FD4                 BL      memset
.text:00030FD8                 LDR     R1, [SP,#0x124+wcstr] ; wcstr
.text:00030FDC                 MOV     R2, #0x100      ; count
.text:00030FE0                 ADD     R0, SP, #0x124+mbstr ; mbstr
.text:00030FE4                 BL      wcstombs
.text:00030FE8                 ADD     R0, SP, #0x124+var_120
.text:00030FEC                 BL      sub_30310
.text:00030FF0                 LDR     R1, [SP,#0x124+var_120] ; wcstr
.text:00030FF4                 MOV     R3, #8
.text:00030FF8                 MOV     R2, #8          ; count
.text:00030FFC                 STR     R3, [SP,#0x124+var_118]
.text:00031000                 ADD     R0, SP, #0x124+var_114 ; mbstr
.text:00031004                 BL      wcstombs
.text:00031008                 ADD     R2, SP, #0x124+var_118
.text:0003100C                 ADD     R1, SP, #0x124+var_114
.text:00031010                 ADD     R0, SP, #0x124+mbstr
.text:00031014                 BL      sub_30570                          //生成注册码
.text:00031018                 MOV     R4, R0
.text:0003101C                 MOV     R3, #1
.text:00031020                 MOV     R2, #4
.text:00031024                 MOV     R1, R4
.text:00031028                 ADD     R0, SP, #0x124+var_11C
.text:0003102C                 BL      sub_3047C      //字符串处理
.text:00031030                 MOV     R0, R4
.text:00031034                 BL      operator delete(void *)
.text:00031038                 ADD     R1, SP, #0x124+var_11C
.text:0003103C                 MOV     R0, R5
.text:00031040                 BL      CString::CString(CString const &)
.text:00031044                 ADD     R0, SP, #0x124+var_11C
.text:00031048                 BL      CString::~CString(void)
.text:0003104C                 ADD     R0, SP, #0x124+var_120
.text:00031050                 BL      CString::~CString(void)
.text:00031054                 ADD     R0, SP, #0x124+wcstr
.text:00031058                 BL      CString::~CString(void)
.text:0003105C                 MOV     R0, R5
.text:00031060                 ADD     SP, SP, #0x118
.text:00031064                 LDMFD   SP!, {R4,R5,PC}
```
/////////////////////////////////////////////////////////////////////////////////////  

//到此我们已经找到了算法休息一下  

////////////////////////////////////////////////////////////////////////////////////  

分析一下 `sub_30570`
```
.text:00030570 sub_30570                               ; CODE XREF: sub_30FB0+64p
.text:00030570

.text:00030570 pbData          = -0x2C
.text:00030570 pdwDataLen      = -0x28
.text:00030570 dwBufLen        = -0x24
.text:00030570 var_20          = -0x20
.text:00030570 hProv           = -0x1C

.text:00030570                 STMFD   SP!, {R4-R8,LR}
.text:00030574                 SUB     SP, SP, #0x14   ; pbData
.text:00030578                 MOV     R4, R0
.text:0003057C                 MOV     R5, R1
.text:00030580                 MOV     R7, R2
.text:00030584                 LDR     R2, =aMicrosoftBaseC ; szProvider
.text:00030588                 MOV     R0, #0xF0000000
.text:0003058C                 STR     R0, [SP,#0x2C+pbData]
.text:00030590                 MOV     R8, #0
.text:00030594                 MOV     R3, #1          ; dwProvType
.text:00030598                 STR     R8, [SP,#0x2C+hProv]
.text:0003059C                 MOV     R1, #0          ; szContainer
.text:000305A0                 ADD     R0, SP, #0x2C+hProv ; phProv
.text:000305A4                 BL      CryptAcquireContextW
.text:000305A8                 CMP     R0, #0
```



## 汇编代码片段

```asm
.text:000305AC                  BEQ     GoToDead
.text:000305B0                  LDR     R0, [SP,#0x2C+hProv]
.text:000305B4                  MOV     R1, R4
.text:000305B8                  BL      sub_3069C
.text:000305BC                  MOVS    R6, R0
.text:000305C0                  BEQ     GoToDead
.text:000305C4                  LDR     R0, [R7]
.text:000305C8                  ADD     R1, SP, #0x2C+var_20
.text:000305CC                  STR     R1, [SP,#0x2C+pdwDataLen]
.text:000305D0                  MOV     R3, #0          ; dwFlags
.text:000305D4                  STR     R0, [SP,#0x2C+var_20]
.text:000305D8                  MOV     R2, #1          ; Final
.text:000305DC                  STR     R0, [SP,#0x2C+dwBufLen]
.text:000305E0                  MOV     R1, #0          ; hHash
.text:000305E4                  MOV     R0, R6          ; hKey
.text:000305E8                  STR     R8, [SP,#0x2C+pbData]
.text:000305EC                  BL      CryptEncrypt
.text:000305F0                  CMP     R0, #0
.text:000305F4                  BEQ     GoToDead
.text:000305F8                  LDR     R0, [SP,#0x2C+var_20]
.text:000305FC                  BL      operator new(uint)
.text:00030600                  LDR     R2, [SP,#0x2C+var_20] ; size_t
.text:00030604                  MOV     R1, #0          ; int
.text:00030608                  MOV     R4, R0
.text:0003060C                  BL      memset
.text:00030610                  LDR     R2, [R7]        ; size_t
.text:00030614                  MOV     R1, R5          ; void *
.text:00030618                  MOV     R0, R4          ; void *
.text:0003061C                  BL      memcpy
.text:00030620                  LDR     R0, [R7]
.text:00030624                  LDR     R3, [SP,#0x2C+var_20]
.text:00030628                  MOV     R2, #1          ; Final
.text:0003062C                  STR     R0, [SP,#0x2C+var_20]
.text:00030630                  ADD     R0, SP, #0x2C+var_20
.text:00030634                  STR     R3, [SP,#0x2C+dwBufLen]
.text:00030638                  MOV     R1, #0          ; hHash
.text:0003063C                  STR     R0, [SP,#0x2C+pdwDataLen]
.text:00030640                  MOV     R3, #0          ; dwFlags
.text:00030644                  MOV     R0, R6          ; hKey
.text:00030648                  STR     R4, [SP,#0x2C+pbData]
.text:0003064C                  BL      CryptEncrypt
.text:00030650                  CMP     R0, #0
.text:00030654                  BNE     loc_3066C
.text:00030658                  MOV     R0, R4
.text:0003065C                  BL      operator delete(void *)
.text:00030660
.text:00030660  GoToDead                        ; CODE XREF: sub_30570+3Cj
.text:00030660                                 ; sub_30570+50j ...
.text:00030660                  MOV     R0, R8
.text:00030664                  ADD     SP, SP, #0x14
.text:00030668                  LDMFD   SP!, {R4-R8,PC}
.text:0003066C  ; ---------------------------------------------------------------------------
.text:0003066C
.text:0003066C  loc_3066C                       ; CODE XREF: sub_30570+E4j
.text:0003066C                  LDR     R0, [SP,#0x2C+var_20]
.text:00030670                  STR     R0, [R7]
.text:00030674                  MOV     R0, R6          ; hKey
.text:00030678                  BL      CryptDestroyKey
.text:0003067C                  LDR     R0, [SP,#0x2C+hProv] ; hProv
.text:00030680                  CMP     R0, #0
.text:00030684                  MOVNE   R1, #0          ; dwFlags
.text:00030688                  BLNE    CryptReleaseContext
.text:0003068C                  MOV     R0, R4
.text:00030690                  ADD     SP, SP, #0x14
.text:00030694                  LDMFD   SP!, {R4-R8,PC}
.text:00030694  ; End of function sub_30570
```

## C 伪代码

```c
DWORD dwProvType = PROV_RSA_FULL;

DWORD dwFlags = 0;

HCRYPTPROV hProv=NULL;

HCRYPTHASH hHash=NULL;

wchar_t szContainer[] = NULL;

wchar_t szProvider[] = L"Microsoft Base Cryptographic Provider v1.0";

if(!CryptAcquireContextW(&hProv,NULL,szProvider,dwProvType,dwFlags))
{
  ASSERT(0);
}

ASSERT(hProv);

ALG_ID Algid = CALG_MD5;//0x8003;
////////////////////////////
//BL      sub_3069C

if(!CryptCreateHash(hProv,Algid,0,0,&hHash))
{
  ASSERT(0);
}

if(!CryptHashData(hHash,szInpuString,nLen,0))
{
  ASSERT(0);
}

DWORD dwDataLen = 0x00000010;
BYTE *bData;
//////////////////////////
CryptEncrypt(HCRYPTKEY hKey,0, true, 0,BYTE* pbData,&dwDataLen, DWORD dwBufLen);

bData = new BYTE[dwDataLen];

CryptEncrypt(HCRYPTKEY hKey,0, true, 0,BYTE* pbData,&dwDataLen, DWORD dwBufLen);
```

注册码就这样出来了

具体数值调试一下就出来了

```asm
.text:00031014                  BL      sub_30570
.text:00031018                  MOV     R4, R0
.text:0003101C                  MOV     R3, #1
.text:00031020                  MOV     R2, #4
.text:00031024                  MOV     R1, R4
.text:00031028                  ADD     R0, SP, #0x124+var_11C
.text:0003102C                  BL      sub_3047C
.text:00031030                  MOV     R0, R4
.text:00031034                  BL      operator delete(void *)
.text:00031038                  ADD     R1, SP, #0x124+var_11C
.text:0003103C                  MOV     R0, R5
.text:00031040                  BL      CString::CString(CString const &)
.text:00031044                  ADD     R0, SP, #0x124+var_11C
.text:00031048                  BL      CString::~CString(void)
.text:0003104C                  ADD     R0, SP, #0x124+var_120
.text:00031050                  BL      CString::~CString(void)
.text:00031054                  ADD     R0, SP, #0x124+wcstr
.text:00031058                  BL      CString::~CString(void)
.text:0003105C                  MOV     R0, R5
.text:00031060                  ADD     SP, SP, #0x118
```

断点设置在 `00031030`，看一下 `r3` 寄存器看他偏移 `0x0c` 的地方就是注册码的 unicode

`pediy` 一下，让他弹出注册码。

## 修改方法

```asm
.text:00031030                  ADD     R0, R3, #0xC
.text:00031034                  MOV     R3, #0          ; uType
.text:00031038                  MOV     R2, R0          ; lpCaption
.text:0003103C                  MOV     R1, R0          ; lpText
.text:00031040                  MOV     R0, #0          ; hWnd
.text:00031044                  BL      MessageBoxW
```

修改方法

先用 `lodepe` 的 `flc` 转化一下要修改的地址，用 `010editor` 打开，

```asm
.text:00031030 0C 00 83 E2                     ADD     R0, R3, #0xC
```

第一个字节 `0c` 代表要加的数字 第二个字节 `00` 的高位表示目的寄存器，第三个字节 `83` 中的低位表示源寄存器，第四个字节表示操作符

```asm
.text:00031034 00 30 A0 E3                     MOV     R3, #0          ; uType
.text:00031038 00 20 A0 E1                     MOV     R2, R0          ; lpCaption
.text:0003103C 00 10 A0 E1                     MOV     R1, R0          ; lpText
.text:00031040 00 00 A0 E3                     MOV     R0, #0          ; hWnd
.text:00031044 F1 77 00 EB                     BL      MessageBoxW
```

到现在还不知道 `BL` 的偏移是怎么算的就用 `Charmed` 修改一下就行了。`Charmed` 虽说可以直接修改成汇编但是在我的店拿上它的成功是随机的。-\_-''郁闷

## Loader 代码

为了开发调试的快一点安装了一下 `vs2005`

记得是 `ppc` 的工程

直接开工

```c
STARTUPINFO si;
PROCESS_INFORMATION pi;
wchar_t  exe[MAX_PATH],*filepath;
HANDLE hFile;

GetModuleFileName(GetModuleHandle(NULL),exe,MAX_PATH);
filepath = wcsrchr(exe,L'\\');
*filepath =0;
wcscat(exe,L"\\IntumiCal.exe");
DWORD fk;

ZeroMemory( &si, sizeof(si) );
si.cb = sizeof(si);
ZeroMemory( &pi, sizeof(pi) );

byte destcode[] = {0x0C,0x00,0x83,0xE2,//add r0,r3,#0c
    0x00,0x30,0xA0,0xE3,//mov r3,#0
    0x00,0x20,0xA0,0xE1,//mov r2,r0
```



```c
    0x00,0x10,0xA0,0xE1,//mov r1,r0  

    0x00,0x00,0xA0,0xE3,//mov r0,#0  

    0xF1,0x77,0x00,0xEB};//messagebox  
```
```
// Start the child process.
```
```c
if(!CreateProcess(  exe,  // No module name (use command line).
    NULL,// Command line.
    NULL,             // Process handle not inheritable.
    NULL,             // Thread handle not inheritable.
    FALSE,            // Set handle inheritance to FALSE.
    CREATE_SUSPENDED,                // No creation flags.
    NULL,             // Use parent's environment block.
    NULL,             // Use parent's starting directory.
    &si,              // Pointer to STARTUPINFO structure.
    &pi )
)
{
    DWORD i = GetLastError();
    ::MessageBox(NULL, TEXT("CreateProcess failed."),TEXT("Error"), MB_OK);  
    //return 0;  
}
else
{
    WriteProcessMemory(pi.hProcess,(LPVOID)0x31030,destcode,24,&fk);
    ResumeThread(pi.hThread);
}
```

记住 `ppc` 是没有

```c
DWORD GetCurrentDirectory(
  DWORD nBufferLength,
  LPTSTR lpBuffer
);
```

好的到此收工，注册机太麻烦。。。主要是懒得再看代码了-\_-''用 `loader` 就可以了。

---------------------------------------------------------------------------------------------

WiNrOOt

winroot@gmail.com

4dd466d0e1e0cc123afbe504da2462ca
