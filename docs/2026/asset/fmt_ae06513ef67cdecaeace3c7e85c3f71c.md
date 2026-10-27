
# TVM 解码分析

```
0xe8 p_a b_ULONG64
0x3F26
*(PULONG64)p_a = b_ULONG64;
v_mov_iregll_ll
----------------------------------------//虚线以上是分析出来的
0x1400d60f5 : mov r9, [rbp+8]
0x1400d6102 : mov r8w, [r9] 这里取2字节 (取虚拟寄存器都是取2字节)
0x1400d6106 : xor r8w, 3F26h 异或 3F26(不同的handl异或的值不同) 得到 a,
0x1400d610c : mov rdx, 0F84A86395161A270h 所以就是取虚拟机寄存器 V_REG_P + a
0x1400d6116 : not rdx
0x1400d6119 : jmp loc_1400D9077
0x1400d9078 : lea rdx, [r10+rdx]
0x1400d907c : movzx r8, r8w
0x1400d9080 : mov rcx, 7B579C6AE9E5D8Eh
0x1400d908a : not rcx
0x1400d908d : add r8, rcx
0x1400d9090 : lea r8, [rdx+r8] p_a = r8 = V_REG_P + a
0x1400d9094 : lea r9, [r9+2] V_RIP+=2
0x1400d909b : mov rdx, [r9] b_ULONG64 从tvm指令表中取8字节
0x1400d909e : jmp loc_1400DA2DD
0x1400da2ed : mov [r8], rdx 放入 [r8],即放入 p_a
0x1400da2f0 : jmp loc_1400DA9AC
0x1400da9b4 : lea r9, [r9+8] V_RIP+=8
0x1400da9be : mov [rbp+8], r9 V_RIP放回[rbp+8]
0x1400da9c8 : jmp loc_1400DAC79
0x1400dac7c : mov r9, [rbp+8] 这里以下属于下一个handle分发，不用看
0x1400dac89 : mov r8b, [r9]
0x1400dac8c : xor r8b, 5Dh
0x1400dac90 : mov rdx, 0D3676A56DAFF3C65h
0x1400dac9a : jmp loc_1400D7763
0x1400d7764 : not rdx
0x1400d7767 : lea rdx, [r9+rdx]
0x1400d776b : mov r9, 2C9895A92500C398h
0x1400d7775 : not r9
0x1400d7778 : lea r9, [rdx+r9]
0x1400d777f : jmp loc_1400D5B91
0x1400d5b95 : mov [rbp+8], r9
0x1400d5b9e : movzx r8, r8b
0x1400d5ba2 : sub r8, 1; switch 200 cases
0x1400d5ba6 : jmp loc_1400D98A8
0x1400d98aa : cmp r8, 0C8h
0x1400d98b1 : jnb def_1400D655C;
```

## 我对 tvmAsm 的命名规则

```
v_opcode_op0_op1_op2_..... _opn
例如:
v_mov_iregw_iregw :
从虚拟寄存器 op1 取 2字节 放入虚拟寄存器 op0
w 表示 USHORT
v_mov_ipreg_iregl :
从虚拟机寄存器 op1 取4字节，放入以虚拟机寄存器 op0作为地址的空间，有点类似与:
mov [reg],reg
v_mov_iregb_b :
从tvm指令表中取1字节放入虚拟机寄存器 op0中,类似于Asm中的立即数赋值:
mov reg,0xff
v_and_oregl_iregl_iregl_oregl:(i 就是 in ，o 就是 out 的意思)
取虚拟寄存器 op1 和 虚拟寄存器 op2进行与运算，并将结果放入虚拟寄存器 op0，与运算后的 rflag 放入 虚拟寄存器 op3
l 表示 ULONG32
v_cmp_iregll_iregll_oregl:
即为 cmp 虚拟寄存器 op0,虚拟寄存器 op1 ，cmp后的rflag 放入 虚拟寄存器 op2
ll 表示 ULONG64
```

## TVM 表

我这里直接将全部 tvmAsm 展示出来：（参考 `idapython/deTvm.py . tvmHandleTableInit()`）

```
TVMTABEL.append("v_sar_oregll_iregll_iregb_oregl",0x01,0xF8BE)
TVMTABEL.append("v_or_oregll_iregll_iregll_oregl",0x1D,0x4AA7)
TVMTABEL.append("v_mov_iregll_iregl",0x2B,0xBF3E)
TVMTABEL.append("v_movzx_iregl_iregb",0x2F,0x7EE9)
TVMTABEL.append("v_ror_oregb_iregb_iregb_oregl",0x3C,0xF8E1)
TVMTABEL.append("v_mov_iregl_iregl",0x4A,0x564B)
TVMTABEL.append("v_mov_iregw_iregw",0x4B,0xD916)
TVMTABEL.append("v_add_oregb_iregb_iregb_oregl",0x4C,0xDD9D)
TVMTABEL.append("v_add_oregll_iregll_iregll",0x4D,0x477D)
TVMTABEL.append("v_add_oregl_iregl_iregl_oregl",0x4E,0xA9C7)
TVMTABEL.append("v_add_oregw_iregw_iregw_oregl",0x4F,0x82BC)
TVMTABEL.append("v_sub_oregl_iregl_iregl_oregl",0x5A,0xC198)
TVMTABEL.append("v_sar_oregl_iregl_iregb_oregl",0x06,0x8374)
TVMTABEL.append("v_and_oregl_iregl_iregl_oregl",0x6A,0x5CF0)
TVMTABEL.append("v_and_oregll_iregll_iregll_oregl",0x6D,0xD9B1)
TVMTABEL.append("v_and_oregl_iregl_iregl",0x6E,0xA1CE)
TVMTABEL.append("v_xor_oregl_iregl_iregl_oregl",0x7A,0xD8ED)
TVMTABEL.append("v_mov_ipreg_iregll",0x7D,0xD878)
TVMTABEL.append("v_shr_oregll_iregll_iregb_oregl",0x09,0x9D87)
TVMTABEL.append("v_int3",[0x9a,0x9b,0x98,0x99,0x9c,0x9d,0xe2,0xe3,0xe0,0xe1,0xe6,0xe7,0xe4,0xeb,0xec,0xed,0xf2,0xf3,0xf0,0xf1,0xf6,0xf7,0xf5,0xfa,0xfb,0xf9,0xfe,0xff,0xc2,0xc0,0xc1,0xc6,0xc7,0xc4,0xc5,0xc8,0xce,0xcf,0xcc,0xcd,0xdb,0xd8,0xde,0xdf,0xdc,0xdd,0x23,0x26,0x27,0x24,0x2a,0x28,0x2e,0x2c,0x2d,0x32,0x33,0x30,0x31,0x36,0x37,0x34,0x35,0x3a,0x3b,0x38,0x39,0x3e,0x3f,0x3d,0x2,0x3,0x0,0x7,0x4,0x5,0xa,0xb,0x8,0xe,0xf,0xd,0x12,0x13,0x10,0x17,0x14,0x15,0x1a,0x1b,0x18,0x1e,0x1f,0x1c,0x63,0x61,0x66,0x67,0x64,0x6b,0x69,0x6f,0x6c,0x71,0x76,0x77,0x74,0x7b,0x79,0x7e,0x7f,0x7c,0x53,0x50,0x55,0x5b,0x58,0x59,0x5e,0x5f,0x5c],0x0000)
TVMTABEL.append("v_jmp_iregxR11",0x9E,0x0AD7)
TVMTABEL.append("v_jmp_iregxR10",0x9F,0x2E72)
TVMTABEL.append("v_shl_oregll_iregll_iregb_oregl",0x11,0x5403)
TVMTABEL.append("v_shl_oregl_iregl_iregb_oregl",0x16,0xEEF7)
TVMTABEL.append("v_not_oregll_iregll",0x19,0x1400)
TVMTABEL.append("v_setz_oregb_iregl",0x20,0x0D45)
TVMTABEL.append("v_movsxd_iregll_iregl",0x21,0x8BC8)
TVMTABEL.append("v_setR8d_iregl",0x22,0x77D7)
TVMTABEL.append("v_movsx_iregl_iregb",0x25,0xD8E4)
TVMTABEL.append("v_movzx_iregl_iregw",0x29,0xF10A)
TVMTABEL.append("v_mov_ipreg_iregb",0x40,0xE304)
TVMTABEL.append("v_mov_iregll_ipreg",0x41,0xE229)
TVMTABEL.append("v_mov_ipreg_iregl",0x42,0x5431)
TVMTABEL.append("v_mov_ipreg_iregw",0x43,0x02CB)
TVMTABEL.append("v_mov_iregb_ipreg",0x44,0xBE8C)
TVMTABEL.append("v_mov_iregll_iregll",0x45,0x58FB)
TVMTABEL.append("v_mov_iregl_ipreg",0x46,0x10BC)
TVMTABEL.append("v_mov_iregw_ipreg",0x47,0x6F62)
TVMTABEL.append("v_mov_iregb_iregb",0x48,0xCFFE)
TVMTABEL.append("v_add_oregll_iregll_iregll_oregl",0x49,0x41AA)
TVMTABEL.append("v_not_oregll_iregll",0x51,0xDB42)
TVMTABEL.append("v_add_oregl_iregl_iregl",0x52,0x77C6)
TVMTABEL.append("v_not_oregb_iregb",0x54,0xDCF3)
TVMTABEL.append("v_not_oregl_iregl",0x56,0xE297)
TVMTABEL.append("v_not_oregw_iregw",0x57,0x666D)
TVMTABEL.append("v_or_oregb_iregb_iregb_oregl",0x60,0xFBFD)
TVMTABEL.append("v_or_oregl_iregl_iregl_oregl",0x62,0x7819)
TVMTABEL.append("v_and_oregll_iregll_iregll_oregl",0x65,0x2954)
TVMTABEL.append("v_and_oregb_iregb_iregb_oregl",0x68,0xD8A5)
TVMTABEL.append("v_and_oregb_iregb_iregb_oregl",0x70,0x64D1)
TVMTABEL.append("v_and_oregl_iregl_iregl_oregl",0x72,0x4A64)
TVMTABEL.append("v_and_oregw_iregw_iregw_oregl",0x73,0xB562)
TVMTABEL.append("v_xor_oregll_iregll_iregll_oregl",0x75,0x69C2)
TVMTABEL.append("v_xor_oregb_iregb_iregb_oregl",0x78,0x19C1)
TVMTABEL.append("v_ret_iregx",0x95,0x805C)
TVMTABEL.append("v_shr_oregb_iregb_iregb_oregl",0x0c,0x62D7)
TVMTABEL.append("v_dec_oregl_iregl_oregl",0xc3,0x467C)
TVMTABEL.append("v_inc_oregb_iregb_oregl",0xc9,0x4267)
TVMTABEL.append("v_inc_oregll_iregll_oregl",0xca,0x6EE0)
TVMTABEL.append("v_inc_oregl_iregl_oregl",0xcb,0x7FB4)
TVMTABEL.append("v_test_iregw_iregw_oregl",0xd0,0x0499)
TVMTABEL.append("v_test_iregb_iregb_oregl",0xd1,0x2A99)
TVMTABEL.append("v_test_iregll_iregll_oregl",0xd2,0x8606)
TVMTABEL.append("v_test_iregl_iregl_oregl",0xd3,0x7FDE)
TVMTABEL.append("v_cmp_iregw_iregw_oregl",0xd4,0x87DF)
TVMTABEL.append("v_cmp_iregb_iregb_oregl",0xd5,0x3728)
TVMTABEL.append("v_cmp_iregll_iregll_oregl",0xd6,0x637D)
TVMTABEL.append("v_cmp_iregl_iregl_oregl",0xd7,0xCBEF)
TVMTABEL.append("v_sbb_oregb_iregb_iregb_oregl",0xd9,0x3F78)
TVMTABEL.append("v_sbb_oregll_iregll_iregll_oregl",0xda,0x0E0D)
TVMTABEL.append("v_jmp_iregxRax",0xe5,0xCD84)
TVMTABEL.append("v_mov_iregll_ll",0xe8,0x3F26)
TVMTABEL.append("v_mov_iregl_l",0xe9,0x448A)
TVMTABEL.append("v_mov_iregll_ll",0xea,0x43EF)
TVMTABEL.append("v_mov_iregw_w",0xee,0x44B1)
TVMTABEL.append("v_mov_iregb_b",0xef,0x144C)
TVMTABEL.append("v_mul_oregll_oregll_iregll_iregll",0xf4,0xEB97)
TVMTABEL.append("v_jmp_ll",0xf8,0x0000)
TVMTABEL.append("v_rep stosb_iregll_iregb_iregll",0xfc,0x8D54)
TVMTABEL.append("v_je_iregb_ll_ll",0xfd,0xDE5E)
```

### 解释

`TVMTABEL.append` 第一个参数是我给 tvmAsm 取的名字，第二个参数就是 `tvmopcode`，第三个参数是：如果这个 handle 要取虚拟寄存器，就必须通过 这个值解密取得虚拟寄存器，就像是我上文中解释的：

```
TVMTABEL.append("v_mov_iregll_ll",0xe8,0x3F26)
```

这里再挑几个特殊说明一下：
