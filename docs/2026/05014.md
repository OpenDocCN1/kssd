
# [原创]验证用`NTHASH`直接登录

见：[NTLM安全认证测试一下子就验证成功了](<https://bbs.kanxue.com/elink@83bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4H3M7r3q4D9L8q4)9J5k6h3u0D9L8$3N6Q4x3X3f1I4y4U0y4Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3M7%4c8S2N6r3W2U0i4K6u0r3x3e0x3%4x3o6M7^5x3U0l9@1x3U0l9I4y4o6b7I4x3U0x3%4y4o6x3K6x3o6y4Q4x3V1k6Q4x3@1k6F1k6i4N6r3L8$3I4D9L8%4N6n7L8r3!0Y4)>)

主机上有个帐号 `trd`，密码 `1122`，他的 `nthash` 为 `9c be f6 79 93 aa e5 09 c5 9c fa 64 7c 8b 13 fb`。

假设现在虚拟机知道了主机的密码哈希，但是密码不清楚是什么来验证通过密码哈希也能成功！

之前已经知道密码哈希的计算是在 `lsass` 进程里面的 `msv1_0.dll` 模块的如下地方：

```text
001b:77c446e4 50              push    eax
001b:77c446e5 56              push    esi
001b:77c446e6 e869ffffff      call    msv1_0!SystemFunction007 (77c44654) 《===在这里面计算密码哈希
001b:77c446eb 8d4330          lea     eax,[ebx+30h]
```

这里通过虚拟机调试修改的办法替换输入的密码哈希使他验证成功。【编程的话注入hook即可】

用如下 `windbg` 脚本：

```text
*trd_hash.x
* 1122
* NTLM 9c be f6 79 93 aa e5 09 c5 9c fa 64 7c 8b 13 fb
bp 77c446eb ".echo =pass=;dS esi;.echo =pwd_hash=;db ebx+10h l10;.echo =1122_hash=;eb ebx+10h 9c be f6 79 93 aa e5 09 c5 9c fa 64 7c 8b 13 fb;g"
```

1、未导入脚本前任意密码测试

```text
d:\test>net use \\xpsp3 /user:trd 123456
发生系统错误 1326。
登录失败: 未知的用户名或错误密码。
```

2、导入脚本后再测试

```text
d:\test>net use \\xpsp3 /user:trd 123456
命令成功完成。
```

```text
============================log================================
nt!DbgBreakPointWithStatus+0x4:
804e45a2 cc              int     3
kd> $<c:\temp\dbg\trd_hash.x
kd> * 1122
kd> * NTLM 9c be f6 79 93 aa e5 09 c5 9c fa 64 7c 8b 13 fb
kd> bp 77c446eb ".echo =pass=;dS esi;.echo =pwd_hash=;db ebx+10h l10;.echo =1122_hash=;eb ebx+10h 9c be f6 79 93 aa e5 09 c5 9c fa 64 7c 8b 13 fb;g"
kd> g
=pass=
00d4f8d2  "123456"
=pwd_hash=
00d4f544  32 ed 87 bd b5 fd c5 e9-cb a8 85 47 37 68 18 d4  2..........G7h..
=1122_hash=
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
