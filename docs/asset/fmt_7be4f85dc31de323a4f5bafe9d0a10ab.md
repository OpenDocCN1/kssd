
### 4027:0000h A38h FECh
Not decompressed yet `nnoprom.bin`, explanation N/A

### 4007:0000h 1493h 2280h
Not decompressed yet `antivir.bin`, this is BIOS antivirus code

### 4028:0000h F63Ah 14380h
Not decompressed yet `ROSUPD.bin`, seems to be custom Logo display procedure

### 5000:0000h 15509h 20000h
Decompressed to RAM beginning at address in column one. `original.tmp`, the system BIOS

> **注意：** 绿色覆盖的解压地址被另外的方法处理：

A. 上面解释的部分不是真正的被解压区域。只是某种真正解压区域的占有区域，稍后由 `original.tmp` 处理。结论是：在 `bootblock` 中只有 `original.tmp` 和 `awardext.rom` 被 `Decompress_System_Bios` 解压缩。如果你想改变这个，那么试着计算被解压代码的大小总和，他不会合适的！

B. 所有的这些被解压段地址部分被 `Decompression_Ngine` procedure 变换到 `4000h`，就像你看到的在例程里面地址 `2000:7842h`。

C. “（被解压）开始地址。。。”中的 `40xxh` 实际上是一个 ID，工作如下：`40`（高字节）是 ID，标示它是一个扩展 bios，将要在稍后的 `original.tmp` 执行时被解压缩。`Xx` 是一个 id，在 `original.tmp` 用到，标示要被解压缩的部分。这些在下面的 `original.tmp` 中会详细解释。

D. 所有的这些部分都要在 `original.tmp` 执行时被解压缩。解压结果被放在地址 `4000:0000h`，但是不会在同一时刻。有一些（也许所有的）部分也要从那个地址重新定向，在另外的部分在那个地址被解压缩后保留他们的内容。这些在下面的 `original.tmp` 中会详细解释。

### 7.1.6.7. Shadow the BIOS code

7. Shadow bios 代码。假设解压例程成功的完成了，上面的例程接着拷贝被解压得 system bios(`original.tmp`)，从 RAM 中的 `5000:0000h` - `6000:FFFFh` 到 `E_0000h` - `F_FFFFh`。完成如下：

1. 重新编程北桥 shadow RAM 控制寄存器，使能只写到地址 `E_0000h` - `F_FFFFh`，促进写操作这个地址范围到 DRAM（没有到 bios rom 芯片）。
2. 进行一个字符串拷贝操作，拷贝被解压了的 system bios(`original.tmp`)，从 `5000:0000h` - `6000:FFFFh` 到 `E_0000h` - `F_FFFFh`。
3. 重新编程北桥 shadow RAM 控制寄存器，使能只读到地址 `E_0000h` - `F_FFFFh` ，促进读操作这个地址范围到 DRAM（没有到 bios rom 芯片）。这个也是对 system bios 代码写保护

### 7.1.6.8. Enable the microprocessor cache then jump into the decompressed system BIOS

8. 使能微处理高速缓存，然后跳转到压缩的 system bios。这一步是普通 bootblock 代码执行路径的最后一步。使能处理器高速缓存后，代就会跳转到 RAM 地址 `F000:F80Dh` 中的写保护的 system bios(`original.tmp`)，如上面看到的代码。这个跳转的目的地址好像在不同的 award bios 中都一样。

现在我要呈现在跳转到解压缩的 `original.tmp` 之前，压缩的和解压的 bios 部分的内存地图。这个很重要，因为这会在等会的分析解压缩了的 `original.tmp` 方便我们。现在我们不得不注意，所有的代码都在 RAM 中之行，在没有代码在 bios rom 芯片中执行了。

| Address Range in RAM | Decompression State (by Bootblock code) | Description |
| :--- | :--- | :--- |
| `0000:6000h` - `0000:6xxxh` | N/A | This area contains the header of the extension component (component other than `original.tmp` and `awardext.rom`) fetched from the compressed BIOS at `8000:0000h` - `9000:FFFFh` (previously BIOS component at `FFFC_0000h` - `FFFD_FFFFh` in the BIOS chip). Note that this is fetched here by part of the bootblock in segment `2000h`. |
| `1000:0000h` - `2000:5531h` | Compressed | This area contains the compressed `original.tmp`. It's part of the copy of the last 128KB of the BIOS (previously BIOS component at `E000:0000h` - `F000:FFFFh` in the BIOS chip). This code is shadowed here by the bootblock in BIOS ROM chip. |
| `2000:5532h` - `2000:5FFFh` | Pure Binary (non-executable) | This area contains only padding bytes. |
| `2000:6000h` - `2000:FFFFh` | Pure binary (executable) | This area contains the bootblock code. It's part of the copy of the last 128KB of the BIOS (previously BIOS component at `E000:0000h` - `F000:FFFFh` in the BIOS ROM chip). This code is shadowed here by the bootblock in BIOS ROM chip. This is where our code currently executing (the "copy" of bootblock in segment `2000h`). |
| `4100:0000h` - `4100:57C0h` | Decompressed | This area contains the decompressed `awardext.rom`. Note that the decompression process is accomplished by part of the bootblock code in segment `2000h`. |
| `5000:0000h` - `6000:FFFFh` | Decompressed | This area contains the decompressed `original.tmp`. Note that the decompression process is accomplished by part of the bootblock code in segment `2000h`. |
| `8000:0000h` - `9000:FFFFh` | Compressed | This area contains the copy of the first/lower 128KB of the BIOS (previously BIOS component at `FFFC_0000h` - `FFFD_0000h` in the BIOS chip). This code is copied here by the bootblock code in segment `2000h`. |
| `E000:0000h` - `F000:FFFFh` | Decompressed | This area contains copy of the decompressed `original.tmp`, which is copied here by the bootblock code in segment `2000h`. |

> 最后要注意：这里解释的 booblock 只涉及到了 normal Bootblock code execution path ，意思是没有解释一旦 `original.tmp` 崩溃时的 bootblock POST。有时间的话，我将要涉及到。所有的 bootblock 如上，我们将要开始研究 `original.tmp`。

## 7.2. System BIOS a.k.a Original.tmp

我们刚进行了上面的 bootblock，我要高亮晦涩的代码执行路径。所以，现在，你正在看我的 bios 的解压了的 `original.tmp` 的反汇编代码。

### 7.2.1. Entry point from "Bootblock in RAM"

| Address | Hex | Mnemonic |
| :--- | :--- | :--- |
| `F000:F80D` | | This code is jumped into by the bootblock code |
| `F000:F80D` | | if everything went OK |
| `F000:F80D` | `E9 02 F6` | `jmp sysbios_entry_point ;` |

这里是在重新定位和写保护 system bios 后，bootblock 跳转的地方。

### 7.2.2. The awardext.rom and Extension BIOS Components (lower 128KB bios-code) Relocation Routine

| Address | Assembly Code |
| :--- | :--- |
| `F000:EE12` | `sysbios_entry_point: ; CODE XREF: F000:F80D` |
| `F000:EE12` | `mov ax, 0` |
| `F000:EE15` | `mov ss, ax ; ss = 0000h` |
| `F000:EE17` | `mov sp, 1000h ; setup stack at 0:1000h` |
| `F000:EE1A` | `call setup_stack ; Call Procedure` |
| `F000:EE1D` | `call init_DRAM_shadowRW ; Call Procedure` |
| `F000:EE20` | `mov si, 5000h ; ds=5000h (look at copy_mem_word)` |
| `F000:EE23` | `mov di, 0E000h ; es=E000h (look at copy_mem_word)` |
| `F000:EE26` | `mov cx, 8000h ; copy 64KByte` |
| `F000:EE29` | `call copy_mem_word ; copy E000h segment routine, i.e.` |
| `F000:EE29` | `; copy 64Kbyte from 5000:0h to E000:0h` |
| `F000:EE2C` | `call j_init_DRAM_shadowR ; Call Procedure` |
| `F000:EE2F` | `mov si, 4100h ; ds = XGroup segment decompressed, i.e.` |
| `F000:EE2F` | `; at this point 4100h` |
| `F000:EE32` | `mov di, 6000h ; es = new XGroup segment` |
| `F000:EE35` | `mov cx, 8000h ; copy 64KByte` |
| `F000:EE38` | `call copy_mem_word ; copy XGroup segment , i.e.` |
| `F000:EE38` | `; 64Kbyte from 4100:0h to 6000:0h` |
| `F000:EE3B` | `call Enter_UnrealMode ; jump below in UnrealMode` |
| `F000:EE3E` | `Begin_in_UnrealMode` |
| `F000:EE3E` | `mov ax, ds` |
| `F000:EE40` | `mov es, ax ; es = ds (3rd entry in GDT)` |
| `F000:EE40` | `; base_addr=0000 0000h;limit 4GB` |
| `F000:EE42` | `assume es:nothing` |
| `F000:EE42` | `mov esi, 80000h ; mov esi,(POST_Cmprssed_Temp_Seg shl 4)` |
| `F000:EE42` | `; relocate lower 128KB bios code` |
| `F000:EE48` | `mov edi, 160000h` |
| `F000:EE4E` | `mov ecx, 8000h` |
| `F000:EE54` | `cld ; Clear Direction Flag` |
| `F000:EE55` | `rep movs dword ptr es:[edi], dword ptr [esi] ; move` |
| `F000:EE55` | `; 128k data to 160000h (phy addr)` |
| `F000:EE59` | `call Leave_UnrealMode ; Call Procedure` |
| `F000:EE59` | `End_in_UnrealMode` |
| `F000:EE5C` | `mov byte ptr [bp+214h], 0 ; mov byte ptr` |
| `F000:EE5C` | `; POST_SPEED[bp],Normal_Boot` |
| `F000:EE61` | `mov si, 626Bh ; offset 626Bh (E000h POST tests)` |
| `F000:EE64` | `push 0E000h ; segment E000h` |
| `F000:EE67` | `push si ; next instruction offset (626Bh)` |
| `F000:EE68` | `retf ; jmp to E000:626Bh` |

____________________________________________________

| Address | Assembly Code |
| :--- | :--- |
| `F000:7440` | `Enter_UnrealMode proc near ; CODE XREF: F000:EE3B` |
| `F000:7440` | `mov ax, cs` |
| `F000:7442` | `mov ds, ax ; ds = cs` |
| `F000:7444` | `assume ds:F000` |
| `F000:7444` | `lgdt qword ptr GDTR_F000_5504 ; Load Global Descriptor Table Register` |
| `F000:7449` | `mov eax, cr0` |
| `F000:744C` | `or al, 1 ; Logical Inclusive OR` |
| `F000:744E` | `mov cr0, eax` |
| `F000:7451` | `mov ax, 10h` |
| `F000:7454` | `mov ds, ax ; ds = 10h (3rd entry in GDT)` |
| `F000:7456` | `assume ds:nothing` |
| `F000:7456` | `mov ss, ax ; ss = 10h (3rd entry in GDT)` |
| `F000:7458` | `assume ss:nothing` |
| `F000:7458` | `retn ; Return Near from Procedure` |
| `F000:7458` | `Enter_UnrealMode endp` |
