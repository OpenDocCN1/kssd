
```
        string lString6;

    begin

//sice的序列号形如xxxx-xxxxxx-xx，保存在注册表里的也是这个格式。

label190:

0067BE:002F:        StrLength(pString0);

0067C3:0021:        lNumber1 = LAST_RESULT;

0067CB:0128:        lNumber1 = lNumber1 != 14;

0067DD:0022:        if (lNumber1 = 0) then

  goto label192;

                    endif;

0067EB:012F:        return(0);

//这里我怀疑有编译错误，应该是如果序列号长14则序次执行到这里，否则返回0。

0067F8:0030:        StrSub(lString1, pString0, 0, 4);

00680A:0030:        StrSub(lString2, pString0, 5, 6);

00681C:0030:        StrSub(lString3, pString0, 12, 2);

//从注册表里的序列号中取除“-”号外的子串。

00682E:0124:        lString6 = lString1 + lString2;

006839:0124:        lString0 = lString6 + lString3;

//再拼起来，也就是相当于序列号去掉“-”号。

//下面仍旧怀疑有误。

006844:0031:        StrFind(lString0, "-");

00684D:0128:        lNumber1 = LAST_RESULT >= 0;

00685F:0022:        if (lNumber1 = 0) then

  goto label193;

                    endif;

00686D:012F:        return(0);

label192:

00687A:0125:        lString5 = SUPPORTDIR ^ "UTILITY.dll";

006890:00B2:        UseDLL(lString5);

//装载utility.dll来准备进行进一步的序列号检验。

006895:0021:        lNumber1 = LAST_RESULT;

00689D:0128:        lNumber1 = lNumber1 = 0;

0068AF:0022:        if (lNumber1 = 0) then

  goto label194;

                    endif;

0068BD:00B4:        UTILITY.DigitCheck(lString0);

  //这一步是关键检验过程。

0068C5:0021:        lNumber0 = LAST_RESULT;

0068CD:00B3:        UnUseDLL(lString5);

label193:

0068D6:0128:        lNumber1 = lNumber0 = 1;

0068E8:0022:        if (lNumber1 = 0) then

  goto label195;

                    endif;

0068F6:012F:        return(1);

      //序列号正确则返回1。

0068FF:002C:        goto label196;

label194:

006908:012F:        return(0);

label195:

006915:00B8:        return;

    end;
```

这段说明，SOFTICE安装时会读取注册表中的序列号信息（如果有的话）并且进行判断，这一段的机制与效果和检验输入的序列号是否正确是一样的。

现在就`UTILITY.DigitCheck(lString0);`是关键了，参数是去掉了“-”号的序列号。

SOFTICE安装程序运行时会在`windows/temp`目录下生成一些临时的目录和文件，其中就包括刚才提到的`utility.dll`，下面是其`DigitCheck`的输出代码。

跟踪过程也不算太复杂，先填序列号，我填的是5100-0099BB-BB，下断`bpx loadlibrarya do "d esp->8"`看载入的DLL文件名，等按“Next”按钮的时候便会装入`UTILITY.DLL`，因此断在系统领空，返回后看看`MOD UTILITY`，我机子上的`BASE`是1BC0000，因此下面的地址都有一步重定位的变换，为了简明起见，我使用1000000做基址，这与W32DASM的反汇编代码相符合。

`digitcheck`的入口是1110，因此下断`bpx 0167:1bc1110`，断后便开始了漫长的跟踪过程。（见下篇）
