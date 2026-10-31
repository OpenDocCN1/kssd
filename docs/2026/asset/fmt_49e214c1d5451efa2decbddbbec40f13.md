
# [原创]堆学习：Unlink attack

## 参考
4e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2L8%4u0Y4i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6#2M7$3g2J5i4K6u0V1L8h3!0V1k6g2)9J5c8X3S2W2j5i4m8Q4x3V1k6H3N6r3#2S2L8r3I4G2j5K6u0Q4x3V1k6A6L8i4m8D9k6h3#2W2L8Y4c8S2N6r3W2G2L8W2)9J5c8X3u0S2M7$3W2U0i4K6u0r3
b9bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2L8%4u0Y4i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6#2M7$3g2J5i4K6u0V1L8h3!0V1k6g2)9J5c8X3S2W2j5i4m8Q4x3V1k6H3N6r3#2S2L8r3I4G2j5K6u0Q4x3V1k6A6L8i4m8D9k6h3#2W2L8Y4c8S2N6r3W2G2L8W2)9J5c8X3k6J5k6h3g2Q4x3V1j5`.
abeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2L8%4u0Y4i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6#2M7$3g2J5i4K6u0V1L8h3!0V1k6g2)9J5c8X3S2W2j5i4m8Q4x3V1k6H3N6r3#2S2L8r3I4G2j5K6u0Q4x3V1k6#2L8X3I4A6L8X3E0Q4x3V1j5`.

---

## 基础知识

### 什么是 Unlink？

在 Linux 的 glibc 堆管理机制中，`unlink` 是一个核心操作，其字面意思为“脱链”。当堆管理器释放某一 `chunk` 时，如果发现其相邻的 `chunk` 处于空闲状态，便会尝试将这两个 `chunk` 合并成一个更大的空闲 `chunk`。为了完成合并，就需要将相邻的空闲 `chunk` 从其所在的双向链表（例如 `small bins`、`unsorted bins` 等）中取出，这个“取出”的过程就是 `unlink`。

以下两段代码分别是 `free` 时的后向合并和前向合并，可以看到都有 `unlink` 操作

```python
/\* consolidate backward \*/
if (!prev\_inuse(p)) {
prevsize = prev\_size(p);
size += prevsize;
p = chunk\_at\_offset(p, -((long) prevsize));
unlink(av, p, bck, fwd);
}
```

```python
// 如果下一个chunk不是top chunk
if (nextchunk != av->top) {
/\* get and clear inuse bit \*/
// 获取下一个 chunk 的使用状态
nextinuse = inuse\_bit\_at\_offset(nextchunk, nextsize);
// 如果不在使用，合并，否则清空当前chunk的使用状态。
/\* consolidate forward \*/
if (!nextinuse) {
unlink(av, nextchunk, bck, fwd);
size += nextsize;
}
```

### Unlink 的攻击原理（Unlink Attack）

`Unlink` 攻击的核心在于，通过漏洞（如堆溢出、`off-by-one` 等）篡改堆块的关键元数据，伪造一个“看似合法”的空闲 `chunk` 结构。当程序后续进行 `unlink` 操作时，会按照我们伪造的指针执行写入操作，从而实现 `任意地址写`。

以下这段源码是从 `ctfwiki` 复制过来的，我觉得里面的注释已经很详细了

```python
/\* Take a chunk off a bin list \*/
// unlink p
#define unlink(AV, P, BK, FD) { \
// 由于 P 已经在双向链表中，所以有两个地方记录其大小，所以检查一下其大小是否一致。
if (\_\_builtin\_expect (chunksize(P) != prev\_size (next\_chunk(P)), 0)) \
malloc\_printerr ("corrupted size vs. prev\_size"); \
FD = P->fd; \
BK = P->bk; \
// 防止攻击者简单篡改空闲的 chunk 的 fd 与 bk 来实现任意写的效果。
if (\_\_builtin\_expect (FD->bk != P || BK->fd != P, 0)) \
malloc\_printerr (check\_action, "corrupted double-linked list", P, AV); \
else { \
FD->bk = BK; \
BK->fd = FD; \
// 下面主要考虑 P 对应的 nextsize 双向链表的修改
if (!in\_smallbin\_range (chunksize\_nomask (P)) \
// 如果P->fd\_nextsize为 NULL，表明 P 未插入到 nextsize 链表中。
// 那么其实也就没有必要对 nextsize 字段进行修改了。
// 这里没有去判断 bk\_nextsize 字段，可能会出问题。
&& \_\_builtin\_expect (P->fd\_nextsize != NULL, 0)) { \
// 类似于小的 chunk 的检查思路
if (\_\_builtin\_expect (P->fd\_nextsize->bk\_nextsize != P, 0) \
|| \_\_builtin\_expect (P->bk\_nextsize->fd\_nextsize != P, 0)) \
malloc\_printerr (check\_action, \
"corrupted double-linked list (not small)", \
P, AV); \
// 这里说明 P 已经在 nextsize 链表中了。
// 如果 FD 没有在 nextsize 链表中
if (FD->fd\_nextsize == NULL) { \
// 如果 nextsize 串起来的双链表只有 P 本身，那就直接拿走 P
// 令 FD 为 nextsize 串起来的
if (P->fd\_nextsize == P) \
FD->fd\_nextsize = FD->bk\_nextsize = FD; \
else { \
// 否则我们需要将 FD 插入到 nextsize 形成的双链表中
FD->fd\_nextsize = P->fd\_nextsize; \
FD->bk\_nextsize = P->bk\_nextsize; \
P->fd\_nextsize->bk\_nextsize = FD; \
P->bk\_nextsize->fd\_nextsize = FD; \
} \
} else { \
// 如果在的话，直接拿走即可
P->fd\_nextsize->bk\_nextsize = P->bk\_nextsize; \
P->bk\_nextsize->fd\_nextsize = P->fd\_nextsize; \
} \
} \
} \
}
```

分别对应以下三个检查

```python
// 由于 P 已经在双向链表中，所以有两个地方记录其大小，所以检查一下其大小是否一致(size检查)
if (\_\_builtin\_expect (chunksize(P) != prev\_size (next\_chunk(P)), 0)) \
malloc\_printerr ("corrupted size vs. prev\_size"); \
// 检查 fd 和 bk 指针(双向链表完整性检查)
if (\_\_builtin\_expect (FD->bk != P || BK->fd != P, 0)) \
malloc\_printerr (check\_action, "corrupted double-linked list", P, AV); \
// largebin 中 next\_size 双向链表完整性检查
if (\_\_builtin\_expect (P->fd\_nextsize->bk\_nextsize != P, 0) \
|| \_\_builtin\_expect (P->bk\_nextsize->fd\_nextsize != P, 0)) \
malloc\_printerr (check\_action, \
"corrupted double-linked list (not small)", \
P, AV);
```

针对这些检查，攻击者需要精心构造 `fd` 和 `bk` 指针来绕过。一个经典的绕过方法是：将伪造 `chunk` 的 `fd` 指针设置为 `&target_address - 0x18`。将 `bk` 指针设置为 `&target_address - 0x10`。这样，在检查 `FD->bk`（即 `*(fd + 0x18)`）和 `BK->fd`（即 `*(bk + 0x10)`）时，计算出的地址都恰好指向 `target_address` 本身，如果 `target_address` 处存储的正是指向这个伪造 `chunk` 的指针 `P`，检查就能被绕过。随后，`unlink` 操作会执行 `FD->bk = BK` 和 `BK->fd = FD`，其最终效果是将 `target_address` 处的值修改为 `&target_address - 0x18`。如果 `target_address` 是某个全局函数指针（如 `free@got`）或重要数据，攻击者就获得了修改它的能力。具体操作可以看例题一的分析过程。

`wget https://launchpadlibrarian.net/353523729/libc6_2.23-0ubuntu10_amd64.deb`
`dpkg -x libc6_2.23-0ubuntu10_amd64.deb libc6_2.23-0ubuntu10_amd64`
`patchelf --set-interpreter ./libc6_2.23-0ubuntu10_amd64/lib/x86_64-linux-gnu/ld-2.23.so ./源程序`
`patchelf --replace-needed libc.so.6 ./libc6_2.23-0ubuntu10_amd64/lib/x86_64-linux-gnu/libc-2.23.so ./源程序`

## 2014_hitcon_stkof

查看保护，Partial RELRO 说明 `got` 表可写

![](upload/attach/202603/1002337_SB44VH3ZF4RM96Z.png)

从 `main` 函数可以看到 `v3` 有 4 种取值：1 2 3 4

```python
\_\_int64 \_\_fastcall main(int a1, char \*\*a2, char \*\*a3)
{
int v3; // eax
int v5; // [rsp+Ch] [rbp-74h]
char nptr[104]; // [rsp+10h] [rbp-70h] BYREF
unsigned \_\_int64 v7; // [rsp+78h] [rbp-8h]
v7 = \_\_readfsqword(0x28u);
alarm(0x78u);
while ( fgets(nptr, 10, stdin) )
{
v3 = atoi(nptr);
if ( v3 == 2 )
{
v5 = sub\_4009E8();
goto LABEL\_14;
}
if ( v3 > 2 )
{
if ( v3 == 3 )
{
v5 = sub\_400B07();
goto LABEL\_14;
}
if ( v3 == 4 )
{
v5 = sub\_400BA9();
goto LABEL\_14;
}
}
else if ( v3 == 1 )
{
v5 = sub\_400936();
goto LABEL\_14;
}
v5 = -1;
LABEL\_14:
if ( v5 )
puts("FAIL");
else
puts("OK");
fflush(stdout);
}
return 0LL;
}
```

当 `v3` = 1 时，可以申请自定义大小的内存，返回的数据部分地址存入了全局变量 `s`，`dword_602100` 是已申请的个数。(`&::s`)[++`dword_602100`] = `v2` 等价于 `dword_602100` += 1; `s`[`dword_602100`] = `v2`;

```python
\_\_int64 sub\_400936()
{
\_\_int64 size; // [rsp+0h] [rbp-80h]
char \*v2; // [rsp+8h] [rbp-78h]
char s[104]; // [rsp+10h] [rbp-70h] BYREF
unsigned \_\_int64 v4; // [rsp+78h] [rbp-8h]
v4 = \_\_readfsqword(0x28u);
fgets(s, 16, stdin);
size = atoll(s);
v2 = (char \*)malloc(size);
if ( !v2 )
return 0xFFFFFFFFLL;
(&::s)[++dword\_602100] = v2;
printf("%d\n", (unsigned int)dword\_602100);
return 0LL;
}
```

第一次申请时，就是把返回的地址存入 `0x602148` 这个位置，也就是 `s[1]`，从 1 开始存。

![](upload/attach/202603/1002337_EHCCH4UCW7A4DJB.png)

当 `v3` = 2 时，可以自定义写入内存的长度，存在堆溢出，可以覆写高地址 `chunk` 的内容。
