
```
00FE10F3    68 00800000              push 8000  

00FE10F8    6A 00                    push 0  

00FE10FA    50                       push eax  

00FE10FB    FF95 7D294400            call dword ptr ss:\[ebp+44297D\]  

00FE1101    8D0E                     lea ecx,dword ptr ds:\[esi\]  

00FE1103    8551 2C                 test dword ptr ds:\[ecx+2C\],edx  

00FE1106    44                       inc esp  

00FE1107    07                       pop es  

00FE1108    50                       push eax  

00FE1109    C3                       retn  
```

--------------------------------------------------------------------------------

【版权声明】本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!
