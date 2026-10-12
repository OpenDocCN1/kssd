
```cpp
{  
  CString temp;  
  // TODO: Add your control notification handler code here  
    unsigned char *v;  
  CString a,b,c;  
  CBigInt m,e,n,KeyL,KeyR;  
  unsigned char k[17]="You are big pig.";//key  
  unsigned long w[2];  
  this->UpdateData();  
  temp=m_UserName;  
  if(m_UserName.GetLength()<8)  
  {  
    if(m_UserName.GetLength()==0)  
    {  
      m_UserName.Insert(0,"QQKav");  
      temp="QQKav";  
    }  
  m_UserName.Insert(m_UserName.GetLength(),"请 Cracker 手下留情");  
  }  
  v=(unsigned char *)m_UserName.GetBuffer(m_UserName.GetLength());  
    tiny_encrypt((unsigned long *const)v,w,(unsigned long *const)k);  
  w[1]=w[0];//由于软件并没用到输出端的后四个字节，所以我这里就用来保存前四个字节，等到下面计算注册码后部份要用到  
    w[0]=w[0] & 0x3FFFFFFF;  
    w[0]+=2;  
  w[1]=w[1]>>0x1E;//保存的前四个字节  
  w[1]=w[1]+0x24F80052;//简单的运算后，参与 rsa 解密，算出注册码后半部份  
  a.Format("%d",w[0]);  
  m.Get(a,DEC);  
  e.Get(CString("2C86F9"));  
  n.Get(CString("69AAA0E3"));  
  KeyL=m.RsaTrans(e,n);  
  KeyL.Put(m_KeyL);  
    KeyR=KeyL.Add(w[1]);  
  KeyR.Put(m_KeyR);  
    KeyR=KeyR.RsaTrans(e,n);  
  KeyR.Put(m_KeyR);  
  m_UserName=temp;  
    this->UpdateData(FALSE);  
}  
```

搞定，收工。谢谢大家观看！！！

`4dd466d0e1e0cc123afbe504da2462ca`
