
# MD5 值

1. `00577F1E |. A5` 指令：`movs dword ptr es:[edi], dword ptr [esi]`
   - `ds:[esi]` = `stack [0012F288]` = `A322009A`
   - `es:[edi]` = `stack [0012F244]` = `00000000`

2. `00577F1F |. A5` 指令：`movs dword ptr es:[edi], dword ptr [esi]`
   - `ds:[esi]` = `stack [0012F28C]` = `3800B6F8`
   - `es:[edi]` = `stack [0012F248]` = `00000000`

3. `00577F20 |. A5` 指令：`movs dword ptr es:[edi], dword ptr [esi]`
   - `ds:[esi]` = `stack [0012F290]` = `DC973D09`
   - `es:[edi]` = `stack [0012F24C]` = `00000000`

4. `00577F21 |. A5` 指令：`movs dword ptr es:[edi], dword ptr [esi]`
   - `ds:[esi]` = `stack [0012F294]` = `3EB52A2D`
   - `es:[edi]` = `stack [0012F250]` = `00000000`

## 运算结果

取 `B` 后与 MD5 运算转换的结果：
`A3 22 00 9A` `38 00 B6 F8` `DC 97 3D 09` `3E B5 2A 2D`

一系列的运算后得到真注册码：
`9a 00 22 a3` `f8 b6 00 38` `09 3d 97 dc` `2d 2a b5 3e`

## 标准 MD5 初始化的四个常数

```c
state[0] = 0x67452301;
state[1] = 0xefcdab89;
state[2] = 0x98badcfe;
state[3] = 0x10325476;
```

## 版权声明

本文纯属技术交流，转载请注明作者信息并保持文章的完整，谢谢！

`4dd466d0e1e0cc123afbe504da2462ca`
