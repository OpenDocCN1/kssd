
1. IDA Pro scripts only recognize one type of variable, i.e. `auto`. There are no other variable types such as `int`, `char`, etc. The declaration of variable in an IDA Pro script is as follows: `auto variable_name;`.

2. Every statement in an IDA Pro script ends with a semicolon (`;`), just like in the C programming language.

3. Function can return a value or not, but there's no return type declaration. The syntax is as follows: `static function_name(parameter1, parameter2, parameter_n, ...)`.

4. Comment in an IDA Pro script starts with double-slash (`//`). The IDA Pro scripting engine ignores anything after the comment in the corresponding line.

5. `// comment`

6. `statement; // comment`

7. IDA Pro "exports" its internal functionality to the script that we build by using header files. These header files must be "included" in our script so that we are able to access that functionality. At least one header file must be included in any IDA Pro script, i.e. `idc.idc`. The header files are located inside a folder named `idc` in the IDA Pro installation directory. One must read the `*.idc` files inside this directory to learn about the functions that are exported by IDA Pro. The most important header file to learn is `idc.idc`. The syntax used to include a header file in an IDA Pro script is:

8. `#include < header_file_name >`

9. The entry point of an IDA Pro script is the `main` function, just as in the C programming language.

Now is the time to put the theory into a simple working example, an IDA Pro sample script.

```
#include <idc.idc>

// relocate one segment

static relocate_seg(src, dest)
{
        auto ea_src, ea_dest, hi_limit;
        hi_limit = src + 0x10000;
        ea_dest = dest;
        for(ea_src = src; ea_src < hi_limit ; ea_src = ea_src + 4 )
          {
          PatchDword( ea_dest, Dword(ea_src));
          ea_dest = ea_dest + 4;
        }

  Message("segment relocation finished (inside relocate_seg function)..\n");
}

static main()
{
  Message("creating target segment (inside entry point function main)...\n");
  SegCreate([0xF000, 0], [0x10000, 0], 0xF000, 0, 0, 0);
  SegRename([0xF000, 0], "_F000");
  relocate_seg([0x7000,0], [0xF000, 0]);
}
```

The square bracket, i.e. `[ ]` in the script above is an operator used to form the linear address from its parameters by shifting the first parameter to left four bits and then adding the second parameter into the result, for example: `[0x7000, 0]` means `(0x7000 << 4) + 0`, i.e. `0x7_0000` linear address. This operator is just the same as `MK_FP( , )` operator in previous versions of IDA Pro. One must read `idc.idc` file to see the "exported" function definition to understand this script completely, such as the `Message`, `SegCreate` and `SegRename` functions. Another "exported" function that may be of interest can be found in numerous `*.idc` files in the `idc` directory of IDA Pro installation folder. To be able to use a function, its definition has to be looked up in the exported function definition in the corresponding `*.idc` header file. For example, `SegCreate` function is defined in `idc.idc` as follows:

```
// Create a new segment
//      startea  - linear address of the start of the segment
//      endea    - linear address of the end of the segment
//                 this address will not belong to the segment
//                 'endea' should be higher than 'startea'
//      base     - base paragraph or selector of the segment.
//                 a paragraph is 16byte memory chunk.
//            If a selector value is specified, the selector should be
//                 already defined.
//      use32    - 0: 16bit segment, 1: 32bit segment
//      align    - segment alignment. see below for alignment values
//      comb  - segment combination. see below for combination values.
// returns: 0-failed, 1-ok
success SegCreate( long startea,long endea,long base,  long use32,
                    long align,long comb);
```

A 512KB BIOS binary file must be opened in IDA Pro with the loading address set to `0000h` to be able to execute the sample script above. This loading scheme is the same as explained in the previous sub-section. In this case, we will just open the binary file of Supermicro H8DAR-8 motherboard as in the previous sub-section and then execute the script. First, we must type the script above in a plain text file. We can use notepad or another ASCII file editor for this purpose. We will name the file as `function.idc`. The script is then executed by clicking on the `File | IDC file...` menu or by pressing F2, then the dialog box below will be shown.

Just select the file and click open to execute the script. If there's any mistake in the script, IDA Pro will warn you with a warning dialog box. Executing the script will display the corresponding message in the message pane of IDA Pro as shown below.

The script above relocates the last segment (64KB) of the Supermicro H8DAR-8 BIOS code to the right place. One must be aware that IDA Pro is only an advanced tool to help the reverse code engineering task, it's not a magical tool that's going to reveal the overall structure of the BIOS binary without us being significantly involved in the process. The script relocates/copies BIOS code from physical/linear address `0x7_0000-0x7_FFFF` to `0xF_0000-0xF_FFFF`. The logical reason behind this algorithm is explained below.

AMD-8111 HyperTransport IO Hub Datasheet chapter 4 page 153 says that:

> Note: The following ranges are always specified as BIOS address ranges. See `DevB:0x80` for more information about how access to BIOS spaces may be controlled.

| Size | Host Address Range[31:0] | Address translation for LPC bus |
|------|--------------------------|----------------------------------|
| 64K bytes | `FFFF_0000h` - `FFFF_FFFFh` | `FFFF_0000h` - `FFFF_FFFFh` |
| 64K bytes | `000F_0000h` - `000F_FFFFh` | `FFFF_0000h` - `FFFF_FFFFh` |

In addition, AMD64 Architecture Programmer's Manual Volume 2: System Programming, February 2005 in section 14.1.5 page 417 says that:

> "Normally within real mode, the code-segment base address is formed by shifting the CS-selector value left four bits. The base address is then added to the value in EIP to form the physical address into memory. As a result, the processor can only address the first 1 Mbyte of memory when in real mode. However, immediately following RESET# or INIT, the CS selector register is loaded with F000h, but the CS base-address is not formed by left-shifting the selector. Instead, the CS base address is initialized to FFFF_0000h. EIP is initialized to FFF0h. Therefore, the first instruction fetched from memory is located at physical-address FFFF_FFF0h (FFFF_0000h+0000_FFF0h). The CS base-address remains at this initial value until the CS selector register is loaded by software. This can occur as a result of executing a far jump instruction or call instruction, for example. When CS is loaded by software, the new base-address value is established as defined for real mode (by left shifting the selector value four bits)."

From the references above, we conclude that address `000F_0000h - 000F_FFFFh` is an alias to address `FFFF_0000h - FFFF_FFFFh`, i.e. they both point to the same physical address range. Whenever the host (CPU) accesses some value in `000F_0000h - 000F_FFFFh` address range, it's actually accessing the value at `FFFF_0000h - FFFF_FFFFh` range and the reverse is also true. From this fact, we know that we have to relocate 64KB of the uppermost BIOS code to address `000F_0000h - 000F_FFFFh` for further investigation. This decision is made based on my previous experience with various BIOS binary files; they generally reference addresses with `F000h` used as the segment value within the BIOS code. Also, note that the last 64KB of the BIOS binary file is mapped to the last 64KB of the 4GB address space, i.e. 4GB-64KB to 4GB, that's why we have to relocate the last 64KB.
