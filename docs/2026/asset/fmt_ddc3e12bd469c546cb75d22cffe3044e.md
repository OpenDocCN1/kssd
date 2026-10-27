
# [原创]易语言代码 手工构造一个COFF/OBJ 可链接

只是构造了简单的一个中文版的hello world

`DUMPBIN` 结果如下

```
Microsoft (R) COFF/PE Dumper Version 10.00.30319.01

Copyright (C) Microsoft Corporation.  All rights reserved.





Dump of file out.obj



File Type: COFF OBJECT



FILE HEADER VALUES

             14C machine (x86)

               4 number of sections

        51C325D9 time date stamp Thu Jun 20 23:55:05 2013

             148 file pointer to symbol table

              10 number of symbols

               0 size of optional header

               0 characteristics



SECTION HEADER #1

   .text name

       0 physical address

       0 virtual address

      21 size of raw data

      B4 file pointer to raw data (000000B4 to 000000D4)

     120 file pointer to relocation table

       0 file pointer to line numbers

       4 number of relocations

       0 number of line numbers

60300020 flags

         Code

         4 byte align

         Execute Read



RAW DATA #1

  00000000: 55 8B EC 6A 00 68 00 00 00 00 68 00 00 00 00 FF  U.ìj.h....h....ÿ

  00000010: 35 00 00 00 00 E8 00 00 00 00 B8 00 00 00 00 C9  5....è....¸....É

  00000020: C3                                               Ã



RELOCATIONS #1

                                                Symbol    Symbol

 Offset    Type              Applied To         Index     Name

 --------  ----------------  -----------------  --------  ------

 00000006  DIR32                      00000000         E  ����

 0000000B  DIR32                      00000000         D  ����

 00000011  DIR32                      00000000         C  ���ھ��

 00000016  REL32                      00000000         B  _MessageBoxA@16



SECTION HEADER #2

   .data name

       0 physical address

       0 virtual address

      22 size of raw data

      D5 file pointer to raw data (000000D5 to 000000F6)

       0 file pointer to relocation table

       0 file pointer to line numbers

       0 number of relocations

       0 number of line numbers

C0300040 flags

         Initialized Data

         4 byte align

         Read Write



RAW DATA #2

  00000000: D5 E2 BE CD CA C7 C1 CB B8 F6 B1 EA CC E2 00 C4  Õâ¾ÍÊÇÁË¸ö±êÌâ.Ä

  00000010: E3 C1 CB B8 F6 BA C3 A3 AC CA C0 C1 CB B8 F6 BD  ãÁË¸öºÃ£¬ÊÀÁË¸ö½

  00000020: E7 00                                            ç.



SECTION HEADER #3

    .bss name

       0 physical address

       0 virtual address

       4 size of raw data

       0 file pointer to raw data

       0 file pointer to relocation table

       0 file pointer to line numbers

       0 number of relocations

       0 number of line numbers

C0300080 flags

         Uninitialized Data

         4 byte align

         Read Write



SECTION HEADER #4

.drectve name

       0 physical address

       0 virtual address

      29 size of raw data

      F7 file pointer to raw data (000000F7 to 0000011F)

       0 file pointer to relocation table

       0 file pointer to line numbers

       0 number of relocations

       0 number of line numbers

     A00 flags

         Info

         Remove

         (no align specified)



RAW DATA #4

  00000000: 2D 64 65 66 61 75 6C 74 6C 69 62 3A 75 73 65 72  -defaultlib:user

  00000010: 33 32 2E 6C 69 62 20 2D 65 6E 74 72 79 3A C6 F4  32.lib -entry:Æô

  00000020: B6 AF D7 D3 B3 CC D0 F2 20                       ¶¯×Ó³ÌÐò



   Linker Directives

   -----------------

   -defaultlib:user32.lib

   -entry:启动子程序



COFF SYMBOL TABLE

000 00000000 DEBUG  notype       Filename     | .file

    纯手工打造

002 43505045 ABS    notype       Static       | @comp.id

003 00000000 SECT1  notype       Static       | .text

    Section length   21, #relocs    4, #linenums    0, checksum        0

005 00000000 SECT2  notype       Static       | .data

    Section length   22, #relocs    0, #linenums    0, checksum        0

007 00000000 SECT3  notype       Static       | .bss

    Section length    0, #relocs    0, #linenums    0, checksum        0

009 00000000 SECT4  notype       Static       | .drectve

    Section length   29, #relocs    0, #linenums    0, checksum        0

00B 00000000 UNDEF  notype ()    External     | _MessageBoxA@16

00C 00000000 SECT3  notype       Static       | ���ھ��

00D 00000000 SECT2  notype       Static       | ����

00E 0000000F SECT2  notype       Static       | ����

00F 00000000 SECT1  notype ()    External     | _����ӳ���



String Table Size = 0x20 bytes



  Summary



           4 .bss

          22 .data

          29 .drectve

          21 .text

```

详细见附件： [`obj.rar`](https://bbs.kanxue.com/attach-download-80138.htm)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`obj.rar`](javascript:void(0);)  （7.93kb，27次下载）
