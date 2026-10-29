
```
//替换目标函数的前len个字节，使之跳转到hook函数
void change_bytes(intptr_t addr, const char code[], int len)
{
 memcpy((void*)addr, code, len);
}

void func_hook(intptr_t target_addr, void* hook_func)
{
```c
//根据目标函数的地址确定目标函数所在的页，并将该页的权限改为可读可写可执行
intptr_t page_start = target_addr & 0xfffffffff000;
mprotect((void*)page_start, 0x1000, PROT_READ|PROT_WRITE|PROT_EXEC);

int asm_len = get_asm_len(target_addr);
if(asm_len == 0)
{
 change_asm_code(target_addr, temp_func + 13 + 14);
}
if(asm_len == -1)
{
 printf("Error: get_asm_code\n");
 exit(-1);
}

//保存跳板中的地址到rsp中，这样自定义函数返回时可以返回跳板函数继续执行原函数
intptr_t x = *temp_func + 27;
char ret_code[13] = {0x68, x&0xff, (x&0xff00)>>8, (x&0xff0000)>>16, (x&0xff000000)>>24,
0xC7, 0x44, 0x24, 0x04, (x&0xff00000000)>>32, (x&0xff0000000000)>>40, (x&0xff000000000000)>>48,
(x&0xff00000000000000)>>56};
memcpy((void*)temp_func, (void*)ret_code, asm_len);

//跳板地址跳到自定义函数
intptr_t y = hook_func;
char self_code[14] = {0x68, y&0xff, (y&0xff00)>>8, (y&0xff0000)>>16, (y&0xff000000)>>24,
0xC7, 0x44, 0x24, 0x04, (y&0xff00000000)>>32, (y&0xff0000000000)>>40, (y&0xff000000000000)>>48,
(y&0xff00000000000000)>>56, 0xC3};
memcpy((void*)temp_func + 13, (void*)self_code, asm_len);

//复制原始指令
memcpy((void*)temp_func+13+14, (void*)target_addr, asm_len);

//跳转指令，跳回原始地址+asmlen处
intptr_t z = (intptr_t)target_addr + asm_len;
//构造push&ret跳转，填入目标地址
char jmp_code[14] = {0x68, z&0xff, (z&0xff00)>>8, (z&0xff0000)>>16, (z&0xff000000)>>24,
0xC7, 0x44, 0x24, 0x04, (z&0xff00000000)>>32, (z&0xff0000000000)>>40, (z&0xff000000000000)>>48,
(z&0xff00000000000000)>>56, 0xC3};
memcpy((void*)(temp_func+asm_len+13+14), (void*)jmp_code, 14);

//目标地址改为跳到跳板地址
uint32_t relativeAddr = target_addr - (uint32_t)temp_func - 5;
uint8_t jumpCode[5] = {0xe9, 0x00, 0x00, 0x00, 0x00};
memcpy(jumpCode + 1, &relativeAddr, sizeof(uint32_t));
change_bytes(target_addr, (const char*) jumpCode, 5);

//用于后续的hook函数使用
temp_func = temp_func + 60;
```
}

//
void test_hook()
{
 __asm("push %rdi");
 /*__asm{
 push rdi;
 push rsi;
 push rdx;
 push rcx;
 push rax;
 }*/

 printf("everythint is ok\n");
 register int *rax asm ("rax");
 printf("rax is %lx\n",*rax);

```c
/*__asm{
 pop rax;
 pop rcx;
 pop rdx;
 pop rsi;
 pop rdi;
}*/
```
}

//so被加载后会首先执行这里的代码
__attribute__((constructor))
void load()
{
 temp_func = (char*)mmap(tempAddr, 4096, PROT_WRITE|PROT_EXEC|PROT_READ, MAP_ANON|MAP_PRIVATE, -1, 0);
 func_hook(0x666666, (void*)test_hook);

```c
printf("inject suceessfully\n");
```
}

这里就是粗略的实现了，反正能用。。而且是从内网往回敲的，可能有点错误还，反正就是这么个思路，好多不是很优美，等后续有时间了不断完善可以

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
