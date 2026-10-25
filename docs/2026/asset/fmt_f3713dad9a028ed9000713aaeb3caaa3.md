
# [原创]PEiD中unUPX.dll(UPX unpacker)插件的逆向分析笔记

【文章标题】: PEiD中unUPX.dll(UPX unpacker)插件的逆向分析笔记

【文章作者】: the0crat

【作者邮箱】: the0crat.cn\_at\_gmail.com

【作者主页】: fdfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8Z5k6e0m8U0M7X3q4@1i4K6u0W2M7%4m8S2j5$3g2K6i4K6u0W2L8r3W2$3k6g2)9J5k6h3y4G2L8b7`.`.

【生产日期】: 20070821

【编写语言】: VC++ 6.0

【使用工具】: IDA + OD + IceSword

【作者声明】: 本文仅供研究学习，本人对因这篇文章而导致的一切后果，不承担任何法律责任。本文中的不足之处请各位多多指教

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

--------------------------------------------------------------------------------

## 【详细过程】

就是试试逆向,找了个10多KB的,不会太花时间也不会没劲.第一次搞逆向,没什么经验,也不知到底该做到什么程度,不足之处请见谅,多多指教.

`Exports`中有三个函数,

`LoadDLL`

`DoMyJob`

`DllEntryPoint`

其中`LoadDLL`是返回此DLL于PEiD中的名称,如下

```
char* __stdcall LoadDLL ()

{

	return "Unpacker for UPX";

}
```

`DLLEntryPoint`就不用说了,可以参考PEiD的插件编写资料,这里就不放于重点了,此文探查去UPX壳的过程

先附上要用到的两个函数,`DoMyJob()`中的,先提前放上来

```
////////////////////////////////////////////////////////

//

//  _IsFileExisting

//  return : TRUE  if File existing
//           FALSE if File not existing

//

////////////////////////////////////////////////////////

BOOL __stdcall _IsFileExisting (char *psFN)

{

	WIN32_FIND_DATA		WFD;

	if (FindFirstFile (psFN, &WFD) == INVALID_HANDLE_VALUE)

	{

		return FALSE;

	}

	return TRUE;

}
```

```
////////////////////////////////////////////////////////

//

//  _CalcAddrOffset
//	两次用到用来计算偏移量
//  return : Address (used by SetFilePointer)
//
//

////////////////////////////////////////////////////////

int __stdcall _CalcAddrOffset (	char*	pPartOfFileImage,
								int		nBaseAddr,
								int		word_404460)

{

	int*	pn	= 0;

	int		n	= 0;

	int		i	= 0;

	pn = (int*)pPartOfFileImage;
	n = nBaseAddr;
	pn += 3;
	for ( i = 0; i < word_404460; i++ )
	{
		if ( *pn <= n )
		{
			if ( (*pn + *(pn + 1)) > n )
			{
				n = (n - *pn) + *(pn + 2);
			}
		}
		pn += 0xA;
	}
	return n;
}
```

下面是`DoMyJob()`~~~
