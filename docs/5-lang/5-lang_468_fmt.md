
# PowerDirector 1.00.06.9

> 原文：[`www.pediy.com/kssd/pediy04/kanxue386.htm`](https://www.pediy.com/kssd/pediy04/kanxue386.htm)

## PowerDirector 1.00.06.9 破解

『GUEOR(S.M)破解』

欢迎访问我的网页: `sm.longcity.net`

qq: `5537188`

看了高人 `Passion` 的一些文章后,才对破解 `Install sheild` 产生了兴趣.谢谢 `Passion` 的破文,愿 `Passion` 能多写好的破文.

用 `isd` 将 `setup.inx` 反编译.如下:

```
/* 00006383: 0021 */    function_4(g_str28);

/* 0000638C: 0006 */    g_number44 = LAST_RESULT;

/* 00006396: 000D */    g_number56 = g_number44 == 0;

/* 000063A5: 0004 */    if(! g_number56) goto label_16;->不等于 0 跳

/* 000063B1: 0021 */    function_343("INVALIDCDKEY");
```

`return` 的值不能为 `0`,进入 `function_4`.可喜的是比较在脚本中进行,减小了难度,也不用跟踪了.^_^

```
function NUMBER function_4(s0)

    STRING s1;

    STRING s2;

    STRING s3;

    STRING s4;

    STRING s5;

    STRING s7;

    STRING s8;

    NUMBER n1;

    NUMBER n2;

    NUMBER n6;

begin

/* 0000ABC5: 0022 */    // -- Start Function Code -- //

/* 0000ABCE: 0021 */    function_537(s0); ->求注册码长度

/* 0000ABD7: 0006 */    n6 = LAST_RESULT;

/* 0000ABE1: 000E */    n6 = n6 != 16;                                      ->n6=16(要 16 位)

/* 0000ABF0: 0004 */    if(! n6) goto label_146;    ->相等就跳

/* 0000ABFC: 0027 */    // -- Start Return Code -- //

/* 0000AC00: 0023 */    return 0;

label_146:

/* 0000AC0B: 0029 */    StrSub(s1, s0, 0, 2); ->s1 为注册码前 2 位.

/* 0000AC1F: 0014 */    s8 = g_str25 ^ "CUSTOM.INI";

/* 0000AC36: 0021 */    function_384(s8, "PowerDirector", "KEY", g_str28);

/* 0000AC58: 0006 */    n6 = LAST_RESULT;

/* 0000AC62: 000D */    n6 = n6 == 0;

/* 0000AC71: 0004 */    if(! n6) goto label_147;

/* 0000AC7D: 0006 */    s5 = g_str28;

/* 0000AC87: 002B */    StrCompare(s1, s5);

/* 0000AC91: 0006 */    n6 = LAST_RESULT;

/* 0000AC9B: 000E */    n6 = n6 != 0;

/* 0000ACAA: 0004 */    if(! n6) goto label_147;

/* 0000ACB6: 0027 */    // -- Start Return Code -- //

/* 0000ACBA: 0023 */    return 0;

label_147:

/* 0000ACC5: 002B */    StrCompare(s1, "PR");          ->s1="PR",即注册码前 2 位为 PR

/* 0000ACD1: 0006 */    n6 = LAST_RESULT;

/* 0000ACDB: 000E */    n6 = n6 != 0;

/* 0000ACEA: 0004 */    if(! n6) goto label_148;    ->相等就跳

/* 0000ACF6: 0027 */    // -- Start Return Code -- //

/* 0000ACFA: 0023 */    return 0;

label_148:

/* 0000AD05: 0029 */    StrSub(s3, s0, 2, 1);        ->s3 为注册码第 3 位

/* 0000AD19: 0029 */    StrSub(s4, s0, 13, 3);        ->s4 为注册码第 14,15,16 位

/* 0000AD2D: 0007 */    s7 = s3 + s4;                  ->s7 为"(3+14+15+16)"

/* 0000AD3A: 002C */    StrToNum(n1, s7);              ->字符串转换成数字 n1

/* 0000AD44: 0006 */    n6 = LAST_RESULT;

/* 0000AD4E: 0009 */    n6 = n6 < 0;

/* 0000AD5D: 0004 */    if(! n6) goto label_149;

/* 0000AD69: 0027 */    // -- Start Return Code -- //

/* 0000AD6D: 0023 */    return 0;

label_149:

/* 0000AD78: 0029 */    StrSub(s2, s0, 3, 10);         -> s2 为注册码第 4 位~第 13 位(10 个)

/* 0000AD8C: 002B */    StrCompare(s1, "PR");

/* 0000AD98: 0006 */    n6 = LAST_RESULT;

/* 0000ADA2: 000D */    n6 = n6 == 0;

/* 0000ADB1: 0004 */    if(! n6) goto label_150;

/* 0000ADBD: 0021 */    function_5(s2, 32);

/* 0000ADCB: 0006 */    n2 = LAST_RESULT;

label_150:

/* 0000ADD7: 000E */    n6 = n2 != n1;                 ->n1 即(3+14+15+16)

/* 0000ADE4: 0004 */    if(! n6) goto label_151;

/* 0000ADF0: 0027 */    // -- Start Return Code -- //

/* 0000ADF4: 0023 */    return 0;
```

进入 `function_5`:

```
〖function NUMBER function_5(s0, n0)                  --形参 s0=s2  n0=32

    STRING s1;

    STRING s2;

    STRING s3;

    NUMBER n1;

    NUMBER n2;

    NUMBER n3;

    NUMBER n4;

    NUMBER n5;

    NUMBER n6;

    NUMBER n7;

begin

/* 0000AE1A: 0022 */    // -- Start Function Code -- //

/* 0000AE23: 0006 */    n5 = 1;

/* 0000AE2F: 0021 */    function_6(s0, s1);  =>进入 function_6( )

/* 0000AE3B: 0006 */    n6 = LAST_RESULT;    -->return n6=1 s1="00000000000"

/* 0000AE45: 000D */    n7 = n6 == 0;

/* 0000AE54: 0004 */    if(! n7) goto label_153;    jump

/* 0000AE60: 0027 */    // -- Start Return Code -- //

/* 0000AE64: 0023 */    return n5;                ( n5=1) 错!
```

进入 `function_6( )`:

```
【function NUMBER function_6(s0, s1)          ->s0=s2

    NUMBER n0;

    NUMBER n1;

    NUMBER n2;

    NUMBER n3;

    NUMBER n4;

begin

/* 0000B003: 0022 */    // -- Start Function Code -- //

/* 0000B00C: 0021 */    function_537(s0);                  -->求 s0 的长度

/* 0000B015: 0006 */    n1 = LAST_RESULT;                  -->n1=10

/* 0000B01F: 0009 */    n2 = n1 < 0;

/* 0000B02E: 0004 */    if(! n2) goto label_160;

/* 0000B03A: 0027 */    // -- Start Return Code -- //

/* 0000B03E: 0023 */    return 0;

label_160:

/* 0000B049: 0006 */    n0 = 0;

/* 0000B055: 000F */    n2 = n1 - 1;              -->n2=9

label_161:

/* 0000B066: 000B */    n3 = n0 <= n2;                    -->循环 10 次

/* 0000B073: 0004 */    if(! n3) goto label_164;

/* 0000B07F: 001E */    n3 = s0[n0];        \\

/* 0000B08C: 0009 */    n3 = n3 < 48;       |

/* 0000B09B: 001E */    n4 = s0[n0];         |   ->判断 10 个字符在 0~9 之间

/* 0000B0A8: 000A */    n4 = n4 > 57;     /

/* 0000B0B7: 0018 */    n3 = n3 || n4;

/* 0000B0C4: 0004 */    if(! n3) goto label_162;  ->成立其一就不跳,不跳回比较简单.

                                                  s1 会为"00000000000",后面转化为数字

                                                  成为 0.那么填字母即可.

/* 0000B0D0: 001D */    s1[n0] = 48;   s1=0           ->s1="00000000000"

/* 0000B0DF: 0005 */    goto label_163;

label_162:

/* 0000B0EA: 001E */    n3 = s0[n0];            ->填数字的运算.

/* 0000B0F7: 001D */    s1[n0] = n3;

label_163:

/* 0000B106: 0007 */    n0 = n0 + 1;        ->n0++

/* 0000B115: 0005 */    goto label_161;

label_164:

/* 0000B120: 0027 */    // -- Start Return Code -- //

/* 0000B124: 0023 */    return 1;

/* 0000B12D: 0026 */    // -- Create Local Variables -- //

end; 】
```

`return` 全局变量 `s1="00000000000"`.

```
label_153:

/* 0000AE6D: 0006 */    n1 = 1;

label_154:

/* 0000AE7B: 000B */    n7 = n1 <= 2;

/* 0000AE8A: 0004 */    if(! n7) goto label_156;

/* 0000AE96: 0021 */    function_7(s1, s2, s3);      =>进入 function_7( )

/* 0000AEA5: 0006 */    n6 = LAST_RESULT;  =>return s2="000000" s3="000000"

/* 0000AEAF: 000D */    n7 = n6 == 0;

/* 0000AEBE: 0004 */    if(! n7) goto label_155;

/* 0000AECA: 0027 */    // -- Start Return Code -- //

/* 0000AECE: 0023 */    return 0;
```

\=>进入 `function_7( )`

```
【function NUMBER function_7(s0, s1, s2)  --形参 s0="000000000"

    NUMBER n0;

    NUMBER n1;

    NUMBER n2;

    NUMBER n3;

    NUMBER n4;

begin

/* 0000B13B: 0022 */    // -- Start Function Code -- //

/* 0000B144: 0021 */    function_537(s0);        ->求 s0 的长度

/* 0000B14D: 0006 */    n2 = LAST_RESULT;             --n2=10

/* 0000B157: 0009 */    n3 = n2 < 0;

/* 0000B166: 0004 */    if(! n3) goto label_166;

/* 0000B172: 0027 */    // -- Start Return Code -- //

/* 0000B176: 0023 */    return 0;

label_166:

/* 0000B181: 0006 */    n1 = 0;

/* 0000B18D: 0006 */    n0 = 0;

/* 0000B199: 0011 */    n3 = n2 / 2;
```



```
/\* 0000B1A8: 000F \*/    n3 = n3 - 1;           ->n3=4   
label_167:   
/\* 0000B1B9: 000B \*/    n4 = n0 <= n3;       -> 循环 5 次   
/\* 0000B1C6: 0004 \*/    if(! n4) goto label_168;   
/\* 0000B1D2: 001E \*/    n4 = s0[n1];           \   
/\* 0000B1DF: 001D \*/    s1[n0] = n4;           |s1="00000"   
/\* 0000B1EC: 0007 \*/    n1 = n1 + 1;           |                   ->s1,s2 分别赋值   
/\* 0000B1FB: 001E \*/    n4 = s0[n1];           |   
/\* 0000B208: 001D \*/    s2[n0] = n4;         / s2="00000"   
/\* 0000B215: 0007 \*/    n1 = n1 + 1;   
/\* 0000B224: 0007 \*/    n0 = n0 + 1;   
/\* 0000B233: 0005 \*/    goto label_167;   
label_168:   
/\* 0000B23E: 0027 \*/    // -- Start Return Code -- //   
/\* 0000B242: 0023 \*/    return 1;   
/\* 0000B24B: 0026 \*/    // -- Create Local Variables -- //   
end;】   
return s2="000000" s3="000000"   
label_155:   
/\* 0000AED9: 0007 \*/    s1 = s2 + s3;            ->  s1="000000000000"   
/\* 0000AEE6: 0007 \*/    n1 = n1 + 1;               -> n1=2   
/\* 0000AEF5: 0005 \*/    goto label_154;   
label_156:   
/\* 0000AF00: 002C \*/    StrToNum(n2, s2);        ->n2=0   
/\* 0000AF0A: 0006 \*/    n7 = LAST_RESULT;   
/\* 0000AF14: 0009 \*/    n7 = n7 < 0;   
/\* 0000AF23: 0004 \*/    if(! n7) goto label_157;   
/\* 0000AF2F: 0027 \*/    // -- Start Return Code -- //   
/\* 0000AF33: 0023 \*/    return n5;    n5=1   
label_157:   
/\* 0000AF3C: 002C \*/    StrToNum(n3, s3);    -> n3=0   
/\* 0000AF46: 0006 \*/    n7 = LAST_RESULT;   
/\* 0000AF50: 0009 \*/    n7 = n7 < 0;   
/\* 0000AF5F: 0004 \*/    if(! n7) goto label_158;   
/\* 0000AF6B: 0027 \*/    // -- Start Return Code -- //   
/\* 0000AF6F: 0023 \*/    return n5;    n5=1   
label_158:   
/\* 0000AF78: 0008 \*/    n2 = n2 % 1000;      ->n2=0  |              
/\* 0000AF87: 0008 \*/    n3 = n3 % 10000;    ->n3=0  |简单吧!   
/\* 0000AF96: 0007 \*/    n4 = n0 + 924;        ->n4=956   
/\* 0000AFA5: 0007 \*/    n2 = n2 + n4;          ->n2=956   
/\* 0000AFB2: 0007 \*/    n3 = n3 + n4;          ->n3=956   
/\* 0000AFBF: 0010 \*/    n5 = n2 * n3;          ->n5=913936   
/\* 0000AFCC: 0008 \*/    n5 = n5 % 9999;      ->n5=4027   
/\* 0000AFDB: 0007 \*/    n5 = n5 + 1;           ->n5=4028   
/\* 0000AFEA: 0027 \*/    // -- Start Return Code -- //   
/\* 0000AFEE: 0023 \*/    return n5;   
/\* 0000AFF5: 0026 \*/    // -- Create Local Variables -- //   
end;〗   
label_151:   
/\* 0000ADFF: 0027 \*/    // -- Start Return Code -- //   
/\* 0000AE03: 0023 \*/    return 1;   
/\* 0000AE0C: 0026 \*/    // -- Create Local Variables -- //   
end;   
```

根据 `0000B066: 000B` 的循环，注册码第 4 位~第 13 位可填除数字外的任意字符。

即 `serial number:PR4XXXXXXXXXX028`

不过只能运行在 `1024 * 768` 下，靠。只有卸了。

有空别忘了访问我的网页。呵呵呵呵呵呵呵呵.... `4dd466d0e1e0cc123afbe504da2462ca`
