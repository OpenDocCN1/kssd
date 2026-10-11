
```
int WINAPI WinMain (HINSTANCE hInstance, HINSTANCE hPrevInstance, PSTR szCmdLine, int iCmdShow)
{
    DialogBoxA(hInstance, MAKEINTRESOURCE(DLG_MAIN), 0, DlgP);
    return 0;
}
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`KeyGen.RC`\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\`\`\`rc
111 DIALOG 24, 50, 210, 119

STYLE DS_ABSALIGN | DS_MODALFRAME | 0xA04L | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU

CAPTION "KeyGen By 娃娃/[CCG]"

FONT 9, "宋体"

LANGUAGE LANG_FRENCH, SUBLANG_NEUTRAL

{
    CONTROL "", 1009, "EDIT", ES_LEFT | ES_AUTOHSCROLL | ES_OEMCONVERT | WS_CHILD | WS_VISIBLE | WS_BORDER | WS_TABSTOP, 4, 13, 202, 12
    CONTROL "(&G)生成", 1008, "BUTTON", BS_PUSHBUTTON | WS_CHILD | WS_VISIBLE | WS_TABSTOP, 7, 101, 45, 13
    CONTROL "(&X)退出", 2, "BUTTON", BS_PUSHBUTTON | WS_CHILD | WS_VISIBLE | WS_TABSTOP, 59, 101, 45, 13
    CONTROL "", 1010, "EDIT", ES_LEFT | ES_AUTOHSCROLL | ES_READONLY | WS_CHILD | WS_VISIBLE | WS_BORDER | WS_TABSTOP, 4, 73, 202, 12
    CONTROL "用户名称:", -1, "STATIC", SS_LEFT | WS_CHILD | WS_VISIBLE | WS_GROUP, 5, 4, 36, 8
    CONTROL "您的注册码:", -1, "STATIC", SS_LEFT | WS_CHILD | WS_VISIBLE | WS_GROUP, 6, 64, 60, 8
    CONTROL "组织名称:", -1, "STATIC", SS_LEFT | WS_CHILD | WS_VISIBLE | WS_GROUP, 6, 32, 64, 8
    CONTROL "", 1011, "EDIT", ES_LEFT | ES_AUTOHSCROLL | ES_OEMCONVERT | WS_CHILD | WS_VISIBLE | WS_BORDER | WS_TABSTOP, 4, 41, 202, 12
}
\`\`\`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

好久都没有写这种破解文章了 看着以前破解界的朋友现在或忙于工作或投身于编程或者逆向工程 也不知道自己心里是因为技不如人而感到羞愧 还是因为失去一些可以互相交流和帮助的老朋友而感到惋惜 看雪论坛陪我度过了 3 年的时光 无数精彩的回忆都留在这里 不免有的时候想想再过三年之后的看雪论坛 想想三年之后的\[CCG\] 又或者想想三年之后的我 …… …… 无限惆怅

不好意思 一时间有感而发 说多了

还是那句话：仅以此文献给我可爱的组织 CCG 希望他能蒸蒸日上

作者：娃娃 QQ:15110296 EMAil:Nydoll@etang.com

属于中国破解组织 CCG(CHiNA CrACKiNG GrOUp)

写于 2002 年 11 月 8 日

\*/转贴请保持完整 并写明出处:"看雪软件调试论坛" 作者:"娃娃[CCG]"/\*

`4dd466d0e1e0cc123afbe504da2462ca`
