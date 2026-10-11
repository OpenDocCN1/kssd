
```asm
0167:0041031B  PUSH    EDX   
0167:0041031C  CALL    `MFC42!ord_00000B02`   
0167:00410321  ADD     ESP,BYTE +0C   
0167:00410324  LEA     EAX,[ESP+34]   
0167:00410328  LEA     ECX,[ESP+24]   
0167:0041032C  MOV     DWORD [ESP+34],0104   
0167:00410334  PUSH    EAX   
0167:00410335  PUSH    DWORD 0104   
0167:0041033A  CALL    `MFC42!ord_00000B63`   
0167:0041033F  PUSH    EAX   
0167:00410340  CALL    `KERNEL32!GetComputerNameA`<----取机器名   
0167:00410346  PUSH    BYTE -01   
0167:00410348  LEA     ECX,[ESP+28]   
0167:0041034C  CALL    `MFC42!ord_000015C4`   
0167:00410351  MOV     ECX,[ESP+24]   
0167:00410355  PUSH    BYTE +03   
0167:00410357  CMP     DWORD [ECX-08],BYTE +03 <----如果机器名大于 3 字节,则软件编号的前 3 位由机器名中取得,如果小于 3 位,则取软件编号的 3 位.   
0167:0041035B  JNL     0041038B   
0167:0041035D  LEA     EDX,[ESP+2C]   
0167:00410361  LEA     ECX,[ESP+24]   
0167:00410365  PUSH    EDX   
0167:00410366  CALL    `MFC42!ord_0000164E`   
0167:0041036B  PUSH    EAX   
0167:0041036C  LEA     ECX,[ESP+28]   
0167:00410370  MOV     BYTE [ESP+BC],09   
0167:00410378  CALL    `MFC42!ord_0000035A`   
0167:0041037D  MOV     BYTE [ESP+B8],08   
0167:00410385  LEA     ECX,[ESP+28]   
0167:00410389  JMP     SHORT 004103B9   
0167:0041038B  LEA     EAX,[ESP+2C]   
0167:0041038F  PUSH    BYTE +01   
0167:00410391  PUSH    EAX   
0167:00410392  LEA     ECX,[ESP+30]   
0167:00410396  CALL    `MFC42!ord_000010B6`   
0167:0041039B  PUSH    EAX   
0167:0041039C  LEA     ECX,[ESP+28]   
0167:004103A0  MOV     BYTE [ESP+BC],0A   
0167:004103A8  CALL    `MFC42!ord_0000035A`   
0167:004103AD  MOV     BYTE [ESP+B8],08   
0167:004103B5  LEA     ECX,[ESP+28]   
0167:004103B9  CALL    `MFC42!ord_00000320`   
0167:004103BE  LEA     ECX,[ESP+24]   
0167:004103C2  CALL    `MFC42!ord_0000106A`   
0167:004103C7  LEA     ECX,[ESP+20]   
0167:004103CB  LEA     EDX,[ESP+24]   
0167:004103CF  PUSH    ECX   
0167:004103D0  LEA     EAX,[ESP+40]   
0167:004103D4  PUSH    EDX   
0167:004103D5  PUSH    EAX   
0167:004103D6  CALL    `MFC42!ord_0000039A`   
0167:004103DB  PUSH    EAX   
0167:004103DC  LEA     ECX,[ESP+AC]   
0167:004103E3  MOV     BYTE [ESP+BC],0B   
0167:004103EB  CALL    `MFC42!ord_0000035A`   
0167:004103F0  LEA     ECX,[ESP+3C]   
0167:004103F4  MOV     BYTE [ESP+B8],08   
0167:004103FC  CALL    `MFC42!ord_00000320`   
0167:00410401  LEA     ECX,[ESP+30]   
0167:00410405  PUSH    ECX   
0167:00410406  LEA     ECX,[ESP+A4]   
0167:0041040D  CALL    `MFC42!ord_0000035A`   
0167:00410412  LEA     ECX,[ESP+40]   
0167:00410416  MOV     [ESP+A4],EDI   
0167:0041041D  CALL    `MFC42!ord_000009D2`<----这里弹出"注册"对话框.   
```

---

**VC 注册机代码**

```cpp
void CKeyGenDlg::OnOK()   
{   
    // TODO: Add extra validation here   
    this->UpdateData(true);   
    if(this->m_input == "")   
    {   
        this->m_output = "请输入注册用户名";   
        this->m_edit2.SetWindowText(this->m_output);   
        return;   
    }   
    //this->m_input 是输入的用户名，this->m_output 是输出用的注册码。   
    char buf[50];   
    char regcode[20];   
    unsigned long reg;   
    strcpy(buf,this->m_input);   
    reg = hextoi(buf+3);   
    reg -= 0xb4e9d;   
    reg+= 0xbcc09;   
    sprintf(buf,"%8lu",reg);   
    buf[3]= ((buf[0]+buf[1]+buf[2]-0x90) % 7)+0x30;   
    buf[7]= ((buf[4]+buf[5]+buf[6]-0x90) % 7)+0x30;   
    sprintf(regcode,"%c%c%c%c-%c%c%c%c-%c%c%c%c-%c%c%c%c",   
            buf[0]+0x11,   
            buf[6],   
            buf[2]+0x1b,   
            buf[5],   
            buf[7]+0x12,   
            buf[4],   
            buf[3]+0x13,   
            buf[6],   
            buf[5]+0x11,   
            buf[0],   
            buf[4]+0x11,   
            buf[1],   
            buf[6]+0x17,   
            buf[3],   
            buf[2]+0x15,   
            buf[7]);   
    this->m_output = regcode;   
    //--------------------------------------------    
    this->m_edit2.SetWindowText(this->m_output);   
    //以下代码是将注册码写入剪切板中。    
    if(OpenClipboard())   
    {   
    HGLOBAL clipbuffer;   
    char * buffer;   
    EmptyClipboard();   
    clipbuffer = GlobalAlloc(GMEM_DDESHARE, this->m_output.GetLength()+1);   
    buffer = (char*)GlobalLock(clipbuffer);   
    strcpy(buffer, LPCSTR(this->m_output));   
    GlobalUnlock(clipbuffer);   
    SetClipboardData(CF_TEXT,clipbuffer);   
    CloseClipboard();   
    }   
    //*/   
}   

unsigned long CKeyGenDlg::hextoi(char *str) //将十六进制字串转为 LONG 型。   
{   
    char n;   
    int i,len;   
    unsigned long sum,sum1;   
    len  = strlen(str);   
    sum =0;   
    sum1 =1;   
    for(i=0;i<len;i++)   
    {   
        n = str[len-i-1];   
        if( n >='0' && n<='9' )   
        {   
            n -= '0'; //转为数字   
        }   
        else   
        {   
            if(n >='a' && n <='f')   
            {   
                n-= 87; //转为数字   
            }   
            else   
            {   
                if(n>='A' && n <='F')   
                {   
                    n-=55; //转为数字   
                }   
                else   
                { //如果都不是，则返回   
                    break;   
                }   
            }   
        }   
        sum += n * sum1;   
        sum1*=16;   
    }   
    return sum;   
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
