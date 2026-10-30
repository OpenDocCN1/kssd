
## 分析

按理说这里可以`patch`掉来去掉反跳试，但就会出现我前面分析提到的问题。

通过`WriteFile`找到调用`read`/`write`的位置，也就是计算`md5`和获取`md5`的位置。

```
.text:00401D50 ; HANDLE __thiscall f_CalcKeyMd5_401D50(void *this, char *key, size_t len)...

.text:00401E4E                 push    ebx             ; lpOverlapped

.text:00401E4F                 push    eax             ; lpNumberOfBytesWritten

.text:00401E50                 lea     ecx, [esp+344h+Buffer] //用户输入的key相关数据

.text:00401E54                 push    esi             ; nNumberOfBytesToWrite

.text:00401E55                 push    ecx             ; lpBuffer

.text:00401E56                 push    edi             ; hFile

.text:00401E57                 call    ds:WriteFile //计算md5

.text:00401E5D                 test    eax, eax

.text:00401E5F                 jz      short loc_401ED4

.text:00401E61                 lea     edx, [esp+33Ch+NumberOfBytesRead]

.text:00401E65                 push    ebx             ; lpOverlapped

.text:00401E66                 push    edx             ; lpNumberOfBytesRead

.text:00401E67                 lea     eax, [esp+344h+keymd5]

.text:00401E6E                 push    10h             ; nNumberOfBytesToRead

.text:00401E70                 push    eax             ; lpBuffer

.text:00401E71                 push    edi             ; hFile

.text:00401E72                 call    ds:ReadFile //读取md5
```

`f_CalcKeyMd5_401D50`回溯一层就是输入`key`回车的响应函数。 这里先通过`UpdateData(1)`获取输入数据，然后拷贝到局部变量

```
f_UpdateData_41A4F7(1);

  f_CString_copy_417D43((CString *)&key, (LPCSTR *)&v1->key);//用户输入的
```

然后输入进行小写和反转变换

```
f_CString_lwr_4182FA((CString *)&key); //小写

 f_Cstring_rev_41830C((CString *)&key);        // 反转
```

判断输入长度是否为6，不是退出，清除输入，并通过`IsDebuggerPresent`检查是否在调试（OD直接过），是调试也退出，清理出输入。

```
if ( *(_DWORD *)(key - 8) != 6 || IsDebuggerPresent() )

 {

   CString::operator=((CString *)&v1->unk_6c, byte_431398);

   CString::operator=((CString *)&v1->key, byte_431398);

   f_UpdateData_41A4F7(0);

 }
```

满足长度要求，再看驱动是否加载，再调用`f_CalcKeyMd5_401D50`计算`md5`. 也就是调用驱动获取`md5`，记为`KeyMd51`.

```
//.text:004017DE if ( v1->is_drv_run )

    {

      keymd5str = *(_DWORD *)(key - 8);

      v3 = sub_418263(&key, 0);

      f_CalcKeyMd5_401D50(v1, (char *)v3, keymd5str);

    }
```

接着下面两个函数，先调用`f_GetStrMd5_401920`（应用层的`Md5`，通过调试可以很快确认，内部也有`md5`特征）计算`KeyMd51`的`Md5`，记为`KeyMd52`，然后调用`sub_415A78`截取`KeyMd52`从第3为开始的10字符，记为`KeyMd53`。

```
f_GetStrMd5_401920((char)v4, (CString *)keymd5str);// 00943950  37 63 37 36 36 65 32 61 31 63 61 30 35 37 63 37  7c766e2a1ca057c7

                                                // 00943960  62 30 65 39 31 66 39 33 35 65 64 61 61 64 37 33  b0e91f935edaad73

                                                // 

                                                // 

                                                // 

    sub_415A78((LPCSTR *)&keymd5str_obj, (int)&v9, 2, 0xAu);// 截取2开始长度0xA的值

                                                // 00943900  37 36 36 65 32 61 31 63 61 30 00 38 39 30 33 38  766e2a1ca0.89038

                                                // 00943910  33 39 32 36 39 32 65 38 32 64 36 33 62 31 37 64  392692e82d63b17d

                                                //
```

最后`KeyMd53`与`888aeda4ab`比较，成功提示`Success^^`！

```
if ( _mbsicmp(keymd5str_obj, a888aeda4ab) ) // 888aeda4ab

    {

      CString::operator=((CString *)&v1->unk_6c, byte_431398);

      CString::operator=((CString *)&v1->key, byte_431398);

      f_UpdateData_41A4F7(0);

    }    

    else

    {

      f_ShowSuccess_402030(v1);//成功提示

    }
```

### 总结算法

1. `KEY1 = rev(lwr(key))`，`key`长度6，将输入转小写，逆序
2. 反调试成功时`KEY1[0]+=1`, 其他`KEY1[i]+=i`;
3. `KEY2 = DrvMd5(KEY1)`，驱动`MD5`计算
4. `KEY3 = Md5(KEY2)`, 应用层`MD5`计算
5. `KEY4 = KEY3[2:12]`，取第3位开始的10个字符
6. `KEY4 == '888aeda4ab'`

# 0x11. 求解

由于`MD5` `hash`无法逆运算，只能爆破了，刚开始忘了题目`key`只能是数字和字母，结果我跑了全字符，跑了1天多....没出来，卡hi是怀疑自己

后来改成了数字字母，终于得到答案 `su1987`

爆破代码如下：

```
char Seed[/*68*/36] = {	

       'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z',	

       '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',

	};	

	

#define SEED_SIZE 36

typedef struct _THREAD_PARAM

{	

int i1;	

int i2;	

int i3;	

int i2_1;	

int i2_2;

}TPP, *PTPP;



int g_ThreadCnt = 0;

int g_start = 0;

long g_count = 0;



void write_file(char* sz)

{

	HANDLE hFile = CreateFileA("1.log", GENERIC_WRITE|GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);	

	if(hFile)

	{

		SetFilePointer(hFile, 0, 0, FILE_END);

		DWORD dw = 0;

		WriteFile(hFile, sz, strlen(sz), &dw, NULL);

		CloseHandle(hFile);

		hFile = NULL;

	}

}



bool crack1(PTPP p){	

    int i1 = p->i1;	

    int i2 = p->i2;	

    char sss[20] = {0};	

    for(int i3=0; i3<SEED_SIZE; i3++)

    {		

	for(int i4=0; i4<SEED_SIZE; i4++)

		{			

		        for(int i5=0; i5<SEED_SIZE; i5++)

			{				

			        for(int i6=0; i6<SEED_SIZE; i6++)

				{					

				

				       char sza[7] = {Seed[i1], Seed[i2], Seed[i3], Seed[i4], Seed[i5], Seed[i5]};



					g_count ++;					

					char sz[7] = {0};					

					//反转

					sz[0] = Seed[i6]+1;

					sz[1] = Seed[i5]+1;

					sz[2] = Seed[i4]+2;

					sz[3] = Seed[i3]+3;

					sz[4] = Seed[i2]+4;

					sz[5] = Seed[i1]+5;



					FileMD5 fm;					

					char* p = (char*)fm.md5(sz, 6);

					p = (char*)fm.md5(p, 32);					

					strncpy(sss, p+2, 10);					

					if(!stricmp(sss, "888aeda4ab"))

					{						

					       char info[1024] = {0};						

					       sprintf(info, "%c%c%c%c%c%c, => %s，%s\n", 

							Seed[i1], Seed[i2], Seed[i3], Seed[i4], Seed[i5], Seed[i6], 

							sz,

							sss

							);

						write_file(info);					



						int spell = GetTickCount() - g_start;						

						printf("spell time : %d s", spell/1000);



						system("pause");						

						return true;

					}							

				}

			}			//system("cls");

			printf("count: %ld\n", g_count);

		}



	}	return false;

}



void crack3(PTPP p){	

       int i1 = p->i1;	

       int i2_1 = p->i2_1;	

       int i2_2 = p->i2_2;	

       delete[] p;



	TPP p1 = {0};

	p1.i1 = i1;	

	for(int i=i2_1; i<i2_2; i++)

	{

		p1.i2 = i;		

		if(crack1(&p1))

		{			

		    return;

		}

	}

}



void crack2(int i1, int i2_1, int i2_2)

{

	PTPP p = new TPP;//{0};

	if(p == NULL)

	{		

	        printf("!!!!!!!!!!!!没neicun！！");		return;

	}	

	memset(p, 0, sizeof(TPP));

	p->i1 = i1;

	p->i2_1 = i2_1;

	p->i2_2 = i2_2;

	

	HANDLE h = CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)crack3, (PVOID)p, 0, NULL);	

	if(h == NULL)

	{		

	        printf("CreateTHREAD error [%d]\n", g_ThreadCnt);

	}	

	else

	{

		g_Handles[g_ThreadCnt++] = h;

	}

}

void crack()

{	

    for(int i1=0; i1<SEED_SIZE; i1++)

	{		int i2 = 0;#define STEP_SIZE 2

		for(i2 = 0; i2<SEED_SIZE-STEP_SIZE; i2+=STEP_SIZE)

		{

			crack2(i1, i2, i2+STEP_SIZE);

		}

		crack2(i1, i2, SEED_SIZE);

	}

}	



int _tmain(int argc, _TCHAR* argv[])

{	

        int start = GetTickCount();

	g_start = GetTickCount();



	crack();



	WaitForMultipleObjects(g_ThreadCnt, g_Handles, TRUE, INFINITE);	int spell = GetTickCount() - start;	

	printf("spell time : %d s, thread-count: %d\n", spell, g_ThreadCnt);



	getchar();	return 0;

}
```

最后结果

```
su1986, => 79;4yx，888aeda4ab
```

由于算法开始有转小写，所以其时答案中所有字母都可以是大小写选择，答案不唯一。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [vmxdrv.idb](javascript:void(0);)
  （209.01kb，8次下载）
- [CrackMe.exe.idb](javascript:void(0);)
  （3.54MB，10次下载）
