
```
027ce3c8 025163e8 00710150 000003f1 02ccd728 USER32!SetDlgItemTextW+0x21  

027ce3f4 02b9e4d2 02ccd6d0 02e60020 000a5b05 MouCoreUI!GetIWM_MouCoreUI+0x1c98  

00000000 00000000 00000000 00000000 00000000 7z!CreateObject+0x3df62  

# 4 Id: bf8.914 Suspend: 1 Teb: 7ffdb000 Unfrozen  

ChildEBP RetAddr  Args to Child                  

WARNING: Stack unwind information not available. Following frames may be wrong.  

02aefff4 00000000 00000000 00000000 00000000 ntdll!DbgBreakPoint 
```

刘涛涛一直是我的偶像，最近准备做编译器了吗？

发一段我写的编译器的代码，不过只处理 `ascii`，不解析 `unicode`。

```c
#define CHAR_ISDIGIT(c)  ((c)>='0' && (c)<='9')  

#define CHAR_ISCHAR(c)  (((c)>='a' && (c)<='z') || ((c)>='A' && (c)<='Z'))  

#define CHAR_ISHEXCHAR(c) (CHAR_ISDIGIT(c) || ((c)>='a' && (c)<='f') || ((c)>='A' && (c)<='F'))  

#define CHAR_ISBLANK(c) ((c)==' '||(c)=='\t')  

#define CHAR_ISEND(c) ((c)=='\0'||(c)=='\r'||(c)=='\n')  

#define CHAR_ISMATHOPR(c)  ((c)=='+'||(c)=='-'||(c)=='*'||(c)=='/'||(c)=='^'||(c)=='&'||(c)=='|')  

#define ISFULLWORDCHAR(x) (((x)>='a'&&(x)<='z') || ((x)>='A'&&(x)<='Z') || CHAR_ISDIGIT(x) || (x) == '_' || (x) == '$')  

#define SKIPBLANK(x)    while((*x) == ' ' || (*x) == '\t'){(x)++;}    //跳过空格和 tab  

/*  
判断语句是否结尾  
  ;  
*/  

BOOL CScript::IsEndOfLine(const char *str)  
{  
  char* p = (char*)str;  
  SKIPBLANK(p);  
  if(CHAR_ISEND(*p))  
    return TRUE;  
  if(*p != ';')  
    return FALSE;  
  //只能是一个 ';'  
  p++;  
  SKIPBLANK(p);  
  if(CHAR_ISEND(*p))  
    return TRUE;  
  return FALSE;  
}
```
