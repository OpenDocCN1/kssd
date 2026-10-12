
# 流密嵌魔方於存之研究

> 原文：[`www.pediy.com/kssd/pediy10/88968.html`](https://www.pediy.com/kssd/pediy10/88968.html)

1. 名: 流密(`Stream Cipher`)、魔方(`Magic Square`)、存取(`Random Access File`)
2. 篇的英文容的不是很好，大家看故事就可以懂了，多包函。
3. 篇主要是在做什？就是一套加密/解密的件。
4. 用了主要技：流密及魔方。您可以想像家有道一，道都比一道安全。但事是不是真的？就要靠大家追求真相。
5. 根理，我已出件，目前有繁版跟英文版，改天有空我整理好再上(`upload`)大家玩玩看。

A Study on the Stream Cipher Embedded Magic Square of Random Access Files

`rock@phate.tw`

`Abstract`

Magic square and stream cipher issue are very interest well-tried topic. In this paper, we are proposed a new scheme which stream cipher application for random access file that based on magic square method. By the way, there are two thresholds to secure ours data, if decrypts only by the stream cipher. It isn’t to recovery original source. In other hand, we improve the model of cipher stream to strong the defense efficiently, it also own high speed calculate to most parts of key stream generator.

`Keywords`: Stream Cipher, Magic Square, Key Stream Generator, Quadratic Residue

`1. Introduction`

Symmetric key cryptosystems are an important type of modern cryptosystem \[1, 2, 4, 5, 8, 11, 13\]. Symmetric key systems are cryptosystems where the same key is used for both encryption and decryption. This class of cryptosystem is important in modern cryptography because, in general, symmetric key cryptosystems are much faster than public key cryptosystems. Stream ciphers, on the other hand, process plaintext in small blocks (sometimes as small as a single bit). In contrast to block ciphers, stream ciphers keep some sort of memory, or state, as it processes the plaintext and uses this state as an input to the cipher algorithm \[14-15\]. In this paper, we are proposed a new scheme stream cipher contain magic matrix \[3, 6, 7, 9, 10\], to strong a random access file that you already to protect it. Section 2 will describe what kind of cipher schema did we concept in practice. Section 3 is discussing magic square that it will apply in random access file while encryption already implements early. Part 4, it is analysis the reliability with decrypt and encrypt; Conclusion and future research will be mention in final section.

上传的附件

| ![](img/d5c539bacaef0c9233cc56abc062f348.jpg) | A Study on the Stream Cipher Embedded Magic Square of Random Access Files.pdf |
| :--- | :--- |
