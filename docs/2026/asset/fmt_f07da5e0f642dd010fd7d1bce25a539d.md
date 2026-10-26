
```c
size_t decrypt(BYTE* dst, BYTE *src, size_t src_size) 

{

    BYTE *buf=(BYTE*)malloc(0x1000);               

    BYTE *cur_src = src;

    BYTE *cur_dst = dst;

    BYTE *end_dst =  (DWORD)cur_dst + *(DWORD*)cur_src;

    DWORD idx_buff, i, last, c1, c2;



    cur_src += 4;

    memset(buf, 0, 0xFEE);

    idx_buff = 0XFEE;



    c1 = 0; //index byte

    while(1)

    {

        c1 >>= 1;

        if (!(c1 & 0x100)) // c1 bit[9] is 0, it means do 8 times

        {

            c1 = *cur_src;

            c1 |= 0xff00; //make a mark, and to 16bit

            cur_src++;

        }

        if ((BYTE)c1 & 0x1) //copy to buf directly

        {

            buf[idx_buff] = *cur_src;

            *cur_dst = *cur_src;

            idx_buff++;

            idx_buff &= 0xFFF; //cicle buffer

            cur_dst++;

            cur_src++;

            if(cur_dst >= end_dst) 

            {

                free(buf);

                return (size_t)((BYTE*)end_dst - dst);    

            }

        }

        else

        {

            c2 = *(cur_src+1); //index byte2

            i = *cur_src | ((c2 & 0XF0)<<4); //use c1 and c2 (higher 4bits) to determine index

            last = (c2 & 0xf) + i +2; // length = c2 lower 4bit, 2 without length 2 chars

            cur_src += 2; //c1, c2 two index byte



            if(i > end_dst)

            {

                continue;

            }

            do

            {

                buf[idx_buff] = buf[i & 0xFFF];

                *cur_dst = buf[i & 0xFFF];

                idx_buff++;

                idx_buff &= 0xFFF;

                cur_dst++;

                if(cur_dst >= end_dst) 

                {

                    free(buf);

                    return (size_t)((BYTE*)end_dst - dst);  

                }

                i++;

            } while (i <= last);

        }

    }

}
```

并且可以总结出怎么来解密文本了。

### sn.bin 结构

> `0x0` `size` `4`
> `0x4`~ `data`

> `data`部分第一字节为索引，每一位代表当前byte的状态,
> 索引为1：直接copy当前byte到环状缓存区
> 索引为0：则表示当前byte和下一个byte是索引，高12位为位置，低4位为长度。
> 环状缓存区大小`0x1000`，起始位置`0xFEE`。

顺便说一下位运算充当循环遍历的方法吧，

`a = 0b0011111`，`a>>1`需要移位5次`a=0`，则循环次数是5

测试一下文本没问题：

![](https://bbs.kanxue.com/upload/attach/202006/617776_76QJDHZM84EN8J2.png)

## 0x5 后记

之前在分析psv版ida看了半天也没有找到文本位置，

psv的ida loader真的不好用，好多函数识别不出来，字符串也无法定位，ps4版还稍微好点，至少x64比arm汇编看着要舒服。所以这次来分析一下pc版文本。

定位还算简单，然后又稍微逆向了一下算法，还算比较容易。

**iwaihime psv eboot**

![](https://bbs.kanxue.com/upload/attach/202006/617776_8N47YDGYCHRKATV.png)

**iwaihime ps4 eboot**

![](https://bbs.kanxue.com/upload/attach/202006/617776_4RF72GKYAPU7Q57.png)

其实对数据比较敏感的看到`0x1000`，`0XFEE`就知道这其实不是加密，

就是LZSS的压缩，游戏里就是原封不动的把源码复制过去了。

我分析完算法后还以为是变种的RLE结果再仔细想想竟然就是LZSS，以前只是简单了解，

这次逆向算法后基本上对LZSS有了更深刻的了解了。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-6-18 17:57

被kanxue编辑，原因：
