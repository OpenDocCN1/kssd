
```c
case 0x0FF86: //F12=PasteClipData

  InputClipData();

  key=0;

  break;

}

return key;

}
```

`F11` 调用 `LableOut` 查看标签,`F12` 负责处理粘贴

`LableOut` 内部建立一个输入循环处理对标签的操作,`F1` 删除当前标签,`F2` 删除所有标签,`F3` 复制标签的 `VA`

`InputClipData()` 处理粘贴,上面已经提到了写输入缓冲区的方法是无效的,这里利用 `keybd_event` 来模拟键盘输入,就像按键精灵那样

```c
#define assert(a) if(!a){MsgOut("myHiew.dll Error:" #a " return FALSE");return 0;}

BOOL InputClipData()

{

  assert(OpenClipboard(NULL));

  HANDLE hMem=GetClipboardData(CF_TEXT);

  assert(hMem);

  char*ptr=(char*)GlobalLock(hMem);

  assert(ptr);

  strcpy(cliptext,ptr);

  assert(GlobalUnlock(hMem));

  assert(CloseClipboard());

  CreateThread(NULL,0,KeyInput,cliptext,NULL,NULL);

  return 1;

}
```

这里定义一个断言 `assert` 防止出错时非法操作,然后新建一个线程来模拟输入

`CreateThread` 是必须的,如果不建一个新线程会导致当前拦截的输入函数无法返回,实际上和写输入缓冲区一样不能正确的输入

```c
char cliptext[1000];

DWORD WINAPI KeyInput(void*string)

{

  char*ptr=(char*)string;

  SHORT VkKey;

  while(*ptr)

  {

    if(*ptr=='\\n')

    {

      ptr++;

      continue;

    }

    Sleep(50);

    VkKey=VkKeyScan(*ptr);

    if(HIBYTE(VkKey)&1)

    {

      keybd_event(VK_SHIFT,0,0,0);

      keybd_event(LOBYTE(VkKey),0,0,0);

      keybd_event(VK_SHIFT,0,KEYEVENTF_KEYUP,0);

    }

    else

    {

      keybd_event(LOBYTE(VkKey),0,0,0);

    }

    ptr++;

  }

  return 1;

}
```

这里模拟按键,`VkKeyScan` 将字符转换为虚拟键盘码(在低 8 位),而它的最高位表示输入该字符是否需要按 `Shift`

这个方法只能模拟输入英文字母和数字,看起来非常别扭

不过有个好处,就是可以成段的复制输入代码(因为 `keybd_event` 会把回车也模拟出来)

`if(*ptr=='\\n')` 的作用是回车换行("\\r\\n")时只输入回车,这样可以避免跳过两行

下面看 `cinvoke` 的处理(`sinvoke` 和这个相似),原理很简单,逐个读取参数,然后 `push` 参数,`call` 函数,平栈

指令的格式是 `cinvoke proc/p1/p2/p3.....`

`proc` 可以是子函数的 `VA` 地址也可以是 `API` 或者标签,`p1`,`p2`,`p3` 是参数,可以是一般的参数或者标签

```c
  ifins("cinvoke")

  {

    char paramtbl[10][20];

    int nParam=0;

    char*param=strtok(operand,"/");

    while(param&&(nParam<10))

    {

      strcpy(paramtbl[nParam],param);

      param=strtok(NULL,"/");

      nParam++;

    }

    nParam--;

    if(nParam==0)goto invalid;

    if(!GetCallAddr(paramtbl[0],srcaddr))goto invalid;

    int npush=nParam;

    int ndata=0;

    char curins[100];

    int AsmEngRet;

    map<string,DWORD>::iterator mi;

    while(nParam)

    {

      strlwr(paramtbl[nParam]);

      switch(paramtbl[nParam][0])

      {

      case '@':

        mi=LableMap.find(¶mtbl[nParam][1]);

        if(mi==LableMap.end())

          sprintf(curins,"push %s",paramtbl[nParam]);

        else

          sprintf(curins,"push 0%x",mi->second);

        break;

      case '*':

        mi=LableMap.find(¶mtbl[nParam][1]);

        if(mi==LableMap.end())

          sprintf(curins,"push %s",paramtbl[nParam]);

        else

          sprintf(curins,"push d,[0%x]",mi->second);

        break;

      default:

        sprintf(curins,"push %s",paramtbl[nParam]);

        break;

      }

      AsmEngRet=AsmEng(curins,buf+ndata,0x7FFF,srcaddr+ndata);

      if(AsmEngRet>0)

        ndata+=AsmEngRet;

      else

        return AsmEngRet;

      nParam--;

    }

    sprintf(curins,"call %s",paramtbl[0]);

    AsmEngRet=AsmEng(curins,buf+ndata,0x7FFF,srcaddr+ndata);

    if(AsmEngRet>0)

      ndata+=AsmEngRet;

    else

    {

      MyRet=AsmEngRet;

      goto end;

    }

    if(npush>1)

    {

      sprintf(curins,"add esp,0%x",npush*4);

      AsmEngRet=AsmEng(curins,buf+ndata,0x7FFF,srcaddr+ndata);

      if(AsmEngRet>0)

        ndata+=AsmEngRet;

      else

      {

        MyRet=AsmEngRet;

        goto end;

      }

    }

    return ndata;

  }
```

参数允许是标签,以 `@` 开头的标签表示取标签的 `VA`,而以 `*` 开头表示取值(会被转换为 `d,[VA]`),每个参数都以 `/` 分隔(没有想到什么好方法...)

`GetCallAddr` 这里是转换出 `CALL` 的地址,可以是 `VA` `API` 函数名 标签

到这里大部分的处理都完成了,最后只剩下文件的加解密,`Hiew` 里只能进行简单的文件接密

不过我们已经拦截了汇编引擎,所以可以在 `myAsmEng` 里添加 `C` 语言写的加解密代码

这里以 `xor` 这种简单的加密作为例子,在 `myAsmEng` 里加入这样一段代码

```c
  ifins("codexor") //格式:codexor len,key

  {

    if(!strlen(operand))goto invalid;

    if(!strchr(operand,','))goto invalid;

    char*ptok=strchr(operand,',');

    *ptok=0;

    DWORD len=0x200;

    DWORD key=0;

    sscanf(operand,"%X",&len);

    sscanf(ptok+1,"%X",&key);

    if((len>0x100)||(!len))goto invalid;

    len=len/4;

    if(fseek(fh,srcaddr,0))goto invalid;

    DWORD*databuf=new DWORD[len];

    if(fread(databuf,4,len,fh)!=len)

    {

      delete []databuf;

      goto invalid;

    }

    int i;

    for(i=0;i<len;i++)

      *((DWORD*)buf+i)=databuf[i] ^ key;

    MyRet=len*4;

    goto end;

  }
```

这段代码将当前的代码和 `key` 异或,不过可能只能处理 `0x100` 左右的字节(好象 `Hiew` 一次只会打开这么多)

上面完成了所有的功能,但实际上 `myHiew` 仍然有问题,如果输入 `asc 12345678901234567890` 会发现出现非法操作,出错地址是 `0x33343536`

这个地址正好是 `3456` 的 `ASC` 码,感觉上是因为 `myAsmEng` 返回的数据过多导致栈区溢出了!

重新用 `OD` 加载 `unparked.exe`,在 `0041024D` 这里(调用汇编引擎的地方)设中断,`F3`,`F2`,输入 `1212121212\r`

中断下来,看一下栈区

```
0012FD6C    0012FDD9  ASCII "12121212121212121212"

0012FD70    0012FF4C

0012FD74    00007FFF

0012FD78    0002A010
```

存放机器指令的 `buffer=0012FF4C`,显然这个地址也在堆栈里!查看 `0012FF4C`

```
0012FF4C    00000000

0012FF50    0000FF3D

0012FF54    00000000

0012FF58    0040B286  返回到 unpacked.0040B286 来自 unpacked.004243F0

0012FF5C    0040B2F0  返回到 unpacked.0040B2F0 来自 unpacked.0040F370
```

原来 `Hiew` 只在堆栈里分配了 12 个字节来储存机器码!所以 `asc` 指令插入的字符串长度超过 12 时就会溢出!

我们必须修改机器码缓冲区,否则 `invoke` 这些伪指令都可能出错

注意 `0012FF4C` 附近

```
0012FF0C    00430000  unpacked.00430000

0012FF10    0040F6E5  返回到 unpacked.0040F6E5 来自 unpacked.004100A0

0012FF14    0012FF4C  -------->机器码缓冲区,作为参数传递给了另一个函数

0012FF18    00007FFF

0012FF1C    0002A010

0012FF20    00930770

0012FF24    00000000

0012FF28    00000009

0012FF2C    00000000

0012FF30    00000000

0012FF34    00000000

0012FF38    00000003

0012FF3C    00000000

0012FF40    FFFFFFFF

0012FF44    00434B08  unpacked.00434B08

0012FF48    0000000E

0012FF4C    00000000

0012FF50    0000FF3D

0012FF54    00000000
```

显然这个 `CALL` 里为机器码分配了 12 字节的缓冲区,然后作为参数调用了另一个函数

注意 `0040F6E5` 的代码:

```
0040F6CC   |.  8D4C24 24           lea ecx,dword ptr ss:[esp+24]  //ecx 指向机器码缓冲区 拦截点 1

0040F6D0   |.  50                 push eax

0040F6D1   |.  8B049D A0364400    mov eax,dword ptr ds:[ebx*4+4436A> ----------->拦截后跳回地址

0040F6D8   |.  56                 push esi

0040F6D9   |.  50                 push eax

0040F6DA   |.  68 FF7F0000        push 7FFF

0040F6DF   |.  51                 push ecx  //缓冲区

0040F6E0   |.  E8 BB090000        call unpacked.004100A0

0040F6E5   |.  8BF0               mov esi,eax

0040F6E7   |.  83C4 18            add esp,18
```

修改 `0040F6CC` 这里就可以改变缓冲区了,在 `DLL` 里加入下面的函数并导出

```c
char CodeBuffer[0x100];

void*GetCodeBuffer()

{

  return CodeBuffer;

}
```

然后修改 `0040F6CC`,让它跳到一个空白区,然后调用 `GetCodeBuffer` 来获得缓冲区,让 `ecx` 指向缓冲区,然后跳回去

这里跳转指令会占用 5 个字节,所以要先 `push eax`,然后跳回 `0040F6D1`

当然后面肯定还会有读取机器码的地方,在 `0040F6E5` 设中断,输入一个正确的指令,回车,中断下来,`F8` 跟踪

```
0040F725   |.  8B3D 5C3D4400     mov edi,dword ptr ds:[443D5C]

0040F72B   |.  8B2C9D A0364400   mov ebp,dword ptr ds:[ebx*4+4436A0]

0040F732   |.  8BCE             mov ecx,esi

0040F734   |.  8B82 32010000     mov eax,dword ptr ds:[edx+132]

0040F73A   |.  8D7424 20         lea esi,dword ptr ss:[esp+20] //机器码暂存 拦截点 2

0040F73E   |.  2BF8             sub edi,eax

0040F740   |.  8BC1             mov eax,ecx ------------>拦截后跳转点

0040F742   |.  03FD             add edi,ebp

0040F744   |.  C1E9 02           shr ecx,2

0040F747   |.  F3:A5            rep movs dword ptr es:[edi],dword ptr ds:[esi] ---->保存机器码

0040F749   |.  8BC8             mov ecx,eax

0040F74B   |.  83E1 03           and ecx,3
```

很快发现 `0040F73A` 处 `esi` 也是指向机器码缓冲区,修改方法和上面都一样

这样处理以后 `myHiew` 就可以正常工作了

我修改的结果:

[附件:myHiew.rar](http://www.pediy.com/tools/Editors.htm)
