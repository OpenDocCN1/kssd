
# ADSL拨号计时器v5.28算法分析

【破文标题】ADSL拨号计时器v5.28算法分析  
【软件名称】ADSL拨号计时器v5.28

## 【软件介绍】

1. 为方便建立ADSL宽带连接，集成了PPPoE_98c，无需用户额外下载。
2. 联网登录速度极快，通常情况下无停顿。
3. 计时采取“本次时间”、“累计时间”、“单次计时”、“定点计时”四项。
4. 体积小巧，界面简练，功能实用，运行稳定。
5. 用户设置简单，只需设置包月时长与截止日期两项。
6. 查阅记录只需点击一个菜单按钮，便一目了然。
7. 具有屏蔽42种要求安装插件对话框功能，让你上网更安静、安全。
8. 具有屏蔽弹出广告窗口功能，使浏览网页免受打扰。
9. 机器资源占用少，硬件要求低。
10. 安装时，不写注册表，不扩散安装，所有文件都在一个文件夹里，完全等同于自解压。而且，卸载时不留任何垃圾文件。

## 【软件地址】

[`bccK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2F1j5$3I4W2L8h3g2Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3!0%4j5i4u0@1K9h3y4D9k6g2)9J5k6i4m8Z5M7q4)9K6c8X3W2V1i4K6y4p5z5o6f1`.](<https://bbs.kanxue.com/elink@b69K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2F1j5$3I4W2L8h3g2Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3!0%4j5i4u0@1K9h3y4D9k6g2)9J5k6i4m8Z5M7q4)9K6c8X3W2V1i4K6y4p5z5o6f1.>)

## 【破文作者】

KiLlL[DFCG][FCG]

## 【破解时间】

2005-09-16 17:00

## 【破解声明】

仅为技术交流之用！刚加入FCG，高兴～

## 【破解过程】

看到一篇文章

[`a1aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2@1x3K6y4Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8Y4y4G2k6Y4c8Q4x3V1k6C8k6Y4c8I4i4K6u0r3x3U0l9H3y4e0M7^5x3e0p5K6z5o6b7I4i4K6u0W2K9s2c8E0L8l9`.](<https://bbs.kanxue.com/elink@aceK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2@1x3K6y4Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8Y4y4G2k6Y4c8Q4x3V1k6C8k6Y4c8I4i4K6u0r3x3U0l9H3y4e0M7^5x3e0p5K6z5o6b7I4i4K6u0W2K9s2c8E0L8l9.>)

> “具体怎么使用，那要看你自己的意愿了和需要了。反正我是这样做的，我的软件ADSL拨号计时器只在很早版本上出过注册机，后来的v3.70出过破解补丁――其实只是破掉了启动时提示注册的对话框，实质上根本没破解。用了上述的着法以后，到现在的v5.28版本，再没有过什么破解补丁或注册机。如果现在的v5.28版本谁能破解，将立即公布程序源码。”

说的很是有趣，于是下载看看，由于程序现在最新版7.02已经免费，所以看看也无妨了，结果出乎意料。

程序有壳，aspack，脱后`dede`查看：

```
00489004   /.  55           push ebp                        ;  dede可知这个是注册按钮的事件
00489005   |.  8BEC         mov ebp,esp
00489007   |.  6A 00        push 0
00489009   |.  6A 00        push 0
0048900B   |.  53           push ebx
0048900C   |.  8BD8         mov ebx,eax
0048900E   |.  33C0         xor eax,eax
00489010   |.  55           push ebp
00489011   |.  68 A5904800  push unpacked.004890A5
00489016   |.  64:FF30      push dword ptr fs:[eax]
00489019   |.  64:8920      mov dword ptr fs:[eax],esp
0048901C   |.  8D55 FC      lea edx,dword ptr ss:[ebp-4]
0048901F   |.  8B83 FC02000>mov eax,dword ptr ds:[ebx+2FC]
00489025   |.  E8 EA92FBFF  call unpacked.00442314          ;  读取假码
0048902A   |.  837D FC 00   cmp dword ptr ss:[ebp-4],0      ;  假码是否为空？
0048902E   |.  74 57        je short unpacked.00489087
00489030   |.  8BC3         mov eax,ebx
00489032   |.  E8 59FEFFFF  call unpacked.00488E90          ;  对假码进行运算
00489037   |.  84C0         test al,al                      ;  关键跳转
00489039   |.  74 36        je short unpacked.00489071      ;  注册失败就跳走
0048903B   |.  A1 100D4900  mov eax,dword ptr ds:[490D10]
00489040   |.  E8 E712F8FF  call unpacked.0040A32C
00489045   |.  D805 B490480>fadd dword ptr ds:[4890B4]
0048904B   |.  83C4 F4      add esp,-0C
```

跟进关键算法看看：

```
00488E90   /$  55           push ebp                        ;  关键算法
00488E91   |.  8BEC         mov ebp,esp
00488E93   |.  B9 05000000  mov ecx,5
00488E98   |>  6A 00        /push 0
00488E9A   |.  6A 00        |push 0
00488E9C   |.  49           |dec ecx
00488E9D   |.^ 75 F9        \jnz short unpacked.00488E98
00488E9F   |.  51           push ecx
00488EA0   |.  53           push ebx
00488EA1   |.  56           push esi
00488EA2   |.  8945 FC      mov dword ptr ss:[ebp-4],eax
00488EA5   |.  33C0         xor eax,eax
00488EA7   |.  55           push ebp
00488EA8   |.  68 E08F4800  push unpacked.00488FE0
00488EAD   |.  64:FF30      push dword ptr fs:[eax]
00488EB0   |.  64:8920      mov dword ptr fs:[eax],esp
00488EB3   |.  33F6         xor esi,esi
00488EB5   |.  8D55 F8      lea edx,dword ptr ss:[ebp-8]
00488EB8   |.  8B45 FC      mov eax,dword ptr ss:[ebp-4]
00488EBB   |.  E8 D0FAFFFF  call unpacked.00488990
00488EC0   |.  8B45 F8      mov eax,dword ptr ss:[ebp-8]    ;  硬件号 "7FD9534C"
00488EC3   |.  E8 5CBAF7FF  call unpacked.00404924          ;  获得长度
00488EC8   |.  8BD8         mov ebx,eax                     ;  8
00488ECA   |.  85DB         test ebx,ebx
00488ECC   |.  7E 58        jle short unpacked.00488F26
00488ECE   |.  C745 F4 0100>mov dword ptr ss:[ebp-C],1
00488ED5   |>  8D45 F0      /lea eax,dword ptr ss:[ebp-10]  ;  以长度作为循环终止值
00488ED8   |.  50           |push eax
00488ED9   |.  B9 01000000  |mov ecx,1
00488EDE   |.  8B55 F4      |mov edx,dword ptr ss:[ebp-C]
00488EE1   |.  8B45 F8      |mov eax,dword ptr ss:[ebp-8]
00488EE4   |.  E8 93BCF7FF  |call unpacked.00404B7C         ;  mid(code,i,1)
00488EE9   |.  8B45 F0      |mov eax,dword ptr ss:[ebp-10]
00488EEC   |.  E8 2BBCF7FF  |call unpacked.00404B1C
00488EF1   |.  8A00         |mov al,byte ptr ds:[eax]       ;  取得ascii
00488EF3   |.  25 FF000000  |and eax,0FF
00488EF8   |.  03F0         |add esi,eax                    ;  加到esi
00488EFA   |.  8D55 E8      |lea edx,dword ptr ss:[ebp-18]
00488EFD   |.  69C6 4132000>|imul eax,esi,3241              ;  eax=esi*3241
00488F03   |.  E8 24FDF7FF  |call unpacked.00408C2C         ;  转成十进制
00488F08   |.  8B45 E8      |mov eax,dword ptr ss:[ebp-18]  ;  7->707575
00488F0B   |.  8D55 EC      |lea edx,dword ptr ss:[ebp-14]
00488F0E   |.  E8 C5260000  |call unpacked.0048B5D8         ;  较复杂的编码
00488F13   |.  8B55 EC      |mov edx,dword ptr ss:[ebp-14]  ;  707575 －》"2060320645206502065020645")
00488F16   |.  B8 100D4900  |mov eax,unpacked.00490D10
00488F1B   |.  E8 A0B7F7FF  |call unpacked.004046C0
00488F20   |.  FF45 F4      |inc dword ptr ss:[ebp-C]
00488F23   |.  4B           |dec ebx
00488F24   |.^ 75 AF        \jnz short unpacked.00488ED5
00488F26   |>  8D55 DC      lea edx,dword ptr ss:[ebp-24]
00488F29   |.  8B45 FC      mov eax,dword ptr ss:[ebp-4]
00488F2C   |.  8B80 FC02000>mov eax,dword ptr ds:[eax+2FC]
00488F32   |.  E8 DD93FBFF  call unpacked.00442314
```

这里面对硬件号进行了处理，分成两个步骤，首先，逐位取出ascii，加到`esi`，把`esi*3241`的结果转成十进制，得到临时码。再对其进行编码。

```
For i = 1 To Len(machineID)
    eax = Asc(Mid$(machineID, i, 1)) '取出ascii
    esi = esi + eax '相加
    eax = esi * &H3241 '乘以3241
    temp = temp & CStr(encode(eax)) '对结果编码
Next
```

编码过程有点复杂，请看：

```
0048B5D8   /$  55           push ebp                        ;  把数字编码过程
0048B5D9   |.  8BEC         mov ebp,esp
0048B5DB   |.  81C4 DCFEFFF>add esp,-124
0048B5E1   |.  53           push ebx
0048B5E2   |.  56           push esi
0048B5E3   |.  57           push edi
0048B5E4   |.  33C9         xor ecx,ecx
0048B5E6   |.  898D E4FEFFF>mov dword ptr ss:[ebp-11C],ecx
0048B5EC   |.  898D ECFEFFF>mov dword ptr ss:[ebp-114],ecx
0048B5F2   |.  898D E8FEFFF>mov dword ptr ss:[ebp-118],ecx
0048B5F8   |.  898D F0FEFFF>mov dword ptr ss:[ebp-110],ecx
0048B5FE   |.  8BF2         mov esi,edx
0048B600   |.  8945 FC      mov dword ptr ss:[ebp-4],eax
0048B603   |.  8B45 FC      mov eax,dword ptr ss:[ebp-4]    ;  需要编码的数字，以第一个为例： 707575
0048B606   |.  E8 0195F7FF  call unpacked.00404B0C
0048B60B   |.  8DBD F4FEFFF>lea edi,dword ptr ss:[ebp-10C]  ;  707575
0048B611   |.  33C0         xor eax,eax
0048B613   |.  55           push ebp
0048B614   |.  68 5BB74800  push unpacked.0048B75B
0048B619   |.  64:FF30      push dword ptr fs:[eax]
0048B61C   |.  64:8920      mov dword ptr fs:[eax],esp
0048B61F   |.  8BC7         mov eax,edi
0048B621   |.  8B55 FC      mov edx,dword ptr ss:[ebp-4]
0048B624   |.  B9 FF000000  mov ecx,0FF                     ;  707575
0048B629   |.  E8 D292F7FF  call unpacked.00404900          ;  取得位数
0048B62E   |.  8A07         mov al,byte ptr ds:[edi]        ;  6
0048B630   |.  33D2         xor edx,edx                     ;  707575
0048B632   |.  8AD0         mov dl,al
0048B634   |.  42           inc edx
0048B635   |.  8955 F8      mov dword ptr ss:[ebp-8],edx
0048B638   |.  8B55 F8      mov edx,dword ptr ss:[ebp-8]
0048B63B   |.  81E2 0100008>and edx,80000001
0048B641   |.  79 05        jns short unpacked.0048B648
0048B643   |.  4A           dec edx
0048B644   |.  83CA FE      or edx,FFFFFFFE
0048B647   |.  42           inc edx
```
