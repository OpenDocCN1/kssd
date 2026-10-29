
```
int __fastcall inotify_add_watchByTid(int fatherpid, int tid)  

{  

  int fatherpid_1; // ST00_4  

  int tid_1; // ST04_4  

  int v4; // ST00_4  

  int v5; // ST04_4  

  char s; // [sp+8h] [bp-158h]  

  char v8; // [sp+9h] [bp-157h]  

  char v9; // [sp+Ah] [bp-156h]  

  char v10; // [sp+Bh] [bp-155h]  

  char v11; // [sp+Ch] [bp-154h]  

  char v12; // [sp+Dh] [bp-153h]  

  char v13; // [sp+Eh] [bp-152h]  

  char v14; // [sp+Fh] [bp-151h]  

  char v15; // [sp+10h] [bp-150h]  

  char v16; // [sp+11h] [bp-14Fh]  

  char v17; // [sp+12h] [bp-14Eh]  

  char v18; // [sp+13h] [bp-14Dh]  

  char v19; // [sp+14h] [bp-14Ch]  

  char v20; // [sp+15h] [bp-14Bh]  

  char v21; // [sp+16h] [bp-14Ah]  

  char v22; // [sp+17h] [bp-149h]  

  char v23; // [sp+18h] [bp-148h]  

  char v24; // [sp+19h] [bp-147h]  

  char v25; // [sp+1Ah] [bp-146h]  

  char v26; // [sp+1Bh] [bp-145h]  

  char v27; // [sp+1Ch] [bp-144h]  

  char v28; // [sp+1Dh] [bp-143h]  

  char v29; // [sp+1Eh] [bp-142h]  

  char v30; // [sp+1Fh] [bp-141h]  

  char v31; // [sp+24h] [bp-13Ch]  

  char v32; // [sp+25h] [bp-13Bh]  

  char v33; // [sp+26h] [bp-13Ah]  

  char v34; // [sp+27h] [bp-139h]  

  char v35; // [sp+28h] [bp-138h]  

  char v36; // [sp+29h] [bp-137h]  

  char v37; // [sp+2Ah] [bp-136h]  

  char v38; // [sp+2Bh] [bp-135h]  

  char v39; // [sp+2Ch] [bp-134h]  

  char v40; // [sp+2Dh] [bp-133h]  

  char v41; // [sp+2Eh] [bp-132h]  

  char v42; // [sp+2Fh] [bp-131h]  

  char v43; // [sp+30h] [bp-130h]  

  char v44; // [sp+31h] [bp-12Fh]  

  char v45; // [sp+32h] [bp-12Eh]  

  char v46; // [sp+33h] [bp-12Dh]  

  char v47; // [sp+34h] [bp-12Ch]  

  char v48; // [sp+35h] [bp-12Bh]  

  char v49; // [sp+36h] [bp-12Ah]  

  char v50; // [sp+37h] [bp-129h]  

  char v51; // [sp+38h] [bp-128h]  

  char v52; // [sp+39h] [bp-127h]  

  char v53; // [sp+3Ah] [bp-126h]  

  char v54; // [sp+3Bh] [bp-125h]  

  char v55; // [sp+3Ch] [bp-124h]  

  char v56; // [sp+3Dh] [bp-123h]  

  char v57; // [sp+3Eh] [bp-122h]  

  char v58; // [sp+3Fh] [bp-121h]  

  char v59; // [sp+44h] [bp-11Ch]  



  fatherpid_1 = fatherpid;  

  tid_1 = tid;  

  memset(&s, 0, 0x19u);  

  v10 = 5;  

  v11 = 7;  

  v19 = 1;  

  v12 = 26;  

  v8 = -4;  

  v20 = 20;  

  v17 = 17;  

  v26 = 17;  

  v9 = 90;  

  v14 = 90;  

  v18 = 90;  

  v21 = 6;  

  v23 = 90;  

  v27 = 90;  

  v15 = 80;  

  v16 = 25;  

  v24 = 80;  

  v25 = 25;  

  v29 = 16;  

  v13 = 22;  

  v22 = 30;  

  v28 = 24;  

  v30 = 24;  

  ((void (__fastcall *)(char *))DecodeString9)(&s);  

  sprintf(&v59, &s, fatherpid_1, tid_1, fatherpid_1, tid_1);// /proc/16709/task/16709/mem  

  inotify_add_watch_insert_node((int)&v59, 0xFFFu);  

  memset(&v31, 0, 0x1Du);  

  v35 = -9;  

  v36 = -22;  

  v37 = -26;  

  v32 = 99;  

  v40 = -23;  

  v45 = -10;  

  v33 = -86;  

  v38 = -86;  

  v42 = -86;  

      v47 = -86;  

      v51 = -86;  

      v41 = -31;  

      v46 = -18;  

      v54 = -30;  

      v43 = -15;  

      v49 = -23;  

      v55 = -32;  

      v34 = -11;  

      v44 = -28;  

      v50 = -31;  

      v52 = -11;  

      v53 = -28;  

      v57 = -28;  

      v58 = -11;  

      v56 = -24;  

      v39 = -96;  

      v48 = -96;  

      ((void (__fastcall *)(char *, signed int, signed int))DecodeString9)(&v31, 26, 230);  

      sprintf(&v59, &v31, v4, v5);  

      return inotify_add_watch_insert_node((int)&v59, 0xFFFu);  

    }  

}
```

这个函数相对比较简单。

## 3.6 里程碑

至此将目标文件纳入到监控中的相关处理已经分析完了，下面看发生相关事件是的处理流程。

## 3.7 `read_filewatch_event`函数

```
int __fastcall read_filewatch_event1(int a1_FF, int const_1)  

{  

  char *v2; // r1  

  char *v3; // r3  

  struct timeval *timeout; // r4  

  int inotify_init; // r0  

  int v7; // r0  

  int v8; // r3  

  int a1_FF_1; // [sp+8h] [bp-20h]  

  int const_1_1; // [sp+Ch] [bp-1Ch]  



  if ( const_1 <= 0 )  

    return 0;  

  a1_FF_1 = a1_FF;  

  const_1_1 = const_1;  

  setjmp((struct __jmp_buf_tag *)&unk_53D04);  

  g_fileWatch_errno = 0;  

  if ( dword_53E04 )  

  {  

    if ( dword_53E04 <= dword_53E08 - 16 )  

    {  

      v2 = (char *)&dword_53E0C + dword_53E04;  

      dword_63E0C = (int)&dword_53E0C + dword_53E04;  

      v3 = (char *)&(*(struct inotify_event **)((char *)&dword_53E0C + dword_53E04 + 12))[1] + dword_53E04;  

      dword_53E04 = (int)v3;  

      if ( v3 == (char *)dword_53E08 )  

      {  

        dword_53E04 = 0;  

      }  

      else if ( (signed int)v3 > dword_53E08 )  

      {  

        dword_53E08 = (char *)&dword_53E0C + dword_53E08 - v2;  

        memcpy(&dword_53E0C, v2, dword_53E08);  

        return read_filewatch_event1(a1_FF_1, const_1_1);  

      }  

      if ( g_inotify_init_flag1 )  

        p9E01CAAA70B3E111F16A18AB1BC2AB55((int *)v2);  

      return dword_63E0C;  

    }  

  }  

  else  

  {  

    dword_53E08 = dword_53E04;  

  }  

  g_FF = a1_FF_1;  

  timeout = (struct timeval *)&g_FF;  

  dword_53D00 = 0;  

  if ( a1_FF_1 <= 0 )  

    timeout = 0;  

  dword_63E10 = (int)timeout;  

  memset(&g_fds, 0, 0x80u);  

  inotify_init = g_inotify_init;  

  *((_DWORD *)&unk_63DA8 + (g_inotify_init >> 5) + 0x1B) = 1 << (g_inotify_init & 0x1F);  

  v7 = select(inotify_init + 1, (fd_set *)&g_fds, 0, 0, timeout);//  //监控fd的事件。当有事件发生时，返回值>0  

  g_slect_returnValue = v7;  

  if ( v7 < 0 )  

    goto LABEL_21;  

  if ( !v7 )  

    return 0;  

  while ( 1 )  

  {  

    v7 = ioctl(g_inotify_init, 0x541Bu, &g_fileWatch_event_MaxLen);  

    g_slect_returnValue = v7;  

    if ( v7 )  

      break;  

    if ( g_fileWatch_event_MaxLen >= (unsigned int)(16 * const_1_1) )  

      goto LABEL_20;  

  }  

  if ( v7 == -1 )  

    goto LABEL_21;  

LABEL_20:  

  v7 = read(g_inotify_init, &(&dword_53E0C)[4 * dword_53E08], 0x10000 - dword_53E08);  

  g_fileWatch_event_readLen = v7;  

  if ( v7 < 0 )  

  {  

LABEL_21:  

    g_fileWatch_errno = *(_DWORD *)_errno(v7);  

    return 0;  

  }  

  if ( !v7 )  

    return 0;  

  dword_53E08 += v7;  

  dword_63E0C = (int)&dword_53E0C;  

  v8 = dword_53E18 + 0x10;  

  if ( dword_53E18 + 0x10 == dword_53E08 )  

    v8 = 0;  

  dword_53E04 = v8;  

  if ( g_inotify_init_flag1 )  

    p9E01CAAA70B3E111F16A18AB1BC2AB55((int *)&dword_53E0C);  

  return dword_63E0C;  

}
```

`read_filewatch_event`函数步骤如下：

1、调用`select`函数对`inotify`初始化句柄进行阻塞。当发生事件时，则线程唤醒；

2、调用`ioctl`函数获得对应事件的长度；

3、调用`read`函数将发生的事件信息读取到全局变量中。

4、返回对应的事件BUF。

当发生事件时，就开始进行事件处理流程，首先调用`filewatch_Delete`清除watch。

## 3.8 `filewatch_Delete`函数
