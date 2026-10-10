

# DSA 算法理论及应用

> 原文：[`www.pediy.com/kssd/pediy06/pediy6767.htm`](https://www.pediy.com/kssd/pediy06/pediy6767.htm)

DSA 算法的理论，实现，以及在破解中的应用

本节实例：点击此处下载  
相关工具：[`www.pediy.com/tools/Cryptography.htm`](http://www.pediy.com/tools/Cryptography.htm)   
  
  
      \[第一部分： 理论篇\]  
  
    0. 前言  
    我眼中的最有代表性的公匙密码算法 ECC，RSA，DSA。 其中前两种已经被国内众多  
    Cracker 界前辈高人详细的解释并实现，使我等初学者获益匪浅，其中尤为具代表性  
    的两篇如：  
    ●ECC 加密算法入门介绍 \[作者：zmworm/CCG\]  
    ●RSA 与大数运算 \[作者：afanty\]  
    更是使我受益良多。  
    唯独 DSA 算法 由于各种原因鲜有相关文章 国内的共享软件也很少有使用这种加密算  
    法进行注册验证的， 其实 DSA 算法有许多它自己独特的地方 它比 ECC 算法更加便于理  
    解和实现， 比 RSA 产生密匙的速度快很多，且安全性与 RSA 不相上下 但是 DSA 的一个  
    重要特点是两个素数(P,Q)公开，这样，当使用别人的 p 和 q 时，即使不知道私钥，你  
    也能确认它们是否是随机产生的，还是作了手脚。RSA 算法却作不到。  
    我用了一个礼拜时间四处搜集资料 其中包括应用密码学一书中 DSA 算法章节 DSATool  
    中的说明文件 看学学院中 DSA 算法的简单介绍 等诸多权威信息， 经过自己的整理，  
    翻译，理解写出本文，文中少有复杂的数论知识，重在应用和实践，旨在起到一个抛  
    砖引玉的作用 希望这套算法被更多人接受，理解，并得到广泛的应用。不足之处在所  
    难免 希望各位高人能不吝赐教加以指点 ：）  
  
    // 为免译文不准确 部分段落引用英文原文 英文好的朋友可以自行参考  
  
    1. General stuff / 常规资料  
    In  1991  the  Digital Signature  Algorithm(DSA)  has  become the  Digital  
    Signature Standard(DSS). DSA is a public-key signature scheme that uses  a  
    pair of  transformations to  generate and  verify a  digital value  called  
    signature.  
      
    DSA has been  developed by the  US National Security  Agency(NSA) and can   
    -not- be used for encryption or  key distribution. DSA is some variant  of  
    the ElGamal signature algorithm and, as defined in the standard, uses  the  
    Secure Hash Algorithm(SHA/SHA-1) as one-way hash function.  
    // Digital Signature Algorithm (DSA)是 Schnorr 和 ElGamal 签名算法的变种，被美国  
       NIST(美国国家标准局)作为数字签名标准(DigitalSignature Standard)。同样属于  
       公匙密码体系，并使用 Secure Hash Algorithm(SHA/SHA-1)作为中间单向计算算法  
  
    2. Parameters / 参数  
    P = A prime number in range 512 to 1024 bits which must be a multiple of 64  
    P = 一个范围在 512 至 1024 之间的素数 且必须为 64 的倍数  
    Q = A 160 bit prime factor of P-1  
    Q = P - 1 的 160bits 的素因子  
    G = H^((P-1)/Q) mod P. H is any number < P-1 such that H^((P-1)/Q) mod P > 1  
    G = h^((p-1)/q) mod P，H 必须 < p - 1, h^((p-1)/q) mod p > 1  
    X = A number < Q  
    X = 小于 Q 的一个数  
    Y = G^X mod P  
  
    Parameters P, Q, G and  Y are public where Y  is the public key. X  is the  
    private key and must be kept secret! To obtain X from Y one needs to solve  
    the  Discrete  Logarithm  Problem  which  is  virtually  impossible   for   
    -properly- generated parameters of reasonable size.  
    // 以上参数其中 P, Q, G 以及 Y 为公匙， X 为私匙必须保密！任何第三方用户想要从  
    Y 解密成 X 都必须解决整数有限域离散对数难题   
  
    3. Signing a message (M)  / 签名部分  
    To sign M, carry through the following steps:  
    // 若需要对 M 进行数字签名 则需要进行下列运算：  
    - Generate a -random- number K < Q. NEVER use same K twice or more to sign  
      other messages!  
    - 产生一个随机数 K (K < Q),，永远不要将同样的 K 用于进行其他的签名运算！  
    - Compute R = (G^K mod P) mod Q  
    - 计算 R = (G^K mod P) mod Q  
    - Compute S = (K^-1\*(SHA(M) + X\*R)) mod Q  
    - 计算 S = (K^-1\*(SHA(M) + X\*R)) mod Q  
  
    The number pair (R,S) is the signature of M.  
    // R 以及 S 为这次对 M 的数字签名结果  
  
    4. Verifying a signature (R,S) of M / 验证部分  
    - Compute W = S^-1 mod Q  
    - 计算 W = S^-1 mod Q  
    - Compute U1 = (SHA(M) \* W) mod Q  
    - 计算 U1 = (SHA(M) \* W) mod Q  
    - Compute U2 = (R\*W) mod Q  
    - 计算 U2 = (R\*W) mod Q  
    - Compute V = ((G^U1 \* Y^U2) mod P) mod Q  
    - 计算 V = ((G^U1 \* Y^U2) mod P) mod Q  
  
    If V == R the signature is verified.  
    // 若 v = r，则认为签名有效。  
  
    5. DSA 的安全性  
    DSA 主要依赖于整数有限域离散对数难题。素数 P 必须足够大，且 p-1 至少包含一个大  
    素数因子以抵抗 Pohlig & Hellman 算法的攻击。M 一般都应采用信息的 HASH 值(官方推  
    荐为 SHA 算法)。DSA 的安全性主要依赖于 p 和 g，若选取不当则签名容易伪造，应保证 g 对  
    于 p-1 的大素数因子不可约。 个人观点 DSA 的安全性要次于 ECC 与 RSA 不相上下。但是有  
    一点， 就是 DSA 算法的验证过程 R，S 是以明文形式出现的， 这点很容易被利用，在  
    第二篇中各位会体会到这一点。  
  
    6. Cracker 眼中的 DSA  
    DSA 算法鲜有被用于国产共享软件的注册验证部分 即使在国外的共享软件中也远不如  
    RSA，Blowfish 等算法应用广泛。  
    DSA 算法在破解时 关键的参数就是 X 根据 Y = G^X mod P 只要知道 P，G，Y，Q 且能  
    分解出 X 就可以伪造 R，S 写出 KeyGen 了。  
\====================================================================================  
\[第二部分： 签名篇\]  
  
    介绍完理论之后 我们来实践了解一下：  
    目标： pDriLl's Crypto KeygenMe #4  
    主要工具： C32Asm(感谢 pll621/CCG 送给我 Keyfile); Ollydbg1.09d;   
               BigInt Calc Pro 1.2(感谢 Stkman/CCG 送给我 Keyfile)  
               等等....  
  
    先脱壳：用 CoolDumper 找到 OEP 是 407952 然后使用 ImportREC1.6f 修复 IAT 表。  
    用 Ollydbg 载入程序 下断点 BPX GetDigItem 断下后走到不远处：  
  
004012A6   . LEA EBX,DWORD PTR SS:\[EBP-104\]  
004012AC   . MOV DWORD PTR DS:\[EBX\],EAX  
004012AE   . POP EBX  
004012AF   . CMP DWORD PTR SS:\[EBP-1D0\],0  // 该不会连名字都不输入吧  
004012B6   . JNZ SHORT Dumped4W.004012DE     
004012B8   . PUSH 0  
004012BA   . LEA EAX,DWORD PTR SS:\[EBP-3C\]  
004012C0   . PUSH EAX  
004012C1   . LEA EAX,DWORD PTR SS:\[EBP-20\]  
004012C7   . PUSH EAX  
004012C8   . LEA EAX,DWORD PTR SS:\[EBP+8\]  
004012CE   . MOV EAX,DWORD PTR DS:\[EAX\]  
004012D0   . PUSH EAX  
004012D1   . LEA EAX,DWORD PTR DS:\[4017BD\]  
004012D7   . PUSH EAX  
004012D8   .-JMP DWORD PTR DS:\[40E9A4\]                  user32.MessageBoxA  
004012DE     LEA EAX,DWORD PTR SS:\[EBP-22C\]  
004012E4   . PUSH EAX  
004012E5   . CALL Dumped4W.00401900  // 过了这个 CALL， DB EAX 就看到：  
  
0012F878  01 23 45 67 89 AB CD EF FE DC BA 98 76 54 32 10  #Eg 壂惋簶 vT2  
  
See...  MD5 的常量， 不知道为什么 我一见到 MD5 就兴奋 对这个算法很有感情 ：）  
MD5 特点 － 单向不可逆 所以在这里不多说了 主要介绍 DSA。  
  
004012EA   . ADD ESP,4  
004012ED   . MOV CL,BYTE PTR DS:\[40E9C6\]  // 取 Key 的第 7 位  
004012F3   . MOV BYTE PTR DS:\[40E940\],CL    
004012F9   . MOV DL,BYTE PTR DS:\[40E9CD\]  // 取 Key 的第 14 位  
004012FF   . MOV BYTE PTR DS:\[40E941\],DL  
00401305   . MOV AL,BYTE PTR DS:\[40E9D4\]  // 取 Key 的第 21 位  
//所以 Key 最少为 21 位， 重新输入注册信息， Name:LingDi  Key:123456789012345678901  
  
0040130A   . MOV BYTE PTR DS:\[40E942\],AL  
0040130F   . MOV DWORD PTR SS:\[EBP-48\],Dumped4W.0040E>;  ASCII "12FHCF-YEAH!!"  
00401316   . MOV ECX,DWORD PTR SS:\[EBP-104\]  
0040131C   . PUSH ECX  
0040131D   . PUSH Dumped4W.0040D1C0                     ASCII "%d"  
00401322   . MOV EDX,DWORD PTR SS:\[EBP-48\]  
00401325   . PUSH EDX  
00401326   . CALL Dumped4W.00407900  // 将 Key 的位数连接到刚才取出的 3 个 Key 值后.  
0040132B   . ADD ESP,0C  
0040132E   . MOV DWORD PTR SS:\[EBP-48\],Dumped4W.0040E>;  ASCII "FHCF-YEAH!!"  
00401335   . PUSH Dumped4W.0040D1B4                     ASCII "EGBE-YEAH!!"  
0040133A   . PUSH Dumped4W.0040D1B0                     ASCII "%s"  
0040133F   . MOV EAX,DWORD PTR SS:\[EBP-48\]  
00401342   . PUSH EAX  
00401343   . CALL Dumped4W.00407900  // 连接 EGBE-YEAH!!    
// DB 40e940 看到如下结果  
  
0040E940  37 34 31 32 37 45 47 42 45 2D 59 45 41 48 21 21  74127EGBE-YEAH!!  
0040E950  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
  
  
00401348   . ADD ESP,0C  
0040134B   . MOV CL,BYTE PTR DS:\[40E945\]  
00401351   . ADD CL,1  
00401354   . MOV BYTE PTR DS:\[40E945\],CL  
0040135A   . MOV DL,BYTE PTR DS:\[40E946\]  
00401360   . ADD DL,1  
00401363   . MOV BYTE PTR DS:\[40E946\],DL  
00401369   . MOV AL,BYTE PTR DS:\[40E947\]  
0040136E   . ADD AL,1  
00401370   . MOV BYTE PTR DS:\[40E947\],AL  
00401375   . MOV CL,BYTE PTR DS:\[40E948\]  
0040137B   . ADD CL,1  
0040137E   . MOV BYTE PTR DS:\[40E948\],CL  
  
// 00401354 至 0040137E 只不过是分别取 EGBE 的 ASCII 值 并加 1  
// 结果为 FHCF， 好像是个组织名 不过没在 0DayFTP 上看到过这个组织的 Relase  
  
  
00401384   . MOV BYTE PTR DS:\[40E950\],0  
0040138B   . PUSH 10  
0040138D   . PUSH Dumped4W.0040E940  
00401392   . LEA EDX,DWORD PTR SS:\[EBP-22C\]  
00401398   . PUSH EDX  // 啦啦啦～  又看到 MD5 的四个常量了 你猜接下来要干吗？  
00401399   . CALL Dumped4W.00401930   
0040139E   . ADD ESP,0C  
004013A1   . LEA EAX,DWORD PTR SS:\[EBP-22C\]  
004013A7   . PUSH EAX  
004013A8   . LEA ECX,DWORD PTR SS:\[EBP-F0\]  
004013AE   . PUSH ECX  
004013AF   . CALL Dumped4W.004019E0  
004013B4   . ADD ESP,8  
004013B7   . MOV EDX,DWORD PTR SS:\[EBP-1CC\]  
004013BD   . PUSH EDX  
004013BE   . LEA EAX,DWORD PTR SS:\[EBP-F0\]  
004013C4   . PUSH EAX  
// EAX 中保存"74127FHCF-YEAH!!"的 MD5 Hash 结果：AA6E429C590F8579E5D51EBAAE66643A  
  
0012F9B4  AA 6E 42 9C 59 0F 85 79 E5 D5 1E BA AE 66 64 3A  猲 B 淵厃逭寒 fd:  
  
004013C5   . PUSH 10  
004013C7   . CALL Dumped4W.004065C0  
004013CC   . ADD ESP,0C  
004013CF   . PUSH Dumped4W.0040D194                     ASCII "Bn6EN1dDFrupNxw1Wk4WO5=="  
004013D4   . MOV ECX,DWORD PTR SS:\[EBP-FC\]  
004013DA   . PUSH ECX  
004013DB   . CALL Dumped4W.00405E80  
  
01CE2CA0  04 00 00 00 AC 2C CE 01 00 00 00 00 B9 63 E1 A4  ...??....筩幛  
01CE2CB0  55 C3 71 93 BA 6B 31 74 75 43 E8 67 00 00 00 00  U 胵摵 k1tuC 鑗....  
  
004013E0   . ADD ESP,8  
004013E3   . MOV EDX,DWORD PTR SS:\[EBP-1CC\]  
004013E9   . PUSH EDX  
004013EA   . MOV EAX,DWORD PTR SS:\[EBP-FC\]  
004013F0   . PUSH EAX  
004013F1   . CALL Dumped4W.004039C0     
// 这个 CALL 用来比较 Hash 结果是否等于 67E8437574316BBA9371C355A4E163B9  
004013F6   . ADD ESP,8  
004013F9   . TEST EAX,EAX  
004013FB   . JE SHORT Dumped4W.00401423  
  
开始我怎么也弄不明白 Bn6EN1dDFrupNxw1Wk4WO5== 这个字符串是干什么用的，后来在学习一段  
Delphi 源代码的时候才想起来 Base64 这个东西。上面一段代码的作用就是 对比 "74127FHCF-YEAH!!"  
的 MD5 Hash 结果：AA6E429C590F8579E5D51EBAAE66643A 是否等于 "Bn6EN1dDFrupNxw1Wk4WO5=="  
即 67E8437574316BBA9371C355A4E163B9， 不相等则提示 Hmmm.. you don't even pass the first threshold!!  
  
我们来考虑一下如何过这第一关， 过关条件是：  
"Key 的第 7 位 + Key 的第 14 位 + Key 的第 21 位 + Key 的位数 + FHCF-YEAH!!" 这段字符串的 MD5 结果必须  
等于 67E8437574316BBA9371C355A4E163B9  
  
这就要靠猜了 稍微有点 Cracker 头脑的人 都会猜到注册码是 xxxxxx-xxxxxx-xxxxxx-xxxxxx 这种格式的 不然  
就必须编程穷举了。 相当于穷举 5 位密码的 MD5 值，穷举的速度也不慢 如果条件设置得当，理论上 3 分钟左右应该  
就可以得到结果 我是属于没有 Cracker 头脑的人 所以这种格式并不是我想到的  ：（  
验证一下：  MD5("---27FHCF-YEAH!!") = 67E8437574316BBA9371C355A4E163B9  验证通过。  
  
重新输入注册信息，进入下一阶段：  
Name: LingDi  
Key:  123456-890123-567890-234567  
  
00401448   . MOV EDI,Dumped4W.0040E9C0                  ASCII "123456-890123-567890-234567"  
0040144D   . OR ECX,FFFFFFFF  
00401450   . XOR EAX,EAX  
00401452   . REPNE SCAS BYTE PTR ES:\[EDI\]  
00401454   . NOT ECX  
00401456   . ADD ECX,-1  
00401459   . CMP DWORD PTR SS:\[EBP-100\],ECX  
0040145F   . JGE SHORT Dumped4W.004014D5  
00401461   . CMP DWORD PTR SS:\[EBP-100\],6  
00401468   . JNZ SHORT Dumped4W.00401479  
0040146A   . MOV EDX,DWORD PTR SS:\[EBP-100\]  
00401470   . ADD EDX,1  
00401473   . MOV DWORD PTR SS:\[EBP-100\],EDX  
00401479   > CMP DWORD PTR SS:\[EBP-100\],0D  
00401480   . JNZ SHORT Dumped4W.00401491  
00401482   . MOV EAX,DWORD PTR SS:\[EBP-100\]  
00401488   . ADD EAX,1  
0040148B   . MOV DWORD PTR SS:\[EBP-100\],EAX  
00401491   > CMP DWORD PTR SS:\[EBP-100\],14  
00401498   . JNZ SHORT Dumped4W.004014A9  
0040149A   . MOV ECX,DWORD PTR SS:\[EBP-100\]  
004014A0   . ADD ECX,1  
004014A3   . MOV DWORD PTR SS:\[EBP-100\],ECX  
004014A9   > MOV EDX,DWORD PTR SS:\[EBP-108\]  
004014AF   . MOV EAX,DWORD PTR SS:\[EBP-100\]  
004014B5   . MOV CL,BYTE PTR DS:\[EAX+40E9C0\]  
004014BB   . MOV BYTE PTR DS:\[EDX+40E940\],CL  
004014C1   . MOV EDX,DWORD PTR SS:\[EBP-108\]  
004014C7   . ADD EDX,1  
004014CA   . MOV DWORD PTR SS:\[EBP-108\],EDX  
004014D0   .^JMP Dumped4W.00401439  
  
//看结果就知道 以上是个循环 目地是去除 Key 中的 "-"  结果：  
  
0040E940  31 32 33 34 35 36 38 39 30 31 32 33 35 36 37 38  1234568901235678  
0040E950  39 30 32 33 34 35 36 37 00 00 00 00 00 00 00 00  90234567........  
  
004014F0   > MOV EDI,Dumped4W.0040E940                  ASCII "123456890123567890234567"  
004014F5   . OR ECX,FFFFFFFF  
004014F8   . XOR EAX,EAX  
004014FA   . REPNE SCAS BYTE PTR ES:\[EDI\]  
004014FC   . NOT ECX  
004014FE   . ADD ECX,-1  
00401501   . CMP DWORD PTR SS:\[EBP-100\],ECX  
00401507   . JGE SHORT Dumped4W.0040157C  
00401509   . MOV ECX,DWORD PTR SS:\[EBP-100\]  
0040150F   . MOVSX EDX,BYTE PTR DS:\[ECX+40E940\]  
00401516   . CMP EDX,41  
00401519   . JL SHORT Dumped4W.0040152D  
0040151B   . MOV EAX,DWORD PTR SS:\[EBP-100\]  
00401521   . MOVSX ECX,BYTE PTR DS:\[EAX+40E940\]  
00401528   . CMP ECX,46  
0040152B   . JLE SHORT Dumped4W.00401577  
0040152D   > MOV EDX,DWORD PTR SS:\[EBP-100\]  
00401533   . MOVSX EAX,BYTE PTR DS:\[EDX+40E940\]  
0040153A   . CMP EAX,30  
0040153D   . JL SHORT Dumped4W.00401551  
0040153F   . MOV ECX,DWORD PTR SS:\[EBP-100\]  
00401545   . MOVSX EDX,BYTE PTR DS:\[ECX+40E940\]  
0040154C   . CMP EDX,39  
0040154F   . JLE SHORT Dumped4W.00401577  
00401551   > PUSH 0  
00401553   . LEA EAX,DWORD PTR SS:\[EBP-3C\]  
00401559   . PUSH EAX  
0040155A   . LEA EAX,DWORD PTR SS:\[EBP-1C4\]  
00401560   . PUSH EAX  
00401561   . LEA EAX,DWORD PTR SS:\[EBP+8\]  
00401567   . MOV EAX,DWORD PTR DS:\[EAX\]  
00401569   . PUSH EAX  
0040156A   . LEA EAX,DWORD PTR DS:\[4017BD\]  
00401570   . PUSH EAX  
00401571   .-JMP DWORD PTR DS:\[40E9A4\]                  user32.MessageBoxA  
  
// 得到了 Key 的有效范围： 0~9 A~F  
  
004015C1   . LEA ECX,DWORD PTR SS:\[EBP-A4\]  
004015C7   . PUSH ECX  
004015C8   . CALL Dumped4W.00402490 // 跟进这个 CALL 会看到 SHA 算法的 5 个常量  
  
// 00402490  /$ MOV EAX,DWORD PTR SS:\[ESP+4\]  
// 00402494  |. XOR ECX,ECX  
// 00402496  |. MOV DWORD PTR DS:\[EAX\],67452301  
// 0040249C  |. MOV DWORD PTR DS:\[EAX+4\],EFCDAB89  
// 004024A3  |. MOV DWORD PTR DS:\[EAX+8\],98BADCFE  
// 004024AA  |. MOV DWORD PTR DS:\[EAX+C\],10325476  
// 004024B1  |. MOV DWORD PTR DS:\[EAX+10\],C3D2E1F0  
// 004024B8  |. MOV DWORD PTR DS:\[EAX+14\],ECX  
// 004024BB  |. MOV DWORD PTR DS:\[EAX+18\],ECX  
// 004024BE  \\. RETN  
  
004015CD   . ADD ESP,4  
004015D0   . MOV EDX,DWORD PTR SS:\[EBP-1D0\]  
004015D6   . PUSH EDX  
004015D7   . PUSH Dumped4W.0040E8C0                     ASCII "LingDi"  
004015DC   . LEA EAX,DWORD PTR SS:\[EBP-A4\]  // 期待已久的用户名入栈  
004015E2   . PUSH EAX  
004015E3   . CALL Dumped4W.004024C0  
004015E8   . ADD ESP,0C  
004015EB   . LEA ECX,DWORD PTR SS:\[EBP-A4\]  
004015F1   . PUSH ECX  // SHA 计算所需要的常量。  
  
0012FA00  01 23 45 67 89 AB CD EF FE DC BA 98 76 54 32 10  #Eg 壂惋簶 vT2  
0012FA10  F0 E1 D2 C3 30 00 00 00 00 00 00 00 4C 69 6E 67  疳颐 0.......Ling  
  
004015F2   . LEA EDX,DWORD PTR SS:\[EBP-120\]  
004015F8   . PUSH EDX  
004015F9   . CALL Dumped4W.00402760  //  SHA(UserName)  
  
0012F984  9E 80 31 BA 2C 32 5B 7D E8 FA 59 34 69 BD 24 81  瀫 1?2\[}楮 Y4i?  
0012F994  E5 0A 63 0C 00 00 01 01 18 00 00 00 1B 00 00 00  ?c.........  
  
Hash 结果： 9E8031BA2C325B7DE8FA593469BD2481E50A630C   
// 这里有必要说一下， 我试了计算 SHA160(Name)的值 并没有得到这个结果， 根据 DSA  
的官方资料，这里推荐为 SHA-0/SHA-1 所以我猜测上面使用的 HASH 算法是 SHA-0，但由于我  
手头没有计算 SHA-0 的工具，所以暂时无法验证，本文的重点是 DSA，所以这里我们得到 HASH  
的结果就行了，计划下一篇文章写写 SHA 算法，不过那是以后的事情了。 ：）  
  
004015FE   . ADD ESP,8  
00401601   . MOV EAX,DWORD PTR SS:\[EBP-1D4\]  
00401607   . MOV DWORD PTR DS:\[EAX+220\],40  
00401611   . PUSH Dumped4W.0040D170            ASCII "rizMjllW3niYFDZJlEEI7vyix++QkBK7="  
00401616   . MOV ECX,DWORD PTR SS:\[EBP-40\]  
00401619   . PUSH ECX  
0040161A   . CALL Dumped4W.00405E80  
  
092F9320  06 00 00 00 2C 93 2F 09 00 00 00 00 BB 12 90 90  ...,?.....?悙  
092F9330  EF C7 A2 FC EE 08 41 94 49 36 14 98 78 DE 56 59  锴Ⅻ?A 擨 6 榵轛 Y  
092F9340  8E CC 2C AE 00 00 00 00 00 00 00 00 00 00 00 00  幪,?...........  
// P = AE2CCC8E5956DE7898143649944108EEFCA2C7EF909012BB   
  
0040161F   . ADD ESP,8  
00401622   . PUSH Dumped4W.0040D164            ASCII "p911BFFR="  
00401627   . MOV EDX,DWORD PTR SS:\[EBP-44\]  
0040162A   . PUSH EDX  
0040162B   . CALL Dumped4W.00405E80  
  
092F9170  02 00 00 00 7C 91 2F 09 00 00 00 00 51 51 04 75  ...|?.....QQu  
092F9180  DD A7 00 00 00 00 00 00 00 00 00 00 00 00 00 00  荮..............  
// Q = A7DD75045151   
  
00401630   . ADD ESP,8  
00401633   . PUSH Dumped4W.0040D140            ASCII "jfwz34Lt7/VqvRrhYb4X+8H0A9XfEzEG="  
00401638   . MOV EAX,DWORD PTR SS:\[EBP-F8\]  
0040163E   . PUSH EAX  
0040163F   . CALL Dumped4W.00405E80  
  
092FAE50  06 00 00 00 5C AE 2F 09 00 00 00 00 06 31 13 DF  ...\\?.....1  
092FAE60  D5 03 F4 C1 FB 17 BE 61 E1 1A BD 6A F5 EF ED 82  ?袅?綼?絡躏韨  
092FAE70  DF 33 FC 8D 00 00 00 00 00 00 00 00 00 00 00 00  ?鼚............  
// G =8DFC33DF82EDEFF56ABD1AE161BE17FBC1F403D5DF133106   
  
00401644   . ADD ESP,8  
00401647   . PUSH Dumped4W.0040D11C            ASCII "qg1kJK2T1pVDWjoJ+q/VNYYg03Ij7q85="  
0040164C   . MOV ECX,DWORD PTR SS:\[EBP-FC\]  
00401652   . PUSH ECX  
00401653   . CALL Dumped4W.00405E80  
  
092FACA0  06 00 00 00 AC AC 2F 09 00 00 00 00 39 AF EE 23  .../.....9#  
092FACB0  72 D3 20 86 35 D5 AF FA 09 3A 5A 43 95 D6 93 AD  r??寨?:ZC 曋摥  
092FACC0  24 64 0D AA 00 00 00 00 00 00 00 00 00 00 00 00  $d.?...........  
// Y = AA0D6424AD93D695435A3A09FAAFD5358620D37223EEAF39   
  
00401658   . ADD ESP,8  
0040165B   . MOV EDX,DWORD PTR SS:\[EBP-1CC\]  
00401661   . PUSH EDX  
00401662   . LEA EAX,DWORD PTR SS:\[EBP-120\]  
00401668   . PUSH EAX // 用户名的 Hash 结果  
  
0012F984  9E 80 31 BA 2C 32 5B 7D E8 FA 59 34 69 BD 24 81  瀫 1?2\[}楮 Y4i?  
0012F994  E5 0A 63 0C 00 00 01 01 18 00 00 00 1B 00 00 00  ?c.........  
  
// 至此 我们得到了 DSA 的四个公匙：  
P = AE2CCC8E5956DE7898143649944108EEFCA2C7EF909012BB   
G = 8DFC33DF82EDEFF56ABD1AE161BE17FBC1F403D5DF133106   
Y = AA0D6424AD93D695435A3A09FAAFD5358620D37223EEAF39   
Q = A7DD75045151   
  
接下来轮到 BigInt Calc1.2 出场了，它可以很方便的使用表达式辅助计算，方便速度快  
最重要的是 CCG 成员编写的 当然要大力推荐一下咯，具体用法参考说明文件，我就不多说了  
   
根据 Y = G^X mod P 得到：X =  40A6C8A2464A891E99DDBFCFC967BAFD4BAFA67B3ECEDC43   
根据 R = (G^K mod P) mod Q ，设 K 等于 1，计算(G mod P) mod Q 得到： R = 4A2B8273F9AF  
根据 S = (K^-1\*(SHA(M) + X\*R)) mod Q，设 K 等于 1，计算((SHA(M) + X\*R)) mod Q，  
已知： SHA(M) = 9E8031BA2C325B7DE8FA593469BD2481E50A630C  
即： S = (9E8031BA2C325B7DE8FA593469BD2481E50A630C + 12BB32F497704DE84B07A0E581B75D  
3F00EAAFA1FB124DAD4E5F1BBEBCCD)) mod A7DD75045151  
最终得到： S = 2FE47548E4AC  
  
签名算法分析完成， 给出两组有效 KEY，注意大小写，另外任何一个有效的 Name 所对应的 Key  
都不是唯一的 原因在于 K 的值， 比如：  
Name：LingDi  
SN：  4A2B82-73F9AF-2FE475-48E4AC  /  57410B-49E869-63014B-326892  
Name：娃娃  
SN：  4A2B82-73F9AF-2F3777-6CD9C8  
  
通过上面的分析以及计算 想必各位已经了解了 DSA 算法的签名过程 接下来我们仍然结合这个  
KeygenMe 来了解 DSA 算法的验证过程。  
\====================================================================================  
\[第三部分： 验证篇\]  
  
仍然是刚才的 KeygenMe， 上一篇只介绍了 DSA 算法的签名实现步骤，下面让我们通过调试器来看看  
这个 KeygenMe 是如何验证有效 Key 的，通过实践来了解 DSA 算法的验证过程以及计算方法。  
我们以输入有效 Key  
Name：LingDi  
SN：  4A2B82-73F9AF-2FE475-48E4AC  
为例 来分析运算过程：  
  
00401669   . PUSH 14  
0040166B   . CALL Dumped4W.004065C0  
00401670   . ADD ESP,0C  
00401673   . MOV ECX,DWORD PTR SS:\[EBP-E0\]  
00401679   . PUSH ECX  
0040167A   . MOV EDX,DWORD PTR SS:\[EBP-E0\]  
00401680   . PUSH EDX  
00401681   . MOV EAX,DWORD PTR SS:\[EBP-E0\]  
00401687   . PUSH EAX  // EAX = S  
  
009B82B0  02 00 00 00 BC 82 9B 00 00 00 00 00 AC E4 48 75  ...紓?....Hu  
009B82C0  E4 2F 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ?..............  
  
00401688   . MOV ECX,DWORD PTR SS:\[EBP-44\]  
0040168B   . PUSH ECX  // ECX = Q  
  
009B9940  02 00 00 00 4C 99 9B 00 00 00 00 00 51 51 04 75  ...L 櫅.....QQu  
009B9950  DD A7 00 00 00 00 00 00 00 00 00 00 00 00 00 00  荮..............  
  
0040168C   . MOV EDX,DWORD PTR SS:\[EBP-E0\]  
00401692   . PUSH EDX  
00401693   . CALL Dumped4W.00405530  
  
// 以上代码计算 W = S^-1 mod Q  
  
00401698   . ADD ESP,14  
0040169B   . MOV EAX,DWORD PTR SS:\[EBP-124\]  
004016A1   . PUSH EAX  
004016A2   . MOV ECX,DWORD PTR SS:\[EBP-44\]  
004016A5   . PUSH ECX  
004016A6   . MOV EDX,DWORD PTR SS:\[EBP-44\]  
004016A9   . PUSH EDX  
004016AA   . MOV EAX,DWORD PTR SS:\[EBP-E0\]  
004016B0   . PUSH EAX  // EAX = W  
004016B1   . MOV ECX,DWORD PTR SS:\[EBP-E0\]  
004016B7   . PUSH ECX  // ECX = Q  
004016B8   . MOV EDX,DWORD PTR SS:\[EBP-1CC\]  
004016BE   . PUSH EDX  // EDX ＝ SHA(M)  
  
009D5090  05 00 00 00 9C 50 9D 00 00 00 00 00 0C 63 0A E5  ...淧?.....c.  
009D50A0  81 24 BD 69 34 59 FA E8 7D 5B 32 2C BA 31 80 9E  ?絠 4Y}\[2,?€  
  
004016BF   . CALL Dumped4W.00404E50  
  
009D2090  02 00 00 00 9C 20 9D 00 00 00 00 00 6D E0 ED C4  ...??....m 囗  
009D20A0  83 61 00 00 00 00 00 00 00 00 00 00 00 00 00 00  僡..............  
// 上面是用来计算 U1 = (SHA(M) \* W) mod Q  
  
004016C4   . ADD ESP,18  
004016C7   . MOV EAX,DWORD PTR SS:\[EBP-1C8\]  
004016CD   . PUSH EAX  
004016CE   . MOV ECX,DWORD PTR SS:\[EBP-44\]  
004016D1   . PUSH ECX  
004016D2   . MOV EDX,DWORD PTR SS:\[EBP-44\]  
004016D5   . PUSH EDX  
004016D6   . MOV EAX,DWORD PTR SS:\[EBP-E0\]  
004016DC   . PUSH EAX // EAX = W  
  
009D3090  02 00 00 00 9C 30 9D 00 00 00 00 00 9E 1F FE D8  ...??....?  
009D30A0  C9 8F 00 00 00 00 00 00 00 00 00 00 00 00 00 00  蓮..............  
  
004016DD   . MOV ECX,DWORD PTR SS:\[EBP-E0\]  
004016E3   . PUSH ECX  // ECX = Q  
004016E4   . MOV EDX,DWORD PTR SS:\[EBP-DC\]  
004016EA   . PUSH EDX  // EDX = R  
  
009D4090  02 00 00 00 9C 40 9D 00 00 00 00 00 AF F9 73 82  ...淍?....s  
009D40A0  2B 4A 00 00 00 00 00 00 00 00 00 00 00 00 00 00  +J..............  
  
004016EB   . CALL Dumped4W.00404E50  
  
009D1090  02 00 00 00 9C 10 9D 00 00 00 00 00 14 B3 2F B6  ...??....?  
009D10A0  54 6C 00 00 00 00 00 00 00 00 00 00 00 00 00 00  Tl..............  
// 上面的代码用来计算 U2 = (R\*W) mod Q  
  
004016F0   . ADD ESP,18  
004016F3   . MOV EAX,DWORD PTR SS:\[EBP-F4\]  
004016F9   . PUSH EAX  
004016FA   . MOV ECX,DWORD PTR SS:\[EBP-40\]  
004016FD   . PUSH ECX  
004016FE   . MOV EDX,DWORD PTR SS:\[EBP-1C8\]  
00401704   . PUSH EDX  
00401705   . MOV EAX,DWORD PTR SS:\[EBP-FC\]  
0040170B   . PUSH EAX  // EAX = Y  
  
009D6680  06 00 00 00 8C 66 9D 00 00 00 00 00 39 AF EE 23  ...宖?....9#  
009D6690  72 D3 20 86 35 D5 AF FA 09 3A 5A 43 95 D6 93 AD  r??寨?:ZC 曋摥  
009D66A0  24 64 0D AA 00 00 00 00 00 00 00 00 00 00 00 00  $d.?...........  
  
0040170C   . MOV ECX,DWORD PTR SS:\[EBP-124\]  
00401712   . PUSH ECX  // ECX = U1  
00401713   . MOV EDX,DWORD PTR SS:\[EBP-F8\]  
00401719   . PUSH EDX  // EDX = G  
  
009D6830  06 00 00 00 3C 68 9D 00 00 00 00 00 06 31 13 DF  ...br> 009D6840  D5 03 F4 C1 FB 17 BE 61 E1 1A BD 6A F5 EF ED 82  ?袅?綼?絡躏韨  
009D6850  DF 33 FC 8D 00 00 00 00 00 00 00 00 00 00 00 00  ?鼚............  
  
0040171A   . CALL Dumped4W.00405490  
// 计算 Vt = (G^U1 \* Y^U2) mod P  
  
0040171F   . ADD ESP,18  
00401722   . MOV EAX,DWORD PTR SS:\[EBP-44\]  
00401725   . PUSH EAX    // EAX = Q                    /Arg3  
00401726   . MOV ECX,DWORD PTR SS:\[EBP-44\]             |  
00401729   . PUSH ECX                                  |Arg2  
0040172A   . MOV EDX,DWORD PTR SS:\[EBP-F4\]             |  
00401730   . PUSH EDX    // EDX = Vt                   |Arg1  
00401731   . CALL Dumped4W.004044B0                    \\Dumped4W.004044B0  
// 计算 V = Vt mod Q  
  
00401736   . ADD ESP,0C  
00401739   . MOV EAX,DWORD PTR SS:\[EBP-DC\]  
0040173F   . PUSH EAX  // EAX = V  
  
009D4090  02 00 00 00 9C 40 9D 00 00 00 00 00 AF F9 73 82  ...淍?....s  
009D40A0  2B 4A 00 00 00 00 00 00 00 00 00 00 00 00 00 00  +J..............  
  
00401740   . MOV ECX,DWORD PTR SS:\[EBP-F4\]  
00401746   . PUSH ECX  // ECX = R  
  
009D0090  02 00 00 00 9C 00 9D 00 00 00 00 00 AF F9 73 82  ...??....s  
009D00A0  2B 4A 00 00 00 00 00 00 00 00 00 00 00 00 00 00  +J..............  
  
00401747   . CALL Dumped4W.004039C0  // 对比 V 是否等于 R  
0040174C   . ADD ESP,8  
0040174F   . TEST EAX,EAX  
00401751   . JE SHORT Dumped4W.0040177B  
// 得到了 V， 则与 R 对比， 相等则验证通过，不相等自然失败咯～  
  
至此，验证部分的工作过程想必各位都了解了吧，这个 KeygenMe 就此退役了 真的非常  
感谢 pDriLi 编写了这个这么出色的 KeygenMe，结构清晰，代码紧凑 真是一个非常好的  
例子。  ：）  
  
\====================================================================================  
  
    \[第四部分： 应用篇\]  
  
实践过后自然要知道怎么用， 我这篇文章的目地就是希望这种算法能得到广泛的使用  
实际上 DSA 算法在 IC 卡 以及许多银行卡中已经被得到了应用， 但是由于美国的密码算法  
出口限制 这种算法并没有被个人广泛使用  
  
    IMPORTANT FINAL NOTE:  
    Please never forget that even  the most secure cryptographic algorithm  is  
    worthless if  you make  mistakes in  the implementation.  So, if you don't  
    know -exactly- what  you're doing, better  use some 3rd  party sourcecodes  
    made by people you trust and -never- change the recommended standard(s).  
  
// 上面一段是 DSATools 的作者 tE!/TMG 说的， 译文：  
请永远不要忘记， 任何一种安全的加密算法若你在执行的时候出现了一些差错 都可能导致  
密文轻易被攻破， 所以若你并不确定你的程序要做什么和在做什么，或许使用一个第三方编写  
的源代码或插件是个不错的主意。  
  
RSA 算法依靠的是 N 够大，使得你不能分解，无法得到 D，同 RSA 一样 DSA 算法若公匙过于简单，   
很容易让人得到 X， DSA 中的 X 价值相当于 RSA 中的 D， 根据上面的实例各位不难看出， 得到了 X   
剩下事情就是写 KeyGen 了。  
  
DSA 算法在 C/C++ ，Delphi， BC 中都有第三方控件  有兴趣的朋友可以自行搜索一下 通过  
源代码 或许可以更好的理解 DSA 的过程和原理， 我在这里就不贴出了， DSA 有比 RSA 更快的  
速度以及更多优秀的特性， 希望能得到大家妥善的应用。  
  
  
  
    \[第五部分： 写在后面的话\]  
  
必须要说明的是 我不是数论高材生 所以写出来的东西也许只能得到皮毛， 无法深究内  
在精华， 若文中有什么错误 请各位高手不吝赐教，感激不尽  
  
  
计划过几天写一个 DSATools  实现 X 的分解 以及相关运算， 最近事情很多 要写 NFOMaker  
还有忙活 CCG 的官方主页... ...  
  
ThX Fly To：  
－＝  KanXue  Stkman  Ivanopulo  SunBird  pll621  Hoto  时空幻影 nzinzi  ＝－  
－＝  Hying   DingBoy  小楼  Ajj  heXer  Amenesia  ..... 所有帮助过我的朋友  
  
仅以次文献给 CCG 组织 希望它能蒸蒸日上！  
  
娃娃(LingDi)  
属于中国破解组织 CCG  
CHiNA CrACKiNG GrOUp  
\-= iPB =-  -= BCG =-  
2003 年 8 月 12 日  
  
/\*注：  转载请保留作者 并注明转自看雪论坛。  
  
  
\====================================================================================