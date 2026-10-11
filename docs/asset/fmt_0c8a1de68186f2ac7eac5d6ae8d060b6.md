
# wincrack

> 原文：[`www.pediy.com/kssd/pediy02/forum269.htm`](https://www.pediy.com/kssd/pediy02/forum269.htm)

各位高手，大侠请帮我看一下这个程序 `wincrack`，它是用 `UPX` 加的壳，脱壳倒是很简单，但我一直追不出它的注册码，它好象是根据你输入的注册码来比较注册名，不知我看懂没有？？？还请各位大侠指点我。呜。。呜。。呜。。

它的下载地址是：`http://www.imall.com.cn/register/dl.asp?name=filename&url=http://home.online.tj.cn/~whf/whf/wincrack.zip`

`Name:` `郭大志`
`Code:` `1F66167076F15F`

`cracking for fun`

看看就知道算法是怎么样的了。第一个 `for` 循环可以优化，懒得做了。

```c
int k, len;
unsigned char name[7];
unsigned long b[6] = { 0xAD, 0xC0, 0x90, 0x9B, 0x26, 0xC3};
unsigned long a[6] = { 0x8C, 0x5E, 0x1A, 0x35, 0x4D, 0x96};
unsigned long c[6] = { 0xD1, 0xDB, 0x24, 0xBB, 0xC5, 0x82};
unsigned long serial[7], tmp;
char buf[64];

len = GetDlgItemTextA(hDlg, IDC_USERNAME, (char *)name, 7);
if ( (len < 4) || (len > 6))
{
    SetDlgItemTextA(hDlg, IDC_REGCODE, "There must be 4~6 chars in your name.");
    return FALSE;
}

for (k = 0; k < len; k++)
{
    name[k] ^= (a[k] ^ b[k]);
}

//随机数
serial[len] = GetTickCount( ) & 0xFF;

for(k = len -1; k >=0; k--)
{
    c[k] ^= serial[k+1];
    tmp = (unsigned long)name[k];
    if (tmp > c[k])
    {
        serial[k] = c[k] + 0x00FF - tmp;
    }
    else
    {
        serial[k] = c[k] - tmp;
    }
}

for(k =0; k <= len; k++)
{
    wsprintf(&buf[k*2], "%02lX", serial[k]);
}

SetDlgItemTextA(hDlg, IDC_REGCODE, buf);
```

`4dd466d0e1e0cc123afbe504da2462ca`
