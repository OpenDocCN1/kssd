
### 判断操作码

`72 0E`                   `jb`      `Opcode_7X`

-----------------------------------------------------------
判断是否是7字头的操作码（1）

`80 E9 70`                `sub`     `cl, 70h`

`80 E9 10`                `sub`     `cl, 10h`

`72 0E`                   `jb`      `Opcode_7X`

------------------------------------------
判断操作码是否为C2或C3

`add`     `al, 3Eh`

`sub`     `al, 2`

`jb`      `short C2C3`

------------------------------------------
判断操作码是否为06或07

```
debug032:01054540 04 FA                   add     al, 0FAh
debug032:01054542 2C 02                   sub     al, 2
debug032:01054544 72 03                   jb      short lo
```
―――――――――――――――――――――判断操作码是否是E0, E1, E2, E3

`04 20`                   `add`     `al, 20h`

`2C 04`                   `sub`     `al, 4`

`0F 92 C0`                `setb`    `al`

在原程序中的`Emulatee`一共要进入二次，第一次进是为第二次作准备工作。

```
seg024:004E200E 68 14 26 68 6A    push    6A682614h　　　　　　  ; ???
seg024:004E2013 68 BD 46 11 6A    push    6A1146BDh         ; RVA = 6A1F66B3 XOR 6A1146BD
seg024:004E2018 68 88 8B 12 01    push    offset off_1128B88　　; 指针，指向ObjX_1_Fun1
seg024:004E201D E8 F2 48 B7 00    call    Step1
```

函数`Step1()`的工作无它，为第二步真正执行作好准备工作。

`ObjX_1_Fun1`中的数据成员对我们有帮助

+ `18`          `pAspEmulHdr`的数组
+ `1C`          程序中一共有几处`Emulatee`

+
------------------------------------------------------------
程序中一共有五处`Emulatee`

```
debug266:0112A890 B4 8B 12 01       dd offset dword_1128BB4 ; E1382
debug266:0112A894 4C 92 12 01       dd 112924Ch       ; E200E
debug266:0112A898 58 94 12 01       dd 1129458h        ; E50BB
debug266:0112A89C FC 9B 12 01       dd 1129BFCh      ; E53EC
debug266:0112A8A0 70 A7 12 01       dd unk_112A770   ; E6878
```
------------------------------------------------------------

(1) `004E200E`          `FOnClick`         “Project” -> “New Project” Menu Item

(2) `004E1382`          `FOnClick`         “File To Protect”

(3)  `004E50BB`                          “Registration” -> “Generate” Button

(4) `004E53EC`                           “Registration” -> “Check” Button

(5) `004E6878`                           “Protect Original OEP” Checkbox

### 第二次进入

```
seg024:004E2008 53                push    ebx
seg024:004E2009 56                push    esi
seg024:004E200A 8B F2             mov     esi, edx
seg024:004E200C 8B D8             mov     ebx, eax
seg024:004E200E 68 00 00 00 00    push    0                  ; pVirMem
seg024:004E2013 68 BD 46 51 6A    push    6A5146BDh       ; RVA = 6A5146BD XOR 6A1F66B3h
seg024:004E2018 68 18 AC 12 01    push    offset AspEmulHdr  ;
seg024:004E201D E8 1E 49 B7 00    call    Step2
```

```
/*
  [Type List]
  _Aspr_Emul_2_4_11_20_Type_1
  used for protector.dll self
  _Aspr_Emul_2_4_11_20_Type_2
  used for protectee
*/
```

### `_Aspr_Emul_2_4_11_20_Type_1`

// used for protector.dll self

```
#ifdef _Aspr_Emul_2_4_11_20_Type_1
// used for protector.dll self
struct  _AsprEmulInsHdr{
  DWORD  InsHdr_cpTAsprEmul;     //  + 0x000
  DWORD  InsHdr_pInsBody;        //  + 0x004
  DWORD  InsHdr_InsItemNum;      //  + 0x008
  BYTE   InsHdr_FuncInds1[0x0A]; //  + 0x00C
  BYTE   InsHdr_FuncInds2[0x0A]; //  + 0x016
  DWORD  InsHdr_GetFuncs[0x0A];  //  + 0x020
  DWORD  InsHdr_InsItemSize;     //  + 0x048
  DWORD  InsHdr_ImageBase;       //  + 0x04C
  DWORD  InsHdr_RandInt;         //  + 0x050
  DWORD  InsHdr_ProcId ;         //  + 0x054
  DWORD  InsHdr_InsBodySize;     //  + 0x058
} AsprEmulInsHdr, *PAsprEmulInsHdr;

typedef  struct  _AsprEmulInsItem{
    BYTE   SIB;           // + 000
                          // + 001
    BYTE   ModRM          // + 006
    DWORD                 // + 007
    BYTE   FakeOpcode0;   // + 00B
    BYTE                  // + 00C
    DWORD  FakeEip;       // + 00D
                          // + 011
    BYTE   FakeOpcode1;   // + 014
                          // + 015
    DWORD  Disp;          // + 01A
                          // + 01E
    DWORD FakeImm0;       // + 023
    BYTE                  // + 026
    BYTE                  // + 027
    BYTE  Misc;           // + 028
    BYTE                  // + 02C
    BYTE  bAdjust;        // + 02D
                          // + 02E
} AsprEmulInsItem, *PAsprEmulInsItem;
                          // + 033

typedef struct _AsprEmulContext{
                             // + 000
    DWORD  Eax;              // + 020
    DWORD  Ecx;              // + 024
    DWORD  Edx;              // + 028
    DWORD  Ebx;              // + 02C
    DWORD  Esp;              // + 030
    DWORD  Ebp;              // + 034
    DWORD  Esi;              // + 038
    DWORD  Edi;              // + 03C
    DWORD                    // + 040
    DWORD  Eip;              // + 044
    DWORD  Eflags;           // + 048
                             // + 04C
    DWORD  CsBase;           // + 060
    DWORD  SsBase;           // + 064
    DWORD  DsBase;           // + 068
    DWORD  EsBase;           // + 06C
    DWORD  FsBase;           // + 070
    DWORD  GsBase;           // + 074
    WORD   Cs;               // + 078
    WORD   Ss;               // + 07A
    WORD   Ds;               // + 07C
    WORD   Es;               // + 07E
    WORD   Fs;               // + 080
    WORD   Gs;               // + 082
                             // + 084
    DWORD  LowOpcodeNum;     // + 090
    DWORD  HighOpcodeNum;    // + 094
    DWORD  curSegBase;       // + 098
    BYTE   bAdjust;          // + 09C
    DWORD  pOrigERR;         // + 09D
    BYTE   b2BOpcode;        // + 0A1
    DWORD  SrcOperandInfo;   // + 0A2
    BYTE   SrcOperandMod;    // + 0A6
    DWORD  DestOperandInfo;  // + 0A7
    BYTE   DestOperandMod;   // + 0AB
    DWORD  OperandSize;      // + 0AC
    BYTE   PrefixFlags;      // + 0B0
    BYTE                     // + 0B1
    BYTE   FakeOpcode;       // + 0B4
    DWORD  curInsItem;       // + 0B5
    DWORD  nextInsItem;      // + 0B9
} AsprEmulContext, * PAsprEmulContext;
                           // + 0BD
#endif  // #ifdef _Aspr_Emul_2_4_11_20_Type_1
```

### `_Aspr_Emul_2_4_11_20_Type_2`

// used for protectee

```
#ifdef _Aspr_Emul_2_4_11_20_Type_2
// used for protectee
struct  _AsprEmulInsHdr{
  DWORD  InsHdr_cpTAsprEmul;            //  + 0x000
  DWORD  InsHdr_RandInt;                //  + 0x004
  DWORD  InsHdr_pcpTManualInstall;      //  + 0x008
  DWORD  InsHdr_InsItemNum;             //  + 0x00C
  DWORD  InsHdr_ImageBase;              //  + 0x010
  DWORD  InsHdr_ProcId;                 //  + 0x014
  DWORD  InsHdr_pInsBody;               //  + 0x018
  DWORD                                 //  + 0x01C
} AsprEmulInsHdr, *PAsprEmulInsHdr;
                                        //  + 0x020

typedef  struct  _AsprEmulInsItem{
  Byte  FakeOpcode0H4;     // + 000
  Byte  FakeOpcode0L4;     // + 001
  Byte  bInsHdrASM;        // + 002
  Byte  SIB;               // + 003
  DWORD FakeImm0;          // + 004
  DWORD FakeEip;           // + 008
  BYTE                     // + 00C
  BYTE  ModRM;             // + 00D
  DWORD Disp;              // + 00E
  Byte  FakeOpcode1H4;     // + 012
  Byte  FakeOpcode1L4;     // + 013
  Byte  bAdjust;           // + 014
  Byte  Opcode1Flags;      // + 015
  Byte  Opcode0Flags;      // + 016
  BYTE  Misc;              // + 017
} AsprEmulInsItem, *PAsprEmulInsItem;
                           // + 018

typedef  struct _AsprEmulInsHdrASM{
                              // + 000
  BYTE  bInsHdrASM;           // + 002
  DWORD FakeEip;              // + 008
  BYTE  InsItemNum;           // + 00C
  BYTE  CodeSize;             // + 00D
  DWORD InsItemIndex;         // + 00E
                              //
}  AsprEmulInsHdrASM, *PAsprEmulInsHdrASM;
                              // + 018

typedef struct  _AsprEmulInsItemASM{
  BYTE   WorkType;            // + 000
  BYTE   AddrMod;             // + 001
  BYTE   Flags;               // + 002
  BYTE   DestRegCode;         // + 003
  BYTE   SrcRegCode;          // + 004
  DWORD  DestDisp;            // + 005
  DWORD  SrcDispImm;          // + 009
} AsprEmulInsItemASM, *PAsprEmulInsItemASM;
```
