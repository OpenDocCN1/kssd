
# 从 QQ 进程内存中搜索出 QQ 号码

> 原文：[`www.pediy.com/kssd/pediy12/129164.html`](https://www.pediy.com/kssd/pediy12/129164.html)

代码很简单，纯属老菜业余级别，发上来与大家交流交流。

QQ 版本：`QQ2011Beta1(2021)`

内存特征码：`"index?uin="`

![](img/f1cafcdf443f0d3e2fe9557ec2132053.jpg)

字符串搜索采用的是仿 Sunday 算法：

代码:

```
int SearchStr(PTSTR pszString, int iStrLen, PTSTR pszSearchStr)
{
    int i = 0;
    int iSearchStrlen = _tcslen(pszSearchStr);

    while ((i + iSearchStrlen) <= iStrLen )
    {
        int n = 0;
        //先匹配两个字串的第一个字符
        if (pszSearchStr[0] == pszString[i])
        {
            //若相等，则开始逐字符匹配
            for (int j = 0; j < iSearchStrlen; j++)
            {
                //相同位置字符匹配
                if (pszSearchStr[j] == pszString[i + j])
                {
                    //若相同位置字符匹配成功，则计数器加 1
                    n++;
                }
                else  //相同位置字符匹配失败
                {
                    //源字符串位置跳过匹配相同的 n 个字符
                    i = i + n;
                    //跳出当前匹配循环，开始新位置的匹配
                    break;
                }
            }
            //若匹配成功，计数和目标字符串长度相等，则找到目标
            if (iSearchStrlen == n)
            {
                //i 为找到的目标字符串在源字符串中的起点位置，
                //此处 return，即找到的目标字符串首次出现位置
                return i;
                //找到一个目标，后移一位继续找，
                //如要继续找，请注意最后的 return，代码需做点小修改
                //i++;
            }
        }
        else  //若两字符串的第一个字符不同
        {
            //开始反向找源字符串相对目标字符串的后一个字符是否在目标字符串内
            for (int j = iSearchStrlen - 1; j >= 0; j--)
            {
                //找到存在紧跟其后的那个字符
                if (pszSearchStr[j] == pszString[i + iSearchStrlen])
                {
                    //该字符出现在目标字符串中的位置
                    n = j;
                    //只需知道排在倒数第一那个位置，跳出循环开始移动位置
                    break;
                }
            }
            //移动到位置为(一个目标串长度减去出现目标串中匹配字符出现的位置)，
            //即如果出现该字符，则使相同的两个字符对齐，若未出现，直接移动一个目标串长度
            i = i + iSearchStrlen - n;
        }
    }
    return 0;
}
```

完整代码见附件，用 `Code::Blocks` 整的，`unicode` 版。

上传的附件

| ![](img/31a2c422ae120fd0b6c241a4e6e83632.jpg) | `ReadQQ.7z` |
| :--- | :--- |

我也写过 = =3

代码:

```
#define QQNUMBER L"123456789"

VOID __stdcall Joker()
{
  ULONG fnGetSelfUin;
  ULONG currentQQ;

  fnGetSelfUin = (ULONG)GetProcAddress(GetModuleHandleA("KernelUtil"), "?GetSelfUin@Contact@Util@@YAKXZ");
  if ( fnGetSelfUin)
  {
    currentQQ = ((ULONG (__cdecl*())())fnGetSelfUin)();
    if ( currentQQ)
    {
      char buf[64];
      wsprintfA( buf, "QQ: %d", currentQQ);
      OutputDebugStringA( buf);
    }
  }
}
```
