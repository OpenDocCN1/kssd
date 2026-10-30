
# [原创]2017腾讯游戏安全技术竞赛 Round 2 第一题详细题解

## 前言

从第一题题目来看，感觉这次的题目都出得很灵活，主要考的是密码学这一块的东西。这道题目也不例外，由于不是科班出身，根本没有接触过密码学，那只能现学现卖，其中踩了不少坑，但是不要怕，我们可以慢慢爬出来再继续踩。初一看这道题后面的“双重保镖”很是霸气，很容易被吓跑，但是仔细认真做了之后会发现，Oh，hash碰撞也不是那么难的。好了，废话扯多了估计要被拉黑，我选择的依旧是安卓的题目，下面进入正题，我将此题分成两大部分，主要阐述这两大部分。

## 0x1 拉开帷幕

官方具体表演如下：

```
 v17 = 0;

  v15 = "aLSG";

  v16 = "71b";

  v19 = 0;

  v20 = 0;

  v21 = 0;

  v22 = 0;

  v18 = getpid();

  v0 = dlopen("/data/local/tmp/libZapus.so", 0);

  v1 = v0;

  if ( !v0 )

    goto LABEL_18;

  v2 = (void (__fastcall *)(int *))dlsym(v0, "zapus_get");

  if ( !v2 )

    goto LABEL_18;

  v2(&v19);

  dlclose(v1);

  ((void (__fastcall *)(int *))sub_F90)(&v19);

  v3 = memcmp(&v19, &v15, 0x10u);

  if ( v3 )

    goto LABEL_18;

  v4 = fopen("/data/local/tmp/libZapus.so", "rb");

  v5 = v4;

  if ( !v4 )

    goto LABEL_18;

  fseek(v4, 0, 2);

  v6 = ftell(v5);

  fseek(v5, 0, 0);

  v7 = malloc(v6);

  v8 = v7;

  if ( !v7 )
```

1. 判断指定路径有没有so，没有或者是打开失败就跪。
2. 找`zapus_get`符号，符号表里没有就跪。
3. 有了的话，把该函数初始化的指针带进到图中的`sub_F90`函数，然后开始计算。
4. 小插曲，这里说一下这个题目第一次发的有一个bug，解密后的代码与固定两个字符串的指针进行了比较，导致每次对比的值是变的，我做到这里的时候怀疑了一下人生。what？比较的值还是变的？还不能改程序内存？在心中无限崩溃循环中还好官网出来及时修补了bug。

计算过程如下:

```
	unsigned char input[] = "1234567890abcdef";

	unsigned char tmp;

	for ( j = 0; (unsigned int) j < 0x80; j++ )

	{

		tmp = 0;

		for ( k = 0; (unsigned int) k < 0x80; ++k )

		{

			tmp = ((((signed int)(unsigned char)buffer[16 * j + k / 8] >> k % 8) & ((signed int)*( (unsigned char *) input + k / 8) >> (7 - k % 8) ) ) ^ tmp) & 1;	

		}

		s[j/8] |= tmp << (7 - j % 8);

	}
```

实际上第一步要找出这段算法的逆运算。程序中有一个`buffer`，简算过后如下：
