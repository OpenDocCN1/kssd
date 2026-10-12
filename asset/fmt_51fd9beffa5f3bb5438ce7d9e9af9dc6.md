
# PE Header Analysis

## Clean File Analysis

The following table lists the optional header fields for a normal, uninfected file:

| O DOS | O PE | Description |
| :--- | :--- | :--- |
| 0130H | 0030H | Base of Data (.bss ofs.) : 00003000H |
| 0134H | 0034H | Image Base : 00400000H |
| 0138H | 0038H | Section Alignment : 00001000H |
| 013CH | 003CH | File Alignment : 00000200H |
| 0140H | 0040H | Major Operating System Version : 1 |
| 0142H | 0042H | Minor Operating System Version : 0 |
| 0144H | 0044H | Major Image Version : 0 |
| 0146H | 0046H | Minor Image Version : 0 |
| 0148H | 0048H | Major SubSystem Version : 3 |
| 014AH | 004AH | Minor SubSystem Version : 10 |
| 014CH | 004CH | Reserved Long : 43545A41H |
| 0150H | 0050H | Size of Image : 00006600H |
| 0154H | 0054H | Size of Headers : 00000400H |
| 0158H | 0058H | File Checksum : 00000000H |
| 015CH | 005CH | SubSystem : 2 |
| | | -Image runs in the Windows GUI subsystem |
| 015EH | 005EH | DLL Characteristics : 0000H |
| 0160H | 0060H | Size of Stack Reserve : 00100000H |
| 0164H | 0064H | Size of Stack Commit : 00002000H |
| 0168H | 0068H | Size of Heap Reserve : 00100000H |
| 016CH | 006CH | Size of Heap Commit : 00001000H |
| 0170H | 0070H | Loader Flags : 00000000H |
| 0174H | 0074H | Number Directories : 00000010H |

### PE Section Headers

| O DOS | O PE | Description |
| :--- | :--- | :--- |
| 0270H | 0170H | Section name : .reloc |
| 0278H | 0178H | Physical Address : 00001600H |
| 027CH | 017CH | Virtual Address : 00005000H |
| 0280H | 0180H | Size of RAW data : 00001600H |
| 0284H | 0184H | Pointer to RAW data : 00001C00H |
| 0288H | 0188H | Pointer to relocations : 00000000H |
| 028CH | 018CH | Pointer to line numbers : 00000000H |
| 0290H | 0190H | Number of Relocations : 0000H |
| 0292H | 0192H | Number of line numbers : 0000H |
| 0294H | 0194H | Characteristics : F0000060H |
| | | -Section contains code. |
| | | -Section contains initialized data. |
| | | -Section is shareable. |
| | | -Section is executable. |
| | | -Section is readable. |
| | | -Section is writeable. |

## Infected File Analysis

The following file, `GOAT002.EXE`, has been infected by the Aztec virus (Ring-3 example, see text).

### DOS Information

```
-------DOS INFORMATION ------------------------------------------------- 

Analyzed File: GOAT002.EXE 

DOS Reports: 
             ?File Size  - 2600H      (09728d) 
             ?File Time  - 23:20:58   (hh:mm:ss) 
             ?File Date  - 22/06/1999 (dd/mm/yy) 
             ?Attributes : Archive 
```

### PE Header

| O DOS | O PE | Description |
| :--- | :--- | :--- |
| 0100H | 0000H | PE Header Signature - PE/0/0 |
| 0104H | 0004H | The machine for this EXE is Intel 386 (value = 014CH) |
| 0106H | 0006H | Number of sections in the file - 0004H |
| 0108H | 0008H | File was linked at : 23/03/2049 |
| 010CH | 000CH | Pointer to Symbol Table : 00000000H |
| 0110H | 0010H | Number of Symbols : 00000000H |
| 0114H | 0014H | Size of the Optional Header : 00E0H |
| 0116H | 0016H | File Characteristics - 818EH : |
| | | -File is executable |
| | | -Line numbers stripped from file |
| | | -Local symbols stripped from file |
| | | -Bytes of machine word are reversed |
| | | -32 bit word machine |
| | | -Bytes of machine word are reversed |

### PE Optional Header

| O DOS | O PE | Description |
| :--- | :--- | :--- |
| 0118H | 0018H | Magic Value : 010BH |
| 011AH | 001AH | Major Linker Version : 2 |
| 011BH | 001BH | Minor Linker Version : 25 |
| | | Linker Version : 2.25 |
| 011CH | 001CH | Size of Code : 00001200H |
| 0120H | 0020H | Size of Initialized Data : 00000600H |
| 0124H | 0024H | Size of Uninitialized Data : 00000000H |
| 0128H | 0028H | Address of Entry Point : 00005200H |
| 012CH | 002CH | Base of Code (.text ofs.) : 00001000H |
| 0130H | 0030H | Base of Data (.bss ofs.) : 00003000H |
| 0134H | 0034H | Image Base : 00400000H |
| 0138H | 0038H | Section Alignment : 00001000H |
| 013CH | 003CH | File Alignment : 00000200H |
| 0140H | 0040H | Major Operating System Version : 1 |
| 0142H | 0042H | Minor Operating System Version : 0 |
| 0144H | 0044H | Major Image Version : 0 |
| 0146H | 0046H | Minor Image Version : 0 |
| 0148H | 0048H | Major SubSystem Version : 3 |
| 014AH | 004AH | Minor SubSystem Version : 10 |
| 014CH | 004CH | Reserved Long : 43545A41H |
| 0150H | 0050H | Size of Image : 00006600H |
| 0154H | 0054H | Size of Headers : 00000400H |
| 0158H | 0058H | File Checksum : 00000000H |
| 015CH | 005CH | SubSystem : 2 |
| | | -Image runs in the Windows GUI subsystem |
| 015EH | 005EH | DLL Characteristics : 0000H |
| 0160H | 0060H | Size of Stack Reserve : 00100000H |
| 0164H | 0064H | Size of Stack Commit : 00002000H |
| 0168H | 0068H | Size of Heap Reserve : 00100000H |
| 016CH | 006CH | Size of Heap Commit : 00001000H |
| 0170H | 0070H | Loader Flags : 00000000H |
| 0174H | 0074H | Number Directories : 00000010H |

### PE Section Headers

| O DOS | O PE | Description |
| :--- | :--- | :--- |
| 0270H | 0170H | Section name : .reloc |
| 0278H | 0178H | Physical Address : 00001600H |
| 027CH | 017CH | Virtual Address : 00005000H |
| 0280H | 0180H | Size of RAW data : 00001600H |
| 0284H | 0184H | Pointer to RAW data : 00001C00H |
| 0288H | 0188H | Pointer to relocations : 00000000H |
| 028CH | 018CH | Pointer to line numbers : 00000000H |
| 0290H | 0190H | Number of Relocations : 0000H |
| 0292H | 0192H | Number of line numbers : 0000H |
| 0294H | 0194H | Characteristics : F0000060H |
| | | -Section contains code. |
| | | -Section contains initialized data. |
| | | -Section is shareable. |
| | | -Section is executable. |
| | | -Section is readable. |
| | | -Section is writeable. |

## Summary of Changes

Hopefully, this has helped you understand what happens when a PE file is infected by adding its last section. To save you the effort of comparing these tables individually, here is a list of the values that change:

| Values to change | Before | After | Location |
| :--- | :--- | :--- | :--- |
| Address Of Entrypoint | 00001000h | 00005200h | Image File Hdr |
| Reserved1 (inf. mark) | 00000000h | 43545A41h | Image File Hdr |
| Virtual Size | 00001000h | 00001600h | Section header |
| Size Of Raw Data | 00000200h | 00001600h | Section header |
