
# Enigma VM 的分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-570.htm`](https://www.pediy.com/kssd/pediy09/pediy09-570.htm)

> 转载注明来自看雪

> `Enigma` 的 `1.3x` 版本加入了 `VM` 保护，除了 `VM` 外，`Enigma` 的脱法和以前版本类似，`1.3x` 里面还加入了随机异常，让脱壳者不能通过定位异常来判断位置，别的无非就是偷取 `api`、`iat` redirection、偷取库函数等等，都是一些老的技术，本文主要是对其 `VM` 进行一番探索.

> `Enigma` `VM` 执行的过程是：

> 壳程序在每个虚拟块的开头，用

```
0040144C    68 6FB8AB95     push    95ABB86F  //cipher_index
00401451  - E9 86C3E400     jmp     VM_Entry
```

> 当程序运行到这里的时候，会带着 `cipher_index`，进入 `VM_Entry`

> `cipher_index` 是把虚拟块的索引用用随机的 `rc4` 加密，进入 `VM_Entry` 后，会解密回来，通过这个索引找到 `VM_Data`

> 然后把 `VM_Data` 还原成 `x86` 的代码，把这些代码送到中 `VirtualAlloc` 的内存空间里面，然后跳到这个 `VirtualAlloc` 中去执行，所以可以看出 `Enigma` 的虚拟机其实不是真正意义上的虚拟机，它没有常见 `VM` 的 `Context` 保护，`PCode` 执行，它只是一个代码还原执行的载体。可想而知，强度就不会太大。

> 首先是存放 `Enigma` `VM` 的数据结构：

> `Enigma` 的虚拟块存放在一个 `List` 里面，在初始化 `VM` 的时候，会给这个 `List` 复制.

> 这个 `List` 的每个 `Item` 结构是：

```
ITEM
{
0h:  index  //这个是虚拟块的索引,通过解密 cipher_index,找到当前虚拟块的数据
4h:  rva  //这个是 VM 块在程序里面的 rva,要加上 ImageBase
8h:  vm_run_addr  //这个是虚拟块还原出来的 x86 的地址,第一次运行虚拟块,这个地址就产生了
Ch:  vm_data    //这个就是最重要的虚拟数据
}
```

> `Enigma` 主程序的虚拟块一共有 `0x577` 处，要想完全修复得花一定功夫.

```
CODE:0041DA21 loop_write1:                         ; CODE XREF: Fix_VM+19Fj
CODE:0041DA21             mov     edi, ebx
CODE:0041DA23             add     edi, edi
CODE:0041DA25             mov     eax, [ebp+0]    ; jmp_list
CODE:0041DA28             lea     edx, [ebx+1]    ; index
CODE:0041DA2B             mov     [eax+edi*8], edx
CODE:0041DA2E             push    offset rand_jmp_key_20h
CODE:0041DA33             mov     eax, [ebp+0]
CODE:0041DA36             lea     eax, [eax+edi*8]
CODE:0041DA39             lea     edx, [esp+14h+var_4]
CODE:0041DA3D             mov     ecx, 4
CODE:0041DA42             call    Cipher_Index
CODE:0041DA47             mov     eax, [ebp+0]
CODE:0041DA4A             mov     eax, [eax+edi*8+4]
CODE:0041DA4E             mov     edx, ds:pImageBase
CODE:0041DA54             add     eax, [edx]
CODE:0041DA56             mov     [esp+10h+var_C_data_p], eax
CODE:0041DA5A             lea     eax, [esp+10h+var_C_data_p]
CODE:0041DA5E             mov     edx, [esp+10h+var_4]
CODE:0041DA62             call    MakePush
CODE:0041DA67             mov     edx, offset VM_Entry
CODE:0041DA6C             sub     edx, [esp+10h+var_C_data_p]
CODE:0041DA70             lea     eax, [esp+10h+var_C_data_p]
CODE:0041DA74             call    MakeJmp
CODE:0041DA79             inc     ebx
CODE:0041DA7A             dec     esi
CODE:0041DA7B             jnz     short loop_write1
```

> 这里就是对 `VM_List` 赋值的地方.

> 下面再看 `vm_data`，`vm_data` 的结构如下

> 第一个 `DWORD` 是虚拟代码的总共个数

> 然后是每个 `block` 的内容，它的 `VM_BLOCK` 的 `size=0x15`，具体见下面

```
typedef struct _VM_BLOCK
{
  DWORD dwVMType;    //代表 VMType
  DWORD dwParam1;
  DWORD dwReg2;
  DWORD dwOffset;
  DWORD dwReg3;
  BYTE bJunk;  //bJunk=1 产生花指令
}VM_BLOCK;
```

> 进入 `VM_Entry` 后，在这里生成还原的 `x86` 代码

```
CODE:0041D2F7 loop_:                          ; CODE XREF: InitVM_Code+129j
CODE:0041D2F7             imul    edi, ebx, 15h   ; block_offset
CODE:0041D2FA             mov     eax, [ebp+var_8_vm_data]
CODE:0041D2FD             cmp     dword ptr [eax+edi], 0
CODE:0041D301             jz      get_next
CODE:0041D307             mov     eax, [ebp+var_28_list]
CODE:0041D30A             mov     edx, [ebp+var_24_mem_p]
CODE:0041D30D             mov     [eax+ebx*4], edx
CODE:0041D310             mov     eax, [ebp+var_8_vm_data]
CODE:0041D313             mov     eax, [eax+edi+4]
CODE:0041D317             mov     [ebp+var_10_dwParam1], eax
CODE:0041D31A             mov     eax, [ebp+var_8_vm_data]
CODE:0041D31D             mov     eax, [eax+edi+8]
CODE:0041D321             mov     [ebp+var_14_dwReg2], eax
CODE:0041D324             mov     eax, [ebp+var_8_vm_data]
CODE:0041D327             mov     eax, [eax+edi+0Ch]
CODE:0041D32B             mov     [ebp+var_18_offset], eax
CODE:0041D32E             mov     eax, [ebp+var_8_vm_data]
CODE:0041D331             mov     eax, [eax+edi+10h]
CODE:0041D335             mov     [ebp+var_1C_dwReg3], eax
CODE:0041D338             mov     eax, [ebp+var_8_vm_data]
CODE:0041D33B             cmp     byte ptr [eax+edi+14h], 0    //bJunk 是否要产生花指令
CODE:0041D340             jz      short flag_0
CODE:0041D342             lea     eax, [ebp+var_24_mem_p]
CODE:0041D345             call    JunkProduce
CODE:0041D34A
CODE:0041D34A flag_0:                       ; CODE XREF: InitVM_Code+B4j
CODE:0041D34A             mov     eax, [ebp+var_14_dwReg2]
CODE:0041D34D             push    eax
CODE:0041D34E             mov     eax, [ebp+var_18_offset]
CODE:0041D351             push    eax
CODE:0041D352             mov     eax, [ebp+var_1C_dwReg3]
CODE:0041D355             push    eax
CODE:0041D356             mov     eax, [ebp+var_8_vm_data]
CODE:0041D359             mov     edx, [eax+edi]  ; type
CODE:0041D35C             lea     eax, [ebp+var_24_mem_p]
CODE:0041D35F             mov     ecx, [ebp+var_10_dwParam1]
CODE:0041D362             call    GetVMCode       ; eax=pMem
CODE:0041D362                            ; ecx=VM_Param1
CODE:0041D362                            ; edx=VM_Type
CODE:0041D367             test    al, al
CODE:0041D369             jnz     short get_next
CODE:0041D36B             mov     eax, [ebp+var_8_vm_data] ;
CODE:0041D36E             mov     eax, [eax+edi]
CODE:0041D371             xor     edx, edx
CODE:0041D373             push    edx
CODE:0041D374             push    eax
CODE:0041D375             lea     eax, [ebp+var_30]
CODE:0041D378             call    @Sysutils@IntToStr$qqrj ; Sysutils::IntToStr(__int64)
CODE:0041D37D             mov     ecx, [ebp+var_30]
CODE:0041D380             lea     eax, [ebp+var_2C]
CODE:0041D383             mov     edx, offset aInvalidVmInstr ; "Invalid VM instruction: "
CODE:0041D388             call    @System@@LStrCat3$qqrv ; System::__linkproc__ LStrCat3(void)
CODE:0041D38D             mov     eax, [ebp+var_2C]
CODE:0041D390             call    @System@@LStrToPChar$qqrx17System@AnsiString ; System::__linkproc__ LStrToPChar

(System::AnsiString)
CODE:0041D395             mov     ds:dword_4486A4, eax
CODE:0041D39A             push    0
CODE:0041D39C             push    offset aLoaderError ; "Loader error!"
CODE:0041D3A1             mov     eax, ds:dword_4486A4
CODE:0041D3A6             push    eax
CODE:0041D3A7             push    0
CODE:0041D3A9             call    sub_42081C
CODE:0041D3AE             call    Fuck_Debugger
CODE:0041D3B3
CODE:0041D3B3 get_next:                    ; CODE XREF: InitVM_Code+75j
CODE:0041D3B3                          ; InitVM_Code+DDj
CODE:0041D3B3             inc     ebx
CODE:0041D3B4             dec     esi         ; size-1
CODE:0041D3B5             jnz     loop_       ; block_offset
```

> `GetVMCode` 是最核心的部分，它通过 `VMType`，一共处理了 `0xDF` 中代码，具体见程序.
