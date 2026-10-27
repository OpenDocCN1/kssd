
- 设法直接计算私钥，比如参数值过小或曲线选择不合适，导致私钥可以被直接计算出来（如2019 ASIS CTF Quals-Halloween Party题目，给定y坐标求x坐标，计算2在模`P.order()`上的逆并将结果乘`2*P`直接得到`P`）

- 设法恢复私钥，即通过若干特殊明文签名对，采用建立方程等方式重构出密钥（如2019 DEFCON CTF Quals-Tania题目，通过`LLL`算法和`Babai`最近平面算法恢复出密钥）

- 设法伪造签名，即在不知道私钥的情况下，直接构造出特定字符串的签名来拿到`FLAG`（如2019 RealWorld CTF-bank题目，通过`rogue-key attack`伪造`Schnorr`比特币签名算法的银行提款签名）。

### 2.2.4 哈希函数

CTF中考察哈希函数类问题主要以两类场景进行展开：

一类是哈希碰撞类场景，常见的攻击类型包括：

- 同谋碰撞攻击：生成任意两个不同的消息和`x`和`y`，使得哈希值`f(x)=f(y)`
- 第二原像攻击：给定任意一个`x`及其哈希值`f(x)`，可以生成一个与之不同的`y`使得哈希值`f(x)=f(y)`
- 相同前缀攻击：给定任意一个前缀`p`，可以生成两个不同的后缀和`x`和`y`，使得哈希值`f(p+x)=f(p+y)`
- 选择前缀攻击：给定任意两个不同的前缀`p`和`q`，可以生成两个不同的后缀`x`和`y`，使得哈希值`f(p+x)=f(q+y)`

在考察形式上，题目通常会直接要求选手向服务器提交`A`、`B`两个输入，如果满足`AB`但`HASH(A)=HASH(B)`，则判断碰撞成功。`HASH`碰撞类题目虽然场景简单，但通常题目难度较大，而且很多时候都是出题人魔改之后的哈希算法，因此要求选手具备较强的分析能力，在攻击上多以考察代数攻击为主，如：

- 利用`small capacity parameter`误用问题构造`SHA3 Keccak`海绵结构碰撞（2019 0CTF/TCTF Quals-babysponge）
- 利用`Petit-Lauter`攻击构造超奇异同构图上`Charles-Goren-Lauter`哈希函数的弱实例碰撞（2017 HXP CTF-categorical）

另一类则是哈希伪造场景，虽然很多时候伪造哈希也可以通过寻找碰撞的方式来实现，但是它和碰撞场景下的考察侧重点通常不同，一种典型的攻击手法即哈希长度扩展攻击，即在已知的值和的长度的情况下（这里的值未知，即代表哈希时的`salt`），要求选手对于任意一个的值。对于哈希伪造的场景，我们的重点一般直接从攻击手法入手，而不需要对源码进行审计（绝大多数情况下，这类场景也不会给出哈希算法细节，而是直接调用现成的`MD5`、`SHA1`库函数）。

## 3 读paper的能力

自2018年起，大量论文题开始登上CTF历史舞台，然而一场标准的CTF国际赛通常为48小时，留给选手的分析时间是有限的，因此对于很多国际赛上的密码分析类题目来讲，选手是很难在短时间内提出一种可行的攻击手段的。这个时候，就需要快速提炼出题目当中的模型、场景等关键内容，然后去检索并阅读与之对应的会议、期刊论文，从中寻找攻击思想或手段。因此，选手的论文阅读理解能力（尤其是英文文献的阅读能力）对于密码学方向来讲就显得尤为重要。近年来国际赛当中CTF题目与学术界论文或研究成果联系紧密，如：

- 2018 PlaidCTF-braid题目，考察选手通过阅读论文[1]，解决一个辫子群（`Braid Group`）下的 `Conjugacy Search` 问题。
- 2018 Hack.lu CTF- Escape the grid题目，考察选手通过阅读论文[2]，攻破一个使用了非安全函数产生仿射变换的 `rasta` 密码系统。
- 2019 0CTF/TCTF Quals-zer0lfsr题目，考察选手通过通过阅读论文[3]，利用`Fast Correlation Attack`恢复出一个`Meier-Staffelbach`模型下0.75相关性的3个`LFSR`的初始状态（虽然这道题在当时比赛时被`z3`非预期了）。
- 2019 PwnThyBytes CTF-Wrong Ring题目，考察选手通过阅读论文[4]，通过将环上带误差学习（`RLWE`）样本转化为多项式带误差学习(`PLWE`)样本，攻击一个（non-dual）`RLWE`的实例来求私钥。

还有很多题目直接以某一论文或研究成果为背景进行设计，如：

- 2017 ASIS CTF Final-Marijuana

2017年5月，`Divesh Aggarwal`等学者提出了一种基于梅森素数的公钥密码体系[5]，2个月后，该密码系统被`Marc Beunardeau`等学者攻破，相关成果以论文形式发表[6]，同年9月，该事件以CTF形式出现在当年的ASIS CTF竞赛当中，预期选手通过阅读论文使用随机划分和`LLL`攻击攻破密码系统。

- 2017 HXP CTF-notsosmart

2017年2月，马萨里克大学的`Matus Nemec`和`Marek Sys`等学者发现英飞凌科技提供的`RSALib`库中的密钥生成算法存在漏洞，并据此提出了一种可行的`RSA ROCA`攻击。同年11月，相关攻击细节以论文形式披露[7]（该攻击同步影响了`TPM 1.2`, `4.3.5`版本前的`YubiKey 4`等产品，`CVE`编号：`CVE-2017-15361`），两周后，该事件以CTF形式出现在当年的HXP CTF竞赛当中，预期选手通过阅读论文使用`ROCA`攻击攻破一个`RSA`密钥生成算法。

## 4 扎实的编程功底

对于绝大多数情况下，当选手通过审计源码或者阅读论文整理出一种攻击手段之后，接下来都需要通过编程的方式写一个`solver`或`exp`来计算出你所需要的数据或实现你的攻击方式，这个时候就需要选手具备良好的编程功底，因为很多时候将复杂的数学模型转化成可行的脚本并不是一件容易的事。

这里所提到的编程主要可以看成两个部分：

一类是使用`python`、`C/C++`等这类语言进行编程，它是我们主要使用的编程语言，我们通过代码来描述我们的攻击过程，继而实现攻击。

第二类是针对某一工具的编程，常用的包括`SageMath`、`MatLab`、`Mathematica`编程等，这一类的编程往往起到的不是描述而是计算或辅助绘图的作用，帮助我们更好的完成解题过程。其中尤其以`SageMath`最为常用，如针对群、环、域等代数结构的计算，在`SageMath`中都可以很方便的进行操作，而不需要进行代数结构的二次描述，另外很多常用算法都以内置函数的形式在`SageMath`当中集成，可以很方便的供选手使用。

## 5 值得关注的CTF

由于CTF竞赛中专职密码学方向的选手通常较少，因此一场比赛是否有较高质量的密码学题目其实往往取决于办比赛的战队中有没有一个优秀的密码学选手，目前有一些比赛的密码学题目质量是要远高于其他竞赛的，值得参赛选手去重点关注（另外有一些战队虽然拥有很强的密码学选手，但是战队办比赛较少，可以关注其每次比赛赛后发布的`writeup`来了解其做题时的一些思路和解法）：

- Teaser Dragon CTF（波兰Google安全团队Dragon Sector战队主办）
- 0CTF/TCTF （中国腾讯安全A*0*E战队主办）
- HXP CTF （德国慕尼黑工业大学HXP战队主办）
- ASIS CTF & Crypto CTF （伊朗谢里夫理工大学ASIS战队主办）
- CODE BLUE CTF （日本联合战队binja主办）
- PlaidCTF （美国卡耐基梅隆大学PPP战队主办）
- Midnight Sun CTF （瑞典HackingForSoju战队主办）
- SpamAndFlags Teaser CTF （匈牙利布达佩斯技术与经济大学!SpamAndHex战队主办）
- PwnThyBytes CTF （罗马尼亚PwnThyBytes战队主办）

## 6 不只是CTF

对于一名密码学方向的CTF选手来讲，除了通过活跃于各大CTF比赛和在平时保持足够的CTF题目训练强度来提升自己的水平之外，实际上还有一些其他的密码学相关竞赛可以用来作为训练，每年诸如`NSU CRYPTO`、`WhibOx Contest`等解题类密码学竞赛，也可以参加或做赛后练习用，以此来锻炼密码分析能力。

## 7 对密码学的热情

兴趣因素放在本文最后一个环节进行论述，但是实际上这是学习密码学的第一步，也是最重要的一步，拥有对探索密码学领域的无限热情，会使你比任何人都更有可能成为一名优秀的CTF密码学选手。

## 8 参考文献

[1] Ko, K.H., & Lee, J.W. (2006). A fast algorithm to the conjugacy problem on generic braids.
[2] Dobraunig C. et al. (2018) Rasta: A Cipher with Low ANDdepth and Few ANDs per Bit. In: Shacham H., Boldyreva A. (eds) Advances in Cryptology – CRYPTO 2018. CRYPTO 2018. Lecture Notes in Computer Science, vol 10991. Springer, Cham
[3] Meier W. (2011) Fast Correlation Attacks: Methods and Countermeasures. In: Joux A. (eds) Fast Software Encryption. FSE 2011. Lecture Notes in Computer Science, vol 6733. Springer, Berlin, Heidelberg
[4] Elias Y., Lauter K.E., Ozman E., Stange K.E. (2015) Provably Weak Instances of Ring-LWE. In: Gennaro R., Robshaw M. (eds) Advances in Cryptology -- CRYPTO 2015. CRYPTO 2015. Lecture Notes in Computer Science, vol 9215. Springer, Berlin, Heidelberg
[5] Aggarwal D., Joux A., Prakash A., Santha M. (2018) A New Public-Key Cryptosystem via Mersenne Numbers. In: Shacham H., Boldyreva A. (eds) Advances in Cryptology – CRYPTO 2018. CRYPTO 2018. Lecture Notes in Computer Science, vol 10993. Springer, Cham
[6] Beunardeau M., Connolly A., Géraud R., Naccache D. (2019) On the Hardness of the Mersenne Low Hamming Ratio Assumption. In: Lange T., Dunkelman O. (eds) Progress in Cryptology – LATINCRYPT 2017. LATINCRYPT 2017. Lecture Notes in Computer Science, vol 11368. Springer, Cham
[7] Matus Nemec, Marek Sys, Petr Svenda, Dusan Klinec, Vashek Matyas, The Return of Coppersmith's Attack: Practical Factorization of Widely Used RSA Moduli, Proceedings of the 2017 ACM SIGSAC Conference on Computer and Communications Security, October 30-November 03, 2017, Dallas, Texas, USA [doi>10.1145/3133956.3133969]

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#Hash函数](https://bbs.kanxue.com/forum-132-1-146.htm)

[#对称算法](https://bbs.kanxue.com/forum-132-1-147.htm)

[#公开密钥算法](https://bbs.kanxue.com/forum-132-1-148.htm)

[#其他算法](https://bbs.kanxue.com/forum-132-1-149.htm)
