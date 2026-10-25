
### 代码片段

```
ctmp3=suanfa(ctmp1, ctmp2, m_nChk);

testKey[0]=changeinttochar(ctmp3);

ctmp1=changechartoint(testKey[4]);

ctmp2=changechartoint(testKey[1]);

ctmp3=suanfa(ctmp1, ctmp2, m_nChk);

testKey[6]=changeinttochar(ctmp3);

ctmp1=changechartoint(testKey[6]);

ctmp2=changechartoint(testKey[0]);

ctmp3=suanfa(ctmp1, ctmp2, m_nChk);

testKey[3]=changeinttochar(ctmp3);

m_strKey.Format("%s",testKey);

m_strNote.Format("幸福的芝麻制作");  //广告

SetWindowText("http://zhimaxp.126.com");   //广告

UpdateData(FALSE);

}
```

### 方法实现

```
int CNS_KGDlg::changechartoint(char ch)

{

if(ch>=97)

{

ch-=32;

}

if(ch>=65)

{

ch-=7;

}

return ch-48;

}
```

```
int CNS_KGDlg::suanfa(char a, char b, int k)

{

int c;

if(k==0)

{

c=b+a*2+28;

return c%36;

}

else

{

c=b+a*2+29;

return c%36;

}

}
```

```
char CNS_KGDlg::changeinttochar(int nCh)

{

nCh+=48;

if(nCh>57)

nCh+=7;

if(nCh>90)

nCh+=32;

return nCh;

}
```

----------

程序下载：`<http://202.102.229.61/zhima>`
