
## 代码分析与注释

### 加密解密逻辑

`SDPI:0047B8AF`            `add`  `edx`, `32h`

`SDPI:0047B8B2`            `call`   `Crypt_Decrypt_CODE`         ; 传入 `edx`(结束地址) 传入 `EAX`(起始地址)

`SDPI:0047B8B2`            ; 用于加密或解密代码, 加解密操作是互换的

`SDPI:0047B8B2`            ; 是只简单的 `xor`, 因此可以用来加密也可以

`SDPI:0047B8B2`            ; 用来解密

`SDPI:0047B8B7`            `call`   `loc_47B8BD`

`SDPI:0047B8BC`            `nop`

`SDPI:0047B8BD`

`SDPI:0047B8BD` `loc_47B8BD`:          ; CODE XREF: `fnddbg_47B89C`+1B p

`SDPI:0047B8BD`            `pop`  `eax`

`SDPI:0047B8BE`            `add`  `eax`, `37E0h`

`SDPI:0047B8C3`            `call`   `loc_47B8C9`

`SDPI:0047B8C8`            `nop`

`SDPI:0047B8C9`

`SDPI:0047B8C9` `loc_47B8C9`:          ; CODE XREF: `fnddbg_47B89C`+27 p

`SDPI:0047B8C9`            `pop`  `ecx`

`SDPI:0047B8CA`            `add`  `ecx`, `3881h`

`SDPI:0047B8D0`            `push`   `0`

`SDPI:0047B8D2`            `push`   `ecx`

`SDPI:0047B8D3`            `push`   `eax`

`SDPI:0047B8D4`            `push`   `0`

`SDPI:0047B8D6`            `call`   `loc_47B8DC`

`SDPI:0047B8DB`            `nop`

`SDPI:0047B8DC`

`SDPI:0047B8DC` `loc_47B8DC`:          ; CODE XREF: `fnddbg_47B89C`+3A p

`SDPI:0047B8DC`            `pop`  `eax`

`SDPI:0047B8DD`            `add`  `eax`, `11h`

`SDPI:0047B8E2`            `push`   `eax`

`SDPI:0047B8E3`            `jmp`    `MessageBoxA`                ; `MessageBoxA` 函数，壳用的宏来处理，

`SDPI:0047B8E3`            ; 判断函数的前 5 位是否为 `CC`

`SDPI:0047B8E3`            ; 也就是判断有没有下 `int3` 断点

`SDPI:0047B8E3` ; ----------------------------------------------------------

`SDPI:0047B8E8`            `db`  `90h` ; ?

`SDPI:0047B8E9`            `db`  `90h` ; ?

`SDPI:0047B8EA`            `db`  `90h` ; ?

`SDPI:0047B8EB`            `db`  `90h` ; ?

`SDPI:0047B8EC` ; ----------------------------------------------------------

`SDPI:0047B8EC`            `push`   `7`

`SDPI:0047B8EE`            `call`   `loc_47B8F4`

`SDPI:0047B8F3`            `nop`

`SDPI:0047B8F4`

`SDPI:0047B8F4` `loc_47B8F4`:          ; CODE XREF: `fnddbg_47B89C`+52 p

`SDPI:0047B8F4`            `pop`  `eax`

`SDPI:0047B8F5`            `add`  `eax`, `11h`

`SDPI:0047B8FA`            `push`   `eax`

`SDPI:0047B8FB`            `jmp`    `ExitProcess`

`SDPI:0047B8FB` ; ----------------------------------------------------------

`SDPI:0047B900` `aRrrrs@I`        `db` '悙悙?,0,10h,'@',0,'皦?,4

`SDPI:0047B900` `fnddbg_47B89C`    `endp`

`SDPI:0047B900`

`SDPI:0047B90D` ; ----------------------------------------------------------

`SDPI:0047B90D`

`SDPI:0047B90D` `Pass_47B90D`:           ; CODE XREF: `SDPI:0047B89A` j

`SDPI:0047B90D`            `pop`  `large dword ptr fs:0`

`SDPI:0047B914`            `add`  `esp`, `4`

`SDPI:0047B917`            `call`   `loc_47B91D`

`SDPI:0047B91C`            `nop`

`SDPI:0047B91D`

`SDPI:0047B91D` `loc_47B91D`:          ; CODE XREF: `SDPI:0047B917` p

`SDPI:0047B91D`            `pop`  `eax`

`SDPI:0047B91E`            `add`  `eax`, `0FFFFFE1Dh`

`SDPI:0047B923`            `cmp`  `byte ptr [eax]`, `0E9h`

`SDPI:0047B926`            `jnz`  `fnddbg_47B89C`

`SDPI:0047B92C`            `mov`  `byte ptr [eax]`, `0E8h`

`SDPI:0047B92F`            `rdtsc`

`SDPI:0047B931`            `mov`  `ecx`, `eax`

`SDPI:0047B933`            `mov`  `ebx`, `edx`

`SDPI:0047B935`            `rdtsc`

`SDPI:0047B937`            `sub`  `eax`, `ecx`

`SDPI:0047B939`            `sbb`  `edx`, `ebx`

`SDPI:0047B93B`            `cmp`  `edx`, `0`

`SDPI:0047B93E`            `jnz`  `fnddbg_47B89C`

`SDPI:0047B944`            `cmp`  `eax`, `30000000h`

`SDPI:0047B949`            `ja`   `fnddbg_47B89C`              ; 作用用的宏，因此没什么好介绍的了

`SDPI:0047B94F`            `jz`   `short PASS_47B99A`

`SDPI:0047B951`            `jnz`  `short PASS_47B99A`

`SDPI:0047B951` ; ----------------------------------------------------------

`SDPI:0047B953` `aS@I`            `db` '?,0,10h,'@',0,'皦?,4

`SDPI:0047B95C` ; ----------------------------------------------------------

`SDPI:0047B95C`            `mov`  `eax`, `[esp+4]`               ; SEH 处理处

`SDPI:0047B960`            `mov`  `ecx`, `[esp+0Ch]`

`SDPI:0047B964`            `inc`  `dword ptr [ecx+0B8h]`

`SDPI:0047B96A`            `mov`  `eax`, `[eax]`

`SDPI:0047B96C`            `sub`  `eax`, `80000003h`

`SDPI:0047B971`            `jnz`  `short locret_47B999`

`SDPI:0047B973`            `call`   `sub_47B979`

`SDPI:0047B978`            `nop`

`SDPI:0047B979`

`SDPI:0047B979` ; ******************* S U B R O U T I N E *************************

`SDPI:0047B979`

`SDPI:0047B979`

`SDPI:0047B979` `sub_47B979`      `proc near`                ; CODE XREF: `SDPI:0047B973` p

`SDPI:0047B979`            `pop`  `eax`

`SDPI:0047B97A`            `add`  `eax`, `0FFFFFDC1h`

`SDPI:0047B97F`            `cmp`  `byte ptr [eax]`, `0E8h`

`SDPI:0047B982`            `jnz`  `fnddbg_47B89C`

`SDPI:0047B988`            `mov`  `byte ptr [eax]`, `0E9h`

`SDPI:0047B98B`            `xor`  `eax`, `eax`

`SDPI:0047B98D`            `mov`  `[ecx+4]`, `eax`

`SDPI:0047B990`            `mov`  `[ecx+8]`, `eax`

`SDPI:0047B993`            `mov`  `[ecx+0Ch]`, `eax`

`SDPI:0047B996`            `mov`  `[ecx+10h]`, `eax`

`SDPI:0047B999`

`SDPI:0047B999` `locret_47B999`:       ; CODE XREF: `SDPI:0047B971` j

`SDPI:0047B999`            `retn`

`SDPI:0047B999` `sub_47B979`      `endp` ; sp =  4

`SDPI:0047B999`

`SDPI:0047B99A` ; ----------------------------------------------------------

`SDPI:0047B99A`

`SDPI:0047B99A` `PASS_47B99A`:           ; CODE XREF: `SDPI:0047B94F` j

`SDPI:0047B99A`            ; `SDPI:0047B951` j

`SDPI:0047B99A`            `pop`  `eax`

`SDPI:0047B99B`            `call`   `loc_47BB7D`

`SDPI:0047B99B` ; ----------------------------------------------------------

`SDPI:0047B9A0`            `dd` `401000h`

`SDPI:0047B9A4`            `dd` `15C56BEh`

`SDPI:0047B9A8` ; ----------------------------------------------------------

`SDPI:0047B9A8`

`SDPI:0047B9A8` `loc_47B9A8`:          ; CODE XREF: `SDPI:0047BBA6` p

`SDPI:0047B9A8`            `pop`  `ebp`

`SDPI:0047B9A9`            `pop`  `eax`

`SDPI:0047B9AA`            `jmp`  `loc_47BBAB`

`SDPI:0047B9AF` ; ----------------------------------------------------------

`SDPI:0047B9AF`            `mov`  `ecx`, `0FFFFFF00h`

`SDPI:0047B9B4`            `push`   `fs`

`SDPI:0047B9B6`            `jz`   `short loc_47B9C2`

`SDPI:0047B9B8`            `jnz`  `short loc_47B9C2`

`SDPI:0047B9B8` ; ----------------------------------------------------------

`SDPI:0047B9BA`            `dd` `401000h`

`SDPI:0047B9BE`            `dd` `49C89B0h`

`SDPI:0047B9C2` ; ----------------------------------------------------------

`SDPI:0047B9C2`

`SDPI:0047B9C2` `loc_47B9C2`:          ; CODE XREF: `SDPI:0047B9B6` j

`SDPI:0047B9C2`            ; `SDPI:0047B9B8` j

`SDPI:0047B9C2`            `pushfw`

`SDPI:0047B9C4`            `push`   `eax`

`SDPI:0047B9C5`            `mov`  `eax`, `ebx`

`SDPI:0047B9C7`            `push`   `ebx`
