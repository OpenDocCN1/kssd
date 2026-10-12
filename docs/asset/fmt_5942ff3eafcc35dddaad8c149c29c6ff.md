
### `Modrm_Reg_Reg`

```
    call get_reg_32_no_stack;产生 mov r1，r2   
    add eax,ebx ; free reg  
    add eax,0c0h  
    stosb  
    jmp Modrm_End  
```

### `Modrm_reg1_reg1`

```
    call get_reg_32 ; 产生 mov r1/r1  
    mov ah,al  
    rol al,3  
    add al,ah  
    add al,0c0h  
    stosb               
```

### `Modrm_End`

```
    pop ebx  
    retn  
```

### 4. `sub_call` 方式

1. 检测配置中 `poly_subroutines_count` 是否为 0？是 0 则退出，否则步骤 2
2. 检测配置中是否设置 `subroutine` 标志，没设置退出
3. 检测 `poly_subroutines_table` 中是否参数标志，因为产生的 `call` 是按 `__cdecl` 方式压栈的
4. 如果是有参数的情况，负责清栈。
5. 调用 `Garble_Create_Push`，产生不同类型的 `push 32_bit` / `push 8_bit` / `push rx` / `push [rx + rx]`
6. 生成 `call` 指令

### `Garble_Sub_Call`

```
    push ecx  
    push ebx  
    cmp dword ptr [ebp].poly_subroutines_count,0   ; 检测是否设置的 sub_call 方式  
    je Garble_Sub_Call_End  
    call Is_In_Subr ;   
    test eax,eax  
    jnz Garble_Sub_Call_End  
    mov eax,dword ptr [ebp].poly_subroutines_count ;读取计数  
    call random  
    lea eax,[eax * 8]  
    lea ebx,[ebp].poly_subroutines_table  
    add ebx,eax  
    cmp dword ptr [ebp].poly_junk_mem_pos,0        ; 比较是否初始化 junk 内存数据  
    je Garble_Sub_No_Save1  
    mov ecx,dword ptr [ebp].poly_junk_mem_reg  
    add ecx,50h  
    mov byte ptr [edi],cl ;   
    inc edi  
```

### `Garble_Sub_No_Save1`

```
    mov ecx,dword ptr [ebx + 4] ;     
    test ecx,ecx  
    jz Garble_Sub_No_Arg                        ;生成无参数 call 指令  
```

### `Garble_Sub_Arg`

```
    call Garble_Create_Push  
    loop Garble_Sub_Arg  
```

### `Garble_Sub_No_Arg`

```
    mov eax,dword ptr [ebx]  
    sub eax,edi  
    sub eax,5  
    mov byte ptr [edi],0e8h  
    inc edi  
    stosd        
    mov eax,dword ptr [ebx + 4] ;   
    test eax,eax  
    jz Garble_Sub_No_Save2  
    imul eax,4  
    rol eax,16  
    add eax,9000c483h        ； 产生 sub esp , xxxx,清空堆栈                 
    stosd  
    dec edi  
```

### `Garble_Sub_No_Save2`

```
    cmp dword ptr [ebp].poly_junk_mem_pos,0  
    je Garble_Sub_Call_End  
    mov eax,dword ptr [ebp].poly_junk_mem_reg  
    add eax,58h  
    stosb ;         pop rx          
```

### `Garble_Sub_Call_End`

```
    pop ebx  
    pop ecx  
    retn  
```

### 5. `loop` 方式

1. 检测配置中是否设置 `loop` 方式，随机设置 `loop` 的循环次数，范围 `10000h` ~ `1000h`
2. 初始化 `loop` 所用的寄存器
3. 产生一个对 `rx` 赋值循环计数的指令，支持的格式包括 `mov rx,cnt` / `lea rx ,[cnt]` / `push cnt ,pop rx`
4. 在 `loop` 插入仿真无效指令
5. 递减计数

### `Garble_Loop`

```
    call Is_In_Loop  
    test eax,eax  
    jnz Garble_Loop_End  
    cmp ecx,5  
    jbe Garble_Loop_End  
    call get_free_reg_32  
    cmp eax,-1  
    je Garble_Loop_End  
    call Set_In_Loop ; 保存设置  
    mov ebx,eax  
    mov eax,POLY_LOOP_ITERATION_MAX ;   最大 loop 计数  
    call random  
    add eax,POLY_LOOP_ITERATION_MIN   ; 获得一个随机的 loop 计数  
    push ecx  
    mov ecx,eax  
    call Asm_Mov_Reg_Imm_32          ;产生一个 mov rx ，imm32 指令  
    pop ecx  
    push edi  
    push ebx  
    dec ecx  
    call Garble ;插入无效指令  
    pop eax  
    mov ebx,eax  
    call unset_reg_32 ; 递减计数    
    mov al,48h  
    add al,bl  
    stosb  
    mov al,085h ;测试计数  
    stosb  
    mov eax,ebx  
    rol eax,3  
    add eax,ebx  
    add eax,0c0h  
    stosb  
    pop eax ; 产生一个 jcc  
    sub eax,edi  
    push eax  
    not eax  
    cmp eax,255 / 2  
    pop eax  
    jbe Garble_Loop_Short  
    sub eax,6    ;产生一个 near jcc  
    mov word ptr [edi],0850fh  
    mov dword ptr [edi + 2],eax  
    add edi,6  
    jmp Garble_Loop_Cont        ;产生一个 short jcc          
```

\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

经过上述变换，产生的无效指令效果如下，可以看到如果不经过分析，是不容易利用程序分析出这些是无效指令的。

### `loop`

```
    00A00298    68 DE320000     push 32DE  
    00A0029D    5E              pop esi  
    00A0029E    337C24 00       xor edi,dword ptr ss:[esp]  
    00A002A2    4E              dec esi  
    00A002A3    85F6           test esi,esi  
    00A002A5  ^ 75 F7          jnz short 00A0029E  
```

### `push`/`pop`

```
    00A009D8    54              push esp  
    00A009D9    8B5C24 00       mov ebx,dword ptr ss:[esp]  
    00A009DD    5A              pop edx  
    00A009DE    BF 7B3DF36F     mov edi,6FF33D7B  
    00A009E3    FF75 F0         push dword ptr ss:[ebp-10]  
    00A009E6    53              push ebx  
```

### `jump`

```
    00A009EB  /74 04            je short 00A009F1  
    00A009ED  |40               inc eax  
    00A009EE  |66:33F0          xor si,ax  
    00A009F1  \\FF7424 00       push dword ptr ss:[esp]  
    00A009F5    4A              dec edx  
    00A009F6    8BC0            mov eax,eax  
    00A009F8    5B              pop ebx  
```

### `imm`

```
    00A00A06    BB 14C44F40     mov ebx,404FC414  
    00A00A0B    33DE            xor ebx,esi  
    00A00A0D    0B4D 00         or ecx,dword ptr ss:[ebp]  
    00A00A10    8B7D 00         mov edi,dword ptr ss:[ebp]  
```

### `mod/rm`

```
    00A009CA    8D15 2C8A0000   lea edx,dword ptr ds:[8A2C]  
    00A009D0    66:33C0         xor ax,ax  
    00A009D3    4A              dec edx  
    00A009D4    85D2            test edx,edx  
    00A009D6  ^ 75 F8           jnz short 00A009D0  
```

### `call`

```
    00A02B10    E8 03000000     call 00A02B18  
    00A02B15    234D 00         and ecx,dword ptr ss:[ebp]  
    00A02B18    5F              pop edi  
    00A02B19    81C7 EBD45FFF   add edi,FF5FD4EB  
    00A02B1F    81C7 F52AA000   add edi,0A02AF5  
    00A02B25    8B4C24 00       mov ecx,dword ptr ss:[esp]  
```

### 2.5.3 `poly` engine 的实现

`ope` 采用的是随机密钥+滑动密钥方案，加密支持的指令为(`xor` / `sub` / `add`),滑动密钥支持的指令为（`sub` / `add` / `xor`),加密中 `ope` 维护一个 `poly_vars` 结构体，定义如下:

```
poly_vars struct  
   poly_ptr_code_base_va     dd  ?      ;   要加密数据的 virtual-address  
   poly_ptr_code_base_raw    dd  ?      ;   要加密数据的 raw-address  
   poly_ptr_decrypt_buf_va   dd  ?      ;     
   poly_code_size            dd  ?      ;   加密数据的 size  
   poly_code_entry_offset    dd  ?      ;   加密数据的相对 ep  
   poly_decryptor_base       dd  ?      ;   解密数据的地址（相对虚拟地址），指向 virus_body,而不是开头的 Gen trash data.  
   poly_decryptor_base_va    dd  ?      ;   解密数据的 virtual-address  
   poly_decryptor_size       dd  ?      ;   解密数据的 size   
   poly_options              dd  ?      ;   poly 的设置信息  
   poly_garbage_level        dd  ?      ;   产生仿真无效指令的等级 (1-低 3-中 5-高)  
   poly_read_mem_base        dd  ?      ;     
   poly_read_mem_size        dd  ?      ;     
   poly_algo1                dd  ?      ;   加密算法定义（3 种方案）  
   poly_algo2                dd  ?      ;   滑动密钥算法（2 种方案）  
   poly_key                  dd  ?      ;   encrypt key  
   poly_slide_key            dd  ?      ;   slide key   
   poly_ptr_reg              dd  ?      ;   内存寻址时使用的 register  
   poly_key_reg              dd  ?      ;   保存解密的密钥的 register  
   poly_loop_reg             dd  ?      ;   循环计数  
   poly_store_reg            dd  ?      ;   mov_data_loop 时有效  
   poly_junk_mem_reg         dd  ?      ;   下面几个都是产生仿真无效指令的结构，之前已经分析过  
   poly_junk_mem_pos         dd  ?      ;    
   poly_reg_usage            dd  ?      ;    
   poly_random_seed          dd  ?      ;    
   poly_garbler_flags        dd  ?      ;    
   poly_entry_offset         dd  ?      ;    
   poly_subroutines_count    dd    ?  ; 仅在产生 sub_call 时有效  
   poly_subroutines_table    db    1024 DUP(0)  
poly_vars ends  
```

根据 `poly_vars` 里面数据（`poly_algo1`、`poly_algo2`）配置，可产生不同的加密方案。

整体的加密过程如下:
