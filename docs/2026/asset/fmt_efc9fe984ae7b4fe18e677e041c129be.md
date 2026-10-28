
### 分析函数 `sub_48840`

```
BYTE *__cdecl sub_48840(BYTE *buff1_16, void *src, size_t len_32)
{
  v3 = *buff1_16;
  v4 = 10;
  if ( (v3 & 1) != 0 ) // 0 & 1 != 0 判断失败
    v4 = (*(_DWORD *)buff1_16 & 0xFFFFFFFE) - 1;
  if ( v4 >= len_32 ) // 10 >= 32 判断失败
  {
    ......
  }
  else
  {
    if ( (v3 & 1) != 0 ) // 判断失败
      v5 = *((_DWORD *)buff1_16 + 1);
    else
      v5 = v3 >> 1; // v5 = 0
    sub_488E0(buff1_16, v4, len_32 - v4, v5, 0, v5, len_32, src);
    //
    // sub_488e0(buff1_16,10,22,0,0,0,32,"3390fd362dfdda0030d5737632d3d213")
    // buff1_16[0-3] 存放着申请堆空间大小+1，也就是49
    // buff1_16[4-7] 存放着字符串"3390fd362dfdda0030d5737632d3d213"的长度，也就是32
    // buff1_16[8-11] 存放着字符串"3390fd362dfdda0030d5737632d3d213"的指针
  }
  return buff1_16;
}
```

### 分析函数 `sub_488E0`

这里就是 `buff1` 真正调用的函数，作用是初始化 `buff1` 空间中的数据

- `buff1_60_16[0-3]` 存放着申请堆空间大小+1，也就是49
- `buff1_60_16[4-7]` 存放着字符串 `"3390fd362dfdda0030d5737632d3d213"` 的长度，也就是32
- `buff1_60_16[8-11]` 存放着字符串 `"3390fd362dfdda0030d5737632d3d213"` 的指针

```
// sub_488e0(buff1_60_16,10,22,0,0,0,32,"3390fd362dfdda0030d5737632d3d213")
BYTE *__cdecl sub_488E0(BYTE *buff1_60_16, unsigned int a2, unsigned int a3, int a4, size_t n, int a6, size_t a7, void *src)
{
  if ( 0xFFFFFFEE - a2 < a3 ) // 这里注意a2的类型是无符号类型，-18会转换成一个非常大的正数，所以判断失败
    sub_48A20();
  if ( (*buff1_16 & 1) != 0 ) // buff1_16 == 0，判断失败
    v8 = (BYTE *)*((_DWORD *)buff1_16 + 2);
  else
    v8 = buff1_16 + 1; // v8 = buff1_17
  v18 = v8; // v18 = buff1_17
  v9 = -17; // v9 = -17
  if ( a2 <= 0x7FFFFFE6 ) // 10 <= 0x7ffffff6，判断成功
  {
    v10 = a2 + a3; // v10 = 32; a2 = 10; a3 = 22
    if ( a2 + a3 < 2 * a2 ) // 10 + 22 < 10 * 2 = false, 判断失败
      v10 = 2 * a2;
    v9 = 11; // v9 = 11
    if ( v10 >= 0xB ) // 32 >= 0xb，判断成功
      v9 = (v10 + 16) & 0xFFFFFFF0; // v9 = 48
  }
  v20 = v9; // v20 = 48
  buff_48 = (char *)operator new(v9); // 申请了48个字节的空间
  buff_48_ = buff_48;
  if ( n )
    memcpy(buff_48, v18, n);
  buff_48__ = buff_48_;
  v13 = a6; // v13 = 0
  v14 = a7; // v14 = a7 = 32
  if ( a7 )
  {
    memcpy(&buff_48_[n], src, a7); // buff_48[0-31] = "3390fd362dfdda0030d5737632d3d213"
    v13 = a6; // v13= 0
    v14 = a7; // v14 = 32
  }
  v15 = a4 - v13; // v15 = 0 - 0 = 0
  if ( a4 - v13 != n ) // 0 - 0 != 0 该判断失败
  {
    memcpy(&buff_48__[n + v14], &v18[n + a6], v15 - n);
    v14 = a7;
  }
  if ( a2 != 10 ) // 10 != 10 判断失败
  {
    operator delete(v18);
    v14 = a7;
  }
  result = buff1_16;
  *((_DWORD *)buff1_16 + 2) = buff_48__; // buff1_16[8-11] = buff_48；注意这里是把申请空间的地址放在了这里
  // buff1_16[8-11] 存放了一个堆空间的指针，该指针指向字符串"3390fd362dfdda0030d5737632d3d213"
  *(_DWORD *)buff1_16 = v20 | 1; // buff1_16[0-3] = 0x00000031
  // 小端存放 buff1_16[0-3] = {0x31,00,00,00}
  v17 = v14 + v15; // v17 = 32
  *((_DWORD *)buff1_16 + 1) = v17; // buff1_16[4-7] = 0x00000032
  // buff1_16[4-7] = {0x20,0,0,0}
  buff_48__[v17] = 0; // 字符串"3390fd362dfdda0030d5737632d3d213"后添加0确保截断
  return result;
  // 总结一下：
  // buff1_16[0-3] 存放着申请堆空间大小+1，也就是49
  // buff1_16[4-7] 存放着字符串"3390fd362dfdda0030d5737632d3d213"的长度，也就是32
  // buff1_16[8-11] 存放着字符串"3390fd362dfdda0030d5737632d3d213"的指针
}
```

### 分析函数 `set_buff2`

这个函数比较浅，主要就是计算随机数的 hash，然后初始化 `buff2` 的内存数据，结果是：

- `buff2[0-3] = 19aac+imagebase`
- `buff2[4-19] = 0`
- `buff2[20-531] = const_data ^ 0x0036`
- `buff2[532-595] = hash`

```
unsigned int __cdecl set_buff2(BYTE *buff2)
{
  v31 = __readgsdword(0x14u);
  *(_DWORD *)buff2 = &off_19CAAC; // buff2[0-3] = 19aac+imagebase
  *((_DWORD *)buff2 + 2) = 0; // buff2[4-19] = 0
  *((_DWORD *)buff2 + 1) = 0;
  *((_DWORD *)buff2 + 4) = 0;
  *((_DWORD *)buff2 + 3) = 0;
  memcpy(dest, byte_151F3A, sizeof(dest)); // 把固定地址的数据拷贝至dest，这里暂时将这串数据记作const_data
  memcpy(buff2 + 20, dest, 512u); // buff2[20-531] = const_data
  RAND_bytes(v29, 32); // 生成32字节大小的随机数
  v1 = v29[3]; // v1 = v29[3]
  v28[3] = v29[3];
  v2 = v29[2]; // v2 = v29[2]
  v28[2] = v29[2];
  v3 = v29[1]; // v3 = v29[1]
  v28[1] = v29[1];
  v28[0] = v29[0]; // v28[0-3] = v29[0-3]
  *(_QWORD *)(buff2 + 532) = v29[0]; // buff2[532-539] = v29[0]
  *(_QWORD *)(buff2 + 540) = v3; // buff2[540-547] = v3
  *(_QWORD *)(buff2 + 548) = v2; // buff2[548-555] = v2
  *(_QWORD *)(buff2 + 556) = v1; // buff2[556-563] = v1
  *(_QWORD *)(buff2 + 588) = v28[3]; // buff2[588-595] = v1
  v4 = v28[1];
  *(_QWORD *)(buff2 + 564) = v28[0]; // buff2[564-571] = v29[0]
  *(_QWORD *)(buff2 + 580) = v28[2]; // buff2[580-587] = v2
  *(_QWORD *)(buff2 + 572) = v4; // buff2[572-579] = v3
  // 小结一下：
  // buff2[532-563] = buff2[564-595] 是两组相同的随机数
  v5 = -64;
  v6 = 0;
  do
    v6 = *(_WORD *)&buff2[2 * (buff2[v5++ + 596] ^ HIBYTE(v6)) + 20] ^ (v6 << 8);
  while ( v5 );
  // 观察这个公式，buff2[v5++ + 596]指定就是最后的64个随机字节，v5++依次取出，这里将这个值记作rand[i]
  // 观察buff2[2 * (rand[i] ^ HIBYTE(v6)) + 20]，对照当前 buff2 的内存布局，这指的就是之前的 const_data
  // 也明白了buff2的空间为什么是596这个奇怪的大小，它的组成是20 + 256*2 + 64
  // 需要注意的是最后使用了 *(WORD*)&，也就是 const_data 被当做WORD取出
  //
  // 小结一下：
  // 随机字节和v6异或也是随机的，就记为随机
  // 循环将 const_data 以 WORD 类型随机取出，再和上一次结果的高位八位异或
  // 总的来讲就是得到了一个随机数 v6
  v27 = v6;
  *(_WORD *)((char *)v28 + 5) = v6; // 这里修改了v28的第5-6字节为v6
  hmac_ctx_hash((BYTE *)v28, (BYTE *)v29, buff2 + 532);
  // 这是一个已经分析过的函数，hash和data没有关系
  // 把随机数 v29 作为key，hash结果放在buff2[532-595]中
  v7 = -32;
  v8 = 0;
  do
    v8 = *(_WORD *)&buff2[2 * (*((unsigned __int8 *)v29 + v7++) ^ HIBYTE(v8)) + 20] ^ (v8 << 8);
  while ( v7 );
  // 这个循环和上边差不多，无非是随机数在v29这个随机字节数组中取，最终得到一个随机数 v8
  v9 = lrand48() % 12; // v9 是个0-11范围内的随机数
  *(_WORD *)&buff2[v9 + 532] = v8; // 随机数 v8 被随机的写在 buff2[532-544]中，注意 v8 是 WODD 类型
  *(_WORD *)&buff2[v9 + 534] = v27; // 一个 0 被随机的写在 buff2[534-546]中，注意 0 是 WODD 类型
  v10 = _mm_xor_ps(*(__m128 *)(buff2 + 36), (__m128)xmmword_148490);
  *(__m128 *)(buff2 + 20) = _mm_xor_ps(*(__m128 *)(buff2 + 20), (__m128)xmmword_148490);
  // buff2[20-35] ^= 0x00360036003600360036003600360036
  *(__m128 *)(buff2 + 36) = v10; // buff2[36-51] ^= 0x00360036003600360036003600360036
  v11 = _mm_xor_ps(*(__m128 *)(buff2 + 68), (__m128)xmmword_148490);
  *(__m128 *)(buff2 + 52) = _mm_xor_ps(*(__m128 *)(buff2 + 52), (__m128)xmmword_148490);
  //
  // 发现了规律，其实就是，buff2[20-532] 2字节一组与 0x0036 异或，并更新buff2中的值
  // 在这里总结一下：
  // buff2[0-3] = 19aac+imagebase
  // buff2[4-19] = 0
  // buff2[20-531] = const_data ^ 0x0036
  // buff2[532-595] = hash
  ......
  v25 = _mm_xor_ps(*(__m128 *)(buff2 + 516), (__m128)xmmword_148490);
  *(__m128 *)(buff2 + 500) = _mm_xor_ps(*(__m128 *)(buff2 + 500), (__m128)xmmword_148490);
  *(__m128 *)(buff2 + 516) = v25;
  return __readgsdword(0x14u);
}
```

### 分析函数 `set_buff3`

基本上和 `set_buff2` 如出一辙，调用了一个新的函数 `get_hash2`，不过和之前求 hash 函数的也是基本一致，这里直接列出 `buff3` 内存布局：

- `buff3[0-3] = 19ca8c + image_base`（根据后面的分析，这个就是按钮 generate3 中调用的函数）
- `buff3[4-67] = hash`
- `buff3[68-83] = 0`
- `buff3[84-595] = const_data ^ 0x0042`
