
```
signed int sub_7860EFB8()

{

  signed int result; // r0@1

  dword_78626068 = (int)"java/lang/String";
  dword_7862606C = (int)"getBytes";
  dword_78626070 = (int)"()[B";
  dword_78626074 = (int)"android/os/Build$VERSION";
  dword_78626078 = (int)"SDK_INT";
  dword_7862607C = (int)"I";.......
}
```

`sub_7860EFB8` 全是赋值操作，跳过。下面是 F5 之后的 `sub_7860E7C4`，主要执行了 `sub_7860E780` 然后打了个 `registerNatives` 相关的 log，相信作安卓逆向的同学看到 `registerNative` 这个都会比较激动吧，跟进 `sub_7860E780`。

```
int __fastcall sub_7860E7C4(int a1)
{
  int v1; // r4@1

  v1 = sub_7860E780(a1, (int)"com/tencent/StubShell/TxAppEntry", (int)&dword_78626004, 5);
  if ( v1 )
    v1 = 1;
  else
    log(3, (int)"SecShell", (int)"registerNatives Fail");
  return v1;
}
```

F5 后的伪代码如下，亮眼的 `0x35C` 刚好是 `registerNatives` 相对于 `JNINativeInterface` 的偏移量，在这下断。

```
int __fastcall sub_7860E780(int a1, int a2, int a3, int a4)
{
  int v4; // r5@1
  int v5; // r4@1
  int v6; // r6@1
  int J_env; // r1@1
  int v8; // r0@2

  v4 = a4;
  v5 = a1;
  v6 = a3;
  J_env = (*(int (**)(void))(*(_DWORD *)a1 + 0x18))();
  if ( J_env )
  {
    v8 = (*(int (__fastcall **)(int, int, int, int))(*(_DWORD *)v5 + 0x35C))(v5, J_env, v6, v4);// registerNatives
    J_env = 1;
    if ( v8 < 0 )
    {
      ((void (__fastcall *)(signed int, const char *, const char *))log)(3, "SecShell", "register nativers error");
      J_env = 0;
    }
  }
  return J_env;
}
```

触发断点之后查看 `R2` 寄存器的值。

[![](https://bbs.kanxue.com/upload/attach/201705/729487_8cimj3n1ofnywr3.png)](https://bbs.kanxue.com/elink@9f1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5@1i4K6u0W2j5Y4m8Q4x3X3g2T1L8r3!0Y4M7%4m8G2N6q4)9J5k6h3y4G2L8g2)9J5c8W2)9J5k6r3p5I4h3f1^5I4j5@1N6i4L8r3!0g2i4K6u0r3g2#2u0c8h3X3k6%4d9q4S2p5b7@1W2Q4x3V1k6m8b7f1q4m8b7f1q4m8b7f1q4b7y4q4)9J5c8X3A6F1g2p5g2C8f1Y4S2I4c8W2m8C8e0f1x3@1P5h3u0p5x3V1y4K6c8V1#2H3c8Y4b7K6b7Y4S2m8N6V1W2&6k6@1y4x3j5@1u0Q4x3V1k6K6x3e0j5H3x3q4)9J5c8W2y4U0M7X3g2W2L8W2)9J5y4e0u0n7f1$3S2G2N6q4)9J5y4e0u0n7x3U0l9I4y4#2)9J5k6o6l9#2i4K6u0V1x3e0q4Q4x3U0f1J5b7X3q4@1i4K6t1#2x3V1t1I4y4g2)9J5k6e0f1%4i4K6u0W2x3K6S2Q4x3X3g2H3L8X3M7).

可以看到这里的 `RegisterNative` 是用来注册 `native load(Landroid/content/Context;)V`，函数指针则是 `0x78615839`，至于为什么是这样可以参考我[这一篇文章](https://bbs.kanxue.com/elink@40eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8Q4x3X3c8J5k6i4k6Q4x3X3g2T1L8r3!0Y4M7%4m8G2N6q4)9J5k6h3S2C8i4K6u0r3x3U0l9I4y4W2)9J5c8U0p5J5i4K6u0r3j5h3&6V1k6i4k6Q4x3X3c8S2L8X3q4D9P5i4y4A6M7#2)9J5k6o6c8Q4x3X3g2Z5N6r3#2D9)。

接着跳入 `0x78615839`，这里我直接贴 F5 + 重命名函数之后的伪代码了，如图。

[![](https://bbs.kanxue.com/upload/attach/201705/729487_8catlkswo9gx56f.png)](https://bbs.kanxue.com/elink@4a6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5J5i4K6u0W2j5Y4m8Q4x3X3g2T1L8r3!0Y4M7%4m8G2N6q4)9J5k6h3y4G2L8g2)9J5c8W2)9J5k6r3c8f1c8W2y4G2K9i4W2j5j5f1!0K6i4K6u0r3g2#2u0c8j5V1W2p5P5V1V1%4N6f1W2Q4x3V1k6m8b7f1q4m8b7f1q4m8b7f1q4c8c8g2)9J5c8Y4q4K6N6p5!0p5x3g2y4a6M7h3c8Y4z5p5S2r3K9#2y4S2K9p5W2p5L8o6N6o6x3V1R3^5L8q4u0D9f1Y4k6o6b7f1y4x3j5@1u0Q4x3V1k6K6x3e0j5H3x3q4)9J5c8W2y4U0M7X3g2W2L8W2)9J5y4e0u0n7f1$3S2G2N6q4)9J5y4e0u0n7x3U0l9I4y4#2)9J5k6o6l9#2i4K6u0V1x3e0q4Q4x3U0f1J5b7X3q4@1i4K6t1#2x3V1t1I4y4W2)9J5k6e0l9#2i4K6u0W2x3e0y4Q4x3X3g2H3L8X3M7).

可以看到先检查了 sdk 版本，dalvik Or art，是否存在 `zjdroid`，然后在分别进行了两个操作，因为这里我用的是 dalvik，所以跟进 `sub_78614CD0`。
