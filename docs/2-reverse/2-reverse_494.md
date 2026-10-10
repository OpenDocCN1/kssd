

# 逆向分析 C++继承内存分布(带虚函数) 及动态绑定实现

> 原文：[`www.pediy.com/kssd/pediy11/116802.html`](https://www.pediy.com/kssd/pediy11/116802.html)

昨晚断网，于是乎闲来无事，想彻底研究一下 C++的继承关系下的内存结构（带虚函数的），开 IDA 分析，经过不断的分析，发现 C++的内存结构很是复杂，很多东西都是我以前完全不知道的，一下所有内容均是个人午夜至凌晨的原创逆向分析，有很多有用的细节，个人不敢藏着掖着，特攥写此文公布细节，有用的取之，不用的勿拍砖，本人菜鸟一个，大牛请飘过![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")  
  
C++继承关系(含虚函数) 内存结构分析 以及 动态绑定的细节分析  
一．  说明  
本文较为深入的研究 C++的继承(含多重继承)情况下带虚函数时的实例内存结构，较为深入的剖析了继承实例间是如何组织的，以及动态绑定的实现细节。  
  
二．  依据  
以下阐述的细节均依据 VS2005 生成的 DEBUG 模式程序在 IDA Pro5.2 反编译所得，部分数据结构根据程序分析得出。  
  
三．  术语说明  
对于以下将频繁用到的术语，我可能简写为如下  
classX:RTTI\_COL x 原语：classX:RTTI Complete Object Locator {for x}   
classX:RTTI\_CHD   原语：classX:RTTI class Hierarchy Descriptor  
classX:RTTI\_BCD   原语：classX:RTTI Base class Descriptor  
以上简写均采用单词首字母缩写，其中 Derive 表示派生类，x 表示 Derive 的一个基类，classX 表示以上任意类  
  
四．  分析  
1.  派生类内存分布及虚表  
  
                                                           Derive :RTTI\_COL x  
 vfTable for base A        ---------------------------->>   ------------------------------------  
                                                                                virtual Derive::fun1(overwrite   
                                                                                by derive class) for base A  
 data of base A                                                        virtual A::fun2(inherit from   
                                                                                 base A) for base A  
  vfTable for base B                                                                  . . . . . .  
                                                                                ------------------------------------  
  data of base B                                                       virtual fun(if derive own virtual   
    
data of Derive itself                                                  itslef) for derive \* perhaps not  
  
                         Figure 1:派生类实例对象内存分布及虚表映射  
  
如上图，其中左部为派生类 class 的实例对象内存格局（继承来自 A,B），其中第一项为 base A 的虚表地址(A,B 数据在 class 实例中布局顺序是按照 Derive 在声明时继承顺序决定的)，从图中箭头可知 base A 的虚表地址指向了一个数组，里面存放了 Derive 对应的虚函数地址：  
① 首先存放重写过的 base A 声明的的虚函数地址(如果虚函数是从基类继承过来的，那么虚表中存放的也是 base A 中该函数地址)。  
② 然后存放 Derive 自己声明的虚函数(可能没有)。  
  
\*注意：vfTable 地址指向的虚表是从右边表格黄色以下位置开始，黄色以上是虚表附加信息，不为虚表所有，为本人 IDA Pro 反汇编分析所得。  
其中 Derive :RTTI\_COL x 是定位对象所使用的相关数据结构，其结构经反汇编如下：  
  
2.  classX ：RTTI Complete Object Locator {for x}  
Field                        Length           Remark  
reserve\_1                  4               not used,filled with 0x00  
offset\_x                     4               the offset of vfTable( for x) and   
                                                 the base address of classX   
reserve\_3                  4                not used,filled with 0x00  
pTypeDescriptor        4               address of classX:RTTI Type Descriptor  
pHierarchyDescriptor 4               address of classX:RTTI class Hierarchy Descriptor  
             Figure 2:  classX ： RTTI Complete Object Locator {for x}反汇编结构  
  
offset\_x 字段描述 基类 x 在 classX 实例内存空间中的起始地址相对于 classX 基址的偏移(也就是 x 的 vfTable 字段在 classX 内存布局中的偏移)  
pHierarchyDescriptor 字段存放 classX：RTTI class Hierarchy Descriptor 结构的地址，该结构描述了 Derive 类的内存结构层次信息。  
  
\*\*\*Remark:对于 classX 为派生类 Derive 情况时，该结构描述了基类 x 相对于 Derive 的信息,此时表格中所有 classX 均应以 Derive 替换；但是当 classX 为基类 x 情况时，该结构描述了基类自身的信息，此时表格中所有 classX 均应以 x 替换  
  
3.  classX:RTTI class Hierarchy Descriptor  
Field                     Length          Remark  
reserve\_1                4              unknown  
reserve\_2                4              unknown  
dwArrayElemCount  4              the count of the element inside pBaseClassArray  
pBaseClassArray     4              address of classX:RTTI Base class Array  
                  Figure 3:  classX:RTTI class Hierarchy Descriptor 反汇编结构  
  
该结构详细描述了类实例中相关重要信息的的索引  
pBaseClassArray 字段存放 classX:RTTI Base class Array 数组基址。  
dwArrayElemCount 字段存放 classX:RTTI Base class Array 数组项数,包括自身一项和每个基类一项。  
  
\*\*\*Remark:对于 classX 为派生类 Derive 情况时，该结构描述了派生类 Derive 的内存结构层次信息,此时表格中所有 classX 均应以 Derive 替换；但是当 classX 为基类 x 情况时，该结构描述了基类自身的内存结构层次信息，此时表格中所有 classX 均应以 x 替换  
  
  
4.  classX:RTTI Base class Array  
此数组存放了该类实例对象所需的所有布局信息。数组中每项均存放了一个类描述符 classX:RTTI\_BCD 的地址，其中实例自身的类描述符地址放在数组第一项，然后存放基类描述符地址  
  
5.  classX:RTTI Base class Descriptor  
Field                     Length            Remark  
pTypeDescriptor       4              address of classX:RTTI Type Descriptor  
dwBaseClassCount  4              the count of Base class of classX  
 Offset\_X                  4              the offset of vfTable( for classX)                                 
                                                and the   base    address of Derive  
reserve\_1                4              unknown,filled with 0xffffffff  
reserve\_2                4              unknown,filled with 0x00000000  
reserve\_3                4              unknown,filled with 0x00000040  
       Figure 4:  classX:RTTI Base class Descriptor 反汇编结构  
        该结构中有几个特别需要注意的地方，首先该结构是从 classX:RTTI class Hierarchy Descriptor >classX:RTTI Base class Array 引出的，所以此结构中的 Offset\_X 数据都是相对于 classX:RTTI class Hierarchy Descriptor 中的 classX 对象而言  
  
举例说明，从 Derive：RTTI class Hierarchy Descriptor 中引出的 x:RTTI Base class Descriptor 中的 Offset\_X 字段是 x 相对于 Derive 而言的；但是如果从 x:RTTI class Hierarchy Descriptor 中引出的 x:RTTI Base class Descriptor 中的 Offset\_X 字段则是 x 相对于 x 而言的  
  
  
小结：根据上述数据结构，我们可以清晰的将一个类继承关系描述出来，当然只包括虚函数部分，我们甚至可以基于此信息来完成一个 PE 的类继承视图的扫描器，我们甚至有可能通过对一个基类的扫描而窥知其派生类的继承视图，对，这是有可能的，以上结构信息我相信还有其他用途可供开发研究，再次本人不予赘述。  
  
补充部分(关于 C++的虚函数的调用过程细节):  
以下结果是根据 IDA PRO5.2 逆向反汇编分析出的，分析版本为 Debug  
  根据逆向分析，可知编译器在虚函数处理过程中完成了不少的工作，当然是编译时工作，根据分析发现虚函数表中函数分布是根据函数声明顺序来存放的，类本身的虚函数置于第一虚表中的末尾位置，这在派生类内存分布及虚表小节中有详细说明，而虚表地址字段又是根据派生类对基类的引用顺序来存放的，那么分析可知，对于某个虚函数而言，无论动态绑定后的结果如何，此虚函数在虚表中的偏移在编译时就能被确定了，那么我们可以认为编译器在对虚函数的绑定技术是基于 虚表基址+偏移地址 取内容来完成的。  
  
  虚函数的编码问题，根据 Debug 版本逆向分析可知，对于虚函数中的 this 引用，编译器采用的是该函数所在虚表地址字段  虚表地址字段相对于本实例内存基址的偏移，这样就可得出该实例的内存基址，其中虚表地址字段相对实例基址的偏移在编译时就能确定。  
  
  thiscall 问题，根据被调用的虚函数所在的虚表来确定传入的 this 指针，虚函数所在虚表在编译时就能确定，此时，将虚表地址字段的地址作为 this 指针传入(其实也可理解为此虚函数声明所在的类实例的起始地址)。通过传入的 this 指针，虚函数根据基址相对偏移可以轻松定位到对象的真正基址，从而就能达到通过指针实现动态绑定技术。  
  
  
以上内容均为原创分析，如需引用请注明出处。  
                                     by kalrey   2010/07/15 午夜至凌晨  
  
  
        不知道看雪的文章怎么支持 word 文档格式，所以上面的格式很乱![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")，勉强把格式调整的能看  
        里面可能有部分数据有误，如发现请告知，不甚感激，文后附有 word 文档供下载

上传的附件
| ![](img/58e0728fcb44eb7297cf87d68539ed4e.jpg) || C++继承 内存分布(带虚函数)及动态绑定细节.doc  
 |