
# 对 CardMaker5.2 部分注册算法研究

> 原文：`www.pediy.com/kssd/pediy11/111543.html`

## 一、软件防调手段

1. 用 `EnableWindow` 方法禁止窗口活动。
2. 验证代码在取得输入函数很远的地方，用 `PostMessage` 方式。
3. 对验证代码之间用 `GetTickCount` 检测执行时间，小于 100ms，失败。

## 二、注册流程

1. 用 `CreateFile` 方式取得硬盘序列号，失败用 `GetColumeInformation`;
2. 对硬盘序列号前 8 个字节作运算得一个 `DWORD` 数;
3. 对上面的 `DWORD` 数查表计算得出一个 `QWORD` 数;
4. 对输入的注册码写入 ini 文件;
5. 对输入前半部分数运算得出个 `DWORD` 数;
6. 对输入后半部分数运算得出个 `DWORD` 数;
7. 浮点指令装入 3 步的 `QWORD` 数，浮点比较 5、6 步产生的两个 `DWORD` 数;
8. 相等就注册成功(一台机器理论有很多注册码，我只找到两个成功注册)。

## 三、遗留的问题

1. 还不能逆 5、6 步的算法，只能穷举，穷兴有点慢，20 分钟才找到 2 个;
2. 软件产生的序列号和硬盘序列号的关系没找到;

## 四、部分伪代码(代码是 OD 动态分析来的)

### 1: 第二步的

```cpp
DWORD CServUPAWDlg::MyCalc1()  
{  
  UpdateData(TRUE);  
  DWORD  myreturn = 0;  
  BYTE  a1,a2,a3,a4;  
  a1 = m_PdHash.GetAt(0) * m_PdHash.GetAt(1);  
  a2 = m_PdHash.GetAt(2) * m_PdHash.GetAt(3);  
  a3 = m_PdHash.GetAt(4) * m_PdHash.GetAt(5);  
  a4 = m_PdHash.GetAt(6) * m_PdHash.GetAt(7);  
  myreturn = a4;  
  myreturn <<= 0x08;  
  myreturn += a3;  
  myreturn <<= 0x08;  
  myreturn += a2;  
  myreturn <<= 0x08;  
  myreturn += a1;  
  return myreturn;  
}
```

### 2: 第三步的

```cpp
void CServUPAWDlg::MyEncHash()  
{  
  int    i =0;  
  PwHash2 = 0x19a28;  
  PwHash1 = DiskHash;  
  DWORD  temp1 = DiskHash;  
  CString msg;  
  do  
  {  
    temp1 = PwHash1;  
    temp1 += Table5F2CF8[Table5F2D98[i]];  
    PwHash2 ^= MyEnc4228F0(temp1);  
    MyEnc4229A0();  
    i++;  
  }while(i<32);  
  MyEnc4229A0();  
  return;  
}  

DWORD CServUPAWDlg::MyEnc4228F0(DWORD a)  
{  
  DWORD  temp = a;  
  DWORD  bb;  
  DWORD  myreturn = 0;  
  BYTE  a1;  
  temp >>= 0x18;  
  temp &= 0x0f;  
  a1 = Table5F2D78[temp];  
  temp = a;  
  temp >>=0x1c;  
  temp &= 0x0f;  
  myreturn = Table5F2D88[temp];  
  myreturn <<= 4;  
  myreturn |= a1;  
  temp = a;  
  temp >>= 0x14;  
  temp &= 0x0f;  
  a1 = Table5F2D68[temp];  
  myreturn <<= 4;  
  myreturn |= a1;  
  temp = a;  
  temp >>= 0x10;  
  temp &= 0x0f;  
  a1 = Table5F2D58[temp];  
  myreturn <<= 4;  
  myreturn |= a1;  
  temp = a;  
  temp >>= 0x0c;  
  temp &= 0x0f;  
  a1 = Table5F2D48[temp];  
  myreturn <<= 4;  
  myreturn |= a1;  
  temp = a;  
  temp >>= 0x08;  
  temp &= 0x0f;  
  a1 = Table5F2D38[temp];  
  myreturn <<= 4;  
  myreturn |= a1;  
  temp = a;  
  temp >>= 0x04;  
  temp &= 0x0f;  
  a1 = Table5F2D28[temp];  
  myreturn <<= 4;  
  myreturn |= a1;  
  temp = a;  
  temp &= 0x0f;  
  a1 = Table5F2D18[temp];  
  myreturn <<= 4;  
  myreturn |= a1;  
  bb = myreturn;  
  myreturn = (bb >> 0x15 | myreturn << 0x0b);  
  return myreturn;  
}  

void CServUPAWDlg::MyEnc4229A0()  
{  
  //交换 a1 和 a2 的内容  
  __asm  
  {  
    push PwHash1  
    push PwHash2  
    pop  PwHash1  
    pop  PwHash2  
  }  
  return ;  
}
```

### 3: 第 5、6 步的

```cpp
DWORD  MyEncA(CString part1)  
{  
  int    slen = ::strlen(part1);  
  DWORD  A = 0;  
  DWORD  B = 0;  
  int    i = 0;  
  for(i=slen;i>0;i--)  
  {  
    B *= 8;  
    B -= A;  
    B *= 5;  
    B += ChangeChar(part1.GetAt(i-1));//str[i]要经过转换，如 0x39->0x09  
    A = B;  
  }  
  return B;  
}  

DWORD  MyEncB(CString part2)  
{  
  int    slen = ::strlen(part2);  
  DWORD  A = 0;  
  DWORD  B = 0;  
  int    i = 0;  
  for(i=slen;i>0;i--)  
  {  
    B <<= 4;  
    B += A;  
    B <<= 1;  
    B += ChangeChar(part2.GetAt(i-1));//str[i]要经过转换，如 0x39->0x09  
    A = B;  
  }  
  return B;  
}
```
