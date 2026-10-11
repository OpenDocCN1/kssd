
```
switch (ul_reason_for_call)  
  {  
  case DLL_PROCESS_ATTACH:  
  case DLL_THREAD_ATTACH:  
  case DLL_THREAD_DETACH:  
  case DLL_PROCESS_DETACH:  
    break;  
  }  
  return TRUE;  
}
```

```
char cv(int c) // 将 16 进制数字转换为字符  
{  
  if(c>=0 && c<=9)  
    return c+'0';  
  else  
    return c-0xA+'A';  
}
```

```
// feat 为 feature，sign0 为正确的 SIGN，在内存中以 16 进行数字存放，需要转换为相应的字符串  
// 例如，内存数据为 0x11 0x23 0x45 0xAB 0xCD 0xEF，则对应的 SIGN 为“112345ABCDEF”  
// sign1 为输入的错误 SIGN  
FLEXLMDLL_API void FlexlmDll(char * feat,char *sign0,char *sign1)  
{  
  char s0[20];  
  int i;  
  for(i=0;i<6;i++)  
  {  
    s0[i*2]=cv((sign0[i]&0xF0)>>4);  
    s0[i*2+1]=cv(sign0[i]&0x0F);  
  }  
  s0[12]='\0';  
  FILE *fp=fopen("c:\\flexlm_lic.txt","a+");  
  if(fp==NULL)  
    return;  
  fprintf(fp,"%s %s %s\n", feat,s0,sign1);  
  // 为了方便还可以直接生成 license 文件  
  // fprintf(fp,"FEATURE %s xxxx 1.0 permanent uncounted %s HOSTID=ANY\n", feat,s0);  
  // 要注意上面的格式化字符串应与假 license 文件的保持一致  
  fclose(fp);  
}
```

### 第五步 应用前面的劳动成果生成正确的 license

编辑一个伪造的 license 文件，要求 `feature` `version` `vendor` 均正确，`sign` 随便输入即可（字符串长度要够数）

```
SERVER localhost ANY  
VENDOR xxxx  
FEATURE f1 xxxx 1.0 permanent uncounted 123456ABCDEF HOSTID=ANY  
FEATURE f2 xxxx 1.0 permanent uncounted 123456ABCDEF HOSTID=ANY
```

在 `c:\flexlm` 里放置 daemon 程序和 `lmgrd.exe`，在控制台运行 `lmgrd.exe` 后就会发现 `c:\flexlm_lic.txt` 里给出了正确的 `SIGN`

再补充说明一点:

在第二步里确定的关键函数，可能会执行两次，也即 `c:\flexlm_lic.txt` 会相邻两行给出同样的输出；有时第一次生成的 `SIGN` 不能使用（还没搞清楚原因），将这些 `SIGN` 填充到 license 里，再一次运行 `lmgrd.exe` 就会生成正确的 `SIGN` 了

*****

其实在这个函数的两个不同位置处出现了正确的 seed 明码！你知道是哪里吗? `[](img/7c79845954f5fbf9e981f570ed213a8d.jpg "扮鬼脸")` 

*****

`4dd466d0e1e0cc123afbe504da2462ca`
