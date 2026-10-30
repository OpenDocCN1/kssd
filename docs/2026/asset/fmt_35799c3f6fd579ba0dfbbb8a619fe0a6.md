

# [原创] CVE-2010-2883 分析笔记

本来不想写的。。怕技术太差丢人。。。不过还是写下吧，丢了人被人指出来才能提高技术，哈哈。

```c
char __cdecl vunerableFunc(int initial_critical_class, int a2, int a3, int a4)
{
  int critical_class; // edi@1
  bool v5; // zf@1
  int v6; // eax@4
  size_t v7; // eax@13
  int v8; // eax@13
  int v9; // eax@15
  int v10; // eax@16
  int v12; // eax@33
  char v13; // [sp+Ch] [bp-58h]@40
  int v14; // [sp+2Ch] [bp-38h]@13
  int v15; // [sp+34h] [bp-30h]@1
  int v16; // [sp+38h] [bp-2Ch]@23
  int v17; // [sp+3Ch] [bp-28h]@1
  int sing; // [sp+40h] [bp-24h]@4
  int v19; // [sp+44h] [bp-20h]@12
  int v20; // [sp+48h] [bp-1Ch]@2
  int v21; // [sp+4Ch] [bp-18h]@2
  char v22; // [sp+53h] [bp-11h]@3
  int v23; // [sp+60h] [bp-4h]@1
  char v24; // [sp+64h] [bp+0h]@7

  critical_class = initial_critical_class;

  v17 = initial_critical_class;

  v15 = a4;

  sub_8041626();

  v5 = *(_DWORD *)(initial_critical_class + 8) == 3;

  v23 = 0;

  if ( !v5 )
  {
    v20 = 0;

    v21 = 0;

    v5 = *(_DWORD *)(initial_critical_class + 12) == 1;

    LOBYTE(v23) = 1;

    if ( v5 )
    {
      v22 = 0;

      sub_802178F(&v20, initial_critical_class, "name");

      if ( v20 )
        goto LABEL_52;

      procSing((int)&sing, initial_critical_class, "SING");

      v6 = sing;

      LOBYTE(v23) = 2;

      if ( sing )
      {
        if ( !(*(_DWORD *)sing & 0xFFFF) || (*(_DWORD *)sing & 0xFFFF) == 256 )
        {
          v24 = 0;

          strcat(&v24, (const char *)(sing + 16)); // overflow here!!!

          sub_8001243((int *)a2, (int)&v24);

          v6 = sing;
        }

        v22 = 1;
      }

      LOBYTE(v23) = 1;

      if ( v6 )
        sub_80417B9(v6);

      if ( !v22 )
      {
LABEL_52:

        v5 = *(_BYTE *)(initial_critical_class + 188) == 0;

        v19 = 0;

        if ( !v5 )
        {
          v7 = wcslen((const wchar_t *)(initial_critical_class + 124));

          v8 = sub_804007B(initial_critical_class + 124, v7 + 1);

          LOBYTE(v23) = 3;

          sub_800202E(v8);

          LOBYTE(v23) = 1;

          if ( v14 )
            dword_8231224(v14);
        }

        v9 = sub_80165ED(initial_critical_class, &v20);

        v5 = v20 == 0;

        *(_DWORD *)(a2 + 88) = v9;

        if ( !v5 )
        {
          v10 = *(_WORD *)(initial_critical_class + 8);

          LOBYTE(v10) = *(_BYTE *)(initial_critical_class + 68) != 0;

          sub_803C26E(&v20, v10, *(_WORD *)(initial_critical_class + 8));

          critical_class = v17;
        }

        if ( a3 && (!(v19 & 1) || !(v19 & 2) || !(v19 & 4)) )
          *(_DWORD *)(critical_class + 8) = 0x40000000;

        if ( !(v19 & 8) )
        {
          unknown_libname_1(&v16);

          LOBYTE(v23) = 4;

          sub_800202E(&v16);

          LOBYTE(v23) = 1;

          if ( v16 )
            dword_8231224(v16);
        }
      }
    }

    if ( !shellcode_exec(critical_class, a2, (int)&v20) ) // shell code executed here!!!
    {
      if ( v20 )
        sub_80417B9(v20);

      return 0;
    }

    LOBYTE(v23) = 0;

    if ( v20 )
      sub_80417B9(v20);
  }

  if ( !(unsigned __int8)sub_803B556(critical_class, v15) )
    return 0;

  sub_803C9E9(critical_class, a2);

  sub_803B465(critical_class);

  sub_803B338();

  v12 = *(_DWORD *)(critical_class + 8);

  if ( v12 != 1000 )
  {
    if ( !*(_DWORD *)(a2 + 56)
      && *(_DWORD *)a2
      && (!v12 || v12 == 3)
      && !*(_DWORD *)(critical_class + 12)
      && !*(_DWORD *)(a2 + 88) )
    {
      sub_8014F29(&v13);

      LOBYTE(v23) = 5;

      if ( sub_80F60EC(0) )
      {
        if ( (unsigned __int8)sub_80F4859(a2, 1, &v13) )
        {
          sub_8014BE9(&v20);

          LOBYTE(v23) = 6;

          sub_800202E(&v20);

          LOBYTE(v23) = 5;

          if ( v20 )
            dword_8231224(v20);
        }
      }

      LOBYTE(v23) = 0;

      sub_8014F4D(&v13);
    }

    if ( !*(_DWORD *)(a2 + 56) )
    {
      if ( *(_DWORD *)(a2 + 32) )
        sub_800202E(a2 + 32);
      else
        sub_800202E(a2 + 8);
    }
  }

  return 1;
}
```

如代码，`strcat` 溢出点，在一个子函数（`0x08016B96`）执行了 shellcode。跟进去，发现是里面的 `shellcode_exec2` 里面的 `shellcode_exec3`（`call [eax]` 调用的，具体哪个函数我动态跟的）调用的 `call [eax]` 导致的 `eip` 劫持。

