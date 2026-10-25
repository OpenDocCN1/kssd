
# 用软件本身的 DLL 函数写 keygen 手记 [看雪学院 2006 金秋读书季]

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-350.htm`](https://www.pediy.com/kssd/pediy08/pediy8-350.htm)

## 用软件本身的 DLL 函数写 keygen 手记

**readyu**

2006-10-03

**目标软件**：一个小软件，名字不方便列出

**使用工具**：`OllyDbg`, `C32asm`, `Winhex` (查看内存), `PEID`, `VC++`

好些天以前做的了，突然想起来。觉得这个方法比较典型而且也简单实用，整理出来，与大家分享。我尽量写详细，基本上会编程就能看明白。

### 本方法适合：

a. 软件本身有比较规范的计算 key 函数，采用真，假 key 比较型；

b. 此函数没有依赖垃圾全局变量，可以方便调用；

c. 软件加了壳，不好 patch 的。

**可以改进的地方：**

函数地址可采用搜索内存获取，可以比较灵活。

目前的代码是死板定位的。

### 详细步骤：

1. `Setup`, 运行发现采用常见的方法，SN 注册。

   随便输入些字符，跳出“Invalid License Name or License Code”。

2. 用 `OD` 载入，提示加壳过。

   `PEID` 查壳，`Microsoft Visual C++ 6.0 [Overlay]`，没有认出来。

   后来才知道是 `Molebox` 的壳。

3. 直接用把它 dump 出来，当然，运行会出现 XX 初始化失败。

   不用修复，没关系，可以反汇编就行，用 `c32asm`（`IDA Pro` 太慢了，杀鸡不用牛刀）。

   `c32asm` 比 `w32dasm` 好用的多。可惜 `pll621` 意外丢弃源代码后不再写了。一直是 2 年前的版本，但是已经够用了。

4. `c32asm` 中，查找“Invalid License Name or License Code”，如下：

   一般来说，要基于函数的观点来逆向。这样比较能快速抓住关键。

   根据这个 string，找到以下一个函数 `00410940`，主要就是响应注册按钮的。

   代码见后面的 `#code 2`。

   从代码上看，逻辑比较清晰。代码风格比较好。

   别忙调试，把这函数从头到尾，也就 3，4 页。浏览一遍，基本这个注册机的想法就有了。工作完成 30% 了。

   Let's go. 代码附后面 `#code 2`。

5. 可以看出，调用 `aveData.dll` 的函数：`?ge_check@@YAHPBD0@Z` 检查 License。

   用 `peid` 看看这个 `aveData.dll`, 也是加了壳的：`SVKP 1.3x -> Pavol Cerven`

   不爽啊，不好直接反汇编，抠代码了。

6. 在用 `OD` 载入本来的 exe 调试。`bp 004109B1`, 然后可以跟进 `CALL 004195C6`，看看如何校验的。

   忽略所有异常，运行后，跳出注册界面。随便输入用户名 `ada`，注册码 `77498864`，

   `F9` 之后，断在 `004109B1`。

   `F7` 单步。进到 `aveData.dll`，`ge_check` 函数中了。

7. OK，别忙急着跟。先阅读这个函数的一下代码，就在 `OD` 中。

   找到了有趣的一段, 明显是注册码 ascii 格式化的, 注意一下这里：

   单步到这里后果然如此。

   而这前面一大堆 call，主要算法是 `md5`。不用看了。总之到这里，真正的 key 就出来了。

   OK，`00B71E6F 0F85 83000000 jnz aveData.00B71EF8` 则是注册后的跳转。

   后面就是一堆 pop 了，然后返回。

   代码见 `#code 3.`

8. 注意到如果不跳的话，真的 key 就会被清调。返回后，堆栈里也没了。所以这里要搞他一下。

   要让他好好的跳到合适的位置，发现合适的地方，不是跳 83 bytes，而是 7c bytes。把它修改一下：

   ```
   00B71E6F     0F85 83000000          jnz aveData.00B71EF8
   //修改为：
   00B71E6F     90                    NOP     
   00B71E70     E9 7C000000           JMP aveData.00B71EF1
   ```

   当然是内存 patch，不能直接 patch 文件的。我们可以在 keygen 里面写上一段 patch 内存的代码。

   `00B71E6F` 的函数内偏移量是 `0x34F`。把它记下来。写 keygen 时要用到。

   这个跳转之后，在 return 前面，真的 key 位于 `ESP-0x2E4;`

   这一段用嵌入汇编直接 call 即可。

   用 `MOV ECX, dword ptr SS:[ESP-0x2E4]` 来保存即可。

9. 好了。可以写 keygen 了。keygen 测试通过。需要把 `aveData.dll` 放在 keygen 相同目录下。

   keygen 主要代码见 `#code 1`, `VC` 自己补全的部分用...代替了。

10. 结束。

### `#code 1` begin

```cpp
//////   keygen 关键代码   //////
typedef  int (* keygencall)(char *name,char *sn);  //ge_check 函数原型

static FARPROC   proc;

static keygencall kgcall;

static int callbase;   //ge_check 函数 virtual   addr

static int offset = 0x34F;  //需要内存 patch 的函数偏移

static int patch_addr ; //需要内存 patch 的 virtual   addr

static unsigned char patch_code[] = {0x90,0xe9,0x7c};  //0x7c 恰好，需要内存 patch

static int flag = 0 ; //patch 内存 patch 成功标志

static char *fakekey="77498864";  //一个假的 key

/* keygen 窗口,初始化 */

void CKeygenDlg::OnPaint()   

{
         ......   

        CString   Err = "Input a name";
        HINSTANCE     dllinstance;
        dllinstance = LoadLibrary("aveData.dll");     
        if(dllinstance==NULL) {
                Err  = "Can't find aveData.dll";
                SetDlgItemTextA(IDC_EDIT_SN, Err );
                return;
        }
        /* 得到 ?ge_check@@YAHPBD0@Z 函数地址 */
        proc=GetProcAddress(dllinstance,"?ge_check@@YAHPBD0@Z");
        if(proc==NULL) {
                Err= "Internal function error...";
                SetDlgItemTextA(IDC_EDIT_SN, Err );
                return;
        }  
        /* 内存 patch */
        callbase = (int)proc;    
        kgcall = (keygencall)callbase;   
        patch_addr = callbase + offset;    
        memcpy((void*)patch_addr,(void*)patch_code,sizeof(patch_code)); 
        flag = 1 ;   
        SetDlgItemTextA(IDC_EDIT_SN, Err );
        return;
}

/* keygen 函数 */

void CKeygenDlg::OnGenKey() 

{
  char sn[64]={0};  //存放 key,足够大了
  int key = 0;
  CString Name,Serial;
  if(flag == 0)  //没有初始化好
    return;   
  GetDlgItemTextA(IDC_EDIT_Name, Name);
/* 汇编嵌入，直接调用目的函数，得到 key */
__asm
{   
  MOV EAX,dword ptr fakekey;   
  MOV ECX,dword ptr Name;
  LEA EDI,dword ptr Name; 
  PUSH EAX;
  PUSH ECX;
  CALL DWORD PTR kgcall;    //调用目的函数,计算 key
  MOV  ECX, dword ptr SS:[ESP-0x2E4];     
  MOV  key, ECX;
}    
  sprintf(sn, "%s",key);  
  Serial = sn ;
  SetDlgItemTextA(IDC_EDIT_SN ,Serial); //输出 sn
  return;
}
```

### `#code 2` begin

```asm
//////   c32asm 反编译 exe,关键函数。  //////
::00410940::  64:A1 00000000           MOV EAX,FS:[0]                        

::00410940::  64:A1 00000000           MOV EAX,FS:[0]                        

::00410946::  6A FF                    PUSH -1                              

::00410948::  68 C1B64100             PUSH 41B6C1                            \->: ?\x12B

::0041094D::  50                      PUSH EAX                              

::0041094E::  64:8925 00000000        MOV FS:[0],ESP                        

::00410955::  81EC C8010000           SUB ESP,1C8                            

::0041095B::  33C0                    XOR EAX,EAX                            

::0041095D::  53                      PUSH EBX                              

::0041095E::  55                      PUSH EBP                              

::0041095F::  56                      PUSH ESI                              

::00410960::  57                      PUSH EDI                              

::00410961::  8BE9                    MOV EBP,ECX                            

::00410963::  B9 10000000             MOV ECX,10                             

::00410968::  8D7C24 18               LEA EDI,[ESP+18]                        

::0041096C::  F3                      REP STOS DWORD PTR ES:[EDI]           

::0041096D::  AB                      STOS DWORD PTR ES:[EDI]                

::0041096E::  B9 20000000             MOV ECX,20                             

::00410973::  8D7C24 58               LEA EDI,[ESP+58]                        

::00410977::  F3                      REP STOS DWORD PTR ES:[EDI]           

::00410978::  AB                      STOS DWORD PTR ES:[EDI]                

::00410979::  E8 7C910000             CALL 00419AFA                         \:JMPDOWN    >>>: MFC42.DLL:?AfxGetModuleState@@YGPAVAFX_MODULE_STATE@@XZ

::0041097E::  8B58 04                 MOV EBX,[EAX+4]                       

::00410981::  6A 01                   PUSH 1                                

::00410983::  8BCD                    MOV ECX,EBP                            

::00410985::  E8 9A910000             CALL 00419B24                         \:JMPDOWN    >>>: MFC42.DLL:?UpdateData@CWnd@@QAEHH@Z

::0041098A::  8D75 64                 LEA ESI,[EBP+64]                        

::0041098D::  6A 0A                   PUSH A                                

::0041098F::  8BCE                    MOV ECX,ESI     

//很明显的 CWnd，MFC 的不用理他，往下看看。   

...... 

::004109A8::  6A 20                   PUSH 20                                

::004109AA::  8BCE                    MOV ECX,ESI                            

::004109AC::  E8 97910000             CALL 00419B48                         \:JMPDOWN    >>>: MFC42.DLL:?Remove@CString@@QAEHD@Z>

::004109B1::  8B06                    MOV EAX,[ESI]                          

::004109B3::  8B4D 60                 MOV ECX,[EBP+60]                        

::004109B6::  8D7D 60                 LEA EDI,[EBP+60]                        

::004109B9::  50                      PUSH EAX                              

::004109BA::  51                      PUSH ECX     
```



::004109BB:: `E8 068C0000` `CALL 004195C6` `\JMPDOWN` `AVEDATA.DLL:?ge\_check@@YAHPBD0@Z`

::004109C0:: `83C4 08` `ADD ESP,8`

::004109C3:: `85C0` `TEST EAX,EAX`

// 很明显，上面是调用 `AVEDATA.DLL` 的函数： `?ge\_check@@YAHPBD0@Z` 检查 License。

// Ture 则有效。

// 在 `OD` 里里面设一个断点，`bp 004109B1`,然后可以跟进 `CALL 004195C6` ，看看如何校验的。

// 下面的跳转都是很典型注册成功与否，流程很清晰。

// 注册成功后，信息记录在 `data.ini`。

::004109C5:: `75 2B` `JNZ SHORT 004109F2` `\JMPDOWN`

::004109C7:: `6A 40` `PUSH 40`

::004109C9:: `68 98574200` `PUSH 425798` `\->: Sorry`

::004109CE:: `68 70574200` `PUSH 425770` `\->: Invalid License Name or License Code`

::004109D3:: `8BCD` `MOV ECX,EBP`

::004109D5:: `E8 50910000` `CALL 00419B2A` `\JMPDOWN` `MFC42.DLL:?MessageBoxA@CWnd@@QAEHPBD0I@Z`

::004109DA:: `68 FB030000` `PUSH 3FB`

::004109DF:: `8BCD` `MOV ECX,EBP`

::004109E1:: `E8 2A900000` `CALL 00419A10` `\JMPDOWN` `MFC42.DLL:?GetDlgItem@CWnd@@QBEPAV1@H@Z`

::004109E6:: `8BC8` `MOV ECX,EAX`

::004109E8:: `E8 7B8F0000` `CALL 00419968` `\JMPDOWN` `MFC42.DLL:?SetFocus@CWnd@@QAEPAV1@XZ`

::004109ED:: `E9 60010000` `JMP 00410B52` `\JMPDOWN`

::004109F2:: `57` `PUSH EDI` `\BYJMP JmpBy:004109C5,`

::004109F3:: `8D8B CC000000` `LEA ECX,[EBX+CC]`

::004109F9:: `C783 C4000000 01000000` `MOV DWORD PTR [EBX+C4],1`

::00410A03:: `E8 B48C0000` `CALL 004196BC` `\JMPDOWN` `MFC42.DLL:??4CString@@QAEABV0@ABV0@@Z`

::00410A08:: `56` `PUSH ESI`

::00410A09:: `8D8B C8000000` `LEA ECX,[EBX+C8]`

::00410A0F:: `E8 A88C0000` `CALL 004196BC` `\JMPDOWN` `MFC42.DLL:??4CString@@QAEABV0@ABV0@@Z`

::00410A14:: `8B07` `MOV EAX,[EDI]`

::00410A16:: `50` `PUSH EAX`

::00410A17:: `8D8424 DC000000` `LEA EAX,[ESP+DC]`

::00410A1E:: `68 48574200` `PUSH 425748` `\->: Register successful! License to:%s.`

::00410A23:: `50` `PUSH EAX`

::00410A24:: `FF15 88D64100` `CALL [41D688]` `MSVCRT.DLL:sprintf`

::00410A2A:: `83C4 0C` `ADD ESP,C`

::00410A2D:: `8D8C24 D8000000` `LEA ECX,[ESP+D8]`

::00410A34:: `6A 40` `PUSH 40`

::00410A36:: `68 3C574200` `PUSH 42573C` `\->: Thank you`

::00410A3B:: `51` `PUSH ECX`

::00410A3C:: `8BCD` `MOV ECX,EBP`

::00410A3E:: `E8 E7900000` `CALL 00419B2A` `\JMPDOWN` `MFC42.DLL:?MessageBoxA@CWnd@@QAEHPBD0I@Z`

::00410A43:: `8B3F` `MOV EDI,[EDI]`

::00410A45:: `8D4C24 18` `LEA ECX,[ESP+18]`

::00410A49:: `2BCF` `SUB ECX,EDI`

::00410A4B:: `8A07` `MOV AL,[EDI]` `\BYJMP JmpBy:00410A53,`

::00410A4D:: `880439` `MOV [ECX+EDI],AL`

::00410A50:: `47` `INC EDI`

::00410A51:: `84C0` `TEST AL,AL`

::00410A53:: `75 F6` `JNZ SHORT 00410A4B` `\JMPUP`

::00410A55:: `8B36` `MOV ESI,[ESI]`

::00410A57:: `8D4C24 58` `LEA ECX,[ESP+58]`

::00410A5B:: `2BCE` `SUB ECX,ESI`

::00410A5D:: `8A06` `MOV AL,[ESI]` `\BYJMP JmpBy:00410A65,`

::00410A5F:: `880431` `MOV [ECX+ESI],AL`

::00410A62:: `46` `INC ESI`

::00410A63:: `84C0` `TEST AL,AL`

::00410A65:: `75 F6` `JNZ SHORT 00410A5D`

::00410A67:: `8D4C24 10` `LEA ECX,[ESP+10]`

::00410A6B:: `E8 9E8B0000` `CALL 0041960E` `\JMPDOWN` `MFC42.DLL:??0CString@@QAE@XZ`

::00410A70:: `8D5424 14` `LEA EDX,[ESP+14]`

::00410A74:: `C78424 E0010000 00000000` `MOV DWORD PTR [ESP+1E0],0`

::00410A7F:: `52` `PUSH EDX`

::00410A80:: `E8 3BF0FFFF` `CALL 0040FAC0` `\JMPUP`

::00410A85:: `83C4 04` `ADD ESP,4`

::00410A88:: `50` `PUSH EAX`

::00410A89:: `8D4C24 14` `LEA ECX,[ESP+14]`

::00410A8D:: `C68424 E4010000 01` `MOV BYTE PTR [ESP+1E4],1`

::00410A95:: `E8 228C0000` `CALL 004196BC` `\JMPDOWN` `MFC42.DLL:??4CString@@QAEABV0@ABV0@@Z`

::00410A9A:: `8D4C24 14` `LEA ECX,[ESP+14]`

::00410A9E:: `C68424 E0010000 00` `MOV BYTE PTR [ESP+1E0],0`

::00410AA6:: `E8 518B0000` `CALL 004195FC` `\JMPDOWN` `MFC42.DLL:??1CString@@QAE@XZ`

::00410AAB:: `8D4424 10` `LEA EAX,[ESP+10]`

::00410AAF:: `68 30574200` `PUSH 425730` `\->: data.ini`

::00410AB4:: `8D4C24 18` `LEA ECX,[ESP+18]`

::00410AB8:: `50` `PUSH EAX`

::00410AB9:: `51` `PUSH ECX`

::00410ABA:: `E8 458F0000` `CALL 00419A04` `\JMPDOWN` `MFC42.DLL:??H@YG?AVCString@@ABV0@PBD@Z`

::00410ABF:: `50` `PUSH EAX`

::00410AC0:: `8D4C24 14` `LEA ECX,[ESP+14]`

::00410AC4:: `C68424 E4010000 02` `MOV BYTE PTR [ESP+1E4],2`

::00410AF2:: `68 20574200` `PUSH 425720` `\->: License Name`

::00410AF7:: `68 14574200` `PUSH 425714` `\->: Register`

::00410AFC:: `FFD6` `CALL ESI`

::00410AFE:: `8B4C24 10` `MOV ECX,[ESP+10]`

::00410B02:: `8D5424 58` `LEA EDX,[ESP+58]`

::00410B06:: `51` `PUSH ECX`

::00410B07:: `52` `PUSH EDX`

::00410B08:: `68 04574200` `PUSH 425704` `\->: License Code`



::00410B0D::  68 14574200                PUSH 425714                         \->: Register  

::00410B12::  FFD6                       CALL ESI                          

::00410B14::  E8 518C0000               CALL 0041976A                      \:JMPDOWN    >>>: MFC42.DLL:?AfxGetThread@@YGPAVCWinThread@@XZ  

::00410B19::  85C0                       TEST EAX,EAX                      

::00410B1B::  74 0B                      JE SHORT 00410B28                  \:JMPDOWN  

::00410B1D::  8B10                       MOV EDX,[EAX]                    

::00410B1F::  8BC8                       MOV ECX,EAX                      

::00410B21::  FF52 7C                    CALL [EDX+7C]                   

::00410B24::  8BF0                       MOV ESI,EAX                      

::00410B26::  EB 02                      JMP SHORT 00410B2A               \:JMPDOWN  

::00410B28::  33F6                       XOR ESI,ESI                       \:BYJMP JmpBy:00410B1B,  

::00410B2A::  E8 918A0000               CALL 004195C0                     \:JMPDOWN\:BYJMP JmpBy:00410B26,    >>>: AVEDATA.DLL:?ge\_app@@YAPADXZ  

::00410B2F::  50                         PUSH EAX                          

::00410B30::  8BCE                       MOV ECX,ESI                      

::00410B32::  E8 518F0000               CALL 00419A88                     \:JMPDOWN    >>>: MFC42.DLL:?SetWindowTextA@CWnd@@QAEXPBD@Z  

::00410B37::  8D4C24 10                  LEA ECX,[ESP+10]                 

::00410B3B::  C78424 E0010000 FFFFFFFF  MOV DWORD PTR [ESP+1E0],-1       

::00410B46::  E8 B18A0000               CALL 004195FC                     \:JMPDOWN    >>>: MFC42.DLL:??1CString@@QAE@XZ  

::00410B4B::  8BCD                       MOV ECX,EBP                      

::00410B4D::  E8 6A8E0000               CALL 004199BC                     \:JMPDOWN    >>>: MFC42.DLL:?OnOK@CDialog@@MAEXXZ  

::00410B52::  8B8C24 D8010000           MOV ECX,[ESP+1D8]                 \:BYJMP JmpBy:004109ED,  

::00410B59::  5F                         POP EDI                          

::00410B5A::  5E                         POP ESI                          

::00410B5B::  5D                         POP EBP                          

::00410B5C::  5B                         POP EBX                          

::00410B5D::  64:890D 00000000          MOV FS:[0],ECX                   

::00410B64::  81C4 D4010000             ADD ESP,1D4                      

::00410B6A::  C3                         RETN                              

::00410B6A::  C3                         RETN                              

::00410B6B::  90                         NOP                              

::00410B6C::  90                         NOP                              

//这里还有 ge\_buy，也有趣的。贴出来、                 

::00410B70::  6A 05                      PUSH 5                           

::00410B72::  6A 00                      PUSH 0                           

::00410B74::  E8 538A0000               CALL 004195CC                     \:JMPDOWN    >>>: AVEDATA.DLL:?ge\_buy@@YAPADH@Z  

::00410B79::  83C4 04                    ADD ESP,4                      

::00410B7C::  50                         PUSH EAX                          

::00410B7D::  E8 8E9EFFFF               CALL 0040AA10                    \:JMPUP  

::00410B82::  83C4 08                    ADD ESP,8                      

::00410B85::  C3                         RETN                              

::00410B86::  90                         NOP                              

::00410B87::  90                         NOP                              

### code 2 end

### code 3 begin

////  OD 中, aveData.dll 的 ge\_check 函数，这段是得到正确的关键地  ////  

```
00B71E36     2BC2                     sub eax,edx
00B71E38     8D9424 C8010000          lea edx,dword ptr ss:[esp+1C8]
00B71E3F     03C6                     add eax,esi
00B71E41     03C7                     add eax,edi
00B71E43     50                       push eax
00B71E44     68 8CE4B700              push aveData.00B7E48C                  ; ASCII "%08lX"
00B71E49     52                       push edx
00B71E4A     E8 59240000              call aveData.00B742A8
00B71E4F     8B8C24 E8020000          mov ecx,dword ptr ss:[esp+2E8]
00B71E56     8D8424 D4010000          lea eax,dword ptr ss:[esp+1D4]
00B71E5D     6A 08                    push 8
00B71E5F     50                       push eax
00B71E60     51                       push ecx
```

//看 od 寄存器窗口，很可能 EAX 是真的 key 了。果然是的。

```
EAX 0012A12C ASCII "C9575F31"   

ECX 00379990 ASCII "77498864"   

EDX 0012A133

EBX 40C06277
```

```
00B71E61     E8 0A240000              call aveData.00B74270
00B71E66     83C4 18                   add esp,18
00B71E69     85C0                      test eax,eax
00B71E6B     5F                        pop edi
00B71E6C     5E                        pop esi
00B71E6D     5D                        pop ebp
00B71E6E     5B                        pop ebx
00B71E6F     0F85 83000000            jnz aveData.00B71EF8
```

//往后面看看知道，需要在这里跳转，否则会清掉正确的 key。

//在这里做一下内存 patch,跳往我们想要的地方，这样 return 后就得到 key 了。

```
//0F85 83000000 -> 90e9 7c000000 

//(NOP, JMP aveData.00B71EF1)
```

...省略若干行...

```
00B71EC5     E8 A6100000               call    aveData.00B72F70
00B71ECA     8D4C24 00                 lea     ecx,dword ptr ss:[esp]
00B71ECE     C78424 C0020000 FFFFFFFF  mov     dword ptr ss:[esp+2C0],-1
00B71ED9     E8 92100000               call    aveData.00B72F70
00B71EDE     B8 01000000               mov     eax,1
00B71EE3     8B8C24 B8020000           mov     ecx,dword ptr ss:[esp+2B8]
00B71EEA     64:890D 00000000          mov     dword ptr fs:[0],ecx
```

//修改，让他跳到这里 00B71EF1

```
00B71EF1     81C4 C4020000             add     esp,2C4
00B71EF7     C3                        retn
```

//  真的 key 位于 ESP-0x2E4;

//  用 MOV  ECX, dword ptr SS:\[ESP-0x2E4\] 来保存即可。

//本来是跳到 00B71EF8

```
00B71EF8     8D4C24 30                 lea     ecx,dword ptr ss:[esp+30]
00B71EFC     C68424 C0020000 0C        mov     byte ptr ss:[esp+2C0],0C
```

...省略若干行...

```
00B71F61     8B8C24 B8020000           mov     ecx,dword ptr ss:[esp+2B8]
00B71F68     33C0                      xor     eax,eax
00B71F6A     64:890D 00000000          mov     dword ptr fs:[0],ecx
00B71F71     81C4 C4020000             add     esp,2C4
00B71F77     C3                        retn
```

### code 3 end

### EndOfFile  readyu 20061003 

`keygen` 已经好些人用过了，不是纸上谈兵的。:)  

这文章是很久之后抽空写的，把以前的编译时的代码找出来的。  

写的时候，`keygen` 重新编译，测试通过。算了不少的 key。  

这段地址是可写的。它恰好是壳解压后的代码段，没有设置只读保护。  

另外 `memcpy` 是 `Kernel32.dll` 中的 `RtlMoveMemory` 这个 API 的封装，  

与 `WriteProcessMemory` 不同。  

`RtlMoveMemory` 不会检查读写该内存所应有的权限，  

如果不可写，会立刻 XX 内存不能为 written 崩掉的。也能立刻发现。  

引用:  *最初由** cutezhang **发布*  

有疑问:`keygen` 的代码测试通过了吗?DLL 内存代码段怎么随便就修改了,  

似乎还要先设置一下 dll 模块可写
