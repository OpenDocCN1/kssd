
```c
    put_dword_value(temp,char_data+0x08);

    temp=0x00000000;
    dword_count=3;
        XOR_NO=(0x260-0x14)/4;
    pkey=&key_file[edx*4+0x04+0x260];
        data_encryptor(char_data,dword_count,XOR_Factor,XOR_NO,pkey,temp);


/* -----set value of addr ebp+260, ebp+264 and ebp+268 end------*/

/* -----set value of addr ebp+160 : Username ------*/
    printf("Input your name : ");
    scanf("%s",Username);

    temp=0x00000000;
    for(i=0;i<(unsigned char)strlen(Username);i++) char_data[i]=*(Username+i);

    dword_count=strlen(Username)/4+1;
    for(i=strlen(Username);i<(strlen(Username)strlen(Username)%4);i++)
char_data[i]=0x00;
        XOR_NO=(0x160-0x14)/4;
        pkey=&key_file[edx*4+0x04+0x160];
        data_encryptor(char_data,dword_count,XOR_Factor,XOR_NO,pkey,temp);


/* -----set value of addr ebp+160 : Username end------*/

/* -----set value of addr ebp+26C/ebp+28C : S_N head/tail ------*/
    temp=get_dword_value(&key_file[edx*4+0x04+0x268]);


    for(i=0;i<(unsigned char)strlen(SN_Header);i++) char_data[i]=*(SN_Header+i);

    dword_count=strlen(SN_Header)/4+1;
    for(i=strlen(SN_Header);i<(strlen(SN_Header)strlen(SN_Header)%4);i++)
char_data[i]=0x00;
        XOR_NO=(0x26C-0x14)/4;
        pkey=&key_file[edx*4+0x04+0x26C];
        data_encryptor(char_data,dword_count,XOR_Factor,XOR_NO,pkey,temp);


    temp=0x00000000;
    for(i=0;i<(unsigned char)strlen(SN_Tail);i++) char_data[i]=*(SN_Tail+i);

    dword_count=strlen(SN_Tail)/4+1;
    for(i=strlen(SN_Tail);i<(strlen(SN_Tail)strlen(SN_Tail)%4);i++)
char_data[i]=0x00;
        XOR_NO=(0x28C-0x14)/4;
        pkey=&key_file[edx*4+0x04+0x28C];
        data_encryptor(char_data,dword_count,XOR_Factor,XOR_NO,pkey,temp);


/* -----set value of addr ebp+26C/ebp+28C : S_N head/tail end------*/

/* -----set value of addr ebp+0x00, ebp+0x04, ebp+0x08, 0x0C ------ */

/* each value can not equal to 0x00000000 */

    pkey=&key_file[edx*4+0x04];

    temp=Version;
        put_dword_value(temp,pkey);
    temp=0x00000001;
        put_dword_value(temp,pkey+0x04); /* let [ebp+0x04]=0x00000001
*/
    temp=0x00000001;
        put_dword_value(temp,pkey+0x08); /* let [ebp+0x08]=0x00000001
*/
    temp=0x00000001;
        put_dword_value(temp,pkey+0x0C); /* let [ebp+0x0C]=0x00000001
*/

/* -----set value of Next_XOR_Factor1 ------*/

        pkey=&key_file[edx*0x04+0x04+0x02AC];

        /*  Next_XOR_Factor1=[pkey+0x00]=([pkey+0x04]^addr_004256E8)+([pkey+0x08]^addr_004256E4)+([pkey+0x0C]^addr_004256E0)
*/
        /*  here let us make Next_XOR_Factor1=XOR_Factor
*/

    temp=addr_004256E8;
    put_dword_value(temp,pkey+0x00); /* first */
    temp=get_dword_value(key_file+0x10);
    put_dword_value(temp,pkey+0x04); /* second */
    temp=addr_004256E0;
    put_dword_value(temp,pkey+0x08); /* third */

/* -----set value of Next_XOR_Factor1 end------*/

/* -----set value of Next_XOR_Factor2 ------*/

    pkey=&key_file[edx*0x04+0x04+0x02AC+0x02B8];

        /*  Next_XOR_Factor2=[pkey+0x00]=([pkey+0x04]^addr_004256E8)^([pkey+0x08]^addr_004256E4)+([pkey+0x0C]^addr_004256E0)
*/
        /*  here let us make Next_XOR_Factor2=XOR_Factor
*/

    temp=addr_004256E8;
    put_dword_value(temp,pkey+0x00); /* first */
    temp=get_dword_value(key_file+0x10);

    put_dword_value(temp,pkey+0x04); /* second */
    temp=addr_004256E0;
    put_dword_value(temp,pkey+0x08); /* third */

/* -----set value of Next_XOR_Factor2 end------*/

    p1=&key_file[edx*4+0x04+0x14];
    p2=&key_file[edx*4+0x04+0x14+0x02B8];
    for(i=0;i<=0xA5;i++) put_dword_value(get_dword_value(p1+i*4),p2+i*4);


    p1=&key_file[edx*4+0x04+0x14];
    p2=&key_file[edx*4+0x04+0x14+0x02B8+0x02B8];
    for(i=0;i<=0xA5;i++) put_dword_value(get_dword_value(p1+i*4),p2+i*4);


    create_key_file(key_file);
        create_dll_file(key_file+0x20+0x14,XOR_Factor);


        getch();
}

unsigned long get_dword_value(addr)
unsigned char * addr;
{
    unsigned long number=0;

    number=*(addr+3);
    number=number<<0x08;
    number=number|*(addr+2);
    number=number<<0x08;
    number=number|*(addr+1);
    number=number<<0x08;
    number=number|*addr;
    return number;
}

void put_dword_value(number,addr)
unsigned char * addr;
unsigned long number;
{
    unsigned char al=0;

    al=(unsigned char)(number&0x000000FF);
    *(addr+0)=al;
    al=(unsigned char)((number&0x0000FF00)>>0x08);
    *(addr+1)=al;
    al=(unsigned char)((number&0x00FF0000)>>0x10);
    *(addr+2)=al;
    al=(unsigned char)((number&0xFF000000)>>0x18);
    *(addr+3)=al;
}

void data_encryptor(char_data,dword_count,XOR_Factor,XOR_NO,pkey,forward_data)

unsigned char * char_data;
unsigned char dword_count;
unsigned long XOR_Factor;
unsigned long XOR_NO;
unsigned char * pkey;
unsigned long forward_data;
{
    void put_dword_value();
    unsigned long get_dword_value();

    unsigned long temp;
    unsigned char i;

    temp=forward_data;
    put_dword_value(temp,pkey-0x04);
    for(i=0;i<dword_count;i++)
    {
        temp=get_dword_value(pkey+0x04*i-0x04)^get_dword_value(char_data+0x04*i)^XOR_Factor^(XOR_NO+i);

        put_dword_value(temp,pkey+0x04*i);
    }
}

void create_key_file(key_file)
unsigned char key_file[];
{
    FILE *fp;
    int i=0;
    unsigned char * file_name=key_file_name;

    if((fp=fopen(file_name,"wb+"))==NULL)
    {
        printf("\nCan not create %s !\n",file_name);

        exit(0);
    }
    for(;;)
    {
        if(key_file[i]=='T'&&key_file[i+1]=='H'&&key_file[i+2]=='E'&&key_file[i+3]=='E'&&key_file[i+4]=='N'&&key_file[i+5]=='D')
break;
        if(fputc(key_file[i],fp)==EOF)
        {
            printf("\nWrite file
%s error !\n",file_name);
            fclose(fp);
            exit(0);
        }
        i++;
    }
    fclose(fp);
    printf("\n%s has been created successfully !\n",file_name);


}

void create_dll_file(data_addr,XOR_Factor)
unsigned char * data_addr;
unsigned long XOR_Factor;
{
    unsigned char data_000h_to_25Fh[0x0260]=
    {0x4D,0x5A,0x90,0x00,0x03,0x00,0x00,0x00,0x04,0x00,0x00,0x00,0xFF,0xFF,0x00,0x00,

    0xB8,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x40,0x00,0x00,0x00,0x00,0x00,0x00,0x00,

    0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,

    0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0xC0,0x00,0x00,0x00,

    0x0E,0x1F,0xBA,0x0E,0x00,0xB4,0x09,0xCD,0x21,0xB8,0x01,0x4C,0xCD,0x21,0x54,0x68,

    0x69,0x73,0x20,0x70,0x72,0x6F,0x67,0x72,0x61,0x6D,0x20,0x63,0x61,0x6E,0x6E,0x6F,

    0x74,0x20,0x62,0x65,0x20,0x72,0x75,0x6E,0x20,0x69,0x6E,0x20,0x44,0x4F,0x53,0x20,

    0x6D,0x6F,0x64,0x65,0x2E,0x0D,0x0D,0x0A,0x24,0x00,0x00,0x00,0x00,0x00,0x00,0x00,

    0x71,0xD4,0xF7,0xDB,0x35,0xB5,0x99,0x88,0x35,0xB5,0x99,0x88,0x35,0xB5,0x99,0x88,

    0xC9,0x95,0x8B,0x88,0x34,0xB5,0x99,0x88,0xBB,0xAA,0x8A,0x88,0x34,0xB5,0x99,0x88,

    0x52,0x69,0x63,0x68,0x35,0xB5,0x99,0x88,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,

    0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,

    0x50,0x45,0x00,0x00,0x4C,0x01,0x04,0x00,0x34,0x02,0xBC,0x3B,0x00,0x00,0x00,0x00,
```
