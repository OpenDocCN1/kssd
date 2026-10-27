
# UPX 模板修改与源码改造说明

```c
typedef uint uint32_t;
typedef ushort uint16_t;
typedef uchar uint8_t;
struct b_info // 12-byte header before each compressed block
{
uint32_t sz_unc<format = hex, comment = "Uncompressed size">; // uncompressed_size
uint32_t sz_cpr<format = hex, comment = "Compressed size">; // compressed_size
unsigned char b_method<comment = "Compression algorithm">; // compression algorithm
unsigned char b_ftid; // filter id
unsigned char b_cto8; // filter parameter
unsigned char b_unused;
};
struct l_info // 12-byte trailer in header for loader (offset 116)
{
uint32_t l_checksum;
uint32_t l_magic<format = hex, comment = "UPX!">;
uint16_t l_lsize;
uint8_t l_version;
uint8_t l_format;
};
struct p_info // 12-byte packed program header follows stub loader
{
uint32_t p_progid;
uint32_t p_filesize;
uint32_t p_blocksize;
};
typedef struct
{
b_info bheader;
char data[bheader.sz_cpr];
}cblock;
typedef struct
{
uint32 offset_b_info;
uint32 data_size;
}endheader;
```

然后，将原来模板中`file`结构体中对`section`的解析代码去掉，因为UPX压缩后的文件没有相关字段。

```c
// 需要删除的代码
// Find the header name location 869行
local quad section_name_off =
file.elf_header.e_shoff_SECTION_HEADER_OFFSET_IN_FILE +
(file.elf_header.e_shentsize_SECTION_HEADER_ENTRY_SIZE *
file.elf_header.e_shtrndx_STRING_TABLE_INDEX);
// Find the header name block
if (file.elf_header.e_ident.ei_class_2 == ELFCLASS32) {
......
} else {
......
}
local int sec_tbl_cur_elem;
// Find the section headers
if(file.elf_header.e_shnum_NUMBER_OF_SECTION_HEADER_ENTRIES > 0) {
......
}
local int sym_sect;
local int sym_name_sect;
// Find the symbol section
sym_sect = FindNamedSection(".symtab");
if(sym_sect >= 0) {
......
}
// Find the dynamic symbol section
sym_sect = FindNamedSection(".dynsym");
if(sym_sect >= 0) {
......
}
```

最后，在`file`结构体中`program_header_table`结构体生成之后，添加`l_info`结构之后的数据解析代码。

```c
// 需要添加的代码 869行
FSeek(file.elf_header.e_phoff_PROGRAM_HEADER_OFFSET_IN_FILE + file.elf_header.e_phentsize_PROGRAM_HEADER_ENTRY_SIZE_IN_FILE * file.elf_header.e_phnum_NUMBER_OF_PROGRAM_HEADER_ENTRIES);
l_info linfone;
p_info pinfone<comment = "Original file size">;
// Compressed block of PT_LOAD
local uint64 now_offsize;
now_offsize = file.elf_header.e_phoff_PROGRAM_HEADER_OFFSET_IN_FILE + file.elf_header.e_phentsize_PROGRAM_HEADER_ENTRY_SIZE_IN_FILE * file.elf_header.e_phnum_NUMBER_OF_PROGRAM_HEADER_ENTRIES + 24;
local uint64 tmp_offsize = 0;
local uint64 tmp = 0;
struct
{
while (true)
{
tmp_offsize = now_offsize + (7u & (0 - now_offsize));
tmp = ReadInt64(tmp_offsize);
if (((tmp&0xffffffff) == 0x100) && (((tmp>>32)&0xffffffff) == (tmp_offsize + 4)))
{
break;
}
else
{
FSeek(now_offsize);
cblock a;
now_offsize = now_offsize + a.bheader.sz_cpr + 12 ;
}
}
} cpr_load<comment = "Compressed block of PT_LOAD">;
FSeek(tmp_offsize);
endheader headerEND;
local uint64 sizeloader = program_header_table.program_table_element[0].p_filesz_SEGMENT_FILE_LENGTH - tmp_offsize - 8;
uchar loader[sizeloader];
// Compressed block of GAPS
FSeek(program_header_table.program_table_element[0].p_filesz_SEGMENT_FILE_LENGTH);
now_offsize = program_header_table.program_table_element[0].p_filesz_SEGMENT_FILE_LENGTH;
tmp_offsize = 0;
tmp = 0;
struct
{
while (true)
{
tmp_offsize = now_offsize;
tmp = ReadInt64(tmp_offsize);
if (((tmp & 0xffffffff) == 0) && (((tmp >> 32) & 0xffffffff) == (0x21585055)))
{
break;
}
else
{
FSeek(now_offsize);
cblock b;
now_offsize = now_offsize + b.bheader.sz_cpr + 12;
}
}
} cpr_gaps<comment = "Compressed block of GAPS">;
b_info check_upx<comment = "Block end marker (uncompressed size 0, compressed size 559435861)">;
tmp_offsize = tmp_offsize + 12;
tmp_offsize = tmp_offsize + (3u & (0 - tmp_offsize));
FSeek(tmp_offsize);
uchar PackHeader[32]<comment = "PackHeader">;
uint32 size_p_info<format = hex, comment = "Offset of p_info">;
```

最终效果如下，红框中是我们添加的新结构。

![](upload/attach/202301/905958_YVVM5AQ3TW6BWNN.png)

修改完成的模板见结尾文件 `upx.bt`。因为文件格式问题，该模板只适用于 demo 文件压缩后的文件。大家有其他需求可以在此基础上自己修改。

## 源码修改

之前做病毒分析时，碰到了一个无法使用标准UPX程序解压的程序。后来通过搜寻资料和学习，了解到有多种方式可实现该效果。基本思路是修改替换程序中的特征字符，常见的修改方法如下：

1. UPX两个区段名称修改
2. PE中DOS头修改
3. 三个UPX特征码修改（参考EXEINFO等壳识别工具中的特征匹配规则）
4. UPX相关字符串去除或修改

据此，笔者萌生直接对源码进行修改的想法，所以有以下两个简单尝试。

1. 入口点字段修改
2. 压缩数据修改

### 入口点字段修改

本次尝试主要是修改源程序的入口点位置，使常规的UPX解压程序后无法快速找到正确的入口点位置。

#### 代码修改

1. 存储入口点之前，处理该值，这里异或一个常量`0xdeafdeaf`。文件`upx-3.96/src/p_unix.cpp`添加如下代码

```c++
334 int l = fi->readx(hdr_ibuf, hdr_u_len);
335 /********************************************************************************/
336 // add 2022-11-29 20:02
337 // edit 2022-12-12 10:19
338 unsigned char * ttt = (unsigned char *)hdr_ibuf;
339 Elf64_Ehdr *tmp = (Elf64_Ehdr *)ttt;
340 tmp->e_entry = (tmp->e_entry ^ 0xdeafdeaf);
341 /********************************************************************************/
```

2. 解压得到入口点之后，处理该值，也是异或常量`0xdeafdeaf`。这里有两种方式，实现的功能一样。

+ 文件`upx-3.96/src/stub/src/amd64-linux.elf-fold.S`添加汇编代码`xor $3736067759,%eax`

```c
movq %rbp,%arg5 # &decompress: f_expand
call upx_main # Out: %rax= entry
/* entry= upx_main(b_info *arg1, total_size arg2, Elf64_Ehdr *arg3,
Elf32_Auxv_t *arg4, f_decompr arg5, f_unf arg6,
Elf64_Addr elfaddr )
*/ xor $3736067759,%eax // add 2022-12-12
addq $1*NBPW+OVERHEAD,%rsp # toss elfaddr, too
movq %rax,4*NBPW(%rsp) # entry
pop %rbx # fd
```

+ 文件`upx-3.96/src/stub/amd64-linux.elf-fold.h`修改二进制数据，修改三处代码

```c
#define STUB_AMD64_LINUX_ELF_FOLD_SIZE 2251
->
#define STUB_AMD64_LINUX_ELF_FOLD_SIZE 2256
```

```c
unsigned char stub_amd64_linux_elf_fold[2251] = {
->
unsigned char stub_amd64_linux_elf_fold[2256] = {
```

```c
/* 0x01d0 */ 137,232,232,172, 5, 0, 0, 72,129,196, 8, 8, 0, 0, 72,137,
->
/* 0x01d0 */ 137,232,232,177, 5, 0, 0, 53,175,222,175,222, 72,129,196, 8, 8, 0, 0, 72,137,
```

> xor eax, 3736067759 -> 35 AF DE AF DE -> 53,175,222,175,222

#### 验证

1. 使用修改源码后的程序加壳，程序可正常执行。这里的UPX版本是3.96

![](upload/attach/202301/905958_DN9USZZH3HY93HA.png)

2. 在kali上执行加壳后的程序，可正常执行

![](upload/attach/202301/905958_P9E9BG9S6N4WAYJ.png)

3. 使用kali上的UPX程序解压缩该程序（UPX版本为3.96）,执行程序提示段错误

![](upload/attach/202301/905958_KWDEXV388VYG9NC.png)

### 压缩数据修改

上面对于入口点的修改虽然成功了，但是压缩后的程序仍然可被解压。而且从原理来看，解压后是可以看到原程序代码执行逻辑的。所以笔者有了第二次代码修改。主要修改思路是，对压缩之前的数据进行处理，然后再进行压缩。这样标准UPX就无法解压我们自己压缩的程序，比只对入口点的修改效果更好。

#### 代码修改

1. 压缩前修改数据，我们做一个简单的异或处理，这里对所有的数据异或`0xe9`(十进制为233)

文件：`upx-3.96/src/compress.cpp`

`upx_compress`函数修改，核心部分

```c
96 #if 1
97 // debug
98 cresult->method = method;
99 cresult->level = level;
100 cresult->u_len = src_len;
101 cresult->c_len = 0;
102 #endif
103 ////////////////////////////////////////////////////////
104 const unsigned char *tmp = (const unsigned char *)src;
105 unsigned char * tmp0 = (unsigned char*)malloc(src_len*sizeof(char));
106 unsigned char * tmp1 = tmp0;
107 memcpy(tmp0, tmp, src_len);
108 size_t i = 0;
109 for(i =0;i<src_len;i++)
110 {
111 (*tmp0)=(*tmp0)^0xe9;
112 tmp0 = tmp0 + 1;
113 }
114 src=tmp1;
115 ////////////////////////////////////////////////////////
```

`upx_decompress`函数修改，此处修改主要是因为在文件`upx-3.96/src/packer.cpp`的`Packer::compress`函数中，会对压缩后的数据进行解压验证，需要保持压缩和解压函数逻辑对称。

```c
190 else {
191 throwInternalError("unknown decompression method");
192 }
193 ////////////////////////////////////////////////////////
194 unsigned char* dst_tmp=dst;
195 size_t i = 0;
196 for(i =0;i<(*dst_len);i++)
197 {
198 (*dst)=(*dst)^0xe9;
199 dst = dst+ 1;
200 }
201 dst = dst_tmp;
202 ////////////////////////////////////////////////////////
```

2. upx在加载loader是会对其进行压缩处理，而解压算法已写成汇编代码，修改起来比较麻烦。这里我就对loader数据多进行一次处理，以抵消`upx-3.96/src/compress.cpp`文件`upx_compress`函数中添加的异或处理。

文件：`upx-3.96/src/p_lx_elf.cpp`
