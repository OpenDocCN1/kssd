
`malloc_32`后面

![](https://bbs.kanxue.com/upload/attach/202003/840122_32MY25533JGMU54.png)

![](https://bbs.kanxue.com/upload/attach/202003/840122_XZG8FEB72H2DXBC.png)

`sub_9C7AC`函数调用`java`方法`intValue`拿到返回值

后面还有一个解密函数`sub_903F0`，直接运行一下返回值是`0`

对`Java`层传递过来的参数就处理完了

后面对 `sub_9A568`的返回值再次处理，申请`malloc_8`

```
malloc_8= (_DWORD *)malloc_0(8);

  if ( !malloc_8)

  {

    sub_A1308(0, v22, v23, v24, 0, v22);

    __asm { LDCL            p15, c15, [R8], {0xFF} }

  }

  *malloc_8= 0;

  malloc_8[1] = 0;

 v23 = 4 * number;

  *malloc_8 = **(_DWORD **)(hashobj + 4 * number);

  malloc_8[1] = *(_DWORD *)(*(_DWORD *)(hashobj + 4 * number) + 4);

  *(_DWORD *)(malloc_off+ 4 * number) = malloc_8;
```

`malloc_off`是解密函数执行完之后申请的一块

`ArrayLength`*4 大小的内存（

`sub_9A56` 里的

`ArrayLength`

）

![](https://bbs.kanxue.com/upload/attach/202003/840122_7MA8YTBM6NV8WYQ.png)

之后调用 `sub_46708`函数 传入`malloc_off`的指针和一个值为`0x63`的指针

`sub_46708` 函数一共有三次对 `0x9db0` 的调用，没有获取任何返回值，不知道是干什么的。。

![](https://bbs.kanxue.com/upload/attach/202003/840122_4W8CQ2U42CJQXEJ.png)

这个函数地址是`0x9db0` 就是`doCommandNative` 解密处理参数的函数

然后偏移`0x46BD2` 处是个

`Switch` `r1`的值就是`0x4977c`

![](https://bbs.kanxue.com/upload/attach/202003/840122_QGRS9T3G898AMY9.png)

`0x4977c` 里面有个很大`switch`循环，不知道为什么还不能

Specify switch idiom 。只能将就看了

![](https://bbs.kanxue.com/upload/attach/202003/840122_CMF8PWWAC9CXPAN.png)

`switsh_2`

![](https://bbs.kanxue.com/upload/attach/202003/840122_BVZDQGP5DA88R8W.png)

这段数据怎么来的我还没找到，不过我重打包`apk`和修改`key`的值都不会正常返回`sign`，这段数据应该就是`apk`的签名了，这四层`switch`循环弄得像个迷宫一样

![](https://bbs.kanxue.com/upload/attach/202003/840122_U9WFADEWBXW5RDX.jpg)

`key_2`并不是由`key`加密或者其他方式得来的，猜测于`apk`的签名验证有关

最后的`sign`签名算法是在`0x75dda`方法里的`case0x17`里面。偏移`0x76ff6`,传入的两个参数都是`malloc_56`结构体，第一个结构体的值是`key2`，第二个结构体的值是`hash+time+`“6.3“（`http`心跳检测时就为`time+`“6.3”）

```
int __fastcall Decryptkey(int key2_malloc, int hash)

{

hash_ = hash;

  v48 = *off_ABBA8;

  v3 = 0;

  if ( key2_malloc )

  {

    if ( hash )

    {

      key2_size = *(key2_malloc + 4);

      if ( key2_size )

      {

        if ( *(hash_ + 4) )

        {

          if ( key2_size < 65 )

            v5 = (*(key2_malloc + 40))();       // 创建个malloc_56结构体，将key2copy过去

          else

            sub_78530();

          v7 = v5;

          m_key2 = 0;

          if ( v5 )

          {

            v9 = *(v5 + 4) == 0;

            v10 = 0;

            v11 = 0;

            v3 = 0;

            if ( !v9 )

            {

              v44 = mirror_key2_1;

              key2 = v7;

              v45 = v7;

              aeabi_memclr4(mirror_key2_1, 64);

              aeabi_memclr4(mirror_key2_2, 64);

              memcpy8(mirror_key2_1, *key2, key2[1]);

              memcpy8(mirror_key2_2, *key2, key2[1]);

              i = 0;

              do

              {

                mirror_key2_1[-i] ^= 0x36u;

                mirror_key2_2[-i] ^= 0x5Cu;

                --i;

              }

              while ( i != -64 );

              sub_97DA4();                      // 申请 malloc_56_100

              m_key2 = malloc_56_100;

              (*(malloc_56_100 + 12))();        // copy mirror_key2_1

              (*(m_key2 + 48))(m_key2, hash_);  // mirror_key2_1后面加上hash

              sub_78530();                      // 这里面的算法不想看了

              v16 = v15;

              v11 = 0;

              if ( v15 )

              {

                v3 = 0;

                v7 = v45;

                if ( *(v15 + 4) )

                {

                  sub_98594();                  // 申请malloc_56,里面的数据大小为0x40,并copy过去mirror_key_2_2

                  v18 = v17;

                  (*(v17 + 48))();              // 在刚刚申请的malloc_56的数据mirror_key2_2后面拼接上hash

                  sub_78530();                  // 再次执行这个函数执行后的返回值就是key_3

                  v11 = v18;

                  v7 = v45;

                  v3 = v19;

                }

              }

              else

              {

                v3 = 0;

                v7 = v45;

              }

              v10 = v16;

            }

          }

          else

          {

            v10 = 0;

            v11 = 0;

            v3 = 0;

          }

          v20 = v10;

          free_malloc56(v7, v7, v11, v6, v43, v44, v45, v11);

          free_malloc56(m_key2, v21, v22, v23, v24, v25, v26, v27);

          free_malloc56(v20, v28, v29, v30, v31, v32, v33, v34);

          free_malloc56(v41, v35, v36, v37, v38, v39, v40, v41);

        }

      }

    }

  }

  result = *off_ABBA8 - v48;

  if ( *off_ABBA8 == v48 )

    result = v3;

  return result;

}
```

本来想把算法复现的，可惜调试调试着就蒙圈了[捂脸]

[捂脸] ，现在肺炎也快过去了，不玩了开始找工作去，哈哈哈。

![](https://bbs.kanxue.com/upload/attach/202003/840122_5RZ6U699H9MNUSM.gif)

就这样吧，我已经把原版的`so`发了出来，签名函数位置也说了，大佬们有兴趣可以去看看。

这个函数返回值是一个`malloc` 大小`0x14`的数据(取个名字叫`key_3`)，经过算法

```
  do

          {

            _byte = *(*key_3 - i);              // 作用

            v7 = (_byte >> 4) + 87;             // 循环获取key_3里的一个byte值

            if ( _byte >> 5 <= 4 )              // 如果byte小于0x80,就将byte逻辑右移4位后逻辑或上0x30

                                                //     不小于就byte逻辑右移4位后加上87(0x57)

              v7 = (_byte >> 4) | 0x30;

            offset = 2 * i;

            *(*sign - 2 * i) = v7;              // 得到的值放入sign(结果)中 i是为负数

            v9 = _byte & 0xF;                   // byte与0xf,结果小于0xa就或上0x30

                                                //           否则结果加上0x5c

                                                // 熟悉ascll码就知道0x30-0x39对应0-9，0x61-0x7a对应小写的字母

            if ( v9 < 0xA )

            {

              v11 = v9 | 0x30;

              sign_off = *sign - offset;

            }

            else

            {

              sign_off = *sign - offset;

              v11 = v9 + 87;

            }

            *(sign_off + 1) = v11;              // 放入sign中

            --i;

            key_3 = key_3_;

          }

          while ( -var_0x14 != i );
```

扩展成`0x28`的数据，这个就是最终的`sign`了

最后这个`token` 通过抓包改登录协议的`imei`或者重新注册账号去登录，连我换几个热点都是返回一样的值，估计是使用更底层的设备`id`加密得到的值，，，这就超出了我的认知了，还望大佬们提示一下。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2020-3-16 11:31

被`yezheyu`编辑

，原因： 其他原因
