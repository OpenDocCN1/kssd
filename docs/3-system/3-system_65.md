

# 获取 WINDOWS 可执行文件里面所有 DLL 导入函数信息

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-678.htm`](https://www.pediy.com/kssd/pediy09/pediy09-678.htm)

> 此文章纯属自己学习用，， ，！是摸索篇 所以思绪不是很好!有错误请不惜赐教,指正!  
>   
> 初步了解 PE 文件后才能看懂下面的 PE 略过信息,  
>   
> 用户级别的执行程序，在与 IO 硬件通信时是通过驱动程序这个桥梁的,这只是大致说！  
>   
> 应用程序首先调用自己的函数模块，在与硬件 IO 通信的时候会经过 WINDOWS API,比如调用 MessageBoxA 函数 , 该 API 会通过系统服务接口层 到达 IO 管理层(一些服务例程，检查处理 API 相关参数),经过相应的打包处理 传一个 IRP 结构体(IO 请求包) 到相应的驱动入口处理，接着驱动程序 HAL 调用到 HAL 硬件抽象层,平台相关的操作,最后实现硬件上 IO 显示对话框。  
>   
> 上面是一个用户级别的程序通过 WINDOWS 访问硬件的一些步骤!还有在内核模式下执行的程序,它调用函数不会经过 API 而是直接 通过系统服务接口,调用 NTDLL.DLL 服务例程里的函数(IO 管理器),再与驱动程序通信)  
>   
> 以上内容纯属带过、以下是正题 PE 文件:  
>   
> EXE 执行文件又名 PE 文件(可移植执行文件)，虽然是执行文件 但文件里面不全是执行代码  
>   
> 这要讲到 PE 文件的结构，只简单的介绍一下:  
>   
> 像 DLL、EXE 都是 PE 文件；PE 文件的结构如下:  
>   
> IMAGE\_DOS\_HEADER (DOS 头)  
>   
> 1、DOS 头( 用来存放兼容 DOS 程序 的一些字段 其中 e\_magic 字段 标识"MZ" 表示是 DOS 程序  
>   
> 另外 扩展了 e\_lfanew 字段,它的值是 PE 文件头的 RVA 地址 ）  
>   
> 2、DOS 块(64KB 段 )   ;存放 DOS 代码的区域 ，在 DOS 头里有程序初始环境的字段定义  
>   
> 3、PE 头(PE 文件头 它的 RVA(文件被装入到内存中的偏移地址) 是有 DOS 头中的 e\_ifanew 字段的值)  
>   
> 其中 PE 头结构体中的 signature 字段标识"PE" 表示是 PE 文件  
>   
> 5、节表 (它的 RVA 是紧跟 PE 头后面，所以 用 DOS 头中的 e\_ifanew 加上 sizeof IMAGE\_NT\_HEADER 就是节表的起始地址了) 节表 字段记录着每个节的信息 包括节的可读、可写、可执行等..  
>   
> 而节的总数则在 PE 头的 FileHeader 指针字段里相关的 STRUCT 里)  
>   
> 6、节(就是.data 、.code 等 定义的段 像这些数据段 名，都是可以自定义的只要设置好节的相关属性就行了)  
>   
>   
> 下面说怎么获得执行文件里面的函数信息:  
>   
> 1、我们首先可以用 CreateFile 创建一个文件句柄 因为要对文件进行分析!  
>   
> 2、再用 CreateFileMapping 将刚才的文件 变成内存映射文件 ;相当于直接在内存操作!方便(可以映射 4GB 以下)  
>   
> 3、用 MapViewOfFile 将内存映射文件视图化,这样会得到一个文件数据在映射在内存中开始的地址 保存这个地址(就将它用 lpFile 来表示吧,它就是一个 32 位的内存地址 代表文件在内存映射中的开始的地址，这只是一个 4KB 的内存分页的页面中的地址 是存放在虚拟内存中,不过用 lpFile 也可以直接当内存地址来用，这是 WINDOWS 自动处转换的)  
>   
> 4、得到内存映射文件初始地址后就可以 对照 PE 文件里相应的结构 RVA 来分析了  
>   
> 当然验证改 EXE 文件是否是 PE 文件当然是很重要的 ！这里可不是后缀名是 EXE 就是执行文件了,!  
>   
> 现在让我们来操作这个被保存的内存映射地址、首先判断它是否是正确的 PE 文件 可以先判断 DOS 头的'MZ'字段, lpFile 定义为 IMAGE\_DOS\_HEADER 指针  
>   
> if(lpFile->e\_magic== "MZ") 判断它的字段值是否是"MZ” 如果不是当然不是正确的 DOS 程序文件了!  
>   
> 如果是"MZ“,我还可以进一步判断"PE"标识, 来判断它是 DOS 程序还是 WIN32 程序  
>   
> 从 lpFile->e\_ifanew 中得到 PE 头的起始 RVA 地址  
>   
> 这个时候 lpFile+=lpFile->e\_ifanew ,得到 PE 文件头，再定义 lpFile 指向 IMAGE\_NT\_HEADERS (PE 文件头)  
>   
> if(lpFile->signature=="PE")这个时候可以确定它是真正的 PE 文件了!  
>   
> 那么现在 lpFile 是 PE 头的其实地址 ，要获取被导入函数信息，那么首先要确定这些数据在哪里，  
>   
> 这些数据是在.rdata 节里. 而获取节的信息可以在 PE 头里获取!  
>   
> 那么首先 lpFile=&(lpFIle->OptionalHeader.DataDirectory)+8(因为第 1 个是导出表结构体) 得到 导入表的结构体地址后, 再用 lpFile+lpFile->VirtualAddress 求出导入表结构体所在 RVA!(被装载到内存中的偏移地址)  
>   
> 要知道所有常量、资源等数据都是存放于节区的，节区被程序自己装入到内存后，节在内存中的 RVA 跟在文件中的偏移（EA）是不一样的.那么我们求出来的 lpFile+lpFile->VirtualAddress 这个 RVA 地址并不是文件实际偏移地址而是该执行程序自己运行后在内存中的 RVA，我们现在处理的文件并不是程序自己装载到内存中,而是用 CreateFIleMapping 创建得 ,它创建的内存映射文件在内存中是原封不动数据，当然节区也就相当于文件 EA。那求出来数据的地址不能直接用 lpFile(开始)+RVA 来访问 ,因为 lpFIie(开始)+RVA 是程序自己运行时的地址，而现在并不是程序自己装载到内存的，而是用 CreateFileMapping 装入的,  
>   
> 这就要想办法转把这个 RVA 转换成文件 EA:，我们可以首先求出这个数据属于哪个节！要找节的信息我们得去节表找,,在 p=PE 头起始地址+sizeof IMAGE\_NT\_HEADERS 就是节表的头了,节表是由一系列 IMAGE\_SECTION\_HEADER 结构体组成! 一个节拥有一个结构体,最后留一个字段全为零的 IMAGE\_SECTION\_HEADER 结构体标识结束!那么一个节就是 2 个结构体,二个节就是 3 个结构体!  
>   
> 我们就在这个结构体判断数据 RVA 值与每个节的 RVA 对比!  
>   
> if(p->.VirtualAddress<=数据 RVA <=p->VirtualAddress+p->SizeOfRawData),当数据的 RVA 在此节的起始地址到结束地址之间时候代表属于此节! 那么知道此数据属性这个节了那就好办了,,先将:数据 RVA 减去 p->virtualAddress 得到 数据相当与它所属节的偏移量! 在用 int ea=p->PointerToRawData 求出节在文件中的偏移,p=ea+数据 RVA 得到 导入表结构体在内存映射中的正确地址!  
>   
> 得到导入表正确地址真是太好了!现在说说导入表结构体:  
> 导入表结构体是个 IMAGE\_IMPORT\_DESCRIPTOR 结构体 ,如果导入了一个 DLL 那么会存在两个 IMAGE\_IMPORT\_DESCRIPTOR 结构体 最后一个是全字段为零的，表示结束！ 如果导入了二个 DLL 就会存在三个 IMAGE\_IMPORT\_DESCRIPTOR 结构体,原因不用我说了吧，最后一个全零垫底!  
>   
> 那么知道这个结构体是代表一个 DLL 那么 DLL 里的函数信息当然也就记录在这个结构体里面!现在地 P 已经是第一个导入表结构体的地址了，用 p+sizeof IMAGE\_DESCRIPTOR\_DESCRIPTOR 可以遍历所有的 DLL 直到遇到 0！（这里可以用作外循环遍历有多少个 DLL）  
>   
> p->name1 是一个指向 DLL 名称的字符串指针!  
>   
> 用 p->OriginalFirstThunk 或者 p->FirstThunk 求出函数信息结构体,OriginalFirstThunk 跟 FirstThunk 是指向两个同类型相同内容而不同地址的 IMAGE\_THUNK\_DATA 结构体,!FirstThunk 指向的 IMAGE\_THUNK\_DATA 结构体 在程序装载时会被 WINDOWS 转换成函数真正的地址!在此之前 IMAGE\_THUNK\_DATA 只是一个函数导入方式!,而 OriginalFirstThunk 的字段是 First 备份函数信息用的!  
>   
> 一个 DLL 不可能只能导入一个函数吧，所以 IMAGE\_THUNK\_DATA 结构体也不只一个,它也是以全字段为零表示没有函数可导入了，p=p->FirstThunk 得到 IMAGE\_THUNK\_DATA 开始地址 这个时候可以用 IMAGE\_THUNK\_DATA 结构体来判断函数的导入方式!它是一个 DWORD 结构体，我们可以用 unsigned long(p) & 80000000h 俩比较最高位是否为 1 如果最高位是 1 表示函数是按序号方式导入,如果是 0 表示我们可以的函数字符串信息了！它将是一个指针指向 IMAGE\_IMPORT\_BY\_NAME \*(unsigned long(p) )->Name1 得到函数名，\*(unsigned long(p)) -> Hint 得到函数序号!  
>   
> !到这里 可以循环到下一个 IMAGE\_THUNK\_DATA 直到字段为零表示所有导入函数都被找出来了！  
>   
> 经过 DLL 外循环就可以把 PE 文件里面所有的导入函数信息输出到显示器或者文件里面了！！！就先写到这吧，当中自己学习的笔记吧!!以后学习了还会接着写的,其实 PE 文件虽然用到了很多结构体与指针 不够整体看来还是很简单的明了的!!! 

 4dd466d0e1e0cc123afbe504da2462ca