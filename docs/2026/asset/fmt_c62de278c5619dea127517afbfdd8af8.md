
## 这一段逆向如下

```c
int fun3(char *name,char *code)
{
  int sum,i;
  char buf[12];
  if(strlen(name) <= 0)return 0;
  sum = 1;
  for(i = 0; i < strlen(name); i++)sum += name[i];
  sum = (sum * 3 + 19) / 9 % 16;
  sprintf(buf,"%1X",sum);
  if(strlen(code) < 8)return 0;
  if(code[7] != buf[0])return 0;
  else return 1;
}
```

### 第4处

```asm
.text:00483E80 sub_483E80      proc near
.text:00483E80
.text:00483E80 var_1C          = dword ptr -1Ch
.text:00483E80 var_18          = dword ptr -18h
.text:00483E80 var_C           = dword ptr -0Ch
.text:00483E80 var_4           = dword ptr -4
.text:00483E80
.text:00483E80                 push    0FFFFFFFFh
.text:00483E82                 push    offset unknown_libname_908
.text:00483E87                 mov     eax, large fs:0
.text:00483E8D                 push    eax
.text:00483E8E                 mov     large fs:0, esp
.text:00483E95                 sub     esp, 10h
.text:00483E98                 push    ebp
.text:00483E99                 push    esi
.text:00483E9A                 mov     esi, ecx
.text:00483E9C                 mov     [esp+24h+var_1C], esi
.text:00483EA0                 mov     dword ptr [esi], offset off_500FE0
.text:00483EA6                 mov     eax, [esi+258h]
.text:00483EAC                 mov     [esp+24h+var_4], 4
.text:00483EB4                 mov     cx, [eax]
.text:00483EB7                 cmp     cx, 7D0h
.text:00483EBC                 ja      short loc_483ECF
.text:00483EBE                 jnz     loc_483F74
.text:00483EC4                 cmp     word ptr [eax+2], 5
.text:00483EC9                 jbe     loc_483F74
.text:00483ECF
.text:00483ECF loc_483ECF:
.text:00483ECF                 mov     ax, [eax+4]
.text:00483ED3                 cmp     ax, 1
.text:00483ED7                 jz      short loc_483EE3
.text:00483ED9                 cmp     ax, 2
.text:00483EDD                 jnz     loc_483F74
.text:00483EE3
.text:00483EE3 loc_483EE3:
.text:00483EE3                 cmp     dword_52EAE4, 3B5h
.text:00483EED                 jnz     loc_483F74
.text:00483EF3                 mov     eax, [esi+250h]
.text:00483EF9                 mov     ebp, [eax]         //取用户名
.text:00483EFB                 mov     ecx, [ebp-8]
.text:00483EFE                 test    ecx, ecx           //长度小于等于0则跳
.text:00483F00                 jle     short loc_483F74
.text:00483F02                 xor     eax, eax
.text:00483F04                 push    edi
.text:00483F05                 test    ecx, ecx
.text:00483F07                 mov     edi, 1             //初值1
.text:00483F0C                 jle     short loc_483F1A
.text:00483F0E
.text:00483F0E loc_483F0E:
.text:00483F0E                 xor     edx, edx
.text:00483F10                 mov     dl, [eax+ebp]
.text:00483F13                 add     edi, edx           //用户名ASCII值累加
.text:00483F15                 inc     eax
.text:00483F16                 cmp     eax, ecx
.text:00483F18                 jl      short loc_483F0E
.text:00483F1A
.text:00483F1A loc_483F1A:
.text:00483F1A                 lea     ecx, [edi+0Ah]     //加上10
.text:00483F1D                 mov     eax, 55555556h
.text:00483F22                 imul    ecx                //除以3
.text:00483F24                 mov     eax, edx
.text:00483F26                 shr     eax, 1Fh
.text:00483F29                 add     edx, eax
.text:00483F2B                 shl     edx, 3             //乘以8
.text:00483F2E                 and     edx, 8000000Fh     //模16
.text:00483F34                 jns     short loc_483F3B
.text:00483F36                 dec     edx
.text:00483F37                 or      edx, 0FFFFFFF0h
.text:00483F3A                 inc     edx
.text:00483F3B
.text:00483F3B loc_483F3B:
.text:00483F3B                 push    edx
.text:00483F3C                 lea     ecx, [esp+2Ch+var_18]
.text:00483F40                 push    offset off_523498    ; "%1X"
.text:00483F45                 push    ecx
.text:00483F46                 call    _sprintf            //转换成字符串
.text:00483F4B                 mov     edx, [esi+254h]
.text:00483F51                 add     esp, 0Ch
.text:00483F54                 mov     eax, [edx]          //取注册码
.text:00483F56                 pop     edi
.text:00483F57                 cmp     dword ptr [eax-8], 11
.text:00483F5B                 jl      short loc_483F68    //长度小于11则跳
.text:00483F5D                 mov     al, [eax+0Ah]       //取注册码第11个字节
.text:00483F60                 mov     cl, byte ptr [esp+24h+var_18] //取字符串第1个字节
.text:00483F64                 cmp     al, cl              //比较
.text:00483F66                 jz      short loc_483F74
.text:00483F68
.text:00483F68 loc_483F68:
.text:00483F68                 mov     eax, [esi+25Ch]
.text:00483F6E                 mov     dword ptr [eax], 1  //比较失败则置标志
.text:00483F74
.text:00483F74 loc_483F74:
```

## 这一段逆向如下

```c
int fun4(char *name,char *code)
{
  int sum,i;
  char buf[12];
  if(strlen(name) <= 0)return 0;
  sum = 1;
  for(i = 0; i < strlen(name); i++)sum += name[i];
  sum = (sum + 10) / 3 * 8 % 16;
  sprintf(buf,"%1X",sum);
  if(strlen(code) < 11)return 0;
  if(code[10] != buf[0])return 0;
  else return 1;
}
```

### 第5处

```asm
.text:004872E0 sub_4872E0      proc near
.text:004872E0
.text:004872E0 var_1C          = dword ptr -1Ch
.text:004872E0 var_18          = dword ptr -18h
.text:004872E0 var_C           = dword ptr -0Ch
.text:004872E0 var_4           = dword ptr -4
.text:004872E0
.text:004872E0                 push    0FFFFFFFFh
.text:004872E2                 push    offset unknown_libname_914
.text:004872E7                 mov     eax, large fs:0
.text:004872ED                 push    eax
.text:004872EE                 mov     large fs:0, esp
.text:004872F5                 sub     esp, 10h
.text:004872F8                 push    ebp
.text:004872F9                 push    esi
.text:004872FA                 push    edi
.text:004872FB                 mov     edi, ecx
.text:004872FD                 mov     [esp+28h+var_1C], edi
.text:00487301                 mov     dword ptr [edi], offset off_5012A0
.text:00487307                 mov     eax, [edi+0D8h]
.text:0048730D                 mov     [esp+28h+var_4], 1
.text:00487315                 mov     ebp, [eax]            //取用户名
.text:00487317                 mov     esi, [ebp-8]
.text:0048731A                 test    esi, esi
.text:0048731C                 jle     short loc_48738E      //长度小于等于0则跳
.text:0048731E                 xor     eax, eax
.text:00487320                 mov     ecx, 1                //初值1
.text:00487325                 test    esi, esi
.text:00487327                 jle     short loc_487335
.text:00487329
.text:00487329 loc_487329:
.text:00487329                 xor     edx, edx
.text:0048732B                 mov     dl, [eax+ebp]
.text:0048732E                 add     ecx, edx              //用户名ASCII值累加
.text:00487330                 inc     eax
.text:00487331                 cmp     eax, esi
.text:00487333                 jl      short loc_487329
.text:00487335
.text:00487335 loc_487335:
.text:00487335                 lea     ecx, [ecx+ecx*8+0Ah]  //乘以9加上10
.text:00487339                 mov     eax, 55555556h
.text:0048733E                 imul    ecx                   //除以3
.text:00487340                 mov     eax, edx
.text:00487342                 shr     eax, 1Fh
.text:00487345                 lea     ecx, [edx+eax+24h]    //加上36
.text:00487349                 and     ecx, 8000000Fh        //模16
.text:0048734F                 jns     short loc_487356
.text:00487351                 dec     ecx
.text:00487352                 or      ecx, 0FFFFFFF0h
.text:00487355                 inc     ecx
.text:00487356
.text:00487356 loc_487356:
.text:00487356                 push    ecx
.text:00487357                 lea     edx, [esp+2Ch+var_18]
```
