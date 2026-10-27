
将 `serial` 填入 `inbuff` 的高 64 位，同时也填入低 64 位，`MD5(inbuff)` 得到 `key`

这个 `MD5` 有三处变形，变形没有什么新意，还是老一套：

1. 四个变量的变形
2. 数据填充变形
3. 参与 `HASH` 运算的变量的顺序变形，`HASH` 本身没有变形。

----------------------------------------------------------------------------------------

## 【注册机源码】

因为程序只对 128 位消息进行 `MD5` 编码，为了便于编辑，没有采用类，大家直接看吧。

### // 响应 Generate 按钮

``` 
void CkeygenDlg::OnOK()
{
        // TODO: Add extra validation here
        CkeygenDlg::OnChangeEdit1();
}
```

``` 
void CkeygenDlg::OnChangeEdit1()
{
        // TODO: If this is a RICHEDIT control, the control will not
        // send this notification unless you override the CDialog::OnInitDialog()
        // function and call CRichEditCtrl().SetEventMask()
        // with the ENM\_CHANGE flag ORed into the mask.
        // TODO: Add your control notification handler code here
                UpdateData(true);
        Beep(1000,50);
        char inbuff[16] = {0},name[12];
        unsigned long state[4];
        int i, n, namelen;
        //对用户名处理
        namelen = m\_Edit1.GetLength ();
        n = namelen >> 3 ;
        for(i = 0;i<n;i++){
                strcpy(name,m\_Edit1.Mid (i<<3,8))       //取用户名的8个字符
                strrev(name);                           //将8个字符顺序反转
                memcpy(inbuff,name,8);                  //复制到待加密信息的高64位
                MD5(inbuff,state);                      //MD5编码
                memcpy(inbuff,state,16);                //替换掉待加密信息
        }
        n = namelen & 7 ;                               //判断用户名有没有处理完毕
        strcpy(name,m\_Edit1.Mid (i<<3,n));;             //取出剩余字符
        strrev(name);                                   //剩余字符顺序反转
        i = 8 - n ;                                     //计算补0的个数
        memset(inbuff,0,i);                             //将0填入待加密信息
        memcpy(inbuff+i,name,n);                        //将反转字符填入待加密信息
        MD5(inbuff,state);                              //MD5编码
        m\_Edit2.Format ("%08X",state[3]);               //将 State[3] 转为16进制字符串作为 Serial 的一部分
        m\_Edit2.Insert (0,"0018");                      //serial 前面插入"0018"
        n = rand ();
        sprintf(inbuff,"%04X",n);
        m\_Edit2 += inbuff;                              //serial 的最后四位任意，并且传递给编辑框
        memcpy(inbuff,&n,2);
        memcpy(inbuff+2,state+3,4);
        inbuff[6] = char(0x18);
        inbuff[7] = char(0);
        memcpy(inbuff+8,inbuff,8);                      //serial 填充到 inbuff
        MD5(inbuff,state);                              //对 serial MD5编码
        sprintf(inbuff,"%08X",state[2]);
        m\_Edit3.Format ("%08X",state[3]);               //取 state[2] 和 state[2] 作为 Code
        m\_Edit3 += inbuff;                              //将 code 传递给编辑框
        UpdateData(false);
}
```

### // 将 128 待加密信息填充成 512 位，即16个整数

``` 
void CkeygenDlg::FillBuff(unsigned long \*buff,char \*inbuff)
{
        int \*from=(int \*)inbuff;
        buff[3]=\*from;  buff[2]=\*(from+1);  buff[1]=\*(from+2);  buff[0]=\*(from+3);
        buff[4]=\*from;  buff[5]=\*(from+1);  buff[6]=\*(from+2);  buff[7]=\*(from+3);
        buff[11]=\*from; buff[10]=\*(from+1); buff[9]=\*(from+2);  buff[8]=\*(from+3);
        buff[12]=\*from; buff[13]=\*(from+1); buff[14]=\*(from+2); buff[15]=\*(from+3);
}
```

### //四个函数定义1 FF

``` 
unsigned long CkeygenDlg::FF(unsigned long a, unsigned long b, unsigned long c, unsigned long d, unsigned long x, byte s, unsigned long ac)
{
        a = ((b & c)|((~b)&d)) + a + x + ac;
        a = (a<<s)|(a>>(32-s)) ;
        a += b;
        return a;
}
```

### //四个函数定义2 GG

``` 
unsigned long CkeygenDlg::GG(unsigned long a, unsigned long b, unsigned long c, unsigned long d, unsigned long x, byte s, unsigned long ac)
{
        a = ((b&d)|(c&(~d))) + a + x + ac ;
        a = (a<<s)|(a>>(32-s)) ;
        a += b;
        return a;
}
```

### //四个函数定义3 HH

``` 
unsigned long CkeygenDlg::HH(unsigned long a, unsigned long b, unsigned long c, unsigned long d, unsigned long x, byte s, unsigned long ac)
{
        a = (b^c^d) + a + x + ac ;
        a = (a<<s)|(a>>(32-s)) ;
        a += b;
        return a;
}
```

### //四个函数定义4 II

``` 
unsigned long CkeygenDlg::II(unsigned long a, unsigned long b, unsigned long c, unsigned long d, unsigned long x, byte s, unsigned long ac)
{
        a = ( c^(b|(~d)) ) + a + x + ac ;
        a = (a<<s)|(a>>(32-s)) ;
        a += b;
        return a;
}
```

### //MD5运算主函数 待加密信息保存在在inbuff中，结果保存在state[4]中

``` 
void CkeygenDlg::MD5(char \*inbuff,unsigned long \*state)
{
        unsigned long context[16];
        state[0]=0x1234567;
        state[1]=0x89ABCDEF;
        state[2]=0xBA98FEDC;
        state[3]=0x76504321;
        FillBuff(context,inbuff);
        //------------- round 1 -------------//
        state[3] = FF(state[3],state[2],state[1],state[0],context[0],7,0xd76aa478);  // -1
        state[0] = FF(state[0],state[3],state[2],state[1],context[1],12,0xe8c7b756); // -2
        state[1] = FF(state[1],state[0],state[3],state[2],context[2],17,0x242070db); // -3
        state[2] = FF(state[2],state[1],state[0],state[3],context[3],22,0xc1bdceee); // -4
        state[3] = FF(state[3],state[2],state[1],state[0],context[4],7,0xf57c0faf);  // -5
        state[0] = FF(state[0],state[3],state[2],state[1],context[5],12,0x4787c62a); // -6
        state[1] = FF(state[1],state[0],state[3],state[2],context[6],17,0xa8304613); // -7
        state[2] = FF(state[2],state[1],state[0],state[3],context[7],22,0xfd469501); // -8
        state[3] = FF(state[3],state[2],state[1],state[0],context[8],7,0x698098d8);  // -9
        state[0] = FF(state[0],state[3],state[2],state[1],context[9],12,0x8b44f7af); //-10
        state[1] = FF(state[1],state[0],state[3],state[2],context[10],17,0xffff5bb1);//-11
        state[2] = FF(state[2],state[1],state[0],state[3],context[11],22,0x895cd7be);//-12
        state[3] = FF(state[3],state[2],state[1],state[0],context[12],7,0x6b901122); //-13
        state[0] = FF(state[0],state[3],state[2],state[1],context[13],12,0xfd987193);//-14
        state[1] = FF(state[1],state[0],state[3],state[2],context[14],17,0xa679438e);//-15
        state[2] = FF(state[2],state[1],state[0],state[3],context[15],22,0x49b40821);//-16
        //------------- round 2 -------------//
        state[3] = GG(state[3],state[2],state[1],state[0],context[1],5,0xf61e2562);  //-17
        state[0] = GG(state[0],state[3],state[2],state[1],context[6],9,0xc040b340);  //-18
        state[1] = GG(state[1],state[0],state[3],state[2],context[11],14,0x265e5a51);//-19
        state[2] = GG(state[2],state[1],state[0],state[3],context[0],20,0xe9b6c7aa); //-20
        state[3] = GG(state[3],state[2],state[1],state[0],context[5],5,0xd62f105d);  //-21
        state[0] = GG(state[0],state[3],state[2],state[1],context[10],9,0x2441453);  //-22
        state[1] = GG(state[1],state[0],state[3],state[2],context[15],14,0xd8a1e681);//-23
        state[2] = GG(state[2],state[1],state[0],state[3],context[4],20,0xe7d3fbc8); //-24
        state[3] = GG(state[3],state[2],state[1],state[0],context[9],5,0x21e1cde6);  //-25
        state[0] = GG(state[0],state[3],state[2],state[1],context[14],9,0xc33707d6); //-26
        state[1] = GG(state[1],state[0],state[3],state[2],context[3],14,0xf4d50d87); //-27
        state[2] = GG(state[2],state[1],state[0],state[3],context[8],20,0x455a14ed); //-28
        state[3] = GG(state[3],state[2],state[1],state[0],context[13],5,0xa9e3e905); //-28
        state[0] = GG(state[0],state[3],state[2],state[1],context[2],9,0xfcefa3f8);  //-30
        state[1] = GG(state[1],state[0],state[3],state[2],context[7],14,0x676f02d9); //-31
        state[2] = GG(state[2],state[1],state[0],state[3],context[12],20,0x8d2a4c8a);//-32
        //------------- round 3 -------------//
        state[3] = HH(state[3],state[2],state[1],state[0],context[5],4,0xfffa3942);  //-33
        state[0] = HH(state[0],state[3],state[2],state[1],context[8],11,0x8771f681); //-34
        state[1] = HH(state[1],state[0],state[3],state[2],context[11],16,0x6d9d6122);//-35
```
