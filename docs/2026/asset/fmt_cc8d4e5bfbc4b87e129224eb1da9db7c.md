
`guard` 的初始化位于函数 `__sanitizer_cov_trace_pc_guard_init` 中：

```c
void __sanitizer_cov_trace_pc_guard(uint32_t * guard) {
  __afl_area_ptr[*guard]++;
}
```

```c
void __sanitizer_cov_trace_pc_guard_init(uint32_t * start, uint32_t * stop) {
  u32 inst_ratio = 100;
  u8 * x;
  if (start == stop || *start) return;
  x = getenv("AFL_INST_RATIO");
  if (x) inst_ratio = atoi(x);
  if (!inst_ratio || inst_ratio > 100) {
    fprintf(stderr, "[-] ERROR: Invalid AFL_INST_RATIO (must be 1-100).\n");
    abort();
  }
  *(start++) = R(MAP_SIZE - 1) + 1;
  while (start < stop) {
    /* 这里如果计算 stop - start，就是程序里总计的 edge 数 */
    if (R(100) < inst_ratio) *start = R(MAP_SIZE - 1) + 1;
    else *start = 0;
    start++;
  }
}
```

## 总结

其实在这里最主要的还是 `persistent mode` 中的逻辑，其他的两个模式对于实际的 fuzz 工作已经或尚且没有太大的指导意义。但是研究一下其源码实现，可以发现 AFL 的设计和开发真的很巧妙，其中很多思路值得开发人员和安全人员学习和借鉴。

## 参考链接

1. 6d8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6r3g2J5L8X3q4D9M7$3q4C8N6i4u0S2x3e0y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6S2Q4x3V1j5J5x3#2)9J5c8X3q4X3L8q4)9J5c8R3`.
2. https://bbs.pediy.com/thread-266025.htm

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-9-27 16:59

被有毒编辑，原因：

[#自动化挖掘](https://bbs.kanxue.com/forum-150-1-155.htm)

[#Fuzz](https://bbs.kanxue.com/forum-150-1-157.htm)

[#Linux](https://bbs.kanxue.com/forum-150-1-161.htm)
