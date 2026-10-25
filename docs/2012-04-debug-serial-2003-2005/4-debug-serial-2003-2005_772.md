
# [原创]PC 定时执行专家 1.3.3.破解过程

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-256.htm`](https://www.pediy.com/kssd/pediy07/pediy7-256.htm)

【破解作者】 `lee`

【作者邮箱】 `cracker_lee@126.com`

【使用工具】 `OD`, `PEID`, `IDA`

【破解平台】 `WinXP`

【软件名称】 `PC 定时执行专家`

【软件简介】 `PC 定时执行专家( PCTimerExpert )`是一个 `Visual C++` 制作，简单易用的“定时器”软件。有“定时关机”、“定时日程提醒”等功能。除了具有“`PC 定时关机精灵( PCTaskTimer )`”的全部功能之外，还将会有更多的新功能加入。它是绿色的共享软件，无需安装，欢迎下载试用。注意未注册用户不能使用全部功能，需要注册才能使用的功能请参考本页的“主要功能和特点”。付费用户可享受技术支持和升级服务。

`Unicode` 版主要针对使用外文环境的用户，这个版本可是让你在英文、日文等外文的 `Windows` 系统下能正常地同时显示、输入外文和中文。

`PC 定时执行专家( PCTimerExpert )`的由来：一直用着一个别人写的一个自动关机小程序，觉得界面有些难看，而且在日文的环境下有乱码的问题。索性自己写了一个完全英文的 `PCTimer` 小工具。只有 3 个功能：注销、重启系统、关机。后来，觉得还可以加些功能，就成了现在的 `PC 定时执行专家( PCTimerExpert )`。

【软件大小】 `132k`

【加壳方式】 无

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

【破解内容】

`peid` 查知道该软件无壳 `Microsoft Visual C++ 6.0` 编写，`OD` 载入：

`bp GetWindowTextA`

```assembly
0040410C    .  68 0A040000 push 40A
00404111    .  C74424 20 0>mov dword ptr ss:[esp+20],0
0404119    .  E8 F24A0000 call <jmp.&MFC42.#3097>             // 中断后返回到这里
0040411E    .  8B4424 08   mov eax,dword ptr ss:[esp+8]
00404122    .  8B48 F8     mov ecx,dword ptr ds:[eax-8]       // 取得输入的长度放入 ECX
00404125    .  85C9        test ecx,ecx
00404127    .  75 0B       jnz short PCTimerE.00404134        //判断是否为空，如果不为空的话，就跳到 00404134 出将字符串转化为对应的整数
00404129    .  6A FF       push -1
0040412B    .  6A 40       push 40
0040412D    .  6A 78       push 78
0040412F    .  E9 C5000000 jmp PCTimerE.004041F9              //如果字符为空就直接跳到注册失败出
00404134    >  50          push eax                             ; /s
00404135    .  FF15 B0B540>call dword ptr ds:[<&MSVCRT.atol>] ; \atol
0040413B    .  8BF0        mov esi,eax                         //将转化的结果放入 ESI 中，以便后面做比较
0040413D    .  83C4 04     add esp,4
00404140    .  85F6        test esi,esi
00404142    .  0F84 AB0000>je PCTimerE.004041F3
00404148    .  6A 00       push 0                               ; /pFileSystemNameSize = NULL
0040414A    .  6A 00       push 0                               ; |pFileSystemNameBuffer = NULL
0040414C    .  6A 00       push 0                               ; |pFileSystemFlags = NULL
0040414E    .  8D4C24 18   lea ecx,dword ptr ss:[esp+18]       ; |
00404152    .  6A 00       push 0                               ; |pMaxFilenameLength = NULL
00404154    .  51          push ecx                             ; |pVolumeSerialNumber
00404155    .  6A 00       push 0                               ; |MaxVolumeNameSize = 0
00404157    .  6A 00       push 0                               ; |VolumeNameBuffer = NULL
00404159    .  68 D4F24000 push PCTimerE.0040F2D4               ; |RootPathName = "c:\\"
0040415E    .  C74424 2C 0>mov dword ptr ss:[esp+2C],0         ; |
00404166    .  FF15 70B040>call dword ptr ds:[<&KERNEL32.Ge; \GetVolumeInformationA //取序号
0040416C    .  85C0        test eax,eax
0040416E    .  75 11       jnz short PCTimerE.00404181
00404170    .  50          push eax
00404171    .  6A 30       push 30
00404173    .  68 B0F24000 push PCTimerE.0040F2B0
00404178    .  E8 AB4A0000 call <jmp.&MFC42.#1200>             ; AfxMessageBox
0040417D    .  33C0        xor eax,eax
0040417F    .  EB 15       jmp short PCTimerE.00404196
00404181    >  8B4C24 0C   mov ecx,dword ptr ss:[esp+C]        //取序号成功后就跳到这里，并将序号放入 ECX
00404185    .  8BD1        mov edx,ecx
00404187    .  8BC1        mov eax,ecx                         //将结果放入同时放入到 EDX 和 EAX 方便后面的取高 16 位和低 16 位
00404189    .  C1EA 10     shr edx,10                          //取高 16 位
0040418C    .  2BC2        sub eax,edx                         //减去高 16 位的值
0040418E    .  81E1 FFFF00>and ecx,0FFFF                       //取低 16 位
00404194    .  2BC1        sub eax,ecx                         //减去低 16 位的值
00404196    >  8B15 94C740>mov edx,dword ptr ds:[40C794]      //在取一个常数 34D944EA
0040419C    .  8BC8        mov ecx,eax                         //上面的结果也保存到 ECX 中
0040419E    .  C1E9 10     shr ecx,10                          //取结果的高 16 位
004041A1    .  33D0        xor edx,eax                         //将 34D944EA 与上面的结果异或
004041A3    .  25 FFFF0000 and eax,0FFFF                       //取结果的低 16 位
004041A8    .  2BD1        sub edx,ecx                         //在减去高 16 位
004041AA    .  2BD0        sub edx,eax                         //在减去低 16 位，此时 EDX 就是注册码
004041AC    .  33C0        xor eax,eax
004041AE    .  3BD6        cmp edx,esi                         //关键比较将用户输入的注册码与正确的注册码比较
004041B0    .  0F94C0      sete al
004041B3    .  85C0        test eax,eax
004041B5    .  74 3C       je short PCTimerE.004041F3          //如果前面的不相等的话，就跳到注册失败的地方
004041B7    .  E8 32480000 call <jmp.&MFC42.#1168>             ;AfxGetModuleState(void)
004041BC    .  8B40 04     mov eax,dword ptr ds:[eax+4]
004041BF    .  56          push esi
004041C0    .  68 8CF24000 push PCTimerE.0040F28C               ;  ASCII "REGCODE"
004041C5    .  68 7CF24000 push PCTimerE.0040F27C               ;  ASCII "Registration"
004041CA    .  8BC8        mov ecx,eax
004041CC    .  E8 5D4A0000 call <jmp.&MFC42.#6402>             ;WriteProfileInt(char const \*,char const \*,int)
004041D1    .  85C0        test eax,eax
004041D3    .  74 15       je short PCTimerE.004041EA
004041D5    .  6A FF       push -1
004041D7    .  6A 40       push 40
004041D9    .  6A 79       push 79
004041DB    .  E8 D8470000 call <jmp.&MFC42.#1199>             ;AfxMessageBox(“注册成功”)
004041E0    .  C705 30FB40>mov dword ptr ds:[40FB30],1
004041EA    >  8BCB        mov ecx,ebx
004041EC    .  E8 194A0000 call <jmp.&MFC42.#4853>             ;CDialog::OnOK(void)
004041F1    .  EB 0B       jmp short PCTimerE.004041FE
004041F3    >  6A FF       push -1
004041F5    .  6A 40       push 40
004041F7    .  6A 7A       push 7A
004041F9    >  E8 BA470000 call <jmp.&MFC42.#1199>             ;  AfxMessageBox("注册失败")
004041FE    >  8D4C24 08   lea ecx,dword ptr ss:[esp+8]
00404202    .  C74424 18 F>mov dword ptr ss:[esp+18],-1
0040420A    .  E8 B5470000 call <jmp.&MFC42.#800>
0040420F    .  8B4C24 10   mov ecx,dword ptr ss:[esp+10]
00404213    .  5E          pop esi
00404214    .  5B          pop ebx
00404215    .  64:890D 000>mov dword ptr fs:[0],ecx
0040421C    .  83C4 14     add esp,14
0040421F    .  C3          retn
```

---

总结一下：

系统首先 `GetWindowText` 取到假注册码并转化为对应的整数后放入 `ESI` 中，接着用

`GetVolumeInformationA` 取到序号后，我的为 `421E3A6F`，接着用这个值减去它的高 16 位和低 16 位，即 `421E3A6F-421E-3A6F=421DBDE2`。

接着用一个常数（`34D944EA`）与上面的结果异或，再减去上面结果的高 16 位和低 16 位，即 `34D944EA^421DBDE2-421D-BDE2=76C3F909`（十进制为 `1992554761`）。

所以我的电脑上注册码就是 `1992554761`。

---

注册机的大致编写过程：

```c
DWORD m_sumber;
CString str;
GetVolumeInformation("c:\\", NULL, 0, &m_sumber, NULL, NULL, NULL, NULL);

// 将 m_sumber（10 进制整型）转化为 16 进制整型
__asm
{
    push eax
    push ecx
    push edx
    mov ecx, m_sumber
    mov edx, ecx
    mov eax, ecx
    shr edx, 10H
    sub eax, edx
    and ecx, 0FFFFH
    sub eax, ecx
    mov edx, 34D944EAH
    mov ecx, eax
    shr ecx, 10H
    xor edx, eax
    and eax, 0FFFFH
    sub edx, ecx
    mov m_sumber, edx
    pop edx
    pop ecx
    pop eax
}

// 将 m_sumber（16 进制整型）转化为 10 进制整型
str.Format("%ld", m_sumber);
m_edit.SetWindowText(str);
}
```

---

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
