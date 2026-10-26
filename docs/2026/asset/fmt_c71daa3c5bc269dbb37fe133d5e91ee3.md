
```
void deresourcename(char *name,//name指针指向要解密的字符串

					BYTE len,//字符串长度

					DWORD a,//传入的是在文件读出的数据，此处是0x00013a14

					DWORD b,//传入的是在文件读出的a值后一个双字，此处为0x00016b12

					DWORD num//资源序号

					)

{

	int temp1,temp3;

	BYTE t2,t4;

	BYTE i,j;

	for(i=0;i<len;i++)

	{



		temp1=a;		//temp1=0x00013a14

		temp1*=b;		//temp1*=0x0016b12

		t2=i;

		t2*=0xfc;

		for(j=3;j>=0;j--)

		{

			temp3=temp1;

			temp3>>=j*8;

			t2-=(char)temp3;

		}

		t4=(char)a;		//t4=0x14;

		t4*=(char)b;	//t4*=0x12;

		t2-=t4;

		temp1=num;		

		for(j=3;j>=0;j--)

		{

			temp3=temp1;

			temp3>>=j*8;

			t2-=(char)temp3;

		}



		name[i]+=t2;

	}

}
```

貌似弄成C更难看了- -|||

不过既然叫逆向当然要将算法写成C才行，解密完资源名后，接下来的是一段复制资源名到新分配的内存空间然后连续读取1个字节加4个双字。接下来比较重要的一段代码是：

> 00401BB9 |. 8BB424 280100>|mov esi, dword ptr [esp+128] ; 取文件头地址
> 00401BC0 |. 0FB646 11 |movzx eax, byte ptr [esi+11]
> 00401BC4 |. 8B7D 00 |mov edi, dword ptr [ebp] ; 取脚本名地址
> 00401BC7 |. 8945 08 |mov dword ptr [ebp+8], eax
> 00401BCA |. 0FB64E 10 |movzx ecx, byte ptr [esi+10]
> 00401BCE |. 894D 0C |mov dword ptr [ebp+C], ecx
> 00401BD1 |. 8B56 28 |mov edx, dword ptr [esi+28] ; 将第一次从文件中读出的最后一个双字保存到edx中
> 00401BD4 |. 83C2 29 |add edx, 29 ; 29是第一次读出的大小，跟edx相加表示什么？看来这个就是文件头的大小了
> 00401BD7 |. 8BC7 |mov eax, edi
> 00401BD9 |. 8955 28 |mov dword ptr [ebp+28], edx
> 00401BDC |. E8 EF030000 |call 00401FD0 ; 计算资源名的hash值
> 00401BE1 |. 8BD8 |mov ebx, eax
> 00401BE3 |. 8BCF |mov ecx, edi
> 00401BE5 |. 895D 2C |mov dword ptr [ebp+2C], ebx ; 保存hash值
> 00401BE8 |. 8D41 01 |lea eax, dword ptr [ecx+1]

其中计算hash的那个call代码如下:

> 00401FD0 /$ 56 push esi
> 00401FD1 |. 8BF0 mov esi, eax ; eax=文件名地址
> 00401FD3 |. 8A06 mov al, byte ptr [esi] ; 取第一个字符
> 00401FD5 |. 84C0 test al, al ; 为空则直接返回
> 00401FD7 |. BA 21436587 mov edx, 87654321
> 00401FDC |. 74 10 je short 00401FEE
> 00401FDE |. 8BCE mov ecx, esi
> 00401FE0 |> 0FB6C0 /movzx eax, al
> 00401FE3 |. 83C1 01 |add ecx, 1
> 00401FE6 |. 2BD0 |sub edx, eax
> 00401FE8 |. 8A01 |mov al, byte ptr [ecx]
> 00401FEA |. 84C0 |test al, al
> 00401FEC |.^ 75 F2 \jnz short 00401FE0
> 00401FEE |> 8BC6 mov eax, esi
> 00401FF0 |. 8D70 01 lea esi, dword ptr [eax+1]
> 00401FF3 |> 8A08 /mov cl, byte ptr [eax]
> 00401FF5 |. 83C0 01 |add eax, 1
> 00401FF8 |. 84C9 |test cl, cl
> 00401FFA |.^ 75 F7 \jnz short 00401FF3
> 00401FFC |. 2BC6 sub eax, esi
> 00401FFE |. 0FAFC2 imul eax, edx
> 00402001 |. 5E pop esi
> 00402002 \. C3 retn

写成C代码如下:

```
DWORD namehash(char *name)

{

	DOWRD temp=0x87654321,i=0;

	while(name[i])

	{

		*(char*)&temp-=name[i];

		i++;

	}

	return i*temp;

}
```

接下来就是循环的最后一部分代码:

> 00401C24 |. 8B46 0C |mov eax, dword ptr [esi+C] ; [ESI+C]=0x00016B12
> 00401C27 |. 0FAF46 08 |imul eax, dword ptr [esi+8] ; [esi+8]=0x00013a14
> 00401C2B |. 03C3 |add eax, ebx ; ebx为文件名的hash值
> 00401C2D |. 0FAF45 24 |imul eax, dword ptr [ebp+24] ; [ebp+24]为上面最后一次在文件读取的双字
> 00401C31 |. 8945 14 |mov dword ptr [ebp+14], eax ; 保存到文件头结构偏移为0x14的地方中
> 00401C34 |. 8B4424 14 |mov eax, dword ptr [esp+14]
> 00401C38 |. 83C0 01 |add eax, 1
> 00401C3B |. 3B46 14 |cmp eax, dword ptr [esi+14] ; [esi+14]=要解密文件头个数
> 00401C3E |. 894424 14 |mov dword ptr [esp+14], eax
> 00401C42 |.^ 0F82 F8FCFFFF \jb 00401940 ; 往回跳

现在来看看内存窗口中的数据：

```
01364518  00 00 00 00 00 00 00 00 11 00 11 00 81 01 08 01  ........ . .?  
01364528  B0 45 36 01 02 00 00 00 01 00 00 00 01 00 00 00  癊6  ... ... ...
01364538  16 04 00 00 9F CA 3F 49 00 00 00 00 FF 23 00 00    ..熓?I....#..
01364548  16 04 00 00 2F 1A 00 00 5E 30 00 00 29 79 EE EE    ../ ..^0..)y铑
01364558  18 44 36 01 38 46 36 01 00 00 00 00 00 00 00 00   D6 8F6 ........
01364568  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364578  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364588  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364598  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
013645A8  11 00 11 00 90 01 08 01 62 6F 6F 74 5F 91 E6 88   . .?  boot_戞 
013645B8  EA 8F CD 2E 6E 73 73 00 00 00 00 00 00 00 00 00  陱?nss.........
013645C8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
013645D8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
013645E8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
013645F8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364608  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364618  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364628  00 00 00 00 00 00 00 00 11 00 11 00 E3 01 08 01  ........ . .?  
01364638  C0 46 36 01 02 00 00 00 01 00 00 00 01 00 00 00  繤6  ... ... ...
01364648  F1 03 00 00 82 D5 1A 91 00 00 00 00 15 28 00 00  ?..傉 ?... (..
01364658  F1 03 00 00 59 1A 00 00 5E 30 00 00 EA 7B EE EE  ?..Y ..^0..陒铑
01364668  28 45 36 01 48 47 36 01 00 00 00 00 00 00 00 00  (E6 HG6 ........
01364678  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364688  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01364698  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
013646A8  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
013646B8  11 00 11 00 F2 01 08 01 62 6F 6F 74 5F 91 E6 8E   . .?  boot_戞 
013646C8  B5 8F CD 2E 6E 73 73 00 00 00 00 00 00 00 00 00  祻?nss.........
```

好了，从这些数据中我们可以看出不少有效的信息，根据以上的数据我可以得出以下文件头结构：

```
struct ResourceHeader{

	DWORD nameoffset;			//文件名在内存中的地址

	DWORD resourcetype;			//不确定,在解密过程中读出的第一个字节数据

	DWORD unknown1;				//第一次读出文件的0x29个字节中的第0x10个

	DWORD unknown2;				//第一次读出文件的0x29个字节中的第0xf个

	DWORD unknownsize;			//与从文件中读出的数据有关，具体作用不详

	DWORD key1;				//最后一个变换得出的数值，用于后面解密

	DWORD unknown4;				//解密过程中读出的第一个双字的数据

	DWORD dataRVA;				//数据相对文件头的偏移，解密过程中读出的第二个双字的数据

	DWORD sizeoffile;			//资源文件的实际大小，解密过程中读出的第三个双字的数据

	DWORD unknown5;				//解密过程中读出的最后一个双字

	DWORD sizeofheader;			//所有文件头的大小

	DWORD hash;					//资源名的hash值

	ResourceHeader *Flink;		//指向前继节点

	ResourceHeader *Blink;		//指向后继节点

};
```

其实上面有些数据到目前都不能确定的，是我在后面的调试中得出来的。其实完整的分析到此为止了，后面的分析很乱，而且也没分析个所以然来。一是由于这个游戏的加密算法实在是太变态，二自然是由于我水平不够（在此要再次向写出这个游戏提取软件的痴汉公贼大大膜拜啊）。

后面的断点怎么下？我一开始打算是继续对`CreateFileA`下的，但这样太麻烦，因为程序要不停打开SoftICE跟FileMon等软件的驱动来进行反调试，而且老是喜欢打开资源文件什么都不做又将其关闭，于是我想到对内存中的文件头结构下内存访问断点。

于是断到这个地方：

> 00401840 |> /8B48 2C /mov ecx, dword ptr [eax+2C]
> 00401843 |. |3B4C24 14 |cmp ecx, dword ptr [esp+14] ; 比较hash值
> 00401847 |. |0F85 8E000000 |jnz 004018DB
> 0040184D |. |83FB 04 |cmp ebx, 4
> 00401850 |. |8B5424 18 |mov edx, dword ptr [esp+18]

假如不是要分析`nss.npa`文件的话在第一次断下后可以就在`cmp ebx,4`那里按F4然后继续分析了,但我想要分析脚本文件,于是就取消内存断点,跳转语句下一句语句按f4,知道`boot.nss`的字符串出现到堆栈窗口.当找到目标后,对`CreateFileA`,`SetFilePointer`以及`ReadFile`下断,在对`boot.nss`对应的文件头结构下内存访问断点.在创建完文件后停到以下地方:

> 00401CB8 . 8B40 20 mov eax, dword ptr [eax+20] ; 资源大小的源数据？
> 00401CBB . 57 push edi
> 00401CBC . 83C0 01 add eax, 1
> 00401CBF . E8 8C2A0500 call 00454750
> 00401CC4 . 8BF8 mov edi, eax
> 00401CC6 . 85FF test edi, edi
> 00401CC8 . 75 03 jnz short 00401CCD ; 分配内存失败则返回
> 00401CCA . 5F pop edi
> 00401CCB . 5E pop esi
> 00401CCC . C3 retn
> 00401CCD > 8B86 48010000 mov eax, dword ptr [esi+148] ; 取头文件结构地址
> 00401CD3 . 8B48 28 mov ecx, dword ptr [eax+28] ; 全部头文件的大小
> 00401CD6 . 0348 1C add ecx, dword ptr [eax+1C] ; 资源的内容在文件中的偏移
> 00401CD9 . 8B96 10010000 mov edx, dword ptr [esi+110]
> 00401CDF . 53 push ebx
> 00401CE0 . 6A 00 push 0
> 00401CE2 . 51 push ecx
> 00401CE3 . 52 push edx
> 00401CE4 . E8 D25E0B00 call 004B7BBB ; SetFilePointer以及相关处理工作
> 00401CE9 . 8B9E 48010000 mov ebx, dword ptr [esi+148] ; 取头文件结构地址
> 00401CEF . 8B86 10010000 mov eax, dword ptr [esi+110]
> 00401CF5 . 50 push eax
> 00401CF6 . 8BCB mov ecx, ebx
> 00401CF8 . 8B51 20 mov edx, dword ptr [ecx+20]
> 00401CFB . 52 push edx
> 00401CFC . 6A 01 push 1
> 00401CFE . 57 push edi
> 00401CFF . E8 23640B00 call 004B8127 ; 读取文件以及相关操作
> 00401D04 . 83C4 1C add esp, 1C
