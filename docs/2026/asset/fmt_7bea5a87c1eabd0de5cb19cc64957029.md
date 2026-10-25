
`cseg01:3130 000                mov`

`  di, ax`

` cseg01:3132 000`

`  inc    si              ;`

`cseg01:3133 000                loop`

`  loc\_0\_311F      ;`

注册机的程序如下（C++BUILDER程序主要部分）：

```cpp
void __fastcall TForm1::Button1Click(TObject *Sender)
{
int rk;
   char ctemp[10];
   memset(ctemp,'\0',10);
   randomize();
   if (Edit1->Text.Length()!=10)
{
     Application->MessageBox("请输入10位十进制数字", NULL, MB_OK);
    return;
   }
   memcpy(input,Edit1->Text.c_str(),10);
   input[10]='\0';
   memcpy(sn1,input,5);
sn1[5]='\0';
   memcpy(cs2,input+5,5);
   cs2[5]='\0';
   p1=cs2;
   p2=cs3;
   pt=table;
disk_sno1=atoi(sn1);
   __asm
   {
     pushad
           mov ax,[disk_sno1]
        mov ebx,[p1]
  mov cx,5
         push    ebx
         push    cx
        clc
         mov
  bx, ax
         and    bx,
157h
         cmp    bx, 0
        jnz    loc_0_30D4
        nop
         nop
         xor    bx, 153h
        cmp    bx, 0
      jnz    loc_0_30D4
      nop
         nop
         or    bx, 53h
loc_0_30D4:
         cmp    ax,
bx
         jb    loc_0_30DF
         nop
  nop
         push    ax
         mov    ax, bx
      pop    bx
  cwd
 loc_0_30DF:
         div
  bx
         mov    cx,
0Ah
         mov ebx,[p2]
 loc_0_30E7:
         mov    [k], cx
        xor    ax, dx
      ror    dx, 1
  ror    dx, 1
         ror
  dx, 1
         ror    dx,
1
         ror    dx, 1
        mov    cl, dl
      ror    ax, cl
  xor    ax, dx
         xor
  al, ah
         xor    dl,
al
         mov    cl, dl
        rol    ax, cl
      and    ax, 0F0Fh
    and eax,0ffffh
         mov
  [ebx], ax
         add    ebx,
2
         xor ecx,ecx
  mov    cx, [k]
         loop
  loc_0_30E7
         mov
  cx, 14h
         mov esi,[p2]
        mov edi,[pt]
  mov    bx, 0
 loc_0_311F:
  mov    al, [esi]
         mov
  ah, 0
         mov    ebx,
edi
         and eax,0ffffh
  add    ebx, eax
         mov
  al, [ebx]
         mov    [esi],
al
         mov    eax, edi
        add    eax, 0Ah
      mov    edi, eax
  inc    esi
         loop
  loc_0_311F
         pop    cx
         pop    ebx
      mov    esi, ebx
  mov ebx,[p2]
         and ecx,0ffffh
 loc_0_3078:
         mov    al,
[ebx]
         add    al, 0
        daa
         mov
  ah, al
         mov    al,
[esi]
         and    al, 0Fh
         add    al, ah
      daa
         and
  al, 0Fh
         or    al,
30h
         mov    [esi], al
         inc    esi
      inc    ebx
  loop    loc_0_3078
     }
  p1=cs2;
     __asm
     {
  mov ax,[productsn]
         mov
ebx,[p1]
         mov cx,5
      push    ebx
  push    cx
         clc
         mov    bx, ax
      and    bx, 157h
  cmp    bx, 0
         jnz
  loc_1_30D4
         nop
        nop
         xor
  bx, 153h
         cmp    bx,
0
         jnz    loc_1_30D4
         nop
  nop
         or    bx,
53h
 loc_1_30D4:
         cmp
  ax, bx
         jb    loc_1_30DF
         nop
  nop
         push    ax
         mov    ax, bx
      pop    bx
  cwd
 loc_1_30DF:
         div
  bx
         mov    cx,
0Ah
         mov ebx,[p2]
 loc_1_30E7:
         mov    [k], cx
        xor    ax, dx
      ror    dx, 1
  ror    dx, 1
         ror
  dx, 1
         ror    dx,
1
         ror    dx, 1
        mov    cl, dl
      ror    ax, cl
  xor    ax, dx
         xor
  al, ah
         xor    dl,
al
         mov    cl, dl
        rol    ax, cl
      and    ax, 0F0Fh
    and eax,0ffffh
         mov
  [ebx], ax
         add    ebx,
2
         xor ecx,ecx
  mov    cx, [k]
         loop
  loc_1_30E7
         mov
  cx, 14h
         mov esi,[p2]
        mov edi,[pt]
  mov    bx, 0
         and ecx,0ffffh
 loc_1_311F:
         mov    al,
[esi]
         mov    ah, 0
        mov    ebx, edi
      and eax, 0ffffh
         add
  ebx, eax
         mov    al,
[ebx]
         mov    [esi],
al
         mov    eax, edi
        add    eax, 0Ah
      mov    edi, eax
  inc    esi
         loop
  loc_1_311F
         pop    cx
         pop    ebx
      mov    esi, ebx
  mov ebx,[p2]
         and ecx,0ffffh
 loc_1_3078:
         mov    al,
[ebx]
         add    al, 0
        daa
         mov
  ah, al
         mov    al,
[esi]
         and    al, 0Fh
         add    al, ah
      daa
         and
  al, 0Fh
         or    al,
30h
         mov    [esi], al
         inc    esi
      inc    ebx
  loop    loc_1_3078
     popad
  }
   memset(keycode,'\0',16);
   strcpy(keycode,cs2);
   int cn1=atoi(cs2);
   unsigned short uscn=(unsigned
short)cn1;
   char st1[11];
   char st2[21];
   memset(st2,'\0',21);
   memset(st1,'\0',11);
   char *p3;
   p3=st2;
  st1[5]='\0';
   rk=(unsigned short)rand();
   strcat(st1,"41062");
   sprintf(ctemp,"%05d",rk);
   strcat(st1,ctemp);
char stemp[11];
   memcpy(stemp,st1,11);
 for(signed char jk=4;jk>=0;jk--)
   for(unsigned char jj=0;jj<=9;jj++)
   {
   if
((jk==4)&&(jj>1))
     continue;
   memcpy(st1,stemp,11);
   st1[0]=jk+0x30;
   stemp[0]=jk+0x30;
   st1[1]=jj+0x30;
   stemp[1]=jj+0x30;
   p1=st1;
   __asm
{
     pushad
mov ax,[uscn]
           mov ebx,[p1]
        mov cx,5
  push    ebx
         push
  cx
         clc
      mov    bx, ax
  and    bx, 157h
         cmp
  bx, 0
         jnz    loc_2_30D4
         nop
  nop
         xor    bx,
153h
         cmp    bx, 0
        jnz    loc_2_30D4
        nop
         nop
         or    bx, 53h
loc_2_30D4:
         cmp    ax,
bx
         jb    loc_2_30DF
         nop
  nop
         push    ax
         mov    ax, bx
      pop    bx
  cwd
 loc_2_30DF:
         div
  bx
         mov    cx,
0Ah
         mov ebx,[p3]
 loc_2_30E7:
         mov    [k], cx
        xor    ax, dx
      ror    dx, 1
  ror    dx, 1
         ror
  dx, 1
         ror    dx,
1
         ror    dx, 1
        mov    cl, dl
      ror    ax, cl
  xor    ax, dx
         xor
  al, ah
         xor    dl,
al
         mov    cl, dl
        rol    ax, cl
      and    ax, 0F0Fh
    and eax,0ffffh
         mov
  [ebx], ax
         add    ebx,
2
         xor ecx,ecx
  mov    cx, [k]
         loop
  loc_2_30E7
         mov
  cx, 14h
         mov esi,[p3]
        mov edi,[pt]
  mov    bx, 0
 loc_2_311F:
  mov    al, [esi]
         mov
  ah, 0
         mov    ebx,
edi
         and eax,0ffffh
  add    ebx, eax
         mov
  al, [ebx]
         mov    [esi],
al
         mov    eax, edi
        add    eax, 0Ah
      mov    edi, eax
  inc    esi
         loop
  loc_2_311F
       }
       p3=st2;
       __asm
       {
            pop    cx
            pop    ebx
            mov    esi, ebx
              mov    ebx, [p3]
                and ecx,0ffffh
    loc_2_309D:
  mov    al, [ebx]
    add    al, 0
    daa
mov    ah, al
  mov    al, [esi]
    and    al, 0Fh
      or      al, 10h
          sub    al, ah
            das
        and    al, 0Fh
          or      al, 30h
              mov    [esi], al
                inc    esi
                inc    ebx
                loop    loc_2_309D
       popad
     }
     char cct[5];
     memset(cct,'\0',5);
     memcpy(cct,st1,4);
    signed short il;
     il=(signed short)(atoi(cct));
     if ((il>=0)&&(il<=0x100c))
goto thatisit;
   }
   thatisit:
   strcat(keycode,stemp);
   Edit2->Text=keycode;
 }
```

需要编译好的注册机的请留下EMAIL，我会发给你们。
