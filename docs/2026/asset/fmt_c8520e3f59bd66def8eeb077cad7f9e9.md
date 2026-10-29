
# 入门级 So静态分析 CTF 之 easy-so write up

## 题目要求

> 拿到 flag 值

## 反编译 APK

APK 用 `jadx` 反编译后如图：

![图片描述](https://bbs.kanxue.com/upload/attach/202008/883944_JHPGQFYM8VCGDY7.png)

查看判断函数`cyberpeace.CheckString`代码位置

![图片描述](https://bbs.kanxue.com/upload/attach/202008/883944_2PBFVMVK7AP93X8.png)

该题重点不在 `Java` 层，只是调用了 `so` 计算下结果，然后在 `so` 中进行比对，成功核对 `flag` 则返回 `1`，所以我们只需要研究 `so` 层即可。

## IDA 查看 So

![图片描述](https://bbs.kanxue.com/upload/attach/202008/883944_4YWYXGQUFKGPVFJ.png)

这里用的是静态注册 `Java_com_testjava_jack_pingan2_cyberpeace_CheckString`

![图片描述](https://bbs.kanxue.com/upload/attach/202008/883944_4RMDWSK8V3ZGND6.png)

从末尾往上找发现，最终进行了**字符串比较**，下面是 IDA 反编译出来的伪代码（arm64）：

```c
bool __fastcall Java_com_testjava_jack_pingan2_cyberpeace_CheckString(__int64 a1)
{
  __int64 v1; // x0
  __int64 v2; // x20
  __int64 v3; // x0
  int v4; // w21
  unsigned __int64 v5; // x22
  __int64 v6; // x0
  _BYTE *v7; // x19
  unsigned __int64 v8; // x2
  unsigned __int64 v9; // x20
  _BYTE *v10; // x8
  char v11; // w9
  char v12; // w10
  char v13; // w9
  __int64 v14; // x20
  _BYTE *v15; // x8
  char v16; // w10
  unsigned __int64 v17; // x0
  unsigned __int64 v18; // x8

  v1 = (*(*a1 + 1352LL))();
  v2 = v1;
  v3 = strlen(v1);
  v4 = v3;
  v5 = ((v3 << 32) + 0x100000000LL) >> 32;
  v6 = malloc(v5);
  v7 = v6;
  if ( v5 > v4 )
    v8 = v5 - v4;
  else
    v8 = 0LL;
  memset(v6 + v4, 0LL, v8);
  memcpy(v7, v2, v4);
  if ( strlen(v7) >= 2 )
  {
    v9 = 0LL;
    do
    {
      v10 = &v7[v9];
      v11 = v7[v9 + 16];
      v12 = v7[v9++];
      *v10 = v11;
      v10[16] = v12;
    }
    while ( strlen(v7) >> 1 > v9 );
  }
  if ( *v7 )
  {
    v13 = v7[1];
    v7[1] = *v7;
    *v7 = v13;
    if ( strlen(v7) >= 3 )
    {
      v14 = 0LL;
      do
      {
        v15 = &v7[v14];
        v16 = v7[v14 + 2];
        v15[2] = v7[v14 + 3];
        v15[3] = v16;
        v17 = strlen(v7);
        v18 = v14 + 4;
        v14 += 2LL;
      }
      while ( v17 > v18 );
    }
  }
  return strcmp(v7, "f72c5a36569418a20907b55be5bf95ad") == 0;
}
```

因为最终是用`v7`来做字符串比较，所以我们分析`v7`是怎么来的即可，从下往上看：

```c
if ( *v7 ) // v7 如果存在进入体内
{
  v13 = v7[1];
  v7[1] = *v7;
  *v7 = v13;
  if ( strlen(v7) >= 3 )
  {
    v14 = 0LL;
    do
    {
      v15 = &v7[v14];
      v16 = v7[v14 + 2];
      v15[2] = v7[v14 + 3];
      v15[3] = v16;
      v17 = strlen(v7);
      v18 = v14 + 4;
      v14 += 2LL;
    }
    while ( v17 > v18 );
  }
}
```

这块是一个位符**两两交换的操作**，比如`123456`交换后就是`214365`，我们把`f72c5a36569418a20907b55be5bf95ad`两两交换回去得到`7fc2a5636549812a90705bb55efb59da`。

再往上看：

```c
memset(v6 + v4, 0LL, v8);
memcpy(v7, v2, v4);
if ( strlen(v7) >= 2 )
{
  v9 = 0LL;
  do
  {
    v10 = &v7[v9];
    v11 = v7[v9 + 16];
    v12 = v7[v9++];
    *v10 = v11;
    v10[16] = v12;
  }
  while ( strlen(v7) >> 1 > v9 );
}
```

这是一个腰斩交换，相当于`123456`换成了`456123`。

我们把上一步得到的`7fc2a5636549812a90705bb55efb59da`还回去得到结果 `flag`：

`90705bb55efb59da7fc2a5636549812a`

## 总结

这是 CTF 上一个入门 `so` 题目，很适合新手上手，感谢出题人。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [456c1dab04b24036ba1d6e32a08dc882.apk](javascript:void(0);)

  （1.49MB，70次下载）
