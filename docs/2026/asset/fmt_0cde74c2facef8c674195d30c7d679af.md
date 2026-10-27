
```
}
name = name+123456;
```

## 2、假码的变化

设假码为 `char passcode[4][len]`；

密码表为 `char*`

`table="L5WXT6VMB4GE3RUYJ9HASF8KQNPD7CI"`;

假码变码为 `long code[4]`；

则有：

```c
int num;
char ch;
for(int i=0;i<4;i++)
{
    code[i]=0;
    for(int j=len-1;j>=0;j--)
    {
        ch = passcode[i][j];
        for(int k=0;k<30;k++)
        {
            if(table[k]==ch)
            {
                num=k;
                break;
            }
        }
        code[i]=code[i]*30+num;
    }
}
```

## 3、RSA验证

模 `n =14391002295252874609`

公钥 `d =65537`

可计算出私钥 `e =3084742971672649089`

既然 `假码变码^d mod n = 用户名变码`

所以 `假码变码 = 用户名变码^e mod n`

对以上算法求逆就是注册机算法，留给有兴趣的朋友自己练习

练习吧，呵呵，有一点点难度哦。

## 结语：

如果你在写注册机的过程中对现有的大数运算库不是很满意，敬请关注俺的下篇教程《RSA与大数运算》，谢谢！
