
# [原创]08年用`masm`写的一套源码！

2008年写的！并没有销售过 大家可放心看。里面包含了2个`RadASM` 和 `MASMPlus` 的工程文件，打开`MainDll.app` 可直接编译,当然你要配置一下`masm`的路径，直接打开`Main.exe`可选择系统自带的 `calc.exe`计算器 启动后会自动呼出`dll`，由于现在没有服务器,为了方便直接看已经把里面服务器通信的几个小地方删除，服务器端也是用`masm`写的,注册和充值我是用`.net`实现. 用户直接点击‘登录’就可以看到所有界面 。资源文件`rs`用`vc6.0`以上的工具可查看。

另外，我自己的一些经历，历程想写下来 链接是：

`http://bbs.pediy.com/showthread.php?p=1308408#post1308408`

有兴趣的朋友请支持一下。谢谢

部分代码有：

```
                .data?

hGoodsFile        dword        ?;data.dat文件句柄

hGoodsMap        dword        ?;物品的map

lpGoodsView        dword        ?

hGoodsFile2        dword        ?;data2.dat文件句柄

hGoodsMap2        dword        ?;物品的map

lpGoodsView2        dword        ?

hGoodsFile3        dword        ?;怪物

hGoodsMap3        dword        ?;怪物的map

lpGoodsView3        dword        ?

szGoodsFileBuf        BYTE MAX_PATH DUP(?);sell.txt文件缓存

szGoodCwpBuf        BYTE MAX_PATH DUP(?);save.data文件缓存

                .data

szGoodsFile        byte        'sell.txt',0;sell.txt文件 装有不要扔掉的物品数据

szGoodsCwp        byte        'save.txt',0;save.data文件 指定存入仓库的物品名字

szGoodsS        byte        '无限回城符,素炒竹肉,家常四季豆,炒豌豆角,蒜茸生菜,玉笛谁家听落梅',0;默认存放不扔掉的物品 每个字符之间用空格分隔,最后一个以0结尾

szSaveS                byte        '小块玄晶碎片（1级玄晶）,玄晶碎片（2级玄晶）,小块玄晶残片（3级玄晶）,玄晶残片（4级玄晶）,未雕琢的玄晶（5级玄晶）,有裂痕的玄晶（6级玄晶）,普通的玄晶（7级玄晶）',0

                .code

;查找指定字符串 没有找到返回0 找到返回查找到的字符后面的地址

;_FillStart主块开始地址   _Search要查找的字符串

_SearchString        proc                _FillStart,_Search

                local                @SearchSize,@FillSize,@return ;@SearchSize;要找查的大小 @FillSize主块的大小

                pushad

                invoke        lstrlen,_Search

                mov                @SearchSize,eax;要查找的大小

                invoke        lstrlen,_FillStart

                mov                @FillSize,eax;主块大小

                mov                @return,0

                .if                eax

                                        mov                esi,_Search;要查找的字符串地址

                                        mov                edi,_FillStart        ;主块开始地址

                                        mov                ecx,@FillSize;主块的大小

                                        mov                ebx,@SearchSize;要查找的大小

                                S1:

                                        or                ecx,ecx

                                        jz                _Ret;读取到的已经为0没有查找性了

                                        or                ebx,ebx

                                        jz                _Ret;要比较的为0也没有比较性了

                                        cmp        ecx,ebx

                                        jb                _Ret;读取到的比要查找到的还要小 退出

                                        mov        al,[esi]

                                        cmp        al,[edi]

                                        jz                S2;查到的第一个字符相等,开始进入第二阶段

                                        dec        ecx;读取的-1 表示比较完一次

                                        inc        edi;读取的字符向前移动

                                        jmp        S1;

                                S2:;第一阶段通过后进入此阶段

                                        dec        ebx

                                        dec        ecx

                                        inc        esi

                                        inc        edi

                                        or                ebx,ebx

                                        jz                findf;表示已经查找到了 ;这个需要在下面的or ecx,ecx的上面。不然如果字符在最后的话会不正确的

                                        or                ecx,ecx

                                        jz                _Ret;读取到的已经为0没有查找性了

                                        cmp        ecx,ebx

                                        jb                _Ret;读取到的比要查找到的还要小 退出

                                        mov        al,[esi]

                                        cmp        al,[edi]

                                        jz                S2

                                S3:;到了这个阶段表示在第二阶段也没有找到

                                        mov        esi,_Search

                                        mov        ebx,@SearchSize

                                        jmp        S1

                        findf:;

                                        mov        eax,@FillSize ;主块的大小

                                        sub        eax,ecx                  ;主块的大小-还没有读取的主块大小=已经读取的主块大小

                                        add        eax,_FillStart;已经读取的主块大小+主块起始地址=查找到的字符后面的地址

                                        mov        @return,eax;查找到的字符后面的地址

                                        jmp        _Ret

                .endif

_Ret:

                popad

                mov                eax,@return

                ret

_SearchString        endp

;打开或创建文件,（文件不存在将要创建文件同时放入默认要保存的物品)

_CreateGoods        proc

                local        @buf

                pushad

                invoke        lstrcpy,addr szGoodsFileBuf,addr szDllPath

                invoke        lstrcat,addr szGoodsFileBuf,addr szGoodsFile;取得sell.txt全路径

                invoke        CreateFile,addr szGoodsFileBuf,GENERIC_READ OR GENERIC_WRITE,FILE_SHARE_READ OR FILE_SHARE_WRITE,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL

                .if                eax==INVALID_HANDLE_VALUE;如果文件不存在则创建此文件

                                invoke        CreateFile,addr szGoodsFileBuf,GENERIC_WRITE OR GENERIC_READ,FILE_SHARE_READ OR FILE_SHARE_WRITE,NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,NULL;打开/创建文件

                                .if                eax !=INVALID_HANDLE_VALUE

                                                        mov                hGoodsFile,eax

                                                        invoke        CreateFileMapping,hGoodsFile,NULL,PAGE_READWRITE,0,4096*50,NULL

                                                        mov                hGoodsMap,eax

                                                        invoke        MapViewOfFile,hGoodsMap,FILE_MAP_READ OR FILE_MAP_WRITE,0,0,0

                                                        mov                lpGoodsView,eax

                                                        ;因为文件才创建 所以我们先写入自己预设一点数据

                                                        invoke        lstrcpy,lpGoodsView,addr szGoodsS

                                                        invoke        FlushViewOfFile,lpGoodsView,sizeof szGoodsS

                                .endif

                .else;文件存在则打开map

                                mov                hGoodsFile,eax

                                invoke        CreateFileMapping,hGoodsFile,NULL,PAGE_READWRITE,0,4096*50,NULL

                                mov                hGoodsMap,eax

                                invoke        MapViewOfFile,hGoodsMap,FILE_MAP_READ OR FILE_MAP_WRITE,0,0,0

                                mov                lpGoodsView,eax

                .endif

                invoke        lstrcpy,addr szGoodCwpBuf,addr szDllPath

                invoke        lstrcat,addr szGoodCwpBuf,addr szGoodsCwp;取得sell.txt全路径

                ;仓库物品

                invoke        CreateFile,addr szGoodCwpBuf,GENERIC_READ OR GENERIC_WRITE,FILE_SHARE_READ OR FILE_SHARE_WRITE,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL

                .if                eax==INVALID_HANDLE_VALUE;如果文件不存在则创建此文件

                                invoke        CreateFile,addr szGoodCwpBuf,GENERIC_WRITE OR GENERIC_READ,FILE_SHARE_READ OR FILE_SHARE_WRITE,NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,NULL;打开/创建文件

                                .if                eax !=INVALID_HANDLE_VALUE

                                                        mov                hGoodsFile2,eax

                                                        invoke        CreateFileMapping,hGoodsFile2,NULL,PAGE_READWRITE,0,4096*50,NULL
```
