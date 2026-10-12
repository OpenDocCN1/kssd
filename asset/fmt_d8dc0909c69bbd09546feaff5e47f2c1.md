
```
  00408548  /$  83C4 F8       add     esp, -8

  0040854B  |.  6A 00         push    0                            ; /Arg1 = 00000000

  0040854D  |.  894424 04     mov     dword ptr [esp+4], eax      ; |eax，就是上面常数减去运算后的结果

  00408551  |.  C64424 08 00  mov     byte ptr [esp+8], 0         ; |

  00408556  |.  8D4C24 04     lea     ecx, dword ptr [esp+4]      ; |

  0040855A  |.  8BC2          mov     eax, edx                    ; |

  0040855C  |.  BA 74854000   mov     edx, 00408574               ; |ASCII "%d"//看到这个没有，转化为十进制

```

拿出计算器，呵呵，还真的是这样，就是真的注册码。

```
  00408561  |.  E8 9E0B0000   call    00409104                    ; \Scbook.00409104

  00408566  |.  59            pop     ecx

  00408567  |.  5A            pop     edx

  00408568  \.  C3            retn
```

-----------------

分析关键 `CALL2`:

```
  00403DB0  /$  53            push    ebx

  00403DB1  |.  56            push    esi

  00403DB2  |.  57            push    edi

  00403DB3  |.  89C6          mov     esi, eax                    ;真的注册码

  00403DB5  |.  89D7          mov     edi, edx                    ;假的注册码

  00403DB7  |.  39D0          cmp     eax, edx                    ;真假较量

  00403DB9  |.  0F84 8F000000 je      00403E4E                    ;不相等就 Death
```

------------------

经验总结：

用户名的 `ASCII`

```
    C  C  D  e  a  t  h
```

`ASCII`  `43`  `43` `44` `65`  `61` `74` `68`  上面

位置    `1`   `2`  `3`  `4`   `5`  `6`  `7`   下面

上面与下面对应相乘，在相加 `ASCII(i)*i=43*1+43*2+ 44*3+65*4+64*5+74*6+68*7=00000A9E`

取出一个常数：`000BEAE0-00000A9E=000BE042`.在把 `000BE042` 转化为十进制就是 `778306`.好了 `778306` 就是真的注册码

------------------

先来写内存注册机如下---友情提示：真的注册码放在 `EAX` 上，不是 `EDX`。

中断地址：`0047EFAA`

中断次数：`1`

第一字节：`E8`

指令长度：`5`

------------------

我们来写带有漂亮界面，又有动态光标，又有音乐的注册机。我写的是上次发那篇注册机，这次的注册机就留给想学 `VC` 的思。

来。我看上次的算法：

取得机器码 8 位为：`Y21A5WQC`，接着把机器码转化为 16 进制字符（共 16 个），接着把这 16 个字符倒序。

取出此时机器码前四位为 `N1="3415"`，接着从第五位取出四个字符为 `N2="7553"`.

固定字符出场了 `C26P-Q618`，取出其前四位为 `N3="C26P"`，取出其后五位为 `N4="-Q618"`

开始组合为: `N3+'-'+N1+N4+'-'+N2 ="C26P-3415-Q618-7553"`

源代码：

```cpp
void CCCDeathDlg::OnButGen()
{
    UpdateData(true);
    CString  sMac;//存放机器码字符串
    CString  sMacHex;//存放机器码转化为 16 进制字符串,Hex 代表 16
    CString  sMacRev;//存放机器码倒序后的字符串,Rev 代表 Reverse
    CString  sTemp;//格式化中间字符串

    this->m_Mac.GetWindowText(sMac);//取得机器码
    if(sMac=="")
    AfxMessageBox("请复制机器码，机器码不能为空");

//============把机器码转化十六进制
    for(int i=0;i<sMac.GetLength();i++)
    {
        sTemp.Format("%x",sMac[i]);
        sMacHex= sMacHex+sTemp;

    }

//============把转化后的机器码进行倒序
    char *pstr;
    pstr=(LPSTR)(LPCTSTR)sMacHex;

    sMacRev=fun(pstr);
//============拆分
//1.对固定值进行拆分
    CString staValve="C26P-Q618";//staVale 代表：static valve

    CString staFirst=staValve.Left(4);//N3
    CString staEnd=staValve.Right(5);//N4
//2.对倒序的机器码进行拆分
    CString MacFirst=sMacRev.Left(4);//N1
    CString MacEnd=sMacRev.Mid(4,4);//N2
//=============开始进行连接
    CString sReg;

    sReg=staFirst+'-'+MacFirst+staEnd+'-'+MacEnd;
//最终成功生成注册码
    this->m_Reg.SetWindowText(sReg);
    AfxMessageBox("恭喜注册成功!欢迎使用 CCDeath 注册机");
    UpdateData(false);


}
```

后记：我也对 `VC` 进行编辑框字符串操作，一点了解都没有，一直网找呀，结果找不到资料和一些封装的函数。天呀，我头晕，只好自己看着 `MSDN` 重新写。调试了一天，终于到现在完成了。OK，Let's continue.....

这个软件早在 01 精华集有,但只给中文算法,什么都没有做,大家可以查查,所以我就重新分析....希望多多批评和建议...`CCDeath` 在此感激不尽...Good-night!...

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 09 月 29 日 1:01:43

`4dd466d0e1e0cc123afbe504da2462ca`
