
`mov eax,POLY_LOOP_ITERATION_MAX` ; 最大loop计数

`call random`

`add eax,POLY_LOOP_ITERATION_MIN` ; 获得一个随机的loop计数

`push ecx`

`mov ecx,eax`

`call Asm_Mov_Reg_Imm_32` ;产生一个mov rx ，imm32 指令

`pop ecx`

`push edi`

`push ebx`

`dec ecx`

`call Garble` ;插入无效指令

`pop eax`

`mov ebx,eax`

`call unset_reg_32` ; 递减计数

`mov al,48h`

`add al,bl`

`stosb`

`mov al,085h` ;测试计数

`stosb`

`mov eax,ebx`

`rol eax,3`

`add eax,ebx`

`add eax,0c0h`

`stosb`

`pop eax` ; 产生一个 jcc

`sub eax,edi`

`push eax`

`not eax`

`cmp eax,255 / 2`

`pop eax`

`jbe Garble_Loop_Short`

`sub eax,6` ;产生一个 near jcc

`mov word ptr [edi],0850fh`

`mov dword ptr [edi + 2],eax`

`add edi,6`

`jmp Garble_Loop_Cont` ;产生一个 short jcc

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

经过上述变换，产生的无效指令效果如下，可以看到如果不经过分析，是不容易利用程序分析出这些是无效指令的。

`loop:`

```
00A00298    68 DE320000     push 32DE
00A0029D    5E              pop esi
00A0029E    337C24 00       xor edi,dword ptr ss:[esp]
00A002A2    4E              dec esi
00A002A3    85F6            test esi,esi
00A002A5  ^ 75 F7           jnz short 00A0029E
```

`push/pop:`

```
00A009D8    54              push esp
00A009D9    8B5C24 00       mov ebx,dword ptr ss:[esp]
00A009DD    5A              pop edx
00A009DE    BF 7B3DF36F     mov edi,6FF33D7B
00A009E3    FF75 F0         push dword ptr ss:[ebp-10]
00A009E6    53              push ebx
```

`jump:`

```
00A009EB   /74 04           je short 00A009F1
00A009ED   |40              inc eax
00A009EE   |66:33F0         xor si,ax
00A009F1   \FF7424 00       push dword ptr ss:[esp]
00A009F5    4A              dec edx
00A009F6    8BC0            mov eax,eax
00A009F8    5B              pop ebx
```

`imm:`

```
00A00A06    BB 14C44F40     mov ebx,404FC414
00A00A0B    33DE            xor ebx,esi
00A00A0D    0B4D 00         or ecx,dword ptr ss:[ebp]
00A00A10    8B7D 00         mov edi,dword ptr ss:[ebp]
```

`mod/rm:`

```
00A009CA    8D15 2C8A0000   lea edx,dword ptr ds:[8A2C]
00A009D0    66:33C0         xor ax,ax
00A009D3    4A              dec edx
00A009D4    85D2            test edx,edx
00A009D6  ^ 75 F8           jnz short 00A009D0
```

`call :`

```
00A02B10    E8 03000000     call 00A02B18
00A02B15    234D 00         and ecx,dword ptr ss:[ebp]
00A02B18    5F              pop edi
00A02B19    81C7 EBD45FFF   add edi,FF5FD4EB
00A02B1F    81C7 F52AA000   add edi,0A02AF5
00A02B25    8B4C24 00       mov ecx,dword ptr ss:[esp]
```

### 2.5.3 poly engine的实现

`ope` 采用的是随机密钥+滑动密钥方案，加密支持的指令为(`xor` / `sub` / `add`),滑动密钥支持的指令为（`sub` / `add` / `xor`）,加密中`ope`维护一个`poly_vars`

这样一个结构体，定义如下:

```
poly_vars struct
   poly_ptr_code_base_va     dd  ?      ;   要加密数据的virtual-address
   poly_ptr_code_base_raw    dd  ?      ;   要加密数据的raw-address
   poly_ptr_decrypt_buf_va   dd  ?      ;
   poly_code_size            dd  ?      ;   加密数据的size
   poly_code_entry_offset    dd  ?      ;   加密数据的相对ep
   poly_decryptor_base       dd  ?      ;   解密数据的地址（相对虚拟地址），指向virus_body,而不是开头的 Gen trash data.
   poly_decryptor_base_va    dd  ?      ;   解密数据的virtual-address
   poly_decryptor_size       dd  ?      ;   解密数据的size
   poly_options              dd  ?      ;   poly 的设置信息
   poly_garbage_level        dd  ?      ;   产生仿真无效指令的等级 (1-低 3-中 5-高)
   poly_read_mem_base        dd  ?      ;
   poly_read_mem_size        dd  ?      ;
   poly_algo1                dd  ?      ;   加密算法定义（3种方案）
   poly_algo2                dd  ?      ;   滑动密钥算法（2种方案）
   poly_key                  dd  ?      ;   encrypt key
   poly_slide_key            dd  ?      ;   slide key
   poly_ptr_reg              dd  ?      ;   内存寻址时使用的register
   poly_key_reg              dd  ?      ;   保存解密的密钥的register
   poly_loop_reg             dd  ?      ;   循环计数
   poly_store_reg            dd  ?      ;   mov_data_loop时有效
   poly_junk_mem_reg         dd  ?      ;   下面几个都是产生仿真无效指令的结构，之前已经分析过
   poly_junk_mem_pos         dd  ?      ;
   poly_reg_usage            dd  ?      ;
   poly_random_seed          dd  ?      ;
   poly_garbler_flags        dd  ?      ;
   poly_entry_offset         dd  ?      ;
   poly_subroutines_count    dd    ?  ; 仅在产生sub_call时有效
   poly_subroutines_table    db    1024 DUP(0)
poly_vars ends
```

根据`poly_vars`里面数据（`poly_algo1`、`poly_algo2`）配置，可产生不同的加密方案。

整体的加密过程如下:

步骤 1 :进行初始化工作，调用`PolyInit`(清0，[`poly_vars->poly_algo1` ~ `poly_entry_offset`],设置`esp`,`ebp`为使用的寄存器，初始化随机种子)
2 :为`algo1`随机选择一个加密算法，`algo1`支持3种加密方式（`xor`/`add`/`sub`），为`algo2`随机选择一个加密算法,`algo2`支持2种加密方式(`add`/`sub`)
3 :随机获得存储`key`寄存器，`loop` 寄存器，`slide_key`寄存器，同时产生一个32bit的`key`
4 :加密病毒数据，由4部分组成（`Gen trash data` + `emulator instruction` + `virus body` + `Gen trash data`）
5 :初始化生成解密的相关数据，如待解密数据地址等
6 :在解密代码当前地址处插入仿真无效指令（包含随机的递归层数，所以从引擎设计角度讲该值不宜设置过大）
7 :建立一个栈帧，`push ebp` / `mov ebp,esp`
8 :同步骤6
9 :产生一个`loop`结构，一个`lea rx,key` 指令
10:同步骤6
11:产生一个获得`virus size`给`rx`，可以通过 `push imm` / `pop rx`、`mov rx，imm`、`lea rx,[imm]`
12: 同步骤6
13:产生一个`xor`/`add`/`sub [ptr_reg],key_reg` 结构的指令，指向要解密的数据同`key`运算
14:同步骤6
15:产生一个(`xor`/`add`/`sub key_reg,slide_key`)结构的指令，`slide_key` 同 13步的中间结果运算
16:同步骤6
17:产生一个`dec rx` ，`rx` 为循环计数
18:同步骤6
19:产生一个`loop`
20 同步骤6

由于加解密算法的可逆关系，`ope`记录了加密时的操作顺序。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

产生密钥的算法很简单:

```
generate_key_32:
      push ecx
      push ebx
      mov  ecx,4
generate_key_loop:
      mov eax,0ffh - 10
      call random
      add eax,10
      mov bl,al
      rol ebx,8
      dec ecx
      test ecx,ecx
      jnz generate_key_loop
      mov eax,ebx
      pop ebx
      pop ecx
      retn
```

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

采用的加密算法是在选择不同逻辑操作的同时，配合滑动密钥，下面给出加解密的公式:

`X` -- > 原始数据
`Y` -- > 加密后的数据
`K` -- > 32bit的密钥
`S` -- > 32bit的滑动密钥
`L1` -- > 加密逻辑1（`sub` / `add` / `xor`）
`L2` -- > 加密逻辑2 (`sub` / `add`)

最终的加解密公式:

`Y1 = L1(X,K);`
`Y  = L2(Y1,S);`

代码如下:

；`key`值保存在`eax`中

```
crypt_data:
  pushad
  mov edi,dword ptr [ebp].poly_ptr_code_base_raw
  mov ecx,dword ptr [ebp].poly_code_size
  mov ebx,dword ptr [ebp].poly_slide_key
crypt_loop:
  cmp dword ptr [ebp].poly_algo1,1
  je crypt_algo1
  cmp dword ptr [ebp].poly_algo1,2
  je crypt_algo2
  xor dword ptr [edi],eax
  jmp crypt_algo_c
crypt_algo1:
  sub dword ptr [edi],eax
  jmp crypt_algo_c
crypt_algo2:
  add dword ptr [edi],eax
crypt_algo_c:
  mov dword ptr [esp+1Ch],eax ; save eax
  cmp dword ptr [ebp].poly_algo2,1
  je crypt_slide1
  cmp dword ptr [ebp].poly_algo2,2
  je crypt_slide2
  xor eax,ebx
  jmp crypt_slide_c
crypt_slide1:
  sub eax,ebx
  jmp crypt_slide_c
crypt_slide2:
  add eax,ebx
crypt_slide_c:
  inc edi
  loop crypt_loop
  mov dword ptr [esp+1Ch],eax ; save eax
  popad
  retn
```

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

具体的`poly engine`实现可参考`ope src`，为了便于分析，现将去除掉`junk code`后`decryptor`代码列出:

```
00A02D70    55              push ebp
00A02D71    8BEC            mov ebp,esp
00A02D73    E8 00000000     call 00A02D78
00A02D78    5E              pop esi
00A02D79    81C6 34B2BFFF   add esi,FFBFB234
00A02D7F    81C6 54204000   add esi,402054             ；获得解密数据基址
00A02D85    E8 00000000     call 00A02D8A
00A02D8A    59              pop ecx
00A02D8B    81C1 22B2BFFF   add ecx,FFBFB222
00A02D91    81C1 10000000   add ecx,10
00A02D97    BA 5D0B0000     mov edx,0B5D
```
