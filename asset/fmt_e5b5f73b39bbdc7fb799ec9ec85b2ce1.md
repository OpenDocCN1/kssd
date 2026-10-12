
```c
  ReadFile(hFile, pBuffer, 512, &dwLen, NULL);
  
  p1 = (char*)pBuffer + 0x4f;
  
  p2 = (unsigned long*)p1;
  
  tmp1 =  *p2;
  
  p2++;
  
  tmp2 =  *p2;
  
  for (i = 0x20; i >= 1; i--)
  
  {
    
    tmp3 = i * edx0;
    
    sub(&tmp2, &tmp1, &esi8, &esic, tmp3);
    
    sub(&tmp1, &tmp2, &esi0, &esi4, tmp3);
  
  }
  
  *pwd = tmp1;
  
  pwd = pwd++;
  
  *pwd = tmp2;
  
  pwd++;
  
  *pwd = 0;
  
  printf("\t\t\t password is : %s \n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n", str1);
  
  CloseHandle(hFile);
  
  free(pBuffer);
  
}

void sub(unsigned long *a, unsigned long *b, unsigned long *a1,
unsigned long *b1, unsigned long c1)
{
  unsigned long higt, low, esi1, esi2, ebp0, edx1;
  
  edx1 = c1;
  
  higt =  *a;
  
  low =  *b;
  
  esi1 =  *a1;
  
  esi2 =  *b1;
  
  ebp0 = low << 4;
  
  higt = higt - ebp0;
  
  ebp0 = low ^ esi1;
  
  higt = higt - ebp0;
  
  ebp0 = low >> 5;
  
  ebp0 = ebp0 ^ edx1;
  
  higt = higt - ebp0;
  
  higt = higt - esi2;
  
  *a = higt;
  
}
```

至于还原精灵 6.0 以上版本及还原精灵网络版 6.x 的算法也是一样(至少目前为止),只是加密的字符串不一样.

例如:

解还原精灵网络版 6.02(目前应该算是最新版吧)的密码,只要将上面的代码中的字符串 `"NJYZ-RG-60-TT-60"` 替换为 `"NJZY-NetCard-V60"` 即可.

实际上解还原精灵 6.0 密码可以直接用内存注册机来解,在 `0041616C` 出中断,`EAX` 指向密码地址(一定要先结束掉还原精灵的进程),网络版的无此 BUG.

还原精灵各个版本一般都是把密码的明文或密文写在 0 磁道 8 扇区偏移量为 `04FH`,该扇区并没有什么特殊保护,可以直接用 `CreateFile` 进行读写.

为了便于理解,我把程序写成控制台程序(界面很简陋),大家有兴趣的话可以利用上面的代码,自己编个破解程序(记得把 `FIGO` 替换成自己的名字就行了 (^_^) ).

无论你对这篇文章有何看法,都非常感谢你有耐心看到这里.

由于权限不够不能上传附件,想要还原精灵 6.0 和 网络还原精灵 6.02 的密码读取程序源代码和实例请和我联系.

QQ :382174647

附件下载!

`4dd466d0e1e0cc123afbe504da2462ca`
