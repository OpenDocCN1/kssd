
004077F0 此处用于取`TTMsg.INI`里的数据,以下为VC++伪代码

```
void *__thiscall sub_4077F0(int this, void *a2, LPCSTR lpAppName, LPCSTR lpKeyName, LPCSTR lpDefault, char a6)
{
  const CHAR *v6; // ST14_4@1
  void *v7; // esi@3
  const CHAR *v8; // eax@3
  CHAR Src; // [sp+Ch] [bp-10Ch]@1
  int v11; // [sp+114h] [bp-4h]@1

  v6 = *(const CHAR **)(this + 8);
  v11 = 3;
  GetPrivateProfileStringA(lpAppName, lpKeyName, lpDefault, &Src, 0x100u, v6);
  if ( a6 && strcmp(&Src, (const char *)sub_4900CA(0)) )
  {
    v8 = (const CHAR *)sub_407E00(&Src);
    v7 = a2;
    sub_48FD08(a2, v8);
  }
  else
  {
    v7 = a2;
    sub_48FD08(a2, &Src);
  }
  LOBYTE(v11) = 2;
  sub_48FC9A(&lpAppName);
  LOBYTE(v11) = 1;
  sub_48FC9A(&lpKeyName);
  LOBYTE(v11) = 0;
  sub_48FC9A(&lpDefault);
  return v7;
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
