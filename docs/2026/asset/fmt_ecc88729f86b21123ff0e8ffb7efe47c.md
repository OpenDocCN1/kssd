
`typedef struct {`
`uint32` `next``;`
`uint32` `len``;`
`} LG_mem_free_t;`

`uint32 LG_mem_min;`
`uint32 LG_mem_top;`
`LG_mem_free_t LG_mem_free;`
`char` `*``LG_mem_base;`
`uint32 LG_mem_len;`
`char` `*``Origin_LG_mem_base;`
`uint32 Origin_LG_mem_len;`
`char` `*``LG_mem_end;`
`uint32 LG_mem_left;`

```
#define realLGmemSize(x) (((x) + 7) & (0xfffffff8))
```

`//` `初始化内存管理器`
`//` `baseAddress:  托管的内存的首地址，是一个模拟器内的地址`
`//` `len``:          内存的总长度`
```
void initMemoryManager(uint32 baseAddress, uint32 len) {
printf("initMemoryManager: baseAddress:0x%X len: 0x%X\n", baseAddress, len);
Origin_LG_mem_base = toPtr(baseAddress);
Origin_LG_mem_len = len;

LG_mem_base = (char *)((uint32)(Origin_LG_mem_base + 3) & (~3));
LG_mem_len = (Origin_LG_mem_len - (LG_mem_base - Origin_LG_mem_base)) & (~3);
LG_mem_end = LG_mem_base + LG_mem_len;
LG_mem_free.next = 0;
LG_mem_free.len = 0;
((LG_mem_free_t *)LG_mem_base)->next = LG_mem_len;
((LG_mem_free_t *)LG_mem_base)->len = LG_mem_len;
LG_mem_left = LG_mem_len;
#ifdef MEM_DEBUG
LG_mem_min = LG_mem_len;
LG_mem_top = 0;
#endif
}
```

```
void *my_malloc(uint32 len) {
LG_mem_free_t *previous, *nextfree, *l;
void *ret;

len = (uint32)realLGmemSize(len);
if (len >= LG_mem_left) {
printf("my_malloc no memory\n");
goto err;
}
if (!len) {
printf("my_malloc invalid memory request");
goto err;
}
if (LG_mem_base + LG_mem_free.next > LG_mem_end) {
printf("my_malloc corrupted memory");
goto err;
}
previous = &LG_mem_free;
nextfree = (LG_mem_free_t *)(LG_mem_base + previous->next);
while ((char *)nextfree < LG_mem_end) {
if (nextfree->len == len) {
previous->next = nextfree->next;
LG_mem_left -= len;
#ifdef MEM_DEBUG
if (LG_mem_left < LG_mem_min) LG_mem_min = LG_mem_left;
if (LG_mem_top < previous->next) LG_mem_top = previous->next;
#endif
ret = (void *)nextfree;
goto end;
}
if (nextfree->len > len) {
l = (LG_mem_free_t *)((char *)nextfree + len);
l->next = nextfree->next;
l->len = (uint32)(nextfree->len - len);
previous->next += len;
LG_mem_left -= len;
#ifdef MEM_DEBUG
if (LG_mem_left < LG_mem_min) LG_mem_min = LG_mem_left;
if (LG_mem_top < previous->next) LG_mem_top = previous->next;
#endif
ret = (void *)nextfree;
goto end;
}
previous = nextfree;
nextfree = (LG_mem_free_t *)(LG_mem_base + nextfree->next);
}
printf("my_malloc no memory\n");
err:
return 0;
end:
return ret;
}
```

```
void my_free(void *p, uint32 len) {
LG_mem_free_t *free, *n;
len = (uint32)realLGmemSize(len);
#ifdef MEM_DEBUG
if (!len || !p || (char *)p < LG_mem_base || (char *)p >= LG_mem_end || (char *)p + len > LG_mem_end || (char *)p + len <= LG_mem_base) {
printf("my_free invalid\n");
printf("p=%d,l=%d,base=%d,LG_mem_end=%d\n", (int32)p, len, (int32)LG_mem_base, (int32)LG_mem_end);
return;
}
#endif
free = &LG_mem_free;
n = (LG_mem_free_t *)(LG_mem_base + free->next);
while (((char *)n < LG_mem_end) && ((void *)n < p)) {
free = n;
n = (LG_mem_free_t *)(LG_mem_base + n->next);
}
#ifdef MEM_DEBUG
if (p == (void *)free || p == (void *)n) {
printf("my_free:already free\n");
return;
}
#endif
if ((free != &LG_mem_free) && ((char *)free + free->len == p)) {
free->len += len;
}
else {
free->next = (uint32)((char *)p - LG_mem_base);
free = (LG_mem_free_t *)p;
free->next = (uint32)((char *)n - LG_mem_base);
free->len = len;
}
if (((char *)n < LG_mem_end) && ((char *)p + len == (char *)n)) {
free->next = n->next;
free->len += n->len;
}
LG_mem_left += len;
}
```

由于上面的`my_free()`在释放内存时要求传入释放内存的长度，与c语言的`free()`用法不同，因此还需要增加两个包装后的函数：

```
void *my_mallocExt(uint32 len) {
uint32 *p;
if (len == 0) {
return NULL;
}
p = my_malloc(len + sizeof(uint32));
if (p) {
*p = len;
return (void *)(p + 1);
}
return p;
}

void my_freeExt(void *p) {
if (p) {
uint32 *t = (uint32 *)p - 1;
my_free(t, *t + sizeof(uint32));
}
}
```

`my_mallocExt()`和`my_freeExt()`直接替换系统的`malloc()`和`free()`进行内存管理，通过这种方式获得的内存只需要通过 **地址转换函数** 处理一下就可以在模拟器内外自由使用了。

一个常见的用法是传递字符串：

```
uint32_t copyStrToEmu(char *str) {
if (!str) return 0;
uint32_t len = strlen(str) + 1;
void *p = my_mallocExt(len);
memcpy(p, str, len);
return toAddr(p);
}
```

`//` `str` `将是一个模拟器内的地址，直接传递给模拟器`
`uint32_t` `str` `=` `copyStrToEmu("test.txt");`

`//` `str2 将是一个本地指针，可以直接使用`
`char` `*``str2` `=` `toPtr(str);`
`printf("%s\n", str2);`

## 结束

以上就是我在使用Unicorn用到的最核心技术，如果对实际应用的真实案例感兴趣可以研究下我的开源模拟器 [vmrp](https://bbs.kanxue.com/elink@f73K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6*7k6h3&6Y4L8h3W2F1k6K6l9H3i4K6u0r3N6X3#2J5M7l9`.`.)

2021/04/05 23:11 中国标准时间

原文地址a88K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6\*7k6h3&6Y4L8h3W2F1k6K6l9H3i4K6u0r3N6h3&6A6j5$3!0J5L8W2)9#2k6X3g2F1k6$3W2F1k6g2)9#2k6Y4c8#2N6r3!0J5K9h3q4D9

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

[#VM保护](https://bbs.kanxue.com/forum-4-1-4.htm)

[#其他内容](https://bbs.kanxue.com/forum-4-1-10.htm)
