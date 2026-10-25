
```
int JudgePdfPasswordRC4_128()
{
		int j=0;
		int rc4_count=0;
		UINT4  _State[4];
		UCHAR S[256];
		UINT state[4];

		UCHAR local_converStateEnd[16];
		memcpy(local_converStateEnd,converStateEnd,16);

		j=strlen(pdf_szZu);

		memcpy(CheckArea1,pdf_szZu,j);
		memcpy(CheckArea1+j,ConData,32-j);

		memcpy(_State,g_State,16);//取代reset();
		md5_transform_assembly(_State,CheckArea1);
		md5_transform_assembly(_State,CheckArea1 + 0x40);


		for(int md5_count = 0;md5_count != 50;++md5_count)//50次MD5算法
		{
			memcpy(CheckArea2,_State,16);
			memcpy(_State,g_State,16);//取代reset();
			CMD5::MD5Transform(_State,CheckArea2);
		}

		RC4_assemly(S, (const UCHAR*)_State, local_converStateEnd);
		for (rc4_count=0;rc4_count!=18;rc4_count++)//18次RC4算法
		{

			state[0]=_State[0]^cnt_X[rc4_count];
			state[1]=_State[1]^cnt_X[rc4_count];
			state[2]=_State[2]^cnt_X[rc4_count];
			state[3]=_State[3]^cnt_X[rc4_count];
			RC4_assemly(S, (const UCHAR*)state, local_converStateEnd);
		}

		state[0]=_State[0]^cnt_X[rc4_count];
		state[1]=_State[1]^cnt_X[rc4_count];
		state[2]=_State[2]^cnt_X[rc4_count];
		state[3]=_State[3]^cnt_X[rc4_count];
return RC4AndResult(S,m_lin,local_converStateEnd,sizeof(local_converStateEnd),(char*)state,16);
//第19次RC4算法并作判断
}
```

# RC4-40算法分析

## Passware Kit 11 Demo分析

### 数据处理部分

`00A8D07D`  `|.  310F          xor dword ptr ds:[edi],ecx`

`00A8D07F`  `|.  8B50 04       mov edx,dword ptr ds:[eax+0x4]`

`00A8D082`  `|.  3157 04       xor dword ptr ds:[edi+0x4],edx`

`00A8D085`  `|.  8B48 08       mov ecx,dword ptr ds:[eax+0x8]`

`00A8D088`  `|.  314F 08       xor dword ptr ds:[edi+0x8],ecx`

`00A8D08B`  `|.  8B50 0C       mov edx,dword ptr ds:[eax+0xC]`

`00A8D08E`  `|.  3157 0C       xor dword ptr ds:[edi+0xC],edx`

`00A8D091`  `|.  8B48 10       mov ecx,dword ptr ds:[eax+0x10]`

`00A8D094`  `|.  314F 10       xor dword ptr ds:[edi+0x10],ecx`

`00A8D097`  `|.  8B50 14       mov edx,dword ptr ds:[eax+0x14]`

`00A8D09A`  `|.  3157 14       xor dword ptr ds:[edi+0x14],edx`

`00A8D09D`  `|.  8B48 18       mov ecx,dword ptr ds:[eax+0x18]`

`00A8D0A0`  `|.  314F 18       xor dword ptr ds:[edi+0x18],ecx`

`00A8D0A3`  `|.  8B50 1C       mov edx,dword ptr ds:[eax+0x1C]`

`00A8D0A6`  `|.  3157 1C       xor dword ptr ds:[edi+0x1C],edx`

`00A8D0A9`  `|.  8B4E 3C       mov ecx,dword ptr ds:[esi+0x3C]`

`00A91503`  `|.  E8 F8CBFFFF   call Passware.00A8E100                   ;  MD5算法`

32字节填充数据

`5E4EBF28`

`418A754E`

`564E0064        与U条目异或`

`0801FAFF    ----------------->生成最终数据`

`B6002E2E`

`803E68D0`

`FEA90C2F`

`7A695364`

`0AFFE084  BF289038`

`0AFFE088  754E5E4E`

`0AFFE08C  0064418A  Passware.0064418A`

`0AFFE090  FAFF564E`

`0AFFE094  2E2E0801`

`0AFFE098  68D0B600`

`0AFFE09C  0C2F803E`

`0AFFE0A0  5364FEA9`

`0AFFE0A4  2494D79E`

`0AFFE0A8  888F3421`

`0AFFE0AC  67719CB7`

`0AFFE0B0  9E49855C`

`0AFFE0B4  4B36B2F9`

`0AFFE0B8  9246127F`

`0AFFE0BC  CDAE7226`

`0AFFE0C0  5A206F65                经过两次MD5处理，再经过RC4处理后与最终数据比较`

`0B017674  FFFFFBFC     -------------------------------------------------->      判别结果`

`0B017678  4E09775B`

`0B01767C  4CE72BD3`

`0B017680  4A9C5DA9`

`0B017684  E0B289D5`

`0B017688  00000080`

`0B01768C  00000000`

`0B017690  00000000`

`0B017694  00000000`

`0B017698  00000000`

`0B01769C  00000000`

`0B0176A0  00000000`

`0B0176A4  00000000`

`0B0176A8  00000000`

`0B0176AC  000002A0`

`0B0176B0  00000000`

### 逆向的代码(伪代码)

```
int JudgePassword_RC4_40()
{
	j=strlen(pdf_szZu);//pdf_szZu为传入的密码字符串

	memcpy(CheckArea1,pdf_szZu,j);

	memcpy(CheckArea1 + j,ConData,32-j);

	memcpy(_State,g_State,16);//取代reset();

	CMD5::MD5Transform(_State,CheckArea1);
	CMD5::MD5Transform(_State,CheckArea1 + 0x40);

	return RC4_40_AndResult(S,m_lin,converState_RC4_40,32,(char*)_State,5);

}
```

# AES-128算法分析

## Passware Kit 12 Demo分析

`00B1D8C0  /$  55            push ebp           //Passware Kit 12 Demo 算法入口`

`00B2537B  |.  8B45 08       mov eax,[arg.1]`

`00B2537E  |.  8B5D 0C       mov ebx,[arg.2]`

`00B25381  |.  8BF1          mov esi,ecx`

`00B25383  |.  8B4E 04       mov ecx,dword ptr ds:[esi+0x4]`

`00B25386  |.  8B56 08       mov edx,dword ptr ds:[esi+0x8]`

`00B25389  |.  8945 A4       mov [local.23],eax`

`00B2538C  |.  8D040B        lea eax,dword ptr ds:[ebx+ecx]`

`00B2538F  |.  894D A0       mov [local.24],ecx`

`00B25392  |.  8946 04       mov dword ptr ds:[esi+0x4],eax    //写入1字节数据(长度)`

`00B24E55  |.  8B46 04       mov eax,dword ptr ds:[esi+0x4]`

`00B24E58  |.  03C0          add eax,eax`

`00B24E5A  |.  03C0          add eax,eax`

`00B24E5C  |.  03C0          add eax,eax`

`00B24E5E  |.  85FF          test edi,edi`

`00B24E60  |.  74 02         je short Passware.00B24E64`

`00B24E62  |.  0FC8          bswap eax`

`00B24E64  |>  8BC8          mov ecx,eax`

`00B24E66  |.  8B45 F8       mov eax,[local.2]`

`00B24E69  |.  C1E8 02       shr eax,0x2`

`00B24E6C  |.  8D1438        lea edx,dword ptr ds:[eax+edi]`

`00B24E6F  |.  894C93 F8     mov dword ptr ds:[ebx+edx*4-0x8],ecx  //填充至最后`

### 填充口令后的数据

`0A511350  BF4BF020   餕`

`0A511354  E748B436  6碒`

`0A511358  000080BD  絸..      填充方式：password + U条目最后8个字节 + 80+0 + 长度处理后的数据`

`0A51135C  00000000  ....`

`0A511360  00000000  ....`

`0A511364  00000000  ....`

`0A511368  00000000  ....`

`0A51136C  00000000  ....`

`0A511370  00000000  ....`

`0A511374  00000000  ....`

`0A511378  00000000  ....`

`0A51137C  00000000  ....`

`0A511380  00000000  ....`

`0A511384  00000000  ....`

`0A511388  00000000  ....`

`0A51138C  48000000  ...H`

### SHA256初始化(是SHA256？)

`0A519D50  6A09E667  g?j`

`0A519D54  BB67AE85  叜g`

`0A519D58  3C6EF372  r髇<`

`0A519D5C  A54FF53A  :鮋`

`0A519D60  510E527F  R\_Q`

`0A519D64  9B05688C  宧\_`

`0A519D68  1F83D9AB  ?`

`0A519D6C  5BE0CD19  \_袜[`

### 循环处理

`00977907  |.  E8 84160000   |call Passware.00978F90  //传入字符`

`00978FF2  |.  FFD0          call eax                                 ;  填充字符1`

`  00B2541F  |.  E8 FCD5F2FF   call Passware.00A52A20                   ;  填充字符2`

`0097901B  |.  FFD0          call eax                    ;  8888888888888888`

`00634D7B  |.  FFD2        call edx                                ;GOGO1`

`005F4A68  |.  FFD2          call edx                        ;  GOGO2`

`00B24EA1  |.  FFD0          call eax                  ;  算法处理1`

`00B21344  |.  E8 77C5FFFF   call Passware.00B1D8C0     ;  算法`

`00B24EB5  |.  E8 A6FDFFFF   call Passware.00B24C60 ;  GOGO3   bswap`

`00B24EC4  |.  FFD0          call eax                  ;  数据初始化1`

`00633857   . E8 44A04E00   call Passware.00B1D8A0    ;  数据初始化`

### 判断比较

`SHA256`算法处理后的数据与`U`条目前32个字节比较 `PDF`文件的`U`条目

`0A51E369  9CBD1FAA  ?綔                                0A507210  967D44DD  軩}`

`0A51E36D  CFA79CB3  硿                                0A507214  06E32AD0  ??`

`0A51E371  DB0322CB  ?\_            比较                0A507218  011727E8  ?\_\_   Passware.011727E8`

`0A51E375  52081684  ?R    ------------->                0A50721C  67D6B896  柛謌`

`0A51E379  2C4D47AF  疓M,                                0A507220  8FE78FAC  瑥鐝`

`0A51E37D  2AC42E5D  ].?                                0A507224  FDA33377  w3｝`

`0A51E381  F53D76BF  縱=                                0A507228  46F680B4  磤鯢`

`0A51E385  5C809513  \_晙\                                0A50722C  DC388ADF  邐8`

逆向的代码()
