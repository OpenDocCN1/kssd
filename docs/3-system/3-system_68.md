

# 对导入表结构的学习心得

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-69.htm`](https://www.pediy.com/kssd/pediy07/pediy7-69.htm)

                                    对导入表结构的学习心得  
                                                      by mirrormask  
                                                      2005.2.13  
    学习逆向一个月，这两天才刚刚接触 pe 结构，对导入表的各个字段和结构之间的关系一头雾水。于是做了一些试验，有了几点想法，也不知道对不对，污高手耳目了，请指点：）  
  
    文中涉及的字段和结构：  
    INT：IMPORT NAME TABLE  
    IAT: IMPORT ADDRESS TABLE  两个指针数组  
    OriginalFirstThunk 、 FirstThunk 是 IMAGE\_IMPORT\_DESCRIPTION 结构的两个指针      
    DATADIRETORY 中的第 13 项 IMAGE\_DIRETORY\_ENTRY\_IAT 结构  
  
      
    我对 masm 编译生成的一个 exe 文件进行了静态修改，改变上面提到的字段或结构的内容，看看在 pe 文件装载时它们之间的关系是怎样的.  
    
    如果 pe 文件调用了 dll 文件中的函数,在程序编译时，编译器(只用过 masm32 8.0)确定好 OriginalFirstThunk 指向的 INT 和 FirstThunk 指向的 IAT 的内容。同时在执行代码尾部形成形式如 jump dword ptr ds：\[\*\*\*\*\*\*\*\*\]的一系列指令，\*\*\*\*\*\*\*\*显示的是在编译时确定的相应的 IAT 数组成员的地址，\[\*\*\*\*\*\*\*\*\]是 IAT 数组成员的内容--映射到内存中的 dll 文件的函数地址（函数入口地址）.  
  
1、只修改 FirstThunk：  
    在 pe 装载到内存时的初始阶段，系统搜索遍历 INT 指向的 IMAGE\_THUNK\_DATA 结构，确定每一个函数在内存映射空间中的位置（函数入口地址），然后系统根据 FirstThunk 指向的地址，把该地址向下的每一位置依次填写搜索到的函数入口地址--只要该段的属性是可写的。如果该位置的段属性是不可写，则发生访问违反（ollydbg），windows 提示内存初始化失败。这时用 od 打开修改后的 FirstThunk 指向的地址，会发现该部分内存已被函数入口地址覆盖。  
    同时，如果 OriginalFirstThunk 和 INT 正确（没有修改），系统在装入时可以解析函数的名称（该名称是 INT 指向的 IMAGE\_THUNK\_DATA 结构中的编译时填入的函数名称（IMAGE\_IMORT\_BY\_NAME 结构内容)，而不是函数真实调用的 dll 文件中的函数名称。ollydbg 的注释栏可‘正确’显示函数名，反汇编栏则只显示调用地址）。  
    此时程序仍然按照编译时确定的 IAT 数组内容调用函数，可惜搜索到的函数入口地址没有填入该位置，该位置的内容仍然是一个个的 IMAGE\_IMORT\_BY\_NAME 结构地址,程序无法正常运行.  
  
2、只修改 DATADIRETORY 中的第 13 项 IMAGE\_DIRETORY\_ENTRY\_IAT  
    在加载 pe 文件时，系统首先根据 DATADIRETORY 中的第 13 项 IMAGE\_DIRETORY\_ENTRY\_IAT 中的地址内容（VirtualAddress）改变该地址所在段（SECTION）的属性-----不论该段属性(IMAGE\_SECTION\_HEADER 结构的 CHARACTERISTICTICs 指定)是否可写，在加载阶段，搜索到的函数入口地址均可写入该段（初始化 IAT 数组）。  
    VirtualAddress 的值除了影响该地址所在段在初始化 IAT 数组时的属性外，不会给程序加载和运行造成任何影响。但是该值必须落在 pe 文件本身的某一段内（从 0000 0000——文件结尾的 RVA），对于超出此范围的地址所在的内存空间，程序没有权限改变属性。  
    isize 的值可以变化（改变后程序仍然运行正常），但可以变化的范围不明：（  
     
3、只修改 IAT 的内容  
    IAT 数组在程序装载到内存后动态更新，只要不破坏数组结构，静态修改数组内容不会影响程序运行  
    当空 IAT 出现在 FirstThunk 指向的地址时（该 dword==null），该位置及其以下的 IAT 数组不会被函数入口地址填充。  
4、其他  
    对 DATADIRETORY 中 IMAGE\_DIRETORY\_ENTRY\_IMPORT 的 VirtualAddress 的任何修改都会导致程序错误--因为找不到导入表。  
    对 OriginalFirstThunk、INT 数组成员、IMAGE\_IMORT\_BY\_NAME 的任何修改都会导致程序错误。  
  
    我只是凭兴趣学习，没有这方面的理论修养，有说错话用错字的地方还请各位指点：）写了一下午，晕了  
    感谢看雪学院和罗云彬两部大作的引我入门。 

 4dd466d0e1e0cc123afbe504da2462ca