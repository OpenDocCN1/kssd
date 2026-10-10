
`#define TABLE_SIZE 32`
`#define SERIAL_SIZE 0x1000`
`char name[0x10];`
`char serial[SERIAL_SIZE];`
`unsigned char nametable[10];`
`unsigned char nametablepos[10];`
`unsigned char table[TABLE_SIZE];`

`int namelen = 0, seriallen = 0;`

```c
int findmodifypos()
{
    int i,tableitem = 0;
    for (i=1; i<10; i++)
        if (nametable[i] == 1) nametablepos[tableitem++]=i;
    if(tableitem > 0)
    {
        if (tableitem%2) return 1;    
        else return nametablepos[0]+1;
    }else return 0;   
}
```

```c
void adjusttable( )
{
    int pos;
    while((pos = findmodifypos()) && seriallen<SERIAL_SIZE )
    {
        serial[seriallen]=(10+pos-table[seriallen%(TABLE_SIZE-1)])%10+'0';
        seriallen++;
        if(pos ==1 )nametable[pos]^=1;    
        else nametable[pos]^=1;    
    }
}
```

```c
int work()
{
    // 局部变量
    int i;
    unsigned adder,adder2;
    unsigned namecalc = 0x13572468;

    memset(nametable, 0, 0xa);
    memset(serial, 0, SERIAL_SIZE);
    seriallen = 0;
    // 函数动作
    for(i=0; i<namelen; i++)
    {   // 计算名字特征
        adder = (name[i]+ namecalc)*0x3721273+0x24681357;
        adder2 = adder<<0x19;
        __asm sar adder,7
        namecalc = adder2|adder;
    }

    for (i=1; i<9; i++)
    {   //根据名字特征值的 1-8 位构造名字表
        nametable[i] = (unsigned char)((unsigned char)(namecalc>>i)&1);
    }
    nametable[9] = 1;
    
    for (i=0; i<TABLE_SIZE; i++)    
        table[i] = ((namecalc>>(unsigned char)(i%(TABLE_SIZE-1)))%10);

    adjusttable();
    return 0;
}
```

---

结束语：偶是菜鸟，这个 crackme 中我学到了很多东西，如隐藏字符串，另外就是看大段汇编代码没有那么头疼了;)

总的来说，这个 crackme 难易适中，适合我这种菜菜,毕竟只有 1.6k:)感觉自己杂七杂八的说了，都没说清楚。希望能给你帮助。

关键词:格雷码

代码:

---

```c
DWORD MsGen2(HWND hDlg)
{
  char szName[0x100];
  char szCode[0x400];
  int Len;
  int i;
  int K;
  DWORD N;

  if (Len = GetDlgItemText(hDlg, IDC_EDIT_NAME, szName, 16))
  {
    K = 0x13572468;
    for (i=0;i<Len;i++)
    {
      K = (K + szName[i]) * 0x3721273 + 0x24681357;
      K = (K << 0x19) | (K >> 7);
    }

    Len = 0;
    do    
    {
      Len++;
    } while((Len ^ (Len >> 1)) != ((K >> 1) & 0xFF | 0x100));

    for (i=0;i<Len;i++)
    {
      N = Len - i;
      N ^= N - 1;
      N ^= N >> 1;
      __asm
      {
        bsf eax,N
        mov N,eax
      }
      szCode[i] = ((BYTE)(N + 11 - (((DWORD)K >> (i % 31)) % 10)) % 10) + 0x30;
    }
    szCode[Len] = 0;

    SetDlgItemText(hDlg, IDC_EDIT_CODE, szCode);
  }

  return 1;
}
```

---

`base`=====>为 `namehash`

```c
char buffer[9] = {0};
unsigned int namehash = base;
for (int i=0; i<8; i++)
{
  buffer[i] = (namehash>>(i+1))&0x1;
}
buffer[8] = 1;
```

```c
int nseqidlen;
char seqid[31];
byte nowbyte;

for (i=0; i<nseqidlen; i++)
{
  nowbyte = seqid[i];
  int t1 = (namehash>>(i%31))%10;
  int tem1 = (t1 + nowbyte -0x30)%10;
  if (tem1==1)
  {
    buffer[0] ^= 1;
    continue;
  }
  if (buffer[tem1-2]!=1)
  {
    error;
  }
  if (tem1-2>=1)
  {
    for (i=0; i<tem1-2; i++)
    {
      if (buffer[i] == 1)
      {
        error;
      }
    }
  }
  buffer[tem1-1] ^= 1;
}
```

通过对 `tem1` 的一系列取值,可以达到对其中一位进行反位的功能,前提是这一位前面都为 0.

这个取值是一个数列

让第 1 位反位  1  

让第 2 位反位  1 2 1  

让第 3 位反位  1 2 1 3 1 2 1  

让第 4 位反位  1 2 1 3 1 2 1 4 1 2 1 3 1 2 1  

通项式是 `f(n) = f(n-1)+n+f(n-1);f(1)=1;`

```c
int count;
void get_reserve_string(int num, char *buffer)
{
    if (num==1)
    {
        sprintf(buffer+count, "%1d", 1);
        count++;
        return;
    }
    get_reserve_string(num-1, buffer);
    sprintf(buffer+count, "%1d", num);
    count++;
    get_reserve_string(num-1, buffer);
    return;
}
```

用来生成这个数列.

```c
//这里用的是最正常的办法,没有优化!!!!  对 buffer 中每一个 1 进行反位
            for (i=0; i<9; i++)
            {
                if (buffer[i]==1)
                {
                    memset(stringbuffer, 0, 520);
                    count = 0;//全局置 0
                    get_reserve_string(i+1, stringbuffer);
                    
                    for (int j =0; j<count; j++)
                    {
                        t1 = (namehash>>((key_cur+j)%31))%10;
                        if (t1 <= (stringbuffer[j]-0x30))
                        {
                            key[key_cur+j] = stringbuffer[j] - t1;
                        }
                        else
                            key[key_cur+j] = 10 + stringbuffer[j] - t1;
                    }
                    key_cur += count;      
                }
            }
```

唉,我当时注册的时候,是 12 点以前,但是老是注册不成功,没有办法,运气不好,祝福你们拉

ps  

`sar` 就是带符号的右移,定义成 `int` 就可以了,如果是 `shr` 的话,定义为 `unsigned int`

```c
BYTE Ninekey[9] = {0};
char OutPath[1024] = {0};
int Pathlen = 0;

void UpRing(int idx);
void DownRing(int idx);
```

```c
void DownRing(int idx)
{
  if( idx > 2)
    DownRing(idx-2);
  OutPath[Pathlen] = idx;
  Ninekey[idx-1] = !Ninekey[idx-1];
  Pathlen++;
  if( idx > 2 )
    UpRing(idx-2);
  if( idx > 1 )
    DownRing(idx-1);
}
```

```c
void UpRing(int idx)
{
  if(idx > 1)
    UpRing(idx-1);
  if(idx > 2)
    DownRing(idx-2);
  OutPath[Pathlen] = idx;
  Ninekey[idx-1] = !Ninekey[idx-1];
  Pathlen++;
  if( idx > 2 )  
    UpRing(idx-2);
}
```

关键词：递归

代码:

---

```c
int steplist[1000]={0};  //走的步数
BYTE newtmp1[10] = {0};  //状态
void x(int n);//清空 n
void y(int n);//填上 n
void setstep(int n);//异或一次

void setstep(int n)  //异或一次
{
  steplist[pStep++]=n;
  newtmp1[n] ^= 1;
}

void x(int n)  //清空 n 到最前面的
{
  if (n == 1)
  {
    if (newtmp1[1] == 1) setstep(1);
  }
  else
  {
    y(n-1);//填上 n-1,清空 n-1 前面的
    if (newtmp1[n] == 1) setstep(n);  //如果有值就清空
    x(n-1);//清空前面的
  }
}

void y(int n)  //填上 n，并清空前面的 n-1 个
{
  if (n == 1)
  {
    if (newtmp1[1] == 0) setstep(1);
  }
  else
  {
    y(n-1);//填上 n-1 的，并清空前面的 n-2 个
    if (newtmp1[n] == 0) setstep(n);  //当前如果没值就填上
    x(n-1);//清空前面 n-1
  }
}
```

---

调用的时候

代码:

---

```c
x(9);
```

---

是 gray 码  谁要早提醒我一下就好了 网上一搜一大堆![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")

偶也偶的贴出来,虽然写的很滥.

代码:

---

```asm
GetEdx proc;获取 crackme 里面那个 edx
  mov ecx,1Fh
  cdq
  mov eax,Codelen
  idiv ecx
  mov eax,UserCode
  mov ecx,edx
  xor edx,edx
  shr eax,cl
  mov ecx,0ah
  div ecx
  mov eax,edx
  inc Codelen
  ret
GetEdx endp
GetCode proc ;开始计算注册码
  mov esi,offset lala
  @:
  or ecx,0FFFFFFFFH
  mov edi,offset haha
  mov eax,1
  repne scasb  
  not ecx
  dec ecx
  push ecx
  .if ecx>=9h    
    pop ecx
    ret
  .elseif ecx==8
    pop ecx
    mov edi,offset haha
    jmp @1
  .endif
  mov edi,offset haha
  xor byte ptr [edi+ecx+1],1h
  call GetEdx
  mov ebx,0ah
  pop ecx
  add ebx,ecx
  add ebx,2
  sub ebx,eax
  .if ebx>=0ah
    sub ebx,0ah
  .endif
  add ebx,30h
  mov byte ptr [esi],bl
  inc esi
@1:  xor byte ptr [edi],1h
  call GetEdx
  mov ebx,0Bh
  sub ebx,eax
  .if ebx>=0ah
    sub ebx,0ah
  .endif
  add ebx,30h
  mov byte ptr [esi],bl
  inc esi
  jmp @B
  ret
GetCode endp
```

---

哈哈,偶的长的很难看,大家将就看把

代码:

---
