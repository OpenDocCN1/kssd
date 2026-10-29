
# [原创] iOS inlinehook 绕过反调试

## 开始

之前写过一篇<a href="elink@2e9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0c8U0K9o6p5J5k6s2W2Q4x3X3g2K6K9i4c8W2i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9^5i4K6u0r3x3e0u0Q4x3V1k6^5K9h3p5H3L8r3I4V1j5W2)9J5k6r3q4F1N6r3W2Q4x3X3c8S2L8Y4c8A6i4K6u0V1k6r3g2T1N6h3N6Q4x3V1k6^5K9h3p5H3L8r3I4V1j5W2)9J5k6r3q4F1N6r3W2Q4x3X3c8S2L8Y4c8A6i4K6u0V1k6r3g2T1N6h3N6Q4x3V1j5`.">iOS LLDB 中基于内存单指令 patch 实现反反调试</a>介绍了在 LLDB 中如何通过单指令 patch 的方式去绕过`ptrace`来反调试。

但后面还有一些情况没有解决：如果不是调用`ptrace`函数，而是直接编写内联汇编的方式，调用对应的系统调用来间接实现反调试，那么这种方式之前的那种办法就不再可行。本文就准备解决那种用内联汇编的反调试方式。

## 两种绕过方案

这里有两种方案去绕过这种反调试：

- 静态内存匹配特征 patch
- 实现一个简单的 inlinehook 动态 hook 绕过

## 静态内存匹配特征 patch

在介绍思路之前，先看一遍内联汇编去实现反调试的代码：

```assembly
asm volatile(
"mov x0,#31\n"
"mov x1,#0\n"
"mov x2,#0\n"
"mov x3,#0\n"
"mov x16,#26\n"
"svc #128\n"
);
```

这里可以看出，原理就是调用了 26 号系统调用。那么我们是不是可以去代码段里面搜索，找到满足该特征的代码位置，然后直接将`svc`置为`nop`不就可以了？下面引出两个问题：如何去遍历代码段，以及如何去修改？

### 获取代码段位置以及大小

```c
void getTextSegmentAddr(struct segmentRange *textSegRange){
int offset = 0;
struct mach_header_64* header = (struct mach_header_64*)_dyld_get_image_header(0);
if(header->magic != MH_MAGIC_64) {
return ;
}
offset = sizeof(struct mach_header_64);
int ncmds = header->ncmds;
while(ncmds--) {
/* go through all load command to find __TEXT segment*/
struct load_command * lcp = (struct load_command *)((uint8_t*)header + offset);
offset += lcp->cmdsize;
if(lcp->cmd == LC_SEGMENT_64) {
struct segment_command_64 * curSegment = (struct segment_command_64 *)lcp;
struct section_64* curSection = (struct section_64*)((uint8_t*)curSegment + sizeof(struct segment_command_64));
// check current section of segment is __TEXT?
if(!strcmp(curSection->segname, "__TEXT") && !strcmp(curSection->sectname, "__text")){
uint64_t memAddr = curSection->addr;
textSegRange->start = memAddr + _dyld_get_image_vmaddr_slide(0);
textSegRange->end = textSegRange->start + curSection->size;
break;
}
}
}
return ;
}
```

代码不复杂，就是动态解析了自身内存里面的 Mach-O 文件，根据 Mach-O 文件格式找到代码段`LC_SEGMENT_64(__TEXT)`，然后就能得到`__text`的开始位置以及大小。

### 内存搜索匹配 ptrace 内联汇编代码

```c
void* lookup_ptrace_svc(void* target_addr, uint64_t size){
uint8_t * p = (uint8_t*)target_addr;
for(int i = 0; i < size ;i++ ){
/*
mov x16, #0x1a -> 0xd2800350
svc #0x80 -> 0xd4001001
*/
if (*(uint32_t*)p == 0xd2800350 && *(uint32_t*)p+1 == 0xd4001001) {
return p;
}
p++;
}
return NULL;
}
```

传入的就是代码段的地址以及大小，然后遍历整个代码段，找到满足以下`ptrace`特征的汇编代码：

```assembly
mov x16, #0x1a -> 0xd2800350
svc #0x80 -> 0xd4001001
```

然后就返回该地址。

### patch 代码（将 `svc` 改为 `nop`）

<a href="elink@0f2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0c8U0K9o6p5J5k6s2W2Q4x3X3g2K6K9i4c8W2i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9^5i4K6u0r3x3e0u0Q4x3V1k6^5K9h3p5H3L8r3I4V1j5W2)9J5k6r3q4F1N6r3W2Q4x3X3c8S2L8Y4c8A6i4K6u0V1k6r3g2T1N6h3N6Q4x3V1k6^5K9h3p5H3L8r3I4V1j5W2)9J5k6r3q4F1N6r3W2Q4x3X3c8S2L8Y4c8A6i4K6u0V1k6r3g2T1N6h3N6Q4x3V1k6^5K9h3p5H3L8r3I4V1j5W2)9J5k6r3q4F1N6r3W2Q4x3X3c8S2L8Y4c8A6i4K6u0V1k6r3g2T1N6h3N6Q4x3V1k6^5K9h3p5H3L8r3I4V1j5W2)9J5k6r3q4F1N6r3W2Q4x3X3c8S2L8Y4c8A6i4K6u0V1k6r3g2T1N6h3N6Q4x3V1j5`.">iOS LLDB 中基于内存单指令 patch 实现反反调试</a>这篇文章介绍了如何去 patch 代码的原理，但当时遇到一个 bug：在 iOS 11/12 上面 patch 会失败。后面我花了一段时间去分析失败的原因，后来也写了一篇文章去记录分析的过程，感兴趣的可以访问<a href="elink@e8fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0c8U0K9o6p5J5k6s2W2Q4x3X3g2K6K9i4c8W2i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9^5i4K6u0r3x3e0k6Q4x3V1k6A6e0#2x3I4x3W2)9J5k6s2u0W2L8h3q4H3i4K6u0V1j5Y4g2Y4i4K6u0V1j5h3&6S2L8s2W2\*7k6g2)9J5c8X3W2a6f1K6p5J5i4K6u0V1M7X3g2E0j5i4m8Q4x3X3c8T1N6h3N6Q4x3X3c8S2L8X3q4D9P5i4A6W2i4K6u0r3">iOS12 内存 patch remap bug 分析</a>。

这里我就直接给出 patch 的代码：

```c
uint8_t patch_ins_data[4] = {0x1f, 0x20, 0x03, 0xd5}; // nop
patchCode(ptrace_svc_p+4, patch_ins_data , 4);
```

完整流程代码如下：

```c
- (void)kill_anti_debug{
struct segmentRange textSegRange;
getTextSegmentAddr(&textSegRange);
void* ptrace_svc_p = lookup_ptrace_svc((void*)textSegRange.start, textSegRange.end-textSegRange.start);
if (!ptrace_svc_p) {
ADDLOG(@"[-] not found ptrace svc");
return;
}
ADDXLOG(@"[+] found ptrace svc # address=%p", ptrace_svc_p);
char* ptrace_bytes = hex_dump((void*)ptrace_svc_p, 8);
ADDXLOG(@"[+] read ptrace svc ins address:%p size:0x%x inst_bytes:%s", ptrace_svc_p, 8, ptrace_bytes);
free(ptrace_bytes);
ADDLOG(@"[*] start to ptach ptrace svc to ret");
uint8_t patch_ins_data[4] = {0x1f, 0x20, 0x03, 0xd5};
patchCode(ptrace_svc_p+4, patch_ins_data , 4);
ADDLOG(@"[*] ptach ptrace svc to nop done, read new value");
ptrace_bytes = hex_dump((void*)ptrace_svc_p, 8);
ADDXLOG(@"[+] read ptrace svc ins address:%p size:0x%x inst_bytes:%s", ptrace_svc_p, 8, ptrace_bytes);
free(ptrace_bytes);
}
```

通过比对前后的代码，就发现`svc`处地址的代码已经变成了`nop`，从而绕过了反调试。

## inlinehook 动态 hook 绕过

这种方式主要针对那些混淆了系统调用号或者其他编译版本。其绕过原理是直接 hook `svc`指令，然后判断是否为 26 号系统调用（让其他系统调用正常执行），若满足就直接跳过`svc`指令。

整体流程代码如下：

```c
struct segmentRange textSegRange;
getTextSegmentAddr(&textSegRange);
void* svc_p = lookup_svc_ins((void*)textSegRange.start, textSegRange.end-textSegRange.start);
if (!svc_p) {
ADDLOG(@"[-] not found svc");
return;
}
ADDXLOG(@"[+] found svc # address=%p", svc_p);
char* svc_bytes = hex_dump((void*)svc_p, 4);
ADDXLOG(@"[+] read ptrace svc ins address:%p size:0x%x inst_bytes:%s", svc_p, 4, svc_bytes);
free(svc_bytes);
xia0Hook(svc_p);
```

同样遍历代码段找到所有的`svc`指令，然后进行 hook。下面看 hook 的具体实现：

```c
bool xia0Hook(void* target_addr){
int len = (int)sysconf(_SC_PAGESIZE);
// 1. get target address page and patch offset
unsigned long page_start = (unsigned long) (target_addr) & ~PAGE_MASK;
unsigned long patch_offset = (unsigned long)target_addr - page_start;
printf("[*] Target address:%p Page start:%p Patch offset:%p", target_addr, (void*)page_start, (void*)patch_offset);
// 2. map new page for patch
void *new = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_ANON | MAP_SHARED, -1, 0);
if (!new ){
printf("[-] mmap failed!");
return false;
}
// 3.copy target 4 ins to new page
int copy_size = 4*4;
void* copy_from_addr = target_addr - copy_size;
memcpy((void *)(new), copy_from_addr, copy_size);
/*
cmp x16, #0x1a
b.ne loc_not_ptrace_svc_jmp
ldr x17, #0x8
br x17
orig_svc_next_addr_1
orig_svc_next_addr_2
ldr x17, #0x8
br x17
orig_svc_addr_1
orig_svc_addr_2
*/
uint64_t orig_svc_addr = (uint64_t)target_addr;
uint64_t orig_svc_next_addr = (uint64_t)(target_addr+1*4);
uint8_t check_jmp_data[] = {0x1f, 0x6a, 0x00, 0xf1, 0x51, 0x00, 0x00, 0x58, 0x20, 0x02, 0x1f, 0xd6, orig_svc_next_addr&0xff, (orig_svc_next_addr>>8*1)&0xff, (orig_svc_next_addr>>8*2)&0xff, (orig_svc_next_addr>>8*3)&0xff, (orig_svc_next_addr>>8*4)&0xff, (orig_svc_next_addr>>8*5)&0xff, (orig_svc_next_addr>>8*6)&0xff, (orig_svc_next_addr>>8*7)&0xff, 0x51, 0x00, 0x00, 0x58, 0x20, 0x02, 0x1f, 0xd6, orig_svc_addr&0xff, (orig_svc_addr>>8*1)&0xff, (orig_svc_addr>>8*2)&0xff, (orig_svc_addr>>8*3)&0xff, (orig_svc_addr>>8*4)&0xff, (orig_svc_addr>>8*5)&0xff, (orig_svc_addr>>8*6)&0xff, (orig_svc_addr>>8*7)&0xff};
int check_jmp_data_size = 10*4;
memcpy((void *)(new+4*4), check_jmp_data, check_jmp_data_size);
// 4.patch target address to jmp hook code
void* patch_addr = copy_from_addr;
uint64_t new_p = (uint64_t)new;
/*
ldr x16, #0x8
br x16
hook_code_addr_1
hook_code_addr_2
*/
uint8_t patch_data[] = {0x50, 0x00, 0x00, 0x58, 0x00, 0x02, 0x1f, 0xd6,new_p&0xff, (new_p>>8*1)&0xff, (new_p>>8*2)&0xff, (new_p>>8*3)&0xff, (new_p>>8*4)&0xff, (new_p>>8*5)&0xff, (new_p>>8*6)&0xff, (new_p>>8*7)&0xff};
int patch_data_size = 4*4;
patchCode(patch_addr, patch_data, patch_data_size);
// 5. set new page to r-x
mprotect(new, len, PROT_READ | PROT_EXEC);
return true;
}
```

这里代码比较复杂，大致分为以下步骤：

- `map`一页内存`new`，后面会将 hook 的代码写到里面。
- `copy`原`svc`前的四条指令保存到`new`页（目前没有进行相对寻址修复）。
- 将 hook 判断的代码写到紧接着前面四条指令的后面，汇编代码大致如下：

```assembly
cmp x16, #0x1a
b.ne loc_not_ptrace_svc_jmp
ldr x17, #0x8
br x17
orig_svc_next_addr_1
orig_svc_next_addr_2
ldr x17, #0x8
br x17
orig_svc_addr_1
orig_svc_addr_2
```

就是简单地判断了系统调用号是否为 26：若满足，就跳到`svc`的下一条指令；若不是，则跳回原`svc`指令，以保证其他系统调用正常执行。

- `patch`目标地址进行 hook 跳转。由于进行任意地址跳转需要四条指令大小，所以这里覆盖了`svc`前的四条指令。
