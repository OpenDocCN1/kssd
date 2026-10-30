
```asm
.text:004028F0                 push    offset aLuajit210Beta3 ; "luajit2.1.0-beta3"

.text:004028F5                 push    917h            ; a3

.text:004028FA                 push    offset byte_45A578 ; a2

.text:004028FF                 push    esi             ; a1

.text:00402900                 call    luaL_loadbuffer

.text:00402905                 push    0

.text:00402907                 push    0

.text:00402909                 push    esi

.text:0040290A                 call    luaJIT_setmode

.text:0040290F                 push    0               ; a4

.text:00402911                 push    0               ; a3

.text:00402913                 push    0               ; a2

.text:00402915                 push    esi             ; a1

.text:00402916                 call    lua_pcall

.text:0040291B                 add     esp, 2Ch

.text:0040291E                 test    eax, eax

.text:00402920                 jnz     short loc_40296F

.text:00402922                 push    ebp

.text:00402923                 push    esi

.text:00402924                 call    lua_pushstring

.text:00402929                 push    offset aXut     ; "xut"

.text:0040292E                 push    0FFFFD8EEh

.text:00402933                 push    esi

.text:00402934                 call    lua_setfield

.text:00402939                 push    offset aMyst    ; "myst"

.text:0040293E                 push    0FFFFD8EEh

.text:00402943                 push    esi

.text:00402944                 call    lua_getfield

.text:00402949                 push    0               ; a4

.text:0040294B                 push    1               ; a3

.text:0040294D                 push    0               ; a2

.text:0040294F                 push    esi             ; a1

.text:00402950                 call    lua_pcall

...

.text:00402986                 push    0FFFFFFFFh      ; a2

.text:00402988                 push    esi             ; a1

.text:00402989                 call    lua_tonumber

.text:0040298E                 add     esp, 8

.text:00402991                 test    eax, eax

.text:00402993                 jz      short loc_4029A2

.text:00402995                 push    0FFFFFFFFh      ; a2

.text:00402997                 push    esi             ; a1

.text:00402998                 call    lua_tointeger

.text:0040299D                 add     esp, 8

.text:004029A0                 jmp     short loc_4029A6

.text:004029A2                 mov     eax, [esp+104h+var_F0]

.text:004029A6                 test    eax, eax

.text:004029A8                 jz      short loc_4029B1
```

```lua
xut=sn

if myst()==1 then ok

myst():

    x=xut

    x+=101*1001+(10101+1001*99)*100

    x*=983751509373

    x-=1023*13+1203*13*14+1230*13*14*15+1231*13*14*15*16

    x=(x+1)*2

    expected=1574592838300862641516215149137548264158058079230003764126382984039489925466995870724568174393389905601620735902909057604303543552180706761904

    if (x==expected) return 1

    else return 0
```

## LuaJIT 分析

根据 `luaJIT_setmode` 定位到 `lj_dispatch_update` 函数。

从 `lj_dispatch_update` 定位到 `lj_vm_asm_begin` 与 `lj_bc_ofs`。

在 `lj_vm_asm_begin+lj_bc_ofs[i]` 处下断，分析各个 bytecode 的功能。

```asm
.text:0040AFCE                 call    lj_dispatch_update

.text:0040ACA9                 movzx   esi, ds:lj_bc_ofs+96h

.text:0040ACB0                 movzx   edi, ds:lj_bc_ofs+9Ch

.text:0040ACB7                 movzx   ebp, ds:lj_bc_ofs+0A2h

.text:0040ACBE                 movzx   eax, ds:lj_bc_ofs+0AAh

.text:0040ACC5                 add     esi, offset lj_vm_asm_begin

.text:0040ACCB                 add     edi, offset lj_vm_asm_begin

.text:0040ACD1                 add     ebp, offset lj_vm_asm_begin

.text:0040ACD7                 add     eax, offset lj_vm_asm_begin
```

> KXCTF201710BYLoudy08

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
