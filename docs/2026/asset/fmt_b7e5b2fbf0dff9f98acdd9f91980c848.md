
```
Process 70 stopped

opendirectoryd`crypt_verify:

-> 0x104243f3c <+965>: callq 0x104249e8a; symbol stub for: strcmp

(lldb) x/s $rdi

0x70000665eec0: "*.dAJ47YHEIRE"

(lldb) x/s $rsi

0x7fcb0ddcb851: "*"
```

这些字符串显然不匹配，所以`strcmp`函数不会成功（即返回值不会 `== 0x0`）。因此，我们正在跟踪的被传入的参数不会被设置为`0x0`。

在这一点上，我们对这个参数的作用有了清晰的认识。它是一个指向一个变量的指针，当且仅当密码（散列）匹配时，才从`crypt_verify`函数中设置为`0x0`的`od_verify_crypt_password`传入。因此，我们可以想象下面的伪代码：

```
//verify

// 'match' will be set to 0x0 if verification is ok!
int match = kODErrorCredentialsInvalid;

od_verify_crypt_password(accountHash, providedPassword, &match, ...);
....

//verify by checking hashes
// 'match' will be set to 0x0 if verification is ok!
if(strcmp(providedPWHash, accountPWHash, user) == 0x0) {
   *match = 0x0;
}
```

正如我们前面指出的，只有`od_verify_crypt_password`的返回值会被检查……而不是实际验证的结果！（即所谓“匹配”的变量，`var_54`）。

这可以通过检查以下的反编译来确认，这段反编译代码显示了对`od_verify_crypt_password`的调用。请注意，所谓“匹配”变量（`var_54`）在调用之后从未被检查。相反，升级函数（`sub_13d00`，`sub_14324`）被错误地调用：

![](https://bbs.kanxue.com/upload/attach/201801/697506_G75C75P6BAY9ZD4.jpg)

## 苹果官方的回应

发布这个博客不久之后，苹果发布了一个针对macOS 10.13和10.13.1的补丁。该补丁可以从苹果的支持网站直接下载：

![](https://bbs.kanxue.com/upload/attach/201801/697506_FVMQYZEWQK75ZFJ.jpg)

或者，它应该自动显示为安全更新（在macOS应用商店中）：

![](https://bbs.kanxue.com/upload/attach/201801/697506_TYVV4KBCZBM2REM.jpg)

该错误被分配为CVE-2017-13872，苹果在安全发布说明中指出，这仅仅是“证书验证中存在的逻辑错误”。他们指出，他们的补丁“改进了凭证验证”。

你可能想知道他们是如何修补这个bug的？……我们在这个博客中发现的根本问题是正确的吗？

比较未打补丁和修补的`PlistFile`二进制文件，我们可以看到苹果添加了代码来检测无效的凭据（即，当未经身份验证的攻击者试图设置root密码时）：

![](https://bbs.kanxue.com/upload/attach/201801/697506_BMHF62SPN2B87TA.jpg)

特别要说的是，如预期的那样，他们现在在调用`od_verify_crypt_password`之后也会检查所谓“匹配”变量的结果：

```
lea rbx, qword [rbp+var_54]    ;load addr of 'match' in rbx
mov rcx, rbx                   ;move into arg4 for call
call imp___stubs__od_verify_crypt_password

mov ecx, dword [rbx] ;get value of 'match'
test ecx, ecx        ;is it 0x0?
jne noMatch          ;no, then bail!
```

因此，我们的分析是正确的！

不幸的是，与其他的苹果补丁一样，这个补丁似乎还带来了一些严重的问题。首先，它破坏了不同用户的文件共享：

![](https://bbs.kanxue.com/upload/attach/201801/697506_D9B7HHZM6865GK7.jpg)

正如我指出的（谢谢@alvarnell），这种不兼容性很快就被一个新的补丁修复了（将构建带到`17C1003`）。

更糟糕的是，据Wired报道，如果macOS 10.13上的用户使用该补丁，然后升级到macOS 10.13.1，则会重新引入该错误：

![](https://bbs.kanxue.com/upload/attach/201801/697506_FW7FU8B6DYGBZ5V.jpg)

聪明的Pepijn Bruienne（@bruienne）指出，这可能是由于苹果公司既“没有碰撞内部版本号”，也没有“将`#iamroot` bug的补丁放到10.13.1中”：

![](https://bbs.kanxue.com/upload/attach/201801/697506_KCHZW72TZPM7TV2.jpg)

用户还报告说，在应用补丁后，需要重新启动！

我的好朋友托马斯·里德（@thomasareed）发表了一篇很好的文章，全面总结了苹果公司在这个补丁上的失误。

## 结论

那么，这是一个总结！在本博客中，为了在Apple发布补丁之前揭示时下臭名昭著的`#iamroot` bug的根本原因，我们逆向了`opendirectory`守护进程的各种组件！

我们确定苹果公司忘记检查一个必要的变量（保存有一个账户验证结果）的值。

一旦一个补丁被发布，我们将会逆向它，以确认我们的发现是正确的。万岁！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
