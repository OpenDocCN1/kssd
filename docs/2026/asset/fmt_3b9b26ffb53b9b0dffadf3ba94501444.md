
# [原创]Mac OS X下一软件Voila注册算法分析

Voila keygen

工具: IDA v5.2, `gdb`

IDA可以用Wine在mac os x上跑起来，或者有两台电脑，一台windows，跑IDA，另一台Mac。

在windows平台中打开ida，加载Voila(Mac OS X binary)

找到的关键地方为：

`000A9B8F` `__VLRegister_registerContinue__`  
`000AA67A` `__VLRegister_validateKeyWithInfo_Fortype__`

![](https://bbs.kanxue.com/upload/attach/201001/437_eiikz60em4hhuej.png)

![](https://bbs.kanxue.com/upload/attach/201001/437_io0f475u544d7jk.png)

下面再跟进下面的函数:

在`000AA681`执行后,查看`ecx`:

![](https://bbs.kanxue.com/upload/attach/201001/437_6ekm3iww9ml7vz4.png)

下面,判断使用什么key

![](https://bbs.kanxue.com/upload/attach/201001/437_c64xpdx3dzz3aol.png)

参数`nType_C`是从下面传进来的.

```
__text:000A9C0B C7 44 24 0C 00 00 00 00                 mov     [esp+0B8h+var_AC], 0
```

看看key:

![](https://bbs.kanxue.com/upload/attach/201001/437_25eexl8shbq7lmv.png)

继续:

![](https://bbs.kanxue.com/upload/attach/201001/437_0u6xxvgz6i4wa6d.png)

注意到上图最上面两行, 两个常数, 为方便起见,就当作是一个`int64`整数.

跟进`_GetSystemInfo`

实例化一个`SystemState`类

![](https://bbs.kanxue.com/upload/attach/201001/437_67nduxah071tsk8.png)

进入构造函数 `SystemState::SystemState(unsigned char const*, char const*, long long, EValidationType)`:

![](https://bbs.kanxue.com/upload/attach/201001/437_9m17gdv5jvsneqk.png)

如果没有传入上面提到的`int64`常数,则调用`GetSeed`函数来生成一个`int64`.

继续:

![](https://bbs.kanxue.com/upload/attach/201001/437_7yhr6symiz0r6ej.png)

`SystemState::GetFirstState`用来校验注册码的长度(0x20)和注册码的字符(数字和字母).

再跟进`SystemState::GetState`

![](https://bbs.kanxue.com/upload/attach/201001/437_wq7lnokvxffgt12.png)

首先调用`SystemState::GetDecryptText`,然后对解出来的信息进行校验.

跟进`SystemState::GetDecryptText`:

![](https://bbs.kanxue.com/upload/attach/201001/437_8p4ebl896658n1q.png)

取出序列号的最后一个字符,然后找到这个字符在`NIWHSALRMPJ5EKTUYX3012QV67ZGF984`中的位置（第一个字符位置为0），设为`LastCharIndex`。

下面再从注册码中的`LastCharIndex`开始，取5个字符，设为`Part_5`,将剩余的26个字符，合并为`Part_26`

![](https://bbs.kanxue.com/upload/attach/201001/437_c0477c251znkzgx.png)

再来看看`SystemState::GenerateEncryptedPart`

这个函数先将注册码分割为两部分，再将注册码用其在两个字符串中的位置所代替。

偶数字符从 `SystemState::m_acSet2 db 'BCDORSJAMP5LEKTUYX3012QV67ZGF984',0` 中取

奇数字符从 `SystemState::m_acSet1 db 'NIWHSALRMPJ5EKTUYX3012QV67ZGF984',0` 中取

注册码的第一个字符为0，偶数字符

再往下看

![](https://bbs.kanxue.com/upload/attach/201001/437_10p5c1vvwcl57ga.png)

函数`SystemState::ConvertNToHexa`，将`part_5`转换为密钥（aes128），当成一个`int64`来看。

来看看函数`SystemState::ConvertNToHexa`的关键代码：

![](https://bbs.kanxue.com/upload/attach/201001/437_mplb00k05jyncfi.png)

将`part_5`紧凑起来。因为两个字符级是32，`index`最大31，占5个bit，因此将这些5bit合并，使得每个字节的每个bit都利用。

同时将这个`int64`变成一个字符串。

```
jmp     SystemState::GenerateNinHexa(uchar *,ulong long) ; 将int64变成字符串
```

再看看`part_26`的转换：

![](https://bbs.kanxue.com/upload/attach/201001/437_009talzdghgzwkb.png)

将26个`index`，也紧凑起来。

继续

调用`CAESDecryptor::DecryptText`，将`part_26`用`part_5`作密钥来解密

![](https://bbs.kanxue.com/upload/attach/201001/437_9j4k90kp4j8zpxg.png)

解密之后，回到函数`SystemState::GetState`

校验解出来的16个字节，以及`part_5`

解密出来的16个字节必须是 `ds:__ZL13KeyForGeneric`

![](https://bbs.kanxue.com/upload/attach/201001/437_afip9l0a1pcf5x5.png)

`part_5`得到的`int64`，必须比上面提到的`int64` `0x00000000 001e8480` 要小

Over。

剩下的活，就是写注册机。

该公司的另一款产品web2delight，算法一样。

关键代码为：

```
- (NSString*) doKeygen:(int)indexofproduct atusername:(NSString*)username atEmail:(NSString*)email

{
	NSString *szkey;

	unsigned char indata_web2delight[17] = {0x75,0x18,0x10,0x25,0x86,0x37,0x8B,0xDE,
											0x19,0xA7,0x21,0x90,0xD3,0x12,0xB4,0x86,0x00};
	unsigned char indata[17] = {0x86,0x97,0xB3,0xF1,0x6C,0xD2,0x22,0xE8,
								0x9B,0xCE,0xA7,0xC6,0xAE,0x65,0xB1,0xBB,0x00};
	unsigned char outdata[17] = {0};
	unsigned char aeskeydata[17] = {0};
	
	const char* set1 = "NIWHSALRMPJ5EKTUYX3012QV67ZGF984";
	const char* set2 = "BCDORSJAMP5LEKTUYX3012QV67ZGF984";
	
	const unsigned long long ullKeyLowLimit = 0x0;
	const unsigned long long ullKeyHighLimit = 0x001E8480;//1,1D,1,4,0
	const unsigned long long ullKeyLowLimitWeb2 = 0x0;
	const unsigned long long ullKeyHighLimitWeb2 = 0x00004E20;//
	unsigned long long ullKeyRandom = 0;
	unsigned char key5[5] = {0};
	unsigned char otherkey26[26] = {0};
	unsigned char c = 0;
	char szckey[33] = {0};
	int i = 0;
	unsigned char halfbyte = 0;
	int j = 0;
	
	srand(time(NULL));
	
	if(indexofproduct == 0)
	{
		ullKeyRandom = rand() % (ullKeyHighLimit - ullKeyLowLimit) + ullKeyLowLimit;
	}
	else if(indexofproduct == 1)
	{
		ullKeyRandom = rand() % (ullKeyHighLimitWeb2 - ullKeyLowLimitWeb2) + ullKeyLowLimitWeb2;
	}
	else
	{
		return nil;
	}
		
	for(i = 0; i < 5;i++)
	{
		c = ullKeyRandom >> (i) * 5;
		key5[i] = c & 0x1F;
	}
	
	c = rand() % (31 - 5); //the last char of registration code
	
	for(i = 0; i < 16;i++)
	{
		halfbyte = Gethalfbytefromint64(ullKeyRandom,i);
		aeskeydata[15 - i] = halfbyte2hexchar(halfbyte);
	}
	
	if(indexofproduct == 0)
	{
		aes_encrypt(indata,
				outdata,
				aeskeydata);
	}
	else if(indexofproduct == 1)
	{
		aes_encrypt(indata_web2delight,
					outdata,
					aeskeydata);
	}
	else
	{
		return nil;
	}
	
	//5 * 8 = 40, 40 / 5 = 8
	otherkey26[0] = outdata[0] & 0x1F;
	otherkey26[1] = ((outdata[1] & 0x3) << 3) | (outdata[0] >> 5);
	otherkey26[2] = (outdata[1] >> 0x2) & 0x1F;
	otherkey26[3] = ((outdata[2] & 0xF) << 1) | (outdata[1] >> 7);
	otherkey26[4] = ((outdata[3] & 0x1) << 4) | (outdata[2] >> 4);
	otherkey26[5] = (outdata[3] >> 1) & 0x1F;
	otherkey26[6] = ((outdata[4] & 0x7) << 2) | (outdata[3] >> 6);
	otherkey26[7] = (outdata[4] >> 3) & 0x1F;
	
	otherkey26[8] = (outdata[5]) & 0x1F;
	otherkey26[9] = ((outdata[6] & 0x3) << 3) | (outdata[5] >> 5);
	otherkey26[10] = (outdata[6] >> 2) & 0x1F;
	otherkey26[11] = ((outdata[7] & 0xF) << 1) | (outdata[6] >> 7);
	otherkey26[12] = ((outdata[8] & 0x1) << 4) | (outdata[7] >> 4);
	otherkey26[13] = (outdata[8] >> 1) & 0x1F;
	otherkey26[14] = ((outdata[9] & 0x7) << 2) | (outdata[8] >> 6);
	otherkey26[15] = (outdata[9] >> 3) & 0x1F;
	
	otherkey26[16] = (outdata[10]) & 0x1F;
	otherkey26[17] = ((outdata[11] & 0x3) << 3) | (outdata[10] >> 5);
	otherkey26[18] = (outdata[11] >> 2) & 0x1F;
	otherkey26[19] = ((outdata[12] & 0xF) << 1) | (outdata[11] >> 7);
	otherkey26[20] = ((outdata[13] & 0x1) << 4) | (outdata[12] >> 4);
	otherkey26[21] = (outdata[13] >> 1) & 0x1F;
	otherkey26[22] = ((outdata[14] & 0x7) << 2) | (outdata[13] >> 6);
	otherkey26[23] = (outdata[14] >> 3) & 0x1F;
	
	otherkey26[24] = (outdata[15]) & 0x1F;
	otherkey26[25] = ((outdata[16] & 0x3) << 3) | (outdata[15] >> 5);	
	
	for(i = 0; i < c;i++)
	{
		szckey[i] = otherkey26[i];
		
		j = szckey[i];
		if((i % 2) == 0)
		{
			szckey[i] = set2[j];
		}
		else
		{
			szckey[i] = set1[j];
		}
	}
	
	for(i = c; i < c + 5;i++)
	{
		szckey[i] = key5[i - c];
		
		j = szckey[i];
		if(((i - c) % 2) == 0)
		{
			szckey[i] = set2[j];
		}
		else
		{
			szckey[i] = set1[j];
		}
	}
	
	for(i = c + 5; i < 31;i++)
	{
		szckey[i] = otherkey26[i - 5];
		
		j = szckey[i];
		if(((i - 5) % 2) == 0)
		{
			szckey[i] = set2[j];
		}
		else
		{
			szckey[i] = set1[j];
		}
	}
	
	szckey[31] = set1[c];
	
	NSLog(@" key is %s",szckey);
	
	szkey = [[NSString stringWithCString:(const char*)szckey] retain];
	
	return szkey;
}
```

附：

ObjC使用`msgSend`来调用函数。

比如：调用`-_VLRegister_validateKeyWithInfo_Fortype__`的地方

![](https://bbs.kanxue.com/upload/attach/201001/437_0ply0evl9ufi1bh.png)

`Function_name_ptr`保存在`[esp+4]`中

![](https://bbs.kanxue.com/upload/attach/201001/437_zlpbw35asactbf0.png)

![](https://bbs.kanxue.com/upload/attach/201001/437_pulgrrlye6o549h.png)

查看交叉参考，`__inst_meth:0019417Co`，来到：

![](https://bbs.kanxue.com/upload/attach/201001/437_6evp8hsdfxm3ej1.png)

然后才看到函数地址

我写了个Python脚本，用IDAPython来跑，可以添加注释，显示出函数名，并添加交叉参考。速度比较慢。跑完Voila，要用18分钟。脚本运行后，鼠标最好就不动了。否则IDA会假死。

Python 脚本为：
