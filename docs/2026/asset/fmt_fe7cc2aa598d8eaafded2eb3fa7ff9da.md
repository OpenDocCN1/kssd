
# [原创]高效e人 Ver2.98(Build 243) 算法分析

- 【破文标题】高效e人 Ver2.98(Build 243) 算法分析
- 【破文作者】Blackk
- 【作者邮箱】191477631@qq.com
- 【作者主页】00ad9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8V1u0D9j5h3y4C8K9#2)9J5k6h3y4G2i4K6u0W2j5$3x3\`.
- 【破解工具】OD
- 【破解平台】XP SP3
- 【软件名称】高效e人 Ver2.98(Build 243)
- 【软件大小】3.96 MB
- 【原版下载】11eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6S2L8%4S2A6j5h3!0W2M7X3g2F1i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3X3g2Z5N6r3@1\`.
- 【破解声明】本解密分析文章仅限用于学习和研究目的，不得将上述内容用于商业或者非法用途，否则，一切后果请用户自负。如果您喜欢该程序，请支持正版软件，购买注册，得到更好的正版服务。

---

## 破解过程

`bp MessageBoxExA`

### 算法入口

```
00B4378C  /$  55            push    ebp
00B4378D  |.  8BEC          mov     ebp, esp
00B4378F  |.  33C9          xor     ecx, ecx
00B43791  |.  51            push    ecx
00B43792  |.  51            push    ecx
00B43793  |.  51            push    ecx
00B43794  |.  51            push    ecx
00B43795  |.  51            push    ecx
00B43796  |.  53            push    ebx
00B43797  |.  8BD8          mov     ebx, eax
00B43799  |.  33C0          xor     eax, eax
00B4379B  |.  55            push    ebp
00B4379C  |.  68 A038B400   push    00B438A0
00B437A1  |.  64:FF30       push    dword ptr fs:[eax]
00B437A4  |.  64:8920       mov     dword ptr fs:[eax], esp
00B437A7  |.  8D55 F8       lea     edx, dword ptr [ebp-8]
00B437AA  |.  8BC3          mov     eax, ebx
00B437AC  |.  E8 53010000   call    00B43904                         ;  取用户名
00B437B1  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
00B437B4  |.  E8 CB198CFF   call    00405184                         ;  取用户名长度
00B437B9  |.  85C0          test    eax, eax
00B437BB  |.  75 3E         jnz     short 00B437FB                   ;  验证是否输入用户名
00B437BD  |.  A0 AC38B400   mov     al, byte ptr [B438AC]
00B437C2  |.  50            push    eax
00B437C3  |.  B2 01         mov     dl, 1
00B437C5  |.  A1 08179400   mov     eax, dword ptr [941708]
00B437CA  |.  E8 256DE0FF   call    0094A4F4
00B437CF  |.  8D55 F4       lea     edx, dword ptr [ebp-C]
00B437D2  |.  E8 6962E0FF   call    00949A40
00B437D7  |.  8B55 F4       mov     edx, dword ptr [ebp-C]
00B437DA  |.  33C9          xor     ecx, ecx
00B437DC  |.  A1 E87D8800   mov     eax, dword ptr [887DE8]
00B437E1  |.  E8 3A59D4FF   call    00889120
00B437E6  |.  8B93 28030000 mov     edx, dword ptr [ebx+328]
00B437EC  |.  A1 347D8800   mov     eax, dword ptr [887D34]
00B437F1  |.  E8 8A51D4FF   call    00888980
00B437F6  |.  E8 F1EE8CFF   call    004126EC
00B437FB  |>  E8 D0EAFCFF   call    00B122D0
00B43800  |.  8945 FC       mov     dword ptr [ebp-4], eax
00B43803  |.  33C0          xor     eax, eax
00B43805  |.  55            push    ebp
00B43806  |.  68 7E38B400   push    00B4387E
00B4380B  |.  64:FF30       push    dword ptr fs:[eax]
00B4380E  |.  64:8920       mov     dword ptr fs:[eax], esp
00B43811  |.  8D55 F0       lea     edx, dword ptr [ebp-10]
00B43814  |.  8BC3          mov     eax, ebx
00B43816  |.  E8 95000000   call    00B438B0                         ;  取注册码
00B4381B  |.  8B55 F0       mov     edx, dword ptr [ebp-10]
00B4381E  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
00B43821  |.  E8 86C9E0FF   call    009501AC                         ;  算法CALL
```

### 算法CALL
