
# 扫盲贴，Award 版 Bios 内部结构及加载过程简单说明

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-78.htm`](https://www.pediy.com/kssd/pediy09/pediy09-78.htm)

> 【文章标题】: Award 版 Bios 内部结构及加载过程简单说明  

> 【文章作者】: 火翼\[CCG\]  

> 【作者邮箱】: `firewings_ccg@yahoo.com.cn`  

> 【作者主页】: 无  

> 【作者 QQ 号】: 网上能查到  

> 【软件名称】: Award Bios  

> 【软件大小】: `128K-1024K`  

> 【下载地址】: 无  

> 【加壳方式】: 无  

> 【保护方式】: 无  

> 【编写语言】: ASM  

> 【使用工具】: IDA  

> 【操作平台】: 无  

> 【软件介绍】: 主板 Bios  

> 【作者声明】: 很多人不了解 BIOS 的加载过程，我就写一遍文章简单说一下以前逆向 Award Bios 得到的东西  

> \--------------------------------------------------------------------------------  

>    Award 版的 Bios 在主板中使用非常广泛，文件结构也不是特别复杂，所以用来进行分析还是比较合适的。Award 公司的 Bios 除 `BootBlock` 之外的部分都要使用 `lha` 压缩后合并到最终的 Bios 文件中，也就是说最终的 Bios 文件中只有 `BootBlock` 是能直接执行的，其他部分在执行之前都需要进行解压。Award 版的 Bios 中不同模块使用 `lha` 文件标准定义中的文件日期部分区分，主模块的识别代号是 `50000000`，其他模块也有自己独特的区分代号，例如：ACPI Table 的代号是 `40030000`。  

>    BookBlock 部分就在 Bios 文件的最后 68K 或 128K，机器启动后，`BootBolck` 被映射到 `F0000-FFFFF`(64K)或者 `E0000-FFFFF`(128K),然后机器调用 `FFFF0` 处的代码开始 Post 过程，Bios 的这个位置基本都是一条长跳转(至少我还没见过其他类型的)。在完成最基本的初始化之后，`BootBlock` 进入解压缩模块，对 Bios 的主模块进行完整性校验和解压缩，如果校验失败或者解压缩失败则会跳到 `BootBlock` 中的失败处理中，也就是一个最简单的引导模块，这个引导模块通常只支持很少的硬件设备，这个部分存在的意义就是在发现 Bios 文件出现问题后有一个恢复的机会，但如果是主模块内部出现问题而不是文件封装出现问题，这部分的判断是不可能起作用的。  

>    下面配合代码对上面的说明进行解释。代码来自 Abit 的 kN9 主板 V18 版本 Bios(网上随便找的比较新的 Award Bios)  

```
seg000:FFF0    jmp     far ptr loc_FE05B
```

>    这个就是 Bios 执行的第一句代码，只是简单的跳到 `F000:E05B` 处继续执行  

```
seg000:E05B    jmp     far ptr loc_FE200
```

```
seg000:E200    mov     gs, dx
seg000:E202    cli
seg000:E203    cld
seg000:E204    mov     ax, cs
seg000:E206    mov     ss, ax
seg000:E208    assume ss:nothing
seg000:E208    mov     ecx, 1Bh
seg000:E20E    rdmsr
seg000:E210    test    ax, 100h
seg000:E213    jz      short $+2
seg000:E215    mov     sp, 0E21Bh
seg000:E218    jmp     loc_F7360   ;这样的代码在跳转后的代码执行后会到 sp 内容指向的位置处继续执行，此处就是到 E21D 处继续执行。后面还有很多地方都是这样的跳转。
seg000:E218    ; -------------------------------------------------------------------------
seg000:E21B    dw 0E21Dh
seg000:E21D    ; -------------------------------------------------------------------------
seg000:E21D    mov     sp, 0E223h
seg000:E220    jmp     loc_FF600
seg000:E220    ; -------------------------------------------------------------------------
seg000:E223    dw 0E225h
seg000:E225    ; -------------------------------------------------------------------------
seg000:E225    jmp     loc_FE2BA
seg000:E228    ; -------------------------------------------------------------------------
seg000:E228
seg000:E228 loc_FE228:                                  ; CODE XREF: seg000:E2EAj
seg000:E228    mov     sp, 0E22Eh
seg000:E22B    jmp     loc_FF65F
```

> 一直跟踪 Bios 中 `BootBoock` 的流程,最后会来到此处  

```
seg000:E4DC    mov     ebx, esi
seg000:E4DF    sub     esi, 10000h
seg000:E4E6    mov     edi, 10000h
seg000:E4EC    mov     ecx, 8000h
seg000:E4F2    rep movs dword ptr es:[edi], dword ptr [esi]
seg000:E4F6    mov     esi, ebx
seg000:E4F9    sub     esi, 10000h
seg000:E500    mov     edi, 180000h
seg000:E506    mov     ecx, 8000h
seg000:E50C    rep movs dword ptr es:[edi], dword ptr [esi]
seg000:E510    mov     eax, cr0
seg000:E513    and     al, 0FEh
seg000:E515    mov     cr0, eax
seg000:E518    jmp     short $+2
seg000:E51A    jmp     far ptr 2000h:0E51Fh//复制下面的代码到 2000 段后进续执行
seg000:E51F    ; -------------------------------------------------------------------------
seg000:E51F    xor     ax, ax
seg000:E521    mov     ss, ax
seg000:E523    assume ss:nothing
seg000:E523    mov     sp, 0E00h
seg000:E526    mov     al, 0FFh
seg000:E528    call    sub_FE6E7
seg000:E52B    mov     al, 0Ch
seg000:E52D    out     80h, al         ; 输出 0C 到调试端口，用 Debug 卡看到的就是这个数字
seg000:E52F    call    sub_FE589
seg000:E532    mov     ax, [si+0Eh]
seg000:E535    mov     si, 0
seg000:E538    mov     ds, si
seg000:E53A    assume ds:nothing
seg000:E53A    mov     si, 6000h
seg000:E53D    mov     [si], ax
seg000:E53F    mov     al, 0C3h ; '?'
seg000:E541    out     80h, al         ; 同上，不同的数字代表在执行不同的 Bios 自检过程
seg000:E543    call    DecompressBios
seg000:E546    jmp     short loc_FE56A
seg000:E548    ; -------------------------------------------------------------------------
seg000:E548    ; START OF FUNCTION CHUNK FOR DecompressBios
seg000:E548
seg000:E548 loc_FE548:                                  ; CODE XREF: DecompressBios+C8j
seg000:E548            ; DecompressBios:loc_FFC63j
seg000:E548    push    2000h
seg000:E54B    pop     ds
seg000:E54C    assume ds:nothing
seg000:E54C    mov     dword ptr ds:0FFF4h, 2F3131F0h
seg000:E555    mov     dword ptr ds:0FFF8h, 392F3131h
seg000:E55E    mov     dword ptr ds:0FFFCh, 0CFFC0039h
seg000:E567    mov     ax, 1000h
seg000:E56A
seg000:E56A loc_FE56A:                                  ; CODE XREF: seg000:E546j
seg000:E56A    mov     ds, ax
seg000:E56C    assume ds:nothing
seg000:E56C    push    ax
seg000:E56D    mov     al, 0C5h ; '?'
seg000:E56F    out     80h, al         ; manufacture's diagnostic checkpoint
seg000:E571    call    sub_F8C1C       ;复制解压后的主模块到 E0000-FFFFF
seg000:E574    pop     ax
seg000:E575    cmp     ax, 5000h
seg000:E578    jz      short loc_FE57F
seg000:E57A    jmp     far ptr loc_FF6C0
seg000:E57F    ; -------------------------------------------------------------------------
seg000:E57F
seg000:E57F loc_FE57F:                                  ; CODE XREF: DecompressBios-15CDj
seg000:E57F    mov     al, 0
seg000:E581    call    sub_FE6E7
seg000:E584    jmp     far ptr loc_FF80D//如果主模块成功解压，则会跳转到解压后的主模块中运行，否则则转入简单引导模块提示进行恢复。
```

>    下面再说一下最近很热门的 ACPI 数据的加载，ACPI Table 的内容保存在 `acpitbl.bin`(文件名不重要,关键是识别代号)中,但加载 ACPI 数据的代码一般还是在主模块中的。  

```
seg000:D640    dd 'TDSR'               ;ACPI Table 名列表
seg000:D644    dd 'PCAF'
seg000:D648    dd 'TDSD'
seg000:D64C    dd 'CIPA'
seg000:D650    dd 'TDSS'
seg000:D654    dd 'TARS'
seg000:D658    dd 'GFCM'
seg000:D65C dword_ED65C   dd 'SCAF'               ; DATA XREF: sub_ED838+5r
seg000:D660
seg000:D660    ; ================= S U B R O U T I N E ?=====================================
seg000:D660
seg000:D660
seg000:D660 sub_ED660     proc near           ;ACPI 数据创建模块
seg000:D660    push    ds
seg000:D661    push    es
seg000:D662    call    sub_ED6B1
seg000:D665    pushad
seg000:D667    mov     al, 1
seg000:D669    call    sub_E8BBD
```



```asm
seg000:D66C        push    cs  
seg000:D66D        push    offset loc_ED678  
seg000:D670        push    8BBFh  
seg000:D673        jmp     far ptr loc_E8000  
seg000:D678 ; ---------------------------------------------------------------------------  
seg000:D678  
seg000:D678 loc_ED678:                  ; DATA XREF: sub_ED660+Do  
seg000:D678        call    sub_ED7B8  
seg000:D67B        call    nullsub_1  
seg000:D67E        call    sub_ED7DC  
seg000:D681        call    sub_ED789  
seg000:D684        call    sub_ED7CA  
seg000:D687        call    sub_ED838  
seg000:D68A        call    sub_ED87C  
seg000:D68D        call    sub_EDAB5  
seg000:D690        call    nullsub_2  
seg000:D693        call    sub_EDA3E  
seg000:D696        call    far ptr 1000h:4166h  
seg000:D69B        push    cs  
seg000:D69C        push    offset loc_ED6A7  
seg000:D69F        push    8BCFh  
seg000:D6A2        jmp     far ptr loc_E8000  
seg000:D6A7 ; ---------------------------------------------------------------------------  
seg000:D6A7  
seg000:D6A7 loc_ED6A7:                  ; DATA XREF: sub_ED660+3Co  
seg000:D6A7        xor     al, al  
seg000:D6A9        call    sub_E8BBD  
seg000:D6AC        popad  
seg000:D6AE        pop     es  
seg000:D6AF        pop     ds  
seg000:D6B0        retn  
seg000:D6B0 sub_ED660       endp
```

然后说一下，网上流行的修改 `Bios` 激活 `Vista` 都对 `Bios` 文件进行了什么修改

1. 把主模块和 `ACPI` 数据模块从 `BIOS` 文件中分离出来，并解压缩
2. 如果主模块的 `ACPI Table` 名称列表中不包含 `SLIC`，则要添加 `SLIC` 部分或者把列表中不重要的部分(`SRAT`, `MCFG` 之类的)改为 `SLIC`，如果是添加，还要修改加载 `ACPI` 部分的代码。
3. 在 `ACPI Table` 数据模块中加入 `SLIC Table` 数据。
4. 重新压缩修改后的这两个模块并替换原始文件中的部分并修正文件校验和
5. 刷新主板 `Bios`

上面的 1 和 5 两个步骤，一般都是使用 `ModBin` 和 `Cbrom` 来辅助进行的

最后说一下这个方法在哪些地方可能出现问题

1. 文件封装部分出现问题，虽然标准的 `Award` 版 `Bios` 的模块定义和模块校验和部分保存的地址都是确定的，但由于市场中的主板使用的 `Award` 公司 `Bios` 的版本新旧各不相同，而最新版的 `ModBin` 和 `Cbrom` 并不能从网上下载到，所以有可能会生成校验和错误的 `Bios` 文件，而且还有一些主板厂商(比如华硕)很喜欢自己修改这个地方的定义。
2. 主模块的修改，特别是需要添加 `SLIC` 而不是替换的，由于要修改一段代码，而且不可调试，如果修改错了就很危险。
3. 刷新过程，从 96 年开始我刷新过的主板 `bios` 也有上百块了，不要总说有刷新 `bios` 有多安全，关键是只要出了问题普通用户就很难恢复，极端点的条件，刷新的时候如果突然断电，肯定会失败吧。

\--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

支持楼主

明显专业多了

对此处稍有疑义

引用: `BookBlock` 部分就在 `Bios` 文件的最后 68K 或 128K，机器启动后，`BootBolck` 被映射到 `F0000-FFFFF`(64K)或者 `E0000-FFFFF`(128K),然后机器调用 `FFFF0` 处的代码开始 `Post` 过程。

据 `Intel` 的手册说

引用: 9.1.4 执行第一条指令

在硬件重置后，第一条被取指并执行的指令位于物理地址 `FFFFFFF0H` 处，即比处理器最高的物理地址少 16 字节。包含软件初始化代码的 `EPROM` 必须被定位在这个地址上。

注意一下这个地址（`FFFFFFF0H`）实际上超出了实模式下处理器所能访问的 1MB 寻址范围。下面是关于这个地址产生的解释：`CS` 寄存器被分成程序可见的段选择子和程序不可见的基址两部分。在实地址模式下，通常由 16 位的段选择子左移 4 位产生 20 位的基址。但是，在硬件重置后，`CS` 段寄存器的段选择子装载为 `F000H`，基址装载为 `FFFF0000H`。第一条指令的地址就是基址加上 `EIP` 寄存器的内容（即 `FFFF0000H+FFF0H = FFFFFFF0H`）。

当硬件重置后第一次装入 `CS` 段寄存器时，处理器就遵循实模式下的地址翻译规则（即 `CS` 基址= `CS` 段选择子*16）。为了保证执行 `EPROM` 中的软件初始化代码时基址不改变（`FFFF0000H`），这段代码就不能执行段间跳转（`FAR JUMP`）或者段间调用（`FAR CALL`）或者允许中断（这些都会导致重新装载 `CS`）。

所以，实际上第一条指令从 `FFFFFFF0H` 开始执行

不过，是否某些主板做了双重映射，这是我不知道的

我想 `ypcok` 应该可以解答这些

是做了多重影射,最根本的还是在 4G 寻址范围的最后面的，1M 之内的影射是为了在实模式下进行 `POST` 而设计的

刷新 `Bios` 其实也就是向 4G 寻址范围的最后面写入，不过要做很多设置才能实际写入到 `Bios` 中

`4dd466d0e1e0cc123afbe504da2462ca`
