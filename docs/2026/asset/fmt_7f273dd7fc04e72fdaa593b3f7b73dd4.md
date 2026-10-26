
```text
004CF4BC   \.  C3                retn
```

程序中将用户名和已知的`KEY`进行了好几次异或运算,我觉得如果写注释还不如我将其用高级语言写出来了. 用C语言实现如下,为了方便我取变量名称基本与汇编程序中一样:

```
int length=m_name.GetLength();

unsigned char *name=new unsigned char[length];

unsigned char nameResult[4]={0,0,0,0};

unsigned long nameRR;

strcpy((char*)name,m_name.GetBuffer(0));

int ebxTemp=0;

int temp1;

int ecxTemp;

int eaxTemp=0;

int esiTemp=0;

unsigned char key[20]={0xAA,0x89,0xC4,0xFE,0x46,0x78,0xF0,0xD0,
                                            0x03,0xE7,0xF7,0xFD,0xF4,0xE7,0xB9,0xB5,
                                                        0x1B,0xC9,0x50,0x73};

do
{
        temp1=name[esiTemp];
        name[esiTemp]=name[esiTemp]^key[eaxTemp];
        key[eaxTemp]=temp1;
        eaxTemp++;
        esiTemp++;
        if(eaxTemp==5)
                eaxTemp=0;
}while(esiTemp<length);

ecxTemp=0;
int ediTemp=0;
do
{
        esiTemp=length-ecxTemp;
        esiTemp--;
        temp1=name[esiTemp];
        name[esiTemp]=name[esiTemp]^key[ediTemp+5];
        key[ediTemp+5]=temp1;
        ediTemp++;
        ecxTemp++;
        if(ediTemp==5)
                ediTemp=0;
}while(ecxTemp<length);

ediTemp=0;
esiTemp=0;
do
{
        temp1=name[ediTemp];
        name[ediTemp]=name[ediTemp]^key[esiTemp+0xA];
        key[esiTemp+0xA]=temp1;
        ediTemp++;
        esiTemp++;
        if(esiTemp==5)
                esiTemp=0;
}while(ediTemp<length);

ecxTemp=0;
ediTemp=0;
do
{
        esiTemp=length-ecxTemp;
        esiTemp--;
        temp1=name[esiTemp];
        name[esiTemp]=name[esiTemp]^key[ediTemp+0xF];
        key[ediTemp+0xF]=temp1;
        ediTemp++;
        ecxTemp++;
        if(ediTemp==5)
                ediTemp=0;
}while(ecxTemp<length);

eaxTemp=0;
do
{
        ecxTemp=eaxTemp;
        ecxTemp&=3;
        nameResult[ecxTemp]+=name[eaxTemp];
        eaxTemp++;
}while(eaxTemp<length);
```

而注册码运算过程，如果我们死搬程序用C语言实现可以简单写成下面：

```
unsigned char key2[34*8-10]={

        0x20,0x00,0x20,0x00,0x20,0x00,

        0x20,0x00,0x20,0x00,0x20,0x00,0x20,0x00,

        0x20,0x00,0x20,0x00,0x28,0x00,0x28,0x00,

        0x28,0x00,0x28,0x00,0x28,0x00,0x20,0x00,

        0x20,0x00,0x20,0x00,0x20,0x00,0x20,0x00,

        0x20,0x00,0x20,0x00,0x20,0x00,0x20,0x00,

        0x20,0x00,0x20,0x00,0x20,0x00,0x20,0x00,

        0x20,0x00,0x20,0x00,0x20,0x00,0x20,0x00,

        0x20,0x00,0x48,0x00,0x10,0x00,0x10,0x00,

        0x10,0x00,0x10,0x00,0x10,0x00,0x10,0x00,

        0x10,0x00,0x10,0x00,0x10,0x00,0x10,0x00,

        0x10,0x00,0x10,0x00,0x10,0x00,0x10,0x00,

        0x10,0x00,0x84,0x00,0x84,0x00,0x84,0x00,

        0x84,0x00,0x84,0x00,0x84,0x00,0x84,0x00,

        0x84,0x00,0x84,0x00,0x84,0x00,0x10,0x00,

        0x10,0x00,0x10,0x00,0x10,0x00,0x10,0x00,

        0x10,0x00,0x10,0x00,0x81,0x00,0x81,0x00,

        0x81,0x00,0x81,0x00,0x81,0x00,0x81,0x00,

        0x01,0x00,0x01,0x00,0x01,0x00,0x01,0x00,

        0x01,0x00,0x01,0x00,0x01,0x00,0x01,0x00,

        0x01,0x00,0x01,0x00,0x01,0x00,0x01,0x00,

        0x01,0x00,0x01,0x00,0x01,0x00,0x01,0x00,

        0x01,0x00,0x01,0x00,0x01,0x00,0x01,0x00,

        0x10,0x00,0x10,0x00,0x10,0x00,0x10,0x00,

        0x10,0x00,0x10,0x00,0x82,0x00,0x82,0x00,

        0x82,0x00,0x82,0x00,0x82,0x00,0x82,0x00,

        0x02,0x00,0x02,0x00,0x02,0x00,0x02,0x00,

        0x02,0x00,0x02,0x00,0x02,0x00,0x02,0x00,

        0x02,0x00,0x02,0x00,0x02,0x00,0x02,0x00,

        0x02,0x00,0x02,0x00,0x02,0x00,0x02,0x00,

        0x02,0x00,0x02,0x00,0x02,0x00,0x02,0x00,

        0x10,0x00,0x10,0x00,0x10,0x00,0x10,0x00,

        0x20,0x00,0x00,0x00,0x00,0x00,0x00,0x00

        };

unsigned char code[10]={'1','0','3','9','1','8','4','0','1','2'};

ediTemp=0;
do
{
        eaxTemp=code[ediTemp];
        eaxTemp=key2[eaxTemp*2];
        eaxTemp&=8;
        if(eaxTemp!=0)
                ediTemp++;
}while(eaxTemp!=0);

length=10;
ebxTemp=0;
for(ediTemp=0;ediTemp<length;ediTemp++)
{
        eaxTemp=key2[code[ediTemp]*2];
        eaxTemp&=4;
        if(eaxTemp==0)
                break;
        eaxTemp=ebxTemp+ebxTemp*4;
        ebxTemp=code[ediTemp]+eaxTemp*2-0x30;
}

eaxTemp=ebxTemp;
```

是不是被吓着了阿，我刚开始都被吓着了，但是我见在纸上写了一下`eaxTemp`的计算步骤如下：

```
code[0]

code[0]*5*2+code[1]

(code[0]*5*2+code[1])*5*2+code[2]

......
```

发现没有，哈哈，主要是我算法不熟悉，害得我干了半天，其实运算过程就是注册码的十进制大小了。

这样注册机就很好写了，只要将`TEMP1`计算出来，然后转化为10进制大小就是注册码了。

注册机程序如下：

```
void CcrackDlg::OnGenerate()
{
        UpdateData(TRUE);

        int length=m_name.GetLength();

        unsigned char *name=new unsigned char[length];

        unsigned char nameResult[4]={0,0,0,0};

        unsigned long nameRR;

        strcpy((char*)name,m_name.GetBuffer(0));

        int ebxTemp=0;

        int temp1;

        int ecxTemp;

        int eaxTemp=0;

        int esiTemp=0;

        unsigned char key[20]={0xAA,0x89,0xC4,0xFE,0x46,0x78,0xF0,0xD0,
                                            0x03,0xE7,0xF7,0xFD,0xF4,0xE7,0xB9,0xB5,
                                                        0x1B,0xC9,0x50,0x73};

        do
        {
                temp1=name[esiTemp];
                name[esiTemp]=name[esiTemp]^key[eaxTemp];
                key[eaxTemp]=temp1;
                eaxTemp++;
                esiTemp++;
                if(eaxTemp==5)
                        eaxTemp=0;
        }while(esiTemp<length);

        ecxTemp=0;
        int ediTemp=0;
        do
        {
                esiTemp=length-ecxTemp;
                esiTemp--;
                temp1=name[esiTemp];
                name[esiTemp]=name[esiTemp]^key[ediTemp+5];
                key[ediTemp+5]=temp1;
                ediTemp++;
                ecxTemp++;
                if(ediTemp==5)
                        ediTemp=0;
        }while(ecxTemp<length);

        ediTemp=0;
        esiTemp=0;
        do
        {
                temp1=name[ediTemp];
                name[ediTemp]=name[ediTemp]^key[esiTemp+0xA];
                key[esiTemp+0xA]=temp1;
                ediTemp++;
                esiTemp++;
                if(esiTemp==5)
                        esiTemp=0;
        }while(ediTemp<length);

        ecxTemp=0;
        ediTemp=0;
        do
        {
                esiTemp=length-ecxTemp;
                esiTemp--;
                temp1=name[esiTemp];
                name[esiTemp]=name[esiTemp]^key[ediTemp+0xF];
                key[ediTemp+0xF]=temp1;
                ediTemp++;
                ecxTemp++;
                if(ediTemp==5)
                        ediTemp=0;
        }while(ecxTemp<length);

        eaxTemp=0;
        do
        {
                ecxTemp=eaxTemp;
                ecxTemp&=3;
                nameResult[ecxTemp]+=name[eaxTemp];
                eaxTemp++;
        }while(eaxTemp<length);

        nameRR=nameResult[0]+(nameResult[1]<<8)+(nameResult[2]<<16)+(nameResult[3]<<24);

        char SN[100];
        sprintf(SN,"%d",nameRR);

        m_code=SN;
        UpdateData(FALSE);
}
```

得到一组有用的注册码:

```
laowang
1039184012
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
