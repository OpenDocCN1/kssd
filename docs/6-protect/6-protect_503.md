

# 论引入表的合并【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-536.htm`](https://www.pediy.com/kssd/pediy08/pediy8-536.htm)

写这个文章不是为了脱壳,而是为了加壳.脱壳靠工具(当然,高手都是 diy),加壳就靠自己了(用工具加壳似乎没什么太多意思,毕竟是自己想搞明白).加壳基本原理在我上一篇帖子中写了很多.但具体操作真的完全是两码事.一个比较头疼的问题就是合并引入表.  
  
因为 stub 有自己的引用表,原来的 app 也有一套.我们加壳后需要继续一定的合并,是 app 的引入库加入 packed app 中.否则这个加壳的程序是不稳定的,在某种情况下是崩溃的(当然,不考虑健壮性的不算).一般来说,就把 dll 的名字和第一个 function 的名字引入就可以了.  
  
熟悉 pe 结构的人都知道 iat,但光光 iat 是不够的.关于合并两个文件的引入表,目前世面上的两边书都有一定的介绍.飞天诚信的书写的商业目的太强,你看了就迷茫了,他上面写的使用 memcpy 就可以做到,我的天,如果真这么简单,我必须用 obj 文件加自己的编译器了.看雪老大的书没有具体的操作,但有加密 iat 部分,可以参考.  
  
如果要真正合并所有的引入表,我们就要搞清楚到底怎么下手.  
  
1. 合并的东西不是一个表,而是 4 个表,iat,iid,int, namestr,其中,iat 和 iid 中间一般会有一定 com 信息,都是不能修改的,所以在 rdata 区,一共有 5 个部分,我们要修改 4 个,移动 5 个.  
  
2. 所有的表都不是线形的,所以 memcpy 是必要的,但不是全部.  
  
3. 我们要搞清楚每个部分的起始和大小.iat 的信息是在 datadic 中有. iid 也在 datadic 中有,用 iid-iat 就得到了 com 信息块的信息. int 的起始位置需要用 iid 的大小算出来,int 的大小可以认为和 iat 一样.那剩下的就是 namestr 的大小了. 呵呵,这个东西说起来容易,真到计算的时候可要小心,别搞错了.  
  
4. 要得到原来 app 的 iid 中的信息,主要的就是 dllname,和 functionname.按 functionname 不是一个简单的 str,而且一个 imported\_by\_name 结构,前面有一个 word 呢. 所以每增加一个 dll 引入,整个需要增加的尺寸是 sizeof(IMAGE\_THUNK\_DATA)\*4+strlen(strDllName)+strlen(strFunctionName)+sizeof(IMAGE\_IMPORT\_DESCRIPTOR)+sizeof(IMAGE\_IMPORT\_BY\_NAME)+sizeof(BYTE)\*4+sizeof(WORD) ,如果是绑定的就是 sizeof(IMAGE\_THUNK\_DATA)\*4+strlen(strDllName)+sizeof(BYTE)\*2+sizeof(IMAGE\_IMPORT\_DESCRIPTOR).  
  
5. 由于所有的逻辑结构都要破坏(没有很巧妙的办法,遍历 iid 更新会造成效率的降低和逻辑的昏乱),所以我们可以放弃所有逻辑关系,按照块区数据更新. 更新 iat,先 memcpy 老的数据,再在最后加一个 IMAGE\_THUNK\_DATA,指向 functionname,如果是 ordinal,就把原来的数据 copy 过来,最后补 4 个字节的 0.对所有老的数据,都要加一个偏移,偏移的大小是 sizeof(IMAGE\_THUNK\_DATA)\*4+sizeof(IMAGE\_IMPORT\_DESCRIPTOR. 然后更新 iat 的 size,否则 loader 就再这里找 com 信息了. 另外,delphi 的 iat 数据都是 0!  
  
第二步,memcpy 老的 com 信息过来.  
  
第三步,memcopy 老的 iid,所有 iid 的 int 指针都要那个偏移,而 iat 不用动.name 指针也要加上偏移.在最后 0 的 iid 中加上新的 iid.新的 iid 中 iat 指向我们刚刚加入的 iat,name 指向 dllname,而 int 指针指到 int 的最后.更新 iid 的起始位置和 size  
  
第四步,memcpy 老的 int 过来,所有 int 的信息都加上偏移,最后补上一个 IMAGE\_THUNK\_DATA 指向 functionname.  
  
最后一步,memcpy 老大 namestr 过来,再最后加上 dllname,和 imported\_by\_name 指向的 functionname.  
  
6. 更新 rdata section 的数据.这里的数据需要对齐.  
  
看了上面一大堆,大家都晕了吧,我刚刚开始也一样,看上去简单问题,如果都是指针操作,一会就晕了,所以我放弃了使用遍历 iid 的方法更新所有数据的思路,抛弃了所有逻辑上的关系,完全考对数据功能的理解更新所有的数据.由于所有的表都不是完全线形的,而我们都是要在数据间插入我们的新数据,所以没一步都要小心.最好把所有文件都 map 进来再操作.  
  
不了解 pe 结构,是无从下手的,如果指向靠工具脱壳的人,也不需要看这个东西.   
  
写这个文章是希望能有更多人真正的理解,其实加壳就想给 pe 动手术.  
  
////////////////////////////////////////////////////////////////////  
按照看雪老大的要求,我上传一下代码,大家不要笑话我,写的很乱,对绑定的 iat 处理不是很仔细.  
  
这个代码的功能就是把 app 的第一个 dll 合并到 stub 的表中.如果要合并所有的,遍历就是了. 

相关源码下载：merge.zip

 4dd466d0e1e0cc123afbe504da2462ca