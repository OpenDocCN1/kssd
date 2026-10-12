
```
/* 0001A5E9: 0021 */    function_396("MSG_ErrorSN");  

/* 0001A5FD: 0006 */    s31 = LAST_RESULT;  

/* 0001A607: 0021 */    function_428(-65534, "", s31, n0);  

/* 0001A61B: 0005 */    goto label_483;  

// : Jump Referenced(1):  

// :  0001A5DD,   

label_487:  

/* 0001A626: 0005 */    goto label_489;  

// : Jump Referenced(1):  

// :  0001A5C4,   

label_488:  

/* 0001A631: 0021 */    function_29(g_str27);  

/* 0001A63A: 0006 */    n9 = LAST_RESULT;  

/* 0001A644: 000D */    n9 = n9 == 0;            这里一定把 000D 改成 000E 不等于真码,  

就跳到正确的地方，  

 看到那个 MSG_ErrorSN 这里是报错，说你输入的注册码是                                

           错  

 误，改成 000E 就算输入错误的注册码也可以正确下一步  

                                          000D 是代表相等于，而 000E 是代表不等  

于  

                                          有兴趣的朋友可以参考看雪大哥网站的  

看雪精华 4 的  

                                          标 题:MAYA4.x 中英文双语菜单插件 b1  

版 破解 (3 千字)  

                                          发信人:boolking  

                                          自已参看呀！！！  

/* 0001A653: 0004 */    if(! n9) goto label_489;                              // norma  

l if  

/* 0001A65F: 0021 */    function_396("MSG_ErrorSN");  

/* 0001A673: 0006 */    s31 = LAST_RESULT;  

/* 0001A67D: 0021 */    function_428(-65534, "", s31, n0);  

/* 0001A691: 0005 */    goto label_483;  

// : Jump Referenced(2):  

// :  0001A626, 0001A653,   

label_489:  

/* 0001A69C: 0005 */    goto label_491;  

// : Jump Referenced(1):  

// :  0001A58D,   

label_490:  

/* 0001A6A7: 0021 */    function_29(g_str27);  

/* 0001A6B0: 0006 */    n9 = LAST_RESULT;  

/* 0001A6BA: 000D */    n9 = n9 == 0;                这里也要改它，一样的改  

，把 000D 改成 000E 就可以,简单吧！  

/* 0001A6C9: 0004 */    if(! n9) goto label_491;                              // norma  

l if  

/* 0001A6D5: 0021 */    function_396("MSG_ErrorSN");  

/* 0001A6E9: 0006 */    s31 = LAST_RESULT;  

/* 0001A6F3: 0021 */    function_428(-65534, "", s31, n0);  

/* 0001A707: 0005 */    goto label_483;  

// : Jump Referenced(2):  

// :  0001A69C, 0001A6C9,   

label_491:  

/* 0001A712: 0005 */    goto label_493;  

// : Jump Referenced(1):  

// :  0001A4BD,   

label_492:  

/* 0001A71D: 0021 */    function_532(g_str25, g_str26);  

/* 0001A729: 0021 */    function_531(g_str25);  

/* 0001A732: 0021 */    function_531(g_str26);  

/* 0001A73B: 000D */    n9 = n0 == 12;  

/* 0001A74A: 0004 */    if(! n9) goto label_493;                              // norma  

l if  

/* 0001A756: 0005 */    goto label_481;  

// : Jump Referenced(6):  

// :  0001A510, 0001A712, 0001A74A, 0001A835, 0001A885,   

// :  0001A906,   

label_493:  

/* 0001A761: 0006 */    s0 = "";  

/* 0001A76B: 0006 */    s1 = "";  

/* 0001A775: 0021 */    function_514(s0, s1, g_str9, 0);  

/* 0001A789: 0006 */    n0 = LAST_RESULT;  

/* 0001A793: 0029 */    StrSub(s28, g_str9, 1, 1);  

/* 0001A7A7: 0029 */    StrSub(s29, g_str9, 2, 1);  

/* 0001A7BB: 000D */    n9 = s28 == ":";  

/* 0001A7C9: 000E */    n10 = s29 != "\\";  

/* 0001A7D7: 0019 */    n9 = n9 && n10;  

/* 0001A7E4: 0004 */    if(! n9) goto label_494;                              // norma  

l if  

/* 0001A7F0: 0021 */    function_396("Dialog_PathInputWrong");  

/* 0001A80E: 0006 */    s31 = LAST_RESULT;  

/* 0001A818: 0021 */    function_603(s1, s31, g_str9);  

/* 0001A827: 0021 */    function_427(s1, -65534);  

/* 0001A835: 0005 */    goto label_493;  
```

破解总结用 `UltraEdit` 打开 `setup.inx` 按 `Ctrl+G` 输入要修改偏移值点确定

会看到这些:

把: `0001A644   00 0D 00 03 00 05 92 FF 05 92 FF 07 00`
改成: `00 0E 00 03 00 05 92 FF 05 92 FF 07 00`
还有: `0001A6BA     00 0D 00 03 00 05 92 FF 05 92 FF 07 00`
改成: `00 0D 00 03 00 05 92 FF 05 92 FF 07 00`

保存就 OK 了！

别怀疑就是这么简单

如有错误请指正

我算了好久还是算不出注册码,不知道我是笨还是干吗？

那位大侠算出注册码，请贴之呀！大家学习一下吗？

属于 `BCG`, `YCG`, `FCG`, `DFCG`, `DCM`

我现在正申请加入组织,谢谢~~~

`4dd466d0e1e0cc123afbe504da2462ca`
