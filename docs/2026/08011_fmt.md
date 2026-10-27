
# [原创]关于第七章的`SEH_stack.c`程序调试心得

请注意将`strcpy`函数提前到`__asm int 3`语句之前，否则在`0x002FE98`地址中将看不到`90909090`的数据，我当时就在这里被郁闷了一下午。

```
/***************************************************************************
      To be the apostrophe which changed "Impossible" into "I'm possible"!

POC code of chapter 7.2 in book "Vulnerability Exploit and Analysis Technique"

file name        : SEH_stack.c
author           : failwest
date             : 2007.07.04
description      : demo show of how SEH be exploited

Noticed          : 1 only run on windows 2000
                  2 complied with VC 6.0
                  3 build into release version
                  4 SEH offset and shellcode address may need
                    to make sure via runtime debug

version          : 1.0
E-mail           : failwest@gmail.com

        Only for educational purposes    enjoy the fun from exploiting :)
***************************************************************************/

#include <windows.h>

char shellcode[]=
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90";

DWORD MyExceptionhandler(void)
{
        printf("got an exception, press Enter to kill process!\n");
        getchar();
        ExitProcess(1);
}

void test(char * input)
{
        char buf[200];
        int zero=0;

        strcpy(buf,input); //overrun the stack
        __asm int 3 //used to break process for debug
        __try
        {
                //strcpy(buf,input); //overrun the stack
                zero=4/zero; //generate an exception
        }
        __except(MyExceptionhandler()){}
}

main()
{
        test(shellcode);
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
