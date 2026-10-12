
# 流光 2001

> 原文：[`www.pediy.com/kssd/pediy03/forum789.htm`](https://www.pediy.com/kssd/pediy03/forum789.htm)

去掉`流光 2001 Gold Edition (Preview 1 Build 2262 Patch 2)`中文版`For Win98/ME`部分功能限制

- 查找方法：用`w32dsm`反汇编，查找出错信息所对应的地址，用`s-ice loader`加载后，稍加调试即可。

### 1. 免注册

```
407e53 75(cc)0a->eb0a jnz 407e5f ->jmp 407e5f
```

### 2. 去时限

现在改成只有上线验证一种方式了

```
46cafe 6aff ->c3 push ffffffff ->ret
```

### 3. 去 ip 限制

```
45cf8d 746d ->eb6d jz 45cffc ->jmp 45cffc
```

### 4. 去文件代码自校验

```
406992 7416 ->eb16 jz 4069aa ->jmp 4069aa
4069c1 740b ->eb0b jz 4069ce ->jmp 4069ce
```

修改方法：用`peditor`的`flc`功能转换`virtual address`为`offset`，然后用`winhex`搞定。完活:)

`cracked by zombieys`

`my oicq is 1789655`

`my hp is zombieys.126.com`

`thanks for your supports 4dd466d0e1e0cc123afbe504da2462ca`
