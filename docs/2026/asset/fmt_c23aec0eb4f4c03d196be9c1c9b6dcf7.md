
```
CODE:00466B30                 push    ebp
  CODE:00466B31                 mov     ebp, esp
  CODE:00466B33                 xor     ecx, ecx
  CODE:00466B35                 push    ecx
  CODE:00466B36                 push    ecx
  CODE:00466B37                 push    ecx
  CODE:00466B38                 push    ecx
  CODE:00466B39                 push    ecx
  CODE:00466B3A                 push    ebx
  CODE:00466B3B                 push    esi
  CODE:00466B3C                 mov     ebx, eax
  CODE:00466B3E                 xor     eax, eax
  CODE:00466B40                 push    ebp
  CODE:00466B41                 push    offset loc_466C50
  CODE:00466B46                 push    dword ptr fs:[eax]
  CODE:00466B49                 mov     fs:[eax], esp
  CODE:00466B4C                 lea     edx, [ebp+var_4]
  CODE:00466B4F                 mov     eax, [ebx+34Ch]
  CODE:00466B55                 call    @Controls@TControl@GetText$qqrv ; 从输入框中获取用户名
  CODE:00466B5A                 mov     eax, [ebp+var_4]
  CODE:00466B5D                 push    eax
  CODE:00466B5E                 lea     edx, [ebp+var_8]
  CODE:00466B61                 mov     eax, [ebx+344h]
  CODE:00466B67                 call    @Controls@TControl@GetText$qqrv ; 从输入框中获取序列号
  CODE:00466B6C                 mov     eax, [ebp+var_8]
  CODE:00466B6F                 push    eax
  CODE:00466B70                 lea     edx, [ebp+var_C]
  CODE:00466B73                 mov     eax, [ebx+33Ch]
  CODE:00466B79                 call    @Controls@TControl@GetText$qqrv ; 从输入框中获取注册码
  CODE:00466B7E                 mov     eax, [ebp+var_C] ; 取用户名到eax
  CODE:00466B81                 pop     edx             ; 取注册码到edx
  CODE:00466B82                 pop     ecx             ; 取序列号到ecx
  CODE:00466B83                 call    sub_466024
  CODE:00466B88                 test    al, al
  CODE:00466B8A                 jnz     short loc_466BB1
```

用OD跟到`00466B83`处的`call sub_466024`里看看：

```
CODE:00466024                 push    ebp
  CODE:00466025                 mov     ebp, esp
  CODE:00466027                 add     esp, 0FFFFFFE4h
  CODE:0046602A                 push    ebx
  CODE:0046602B                 push    esi
  CODE:0046602C                 push    edi
  CODE:0046602D                 xor     ebx, ebx
  CODE:0046602F                 mov     [ebp+var_4], ebx
  CODE:00466032                 mov     [ebp+var_18], ebx
  CODE:00466035                 mov     [ebp+var_1C], ebx
  CODE:00466038                 mov     edi, ecx        ; 取注册码到edi
  CODE:0046603A                 mov     esi, edx        ; 取序列号到esi
  CODE:0046603C                 mov     ebx, eax        ; 取用户名到ebx
  CODE:0046603E                 xor     eax, eax        ; 清零eax
  CODE:00466040                 push    ebp
  CODE:00466041                 push    offset loc_4660B0
  CODE:00466046                 push    dword ptr fs:[eax]
  CODE:00466049                 mov     fs:[eax], esp
  CODE:0046604C                 push    offset _str_ACTIVEARP_0.Text ; 压入字符串“activearp”
  CODE:00466051                 push    ebx             ; 压入用户名
  CODE:00466052                 push    esi             ; 压入序列号
  CODE:00466053                 lea     eax, [ebp+var_1C]
  CODE:00466056                 mov     edx, 3
  CODE:0046605B                 call    @System@@LStrCatN$qqrv ; 连接字符串“activearp+用户名+序列号”
  CODE:00466060                 mov     eax, [ebp+var_1C] ; 将得到的字符串放到eax的
  CODE:00466063                 lea     edx, [ebp+var_18]
  CODE:00466066                 call    @Sysutils@UpperCase$qqrx17System@AnsiString ; 将刚才得到的字符串转为大写
  CODE:0046606B                 mov     eax, [ebp+var_18] ; 将转换后到的字符串放到eax的
  CODE:0046606E                 lea     edx, [ebp+var_14]
  CODE:00466071                 call    sub_466624      ; 计算注册码
  CODE:00466076                 lea     eax, [ebp+var_14]
  CODE:00466079                 lea     edx, [ebp+var_4]
  CODE:0046607C                 call    sub_466584      ; 将生成的注册码转换为字符串
  CODE:00466081                 mov     eax, [ebp+var_4] ; 放正确序列号到eax
  CODE:00466084                 mov     edx, edi        ; 放我们输入的序列号到edx
  CODE:00466086                 call    unknown_libname_111 ; 比较注册码
  CODE:0046608B                 mov     ebx, eax
  CODE:0046608D                 xor     eax, eax
  CODE:0046608F                 pop     edx
  CODE:00466090                 pop     ecx
  CODE:00466091                 pop     ecx
  CODE:00466092                 mov     fs:[eax], edx
  CODE:00466095                 push    offset loc_4660B7
  CODE:0046609A
  CODE:0046609A loc_46609A:                             ; CODE XREF: sub_466024+91 j
  CODE:0046609A                 lea     eax, [ebp+var_1C]
  CODE:0046609D                 mov     edx, 2
  CODE:004660A2                 call    @System@@LStrArrayClr$qqrpvi ; System::__linkproc__ LStrArrayClr(void *,int)
  CODE:004660A7                 lea     eax, [ebp+var_4]
  CODE:004660AA                 call    @System@@LStrClr$qqrpv ; System::__linkproc__ LStrClr(void *)
  CODE:004660AF                 retn
```

我们可以看到`CODE:00466071`处的`call sub_466624`就是计算注册码的地方，其实就是将上面得到的长字符串加密成32位的MD5。

算法就是这样，现在来写个注册机吧。

下面给出关键代码：

```
    CMd5 md5;
      UpdateData(true);
      string key("");
      string temp("");
      key.append("activearp");
      key.append(m_user);
      key.append(m_sn);
      m_key=key.data();
      m_key.MakeUpper();
      md5.TargetStr(m_key.GetBuffer(0));
      temp=md5.GetDigestKey();
      m_key=temp.data();
      UpdateData(false);
```

![](http://www.hackerm.com.cn/attachment/arpkeygen.jpg)

下载地址：cbfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2j5$3E0W2M7X3#2Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8X3q4@1N6r3q4U0K9r3#2W2L8Y4c8Q4x3V1k6S2M7Y4m8C8k6i4W2Y4k6h3&6Q4x3X3g2J5j5i4t1.

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

                                                       2009年03月22日 23:12:14

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
