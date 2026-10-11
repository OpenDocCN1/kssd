
# Delphi 内联汇编 API 内存搜索引擎（带注释）

> 原文：[`www.pediy.com/kssd/pediy11/123172.html`](https://www.pediy.com/kssd/pediy11/123172.html)

## 代码

```
FUNCTION GetKernel32Module(): Cardinal; assembler
asm
  mov eax, fs:$30
  mov eax, [eax+$0c]
  mov eax, [eax+$1c]    // 相当于 mov esi, [eax+$1c]
  mov eax, [eax]        // 相当于 lodsd (mov eax, [esi]) 
  mov eax, [eax+$08]    // 返回 k32 的基址
end;

FUNCTION LStrLengthA(const lpStr: PAnsiChar): Cardinal; assembler;
asm
  push edi
  push ebx
  mov edi, eax
  mov ebx, eax
  xor al, al
@@lstrscan:
  scas byte ptr es:[edi]           //字符扫描法检查字符串指针长度 
  jnz @@lstrscan
  dec edi
  sub edi, ebx
  mov eax, edi
  pop ebx
  pop edi
end;

FUNCTION CalcBufferCRC(lpBuffer:PAnsiChar): Cardinal; assembler;
asm
  push ebx
  push edi
  push ecx
  mov ebx, eax
  call LStrLengthA
  mov edi, eax
  shr edi, 2
  xor ecx, ecx
@@loopBegin:
  dec edi
  jl @@loopOver
  xor ecx, dword ptr ds:[ebx]
  add ebx, 4
  jmp @@loopBegin
@@loopOver:
  mov eax, ecx
  pop ecx
  pop edi
  pop ebx
end;


function GetProcAddressA(hModule:Cardinal; dwExportCRC: Cardinal) : Pointer; assembler;
var
  lpProcNameCRC, dwProcNumber: Cardinal;
  pProcAddress, pProcNameAddress, pProcIndexAddress: Pointer;
asm
  push ebx
  push esi
  mov lpProcNameCRC, edx      // edx=函数名 CRC32
  mov ebx, eax                // ebx=基址
  mov eax, [ebx+$3c]          // eax=文件头偏移
  mov esi, [ebx+eax+$78]      // esi=输出表偏移,文件头+可选头的长度=$78
  lea esi, [ebx+esi+$18]      // esi=函数名数量 = 函数数量 [ebx+esi+$14]
  lods dword ptr ds:[esi]
  mov dwProcNumber, eax       // eax=函数名数量
  lods dword ptr ds:[esi]
  mov pProcAddress, eax       // eax=函数偏移量
  lods dword ptr ds:[esi]
  mov pProcNameAddress, eax   // eax=函数名偏移量
  lods dword ptr ds:[esi]
  mov pProcIndexAddress, eax  // eax=序列号偏移量
  mov edx, dwProcNumber       // edx=遍历次数
@@LoopBegin:
  xor eax, eax                // Result = 0
  dec edx
  jl @@LoopEnd
  mov eax, pProcNameAddress
  add eax, ebx                // eax=函数名基地址
  mov eax, dword ptr ds:[eax+edx*4]
  add eax, ebx                // eax=遍历函数名
  call CalcBufferCRC
  cmp eax, lpProcNameCRC      // 对比 CRC32
  jnz @@LoopBegin
  shl edx, 1
  add edx, pProcIndexAddress  // 函数基序列
  movzx eax, word ptr ss:[edx+ebx]
  shl eax, 2
  add eax, pProcAddress       // 函数基地址
  mov eax, [eax+ebx]
  add eax, ebx                // Result = 函数地址
@@LoopEnd:
  pop esi
  pop ebx
end;
```

先用 `LoadLibraryCRC32:= CalcBufferCRC('LoadLibraryA')` 计算出 `LoadLibraryA` 的 CRC32。通常常用的 API 应该事先计算好放在程序内，以便查找，不要在程序调用时出现。杀毒软件对某些 API 名称会敏感，所以才有 CRC32 匹配法。网上的代码基本都是传遍了大江南北，杀软都做了特征。其实你自己可以用很多种方法来计算 API 的加密算法，用我的汇编进行匹配查询。

然后 `GetProcAddressA(GetKernel32Module, LoadLibraryCRC32)` 就可以得到 `LoadLibraryA` 的函数地址了。

效率绝对一流，用 `KOL` 编译出的 3.5k 程序优化后可以在 2k 以内，极端情况不做代表。

本人不提倡到处复制黏贴，但是若要复制，请说明作者和出处！

“没有分享就没有进步的精神永在”！！！

## 代码

```
{\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
本汇编主要功能就是计算短字符串精确 CRC32 计算
参数 eax = 字符串指针
返回 crc32
作者 饿狼传说 （lofullen）
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*}
FUNCTION CalcBufferCRC(lpBuffer:PAnsiChar): Cardinal; assembler;
var
  checksum : DWORD;
asm
  push ebx
  push ecx
  push edx
  push edi
  push esi
  mov ebx, eax
  call LStrLengthA                   // eax = 字符串长度
  xor edx, edx                       // edx = 整除后的余数
  mov ecx, 4
  div ecx
  xor ecx, ecx
@@loopBegin:
  dec eax                            // eax = 整除后的整数
  jl @@loopOver
  xor ecx, dword ptr ds:[ebx]      // ecx = CRC32
  add ebx, 4
  jmp @@loopBegin
@@loopOver:
  test edx, edx                    // 如果有余数则执行
  je @@Nochecksum
  mov eax, ecx
  mov esi, ebx                    // esi = 剩余字节的首指针
  lea edi, checksum
  mov dword ptr ds:[edi], 0        // checksum 归零 这样的做法为了保证只读取 00 之前的剩余字节
  mov ecx, edx
  rep movsb                       // checksum 得到剩余字节(自然是小于 4 咯) 
  lea edi, checksum
  xor eax, dword ptr ds:[edi]      // 最后一次计算
  jmp @@calcEnd
@@Nochecksum:
  mov eax, ecx                     // 直接获取 ECX 中的 CRC32
@@calcEnd:
  pop esi
  pop edi
  pop edx
  pop ecx
  pop ebx
end;
```

## 补充一份改进型 字符串型精确 CRC32 急速算法

代码:

```
// 功能 数字型转成 16 进制 字符串型(非 String 版)
// 参数 EAX = 10 进制数字
// 参数 EDX = 16 进制字符串格式长度
// 参数 ECX = 指向字符串(array[0..100] of char)指针
// 返回有效字符串长度 
//作者 饿狼传说 （lofullen）

function IntToHexA(Value: Integer; Digits: Integer; lpBuffer: Pchar): Integer;
asm
        PUSH    ESI
        MOV     ESI, ESP
        SUB     ESP, 32
        PUSH    EDI
        MOV     EDI, ECX
        MOV     ECX, 16        //  这里改成 XOR ECX, ECX 就本函数就成了 IntToStrA 了
        PUSH    EDX
        PUSH    ESI
@D1:    XOR     EDX,EDX
        DIV     ECX
        DEC     ESI
        ADD     DL,'0'
        CMP     DL,'0'+10
        JB      @D2
        ADD     DL,('A'-'0')-10
@D2:    MOV     [ESI],DL
        OR      EAX,EAX
        JNE     @D1
        POP     ECX
        POP     EDX
        SUB     ECX,ESI
        SUB     EDX,ECX
        JBE     @D5
        ADD     ECX,EDX
        MOV     AL,'0'
        SUB     ESI,EDX
        JMP     @z
@zloop: MOV     [ESI+EDX],AL
@z:     DEC     EDX
        JNZ     @zloop
        MOV     [ESI],AL
@D5:
        MOV     EAX, ECX
        REP     MOVSB
        POP     EDI
        ADD     ESP, 32
        POP     ESI
end;
```

我就不开贴了，本汇编函数附送给大家了，彻底摆脱 Delphi 的 `string` 结构，可以在纯 `KOL` 环境下编译，这个 `string` 结构比较复杂，移植能力非常的差，本汇编可以帮助大家转换，另外一个 `strtoint` 可以用 API 代替。
