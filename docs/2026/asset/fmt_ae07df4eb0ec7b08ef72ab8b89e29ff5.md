
```
char ch[]={"0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"};

char n[13]={"/0"};             //用来存储注册码

srand((unsigned)time(NULL));

for(int i=0;i<3;i++)         //用随机数来产生注册码的前三位
    n[i]=ch[rand()%62];

n[3]=(n[0] | 0X33) % 0xA+0X30;
n[4]=(n[1] | 0X33) % 0x1A+0X41;
n[5]=(n[2] | 0X33) % 0x1A+0x61;
n[6]=(n[0] & 0x11) % 0xA+0X30;
n[7]=(n[1] & 0x11) % 0x1A+0x61;
n[8]=(n[2] & 0x11) % 0x1A+0x41;
n[9]=(n[3] & 0x11) % 0xA+0X30;
n[10]=(n[4] & 0x11) % 0x1A+0x61;
n[11]=(n[5] & 0x11) % 0x1A+0x41;
GetDlgItem(IDC_CODE)->SetWindowText(n);  //把注册码显示出来
```

---

### 经验总结

算法分析要有耐心，否则。。。。。。。。。

---

2009年08月23日 11:56:54

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
