
# [原创]CTF2017 第十二题 kxuectf-apk 解题报告

## dex 部分

放入 JEB，截两段重要源码。

```
this.button.setOnClickListener(new View$OnClickListener() {

            public void onClick(View v) {

			    // 点击事件，组装 ab.a(下面的算法)、ba.a(一个替换算法)、ca.a(base64)

				// 传入 jni 的 check 函数

                String v2 = uv.this.editText.getText().toString().trim();

                if(ua.check(ab.a(v2) + ba.a(v2) + ca.a(v2)) == 1) {

                    bc.showresult(uv.this.tv, true);

                }

                else {

                    bc.showresult(uv.this.tv, false);

                }

            }

        });

		

// 上面的 ab.a 算法

public static String encrypt(String input) {

        String v3 = "";

        int v2;

        for(v2 = 0; v2 < input.length(); ++v2) {

            int v0 = input.charAt(v2);

            if(v0 < 97 || v0 > 109) {

                if(v0 >= 65 && v0 <= 77) {

                    v3 = String.valueOf(v3) + (((char)(v0 + 13)));

                    goto label_21;

                }

                if(v0 >= 110 && v0 <= 122) {

                    v3 = String.valueOf(v3) + (((char)(v0 - 13)));

                    goto label_21;

                }

                if(v0 >= 78 && v0 <= 90) {

                    v3 = String.valueOf(v3) + (((char)(v0 - 13)));

                    goto label_21;

                }

                if(v0 >= 48 && v0 <= 57) {

                    v3 = String.valueOf(v3) + (((char)(v0 ^ 7)));

                    goto label_21;

                }

                v3 = String.valueOf(v3) + (((char)(v0 ^ 6)));

            }

            else {

                v3 = String.valueOf(v3) + (((char)(v0 + 13)));

            }

        label_21:

        }

        return v3;

    }
```

这个 `ab.a` 算法其实有多解，其实本题最后应该有 4 个解：

> `K^ -> X`
>
> `k~ -> x`
>
> `@S -> F`
>
> `M\ -> Z`
>
> `` `s -> f ``
>
> `m| -> z`
>
> `L_ -> Y`

附 Python 编解码：

```python
from collections import defaultdict

def aba_(d):

    if 65 <= d <= 77:

        return chr(d + 13)

    elif 110 <= d <= 122:

        return chr(d - 13)

    elif 78 <= d <= 90:

        return chr(d - 13)

    elif 48 <= d <= 57:

        return chr(d ^ 7)

    elif 97 <= d <= 109:

        return chr(d + 13)

    return chr(d ^ 6)

def aba_enc(s):

    r = ""

    for c in s:

        r += aba_(ord(c))

    return r

def aba_dec(s):

    dt = defaultdict(str)

    for i in xrange(0x20, 0x7F):

        dt[aba_(i)] += chr(i)

    r = ""

    for c in s:

        t = dt[c]

        if len(t) == 1:

            r += t

        else:

            r += '[' + t + ']'

    return r
```

## jni 部分

jni 有 3 个平台可供选择分析，我选择了 IDA 相容性较好的 `armeabi-v7a` 版本。

### 反反调试

jni 部分有数种反调试手段，但是都可以用比较原始的方法去掉：

- 可以注入一个 `so`，修改相关函数的返回。
- 我是直接使用 `keypatch` 修改 `so`，替换原有 APK 安装。

### 主程序逻辑

1. 截取 dex 部分输入的组合字符前 36 字节。
2. 关于输入长度，程序并没有说明，只有大于 120 会失败，其实应该改成不等于 120 失败，就可以不用提示也能做出来。因为没有限定长度，只能靠猜测 36 字节就是 `aba` 加密结果才能作出该题，因为只有 `aba` 是 36 字节时，整体组合字符才会是 120。
3. 在 36 字节最后 padding `4,4,4,4`，组成 40 字节，使用 `FDB4685408CD564E` 作为密钥，用变种的 DES 算法加密后，得到 `desres`（40 字节）。
4. 将 `654832EFBACD564E0F9B1D27` 组合 `desres[32:36]` 成 16 字节密钥，用 RC6 算法加密 `desres[:32]`，得到 `rc6res`（32 字节），与程序内置的结果进行对比。

### 解法步骤

需要使用调试来判断 2 个变种加密算法的修改细节，本文最后一部分会附上 diff。因为题目进行中公开了用户输入的前 8 位 `kxuectf{` 和末位 `}`，本文将在给出提示的前提下提供解法步骤：

1. `aba_enc("kxuectf{")` 得到 `786b68727067737d`。
2. DES 加密后得到 `4cd9a3e6edfed105`。
3. 枚举 `desres[32:36]` 部分，组合 RC6 密钥，解密内置结果的前 16 字节，得到 `desres[:16]`。
4. 对比 `desres[:8]` 与 `4cd9a3e6edfed105`，相同则找到正确的 `desres[32:36]` `136a7e1f`，以及 `desres[:16]`。
5. 用 RC6 解密内置结果的后 16 字节，解出剩下的 `desres[16:32]`；用 DES 解密 `desres[:32]`，解出 `abares[:32]`。
6. 在可见字符范围内（`aba` 输出可见字符），枚举 `abares[32:36]`，padding 上 `[4,4,4,4]` 组合成 8 字节，用变种 DES 加密。
7. 对比加密结果的前 4 字节与 `136a7e1f`，相同则找到正确的 `abares[32:36]` `613e367b`，组合成 `abares`（36 字节）。
8. `aba_dec(abares)` 得到正确解，应该有 4 组，选了一组没有奇怪标点的提交。

## 算法源码

### DES

`set_key` 位移部分不知道有 bug 还是啥，和标准不大一样，[原型代码](<https://bbs.kanxue.com/elink@5f0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8A6j5$3u0Q4x3X3g2S2j5#2)9J5k6h3y4F1i4K6u0r3N6r3!0G2L8s2y4Q4x3V1k6U0M7s2m8Q4x3V1k6E0P5h3y4G2k6r3g2K6i4K6u0r3c8p5g2e0i4K6u0r3c8r3g2K6i4K6u0W2j5%4m8H3>)。

```diff
--- Des.cpp	2017-06-22 21:29:47.000000000 +0800

+++ Des2.cpp	2017-06-25 12:19:36.000000000 +0800

@@ -6,7 +6,7 @@

 */

 //////////////////////////////////////////////////////////////////////////

 

-#include "memory.h"

+#include <string.h>

 #include "Des.h"

 

 //////////////////////////////////////////////////////////////////////////

@@ -163,7 +163,13 @@

             Xor(Ri, Li, 32);

             memcpy(Li, tmp, 32);

         }

+        memcpy(tmp, M, 32);

+        memcpy(M, M + 32, 32);

+        memcpy(M + 32, tmp, 32);

     }else{

+        memcpy(tmp, M, 32);

+        memcpy(M, M + 32, 32);

+        memcpy(M + 32, tmp, 32);

         for(int i=15; i>=0; --i) {

             memcpy(tmp, Li, 32);

             F_func(Li, (*pSubKey)[i]);

@@ -179,10 +185,23 @@

     static bool K[64], *KL=&K[0], *KR=&K[28];

     ByteToBit(K, Key, 64);

     Transform(K, K, PC1_Table, 56);

+    static bool KK[120], *K1=&KK[0], *K2=&KK[60];

+    memcpy(K1, K, 28); memcpy(K1 + 28, K, 28);

+    memcpy(K2, K + 28, 28); memcpy(K2 + 28, K + 28, 28);

+

+    int loop = 0;

     for(int i=0; i<16; ++i) {

-        RotateL(KL, 28, LOOP_Table[i]);

-        RotateL(KR, 28, LOOP_Table[i]);

-        Transform((*pSubKey)[i], K, PC2_Table, 48);

+        loop += LOOP_Table[i];

+        for (int j=0; j<48; ++j) {

+            int k = PC2_Table[j];

+            bool r;

+            if (k < 28) {

+                r = K1[loop + k - 1];

+            } else {

+                r = K1[32 + loop + k - 1];

+            }

+            SubKey[0][i][j] = r;

+        }

     }

 }

 void F_func(bool In[32], const bool Ki[48])

@@ -220,14 +239,18 @@

 }

 void ByteToBit(bool *Out, const char *In, int bits)

 {

+    int t = bits - 1;

+    if (t > 7) t = 7;

     for(int i=0; i<bits; ++i)

-        Out[i] = (In[i>>3]>>(i&7)) & 1;

+        Out[i] = (In[i>>3]>>(t-i%8))&1;

 }

 void BitToByte(char *Out, const bool *In, int bits)

 {

+    int t = bits - 1;

+    if (t > 7) t = 7;

     memset(Out, 0, bits>>3);

     for(int i=0; i<bits; ++i)

-        Out[i>>3] |= In[i]<<(i&7);

+        Out[i>>3] |= In[i]<<(t-i%8);

 }

 //////////////////////////////////////////////////////////////////////////

 // Code ends at Line 231
```

### RC6

RC6 部分赛后 poyoten 发现应该是这个[实现](<https://bbs.kanxue.com/elink@463K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3K9r3q4J5L8%4u0G2k6r3q4Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0f1I4z5o6b7&6x3U0j5J5>)，我为了用 `go` 快速枚举，我是在 [`go-rc6`](<https://bbs.kanxue.com/elink@726K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6%4u0&6M7$3E0A6i4K6u0r3k6$3!0Q4x3X3c8J5j5K6j5>) 这个实现上修改的。
