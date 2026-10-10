

# 记录一段时间学习笔记..

> 原文：[`www.pediy.com/kssd/pediy11/127355.html`](https://www.pediy.com/kssd/pediy11/127355.html)

大致包括三部分:  
①:C 语言的笔记  
②:windows 程序设计笔记  
③:汇编语言的方面的  
笔记已经做的很久了,另外还有几个部分没贴出来,是 windows 内核方面的笔记,一直做为自己的参考资料,都是根据鄙人的需要所做的记录,有写地方并未做的完整,只是贴了一部分代码和参数分析,有用取之..  
  
  
1.  ●●●●●●●C 语言程序设计●●●  
  
●  \\n 只代表一个字符。类似于\\n 的转义字符序列为表示无法输入的字符或不可见  
字符提供了一种通用的可扩充的机制。除此之外，C 语言提供的转义字符序列还包括：\\t 表  
示制表符；\\b 表示回退符；\\"表示双引号；\\\\表示反斜杠符本身  
●  符号常量：用标示符代表一个常量。在Ｃ语言中，可以用一个标识符来表示一个常量，  
称之为符号常量。   
符号常量在使用之前必须先定义，也叫具名常量,其一般形式为：   
#define  标识符(常量名) 常量值  
直接常量:直接常量是指在程序中，以直接明显的形式给出的数据。根据使用的数据类型不同，可分为：字符串常量、数值常量、逻辑常量和日期常量。  
字符串常量：是用两个双引号括起来的一串字符。例如："2345"和"Basic"等。  
数值常量：数值常量就是常数，包括整数、长整数、单精度数和双精度数。例如：2006、3.14159265、2.9D67 等。  
逻辑常量：逻辑常量只有 True 或 False 两个值。  
日期常量：用两个【#】把表示日期和时间的值括起来表示日期常量。  
  
●  :数据类型\[32 位系统\] char(1 字节) bool(1 字节) short(2 字节) WORD(2 字节) DWORD(4 字节) int(4 字节)  LONG(4 字节) float(4 字节) double(8 字节)   
基本的数据类型默认情况下是 signed"有符号的",无符号表示 unsigned 数据类型  
\*\*\*变量使用之前必须要先定义变量。一般情况下，都会在变量定义语句之后才使用变量；若一定要在变量定义语句之前使用变量，也必须借助关键字 extern 来声明变量。使用 extern 声明变量的形式如下：extern 数据类型名 变量名；  
 extern 修饰的变量,表示变量或者函数的定义在别的文件中，提示编译器遇到此变量和函数时在其他模块中寻找其定义,例如系统服务描述表:  
首先定义这个表的结构:结构名(类型名)不一定非要和系统的一致,只要成员类型以及成员名一致即可  
typedef struct SYSTEM\_DESCRIPTOR\_TABLE  
{  
   PVOID ServiceTableBase;  
   PVOID ServiceCounterTable;  
   unsigned int NumberOfServices;  
   PVOID ParamTableBase;  
}SYSTEMSERVICEDESCRIPTORTABLE,\*PSYSTEMSERVICEDESCRIPTORTABLE  
这个结构名是自己定义的,若要与系统的服务描述表链接起来,则要用 extern,系统服务描述表并为导出,只是导出了一个指针 KeServiceDescriptorTable,只要把我们定义的结构地址指向这个指针即可:  
extern PSYSTEMSERVICEDESCRIPTORTABLE KeServiceDescriptorTable;  
●  地址操作符只能使用变量作为操作数，不能将其用在常量前面，要取得一个地址中的内容，可以使用指针操作符\*。指针操作符的操作对象必须为地址  
●  当指针指向直接常量时, 直接常量应该用双引号而不能用单引号,因为单引号是一个 char 型的字符, 不是直接常量.所以必须取地址,赋值的时候并不是把字符串赋值给指针,而是把字符串的首地址赋给指针.  
●  指针和引用的区别:1.指针占四个字节, 引用不占 2.引用申明时必须初始化, 指针不用  
/\*  
  \*p++ 等价于 \*(p++);  先算出\*p, 然后 p 自加  
  \*++p 等价于 \*(++p);  先 p 自加,然后计算\*p  
  ++\*p  \*P 值自加  
\*/  
●  当多个赋值操作符存在于同一条语句中时，按照 C 标准的规定，将会按照从右到左的结合方式先执行最右边的赋值操作符，再依次向左执行其余赋值操作符  
●  当表达式不仅仅是一个变量或一个常量时，需要把表达式放在小括号内。例如：m = (int) (3.7 + 2.5)这时，程序会对 3.7 和 2.5 的和类型转换为 int 型。如果没有括号，只是：m = (int) 3.7 + 2.5  
●  数值运算中的类型转换遵从以下规则：char 型和 short 型数据，不管是 unsigned 还是 usigned，在计算中都要先无条件转换为 int 型，即使是两个 char 型或 short 型数值相加。float 型计算前会无条件转换为 double 型数值，以增加精度。其余类型的数值计算时，都要转换为数据类型级别较高的后再计算。  
●  itoa(   //转换到字符串( Integer To A String)  
  int nsro, //要被转换的整数  
  char \*buffer //转换后存放的缓冲区  
  int  rudix  //以何种进制形式输出  
)  
int atoi(const char \*string) 把字符串转换到整数型  
●  3 个逻辑操作符组合使用时，可以得到多个功能一致但形式不同的逻辑表达式。使用简单的逻辑表达式来代替复杂的表达式是提高程序逻辑性的一个重要方法。下面是两组功能一样的逻辑表达式：第一组表达式如下：!(a || b)!a && !b 第二组表达式如下：!(a && b)!a || !b 使用以上功能相同的两组逻辑表达式可以对一些复杂的、逻辑不清的表达式进行简化。  
●  前置操作符都是一元操作符，其优先级为 2，为第二高。很容易理解将它们得结合性规定为从右到左的原因，例如：+-+a;-++b ;!sizeof(c)按结合性从右到左，等效于：+(-(+a));-(++b);!(sizeof(c));  
●  switch 语句是 C 语言中选择结构的另一个常用的实现方式，十分适用于多路选择的实现,break 语句在 switch 语句中的作用十分重要。在 switch 语句中，遇到 break 便终止执行 switch 语句，跳出本层 switch 体，继续执行后续语句,continue 语句只能使用在 for 语句、while 语句和 do-while 语句的循环体中。其功能为结束本次循环，直接开始下一次循环。goto 语句在 C 语句中可以实现无条件跳转，通过和 if 语句的有效组合也可以实现循环结构。  
●  For 语句的标准形式如下：for ( 表达式 1; 表达式 2; 表达式 3) 表达式 1 用作循环结构的初始化，一般为赋值表达式，设定循环变量及其他变量的初始值；表达式 2 负责循环条件的判断，形式与 if 语句的控制表达式类似，一般为关系表达式或逻辑表达式；表达式 3 负责改变表达式 2 中的循环变量的值，一般也为有赋值作用的表达式。for ( 初始化表达式; 循环判断表达式; 循环变量控制表达式 ))；while ( 表达式 )   
●  初始化数组元素的部分必须是数组中从第 0 个元素开始的连续序列。例如，以下初始化语句是错误的：int window\[6\] = {1, , 3, 4, 5};/\* 第 1 个元素未赋值 \*/和其他类型变量一样，数组元素也具有地址。数组在内存中是作为一个整体分配内存的，数组元素的内存地址是连续的，其差值为数组存储的数据类型的字节长度值。如果存储的是 int 型，那么数组元素的地址都相差 4；如果是 char 型，那么数组元素的地址都相差 1。比较特殊的是，数组变量的值是该数组的第 0 个元素的地址，即数组的首地址。C 语言中的二维数组可以理解为一种特殊的一维数组，可以看作数组元素为一维数组的一维数组。为了方便理解，可以把二维数组看作是一个矩阵，一维容量为行数，二维容量为列数。matrix\[i\]\[j\]即为 matrix 中第 i 行第 j 列的元素；推广到一般情况，对于：数据类型名 array\[M\]\[N\]；array\[i\]\[j\]在其内存中为第(i×N＋j)个元素。同样的，二维数组初始化也可以不指定数组容量，但是只能不指定一维数组容量（第一个数组容量），二维数组容量（第二个数组容量）必须给出。字符串常量在 C 语言中被处理为一维字符数组存储在内存中一块连续的区域内。  
●  数组的特点:  
1, 连续的内存分配  
2, 可以通过下标进行访问  
3, 数组的名字就是数组的首地址  
4, 数组不能动态改变大小  
5, 数组在中间或前端插入或删除一个元素效率低  
  
链表特点:  
1, 不连续的内存分配  
2, 不能通过下标进行访问  
3, 链表可以动态改变其长度  
4, 链表在前面或中间插入元素, 速度快  
●  函数声明也可以称为函数原型，定义了函数作为模块化编程的基本单元的接口：函数值类型对应模块出口，函数名对应模块名，参数列表对应了模块入口。函数返回值类型也称为函数值类型，是由函数带回的值的类型。与变量一样，函数的声明和定义也是有严格区别的。函数声明确定了一个函数的接口，告诉编译器该函数的函数名、函数值类型以及形参列表中形参的个数和顺序；而函数定义则确立了一个函数的功能，不仅仅包含了函数声明所有的信息，还包含了形参的名字和函数体。关键字 extern 和 static 可以用来声明函数，使用 extern 声明的函数为外部函数.C 语言中规定，所有函数默认声明为 extern 类别，即外部函数。也就是说在之前的所有例子中定义的函数都是外部函数。  
●  C 语言中定义了 4 个关键字作为变量的存储类别的修饰词，分别为：auto、static、register 和 extern。变量的存储类别决定了变量在内存中的存储区域。1. 栈是由编译器管理的动态存储区域，用于存储临时变量，即只在需要时才被分配内存，不需要时编译器会自动回收。可存放的数据包括以下几项。函数形参：其只在函数执行期内有效。局部变量（不包括 static 修饰的局部变量）：只在它定义的程序块及其下层程序块的执行期内有效。其他临时变量（例如，a++语句中产生的临时变量）：函数返回时产生的临时变量。2. 堆是由程序管理的动态存储区域，用于分配由程序使用 malloc 函数申请的内存空间，需要由程序自行释放。在内存管理一章中将讨论这块存储区域的使用。堆上分配的内存也是未经初始化的，需要程序显式地初始化。3. 静态存储区用于存储全局变量，该区域的内存在程序开始时就已固定分配完毕，直到程序结束由编译器自动释放，在该区域分配的内存在整个程序执行过程中都是有效的。全局变量全部存放在静态存储区中。静态存储区的内存分配时由编译器自动初始化。auto 变量的内存由编译器自动从栈上分配，因此 auto 变量都是临时变量。使用 auto 声明变量的形式十分简单，如下所示。auto 数据类型 变量名;或数据类型 auto 变量名,所有的局部变量的声明中如果不含存储类别，那么都默认为 auto 型变量。C 语言中提供了一种存储类别 register，这种存储类别的变量的值会被要求直接存储在寄存器中。访问该变量无需从内存中获取它的值，存储该变量时也无需再存回内存，都直接在寄存器上进行操作。由于寄存器的存取速度要远快于内存的存取速度，因此，如果一个该变量在程序中被频繁使用，那么将其声明为 register 变量，将大大提高程序的执行效率。register 型的变量声明如下所示。register 数据类型名 变量名;register 只能修饰函数内的变量，包括局部变量和形式参数 C 程序中的所有变量都有一定的生存期和作用域。生存期是指程序运行时，变量占有内存的时间。变量作用域是指在程序中，变量可以被使用的有效代码区域。为了便于讨论变量的作用域，按程序块间的关系将程序块分为 4 类：本层程序块、上层程序块、下层程序块和外部程序块。如果局部变量没有被显式地初始化，编译器不会自动为其清理内存；如果全局变量没有为其显式初始化，编译器则会自动初始化，将其内存空间清除归零。  
●  new 是一个运算符, 它返回一个分配在堆空间的内存地址,所以 = 左边必须是指针类型  
●  直接从空间的地址获取该内存内容的访问方式叫做内存的“直接访问”。先从其他内存空间获得要访问的内存空间的地址，再根据该地址访问目的空间的方法就是内存的“间接访问”。  
●  在操作系统中，数值一律用补码来存储。一个数值的二进制值可以称其为原码，存储时会将原码表示为补码。补码的最高位为符号位，数值的补码表示可以分为以下两种情况:1．非负数的补码,非负数的补码与原码相同。例如，11 的原码为 00001011，其补码也为 00001011。2．负数的补码负数的补码的符号位为 1，其余位为将该数绝对值的原码按位取反后再加 1 的结果。例如，-15 的补码：因为是负数，则符号位为“1”，整个为 10001111；其余 7 位为-15 的绝对值的原码按位取反，即 0001111 取反后为 1110000，再加 1 为 1110001；加上符号位，最后－15 的补码是 11110001。已知一个数的补码，其求原码的过程与已知原码求补码的过程完全一样：如果补码的符号位为“0”，表示是一个正数，所以补码就是该数的原码。如果补码的符号位为“1”，表示是一个负数，求原码的操作可以是：符号位为 1，其余各位取反加 1。使用补码计算时，可以将符号位和其他位统一处理。注意：两个用补码表示的数相加时，如果最高位（符号位）有进位，则进位被舍弃。减法可按加负数法来处理。C 语言共提供了 6 个位运算操作符，包括取反操作符（~）、位或操作符（|）、位与操作符（&）、异或操作符（^）和位移操作符（>>和<<）取反操作符是一个一元操作符，其使用形式为：~操作数;位或操作符是一个二元操作符，形式如下：数 1 | 数 2 两个位值进行位或运算的规则是：只要有一个数值为 1，位或的结果便为 1；如果都为 0，位或的结果为 0。位与操作符也是一个二元操作符，形式如下：数 1 & 数 2 这两个数可以是常量，也可以是变量。位与操作符将两个操作数逐位进行位与运算。位与运算的规则是：只要有一个数值为 0，位与的结果便为 0；如果都为 1，位与的结果为 1。异或操作符是多元操作符，使用形式如下：数 1 ^ 数 2 表达式将数 1 和数 2 进行异或运算，如果两个数一样，则值为 0；如果不一样，值为 1。右移操作符的作用是将一个数的各二进制值全部右移若干位。其使用形式如下：数值 1 >> 数值 2;该表达式将数值 1 的二进制值右移（数值 2）位，移到右端的低位被舍弃，而高位补入的数值由符号位决定。如果是无符号数，则高位补入的数为 0。左移操作符将一个数的各个二进制位值全部左移若干位。使用形式如下：数值 1 << 数值 2;该表达式将数值 1 的二进制值左移（数值 2）位，移到左端的高位被舍弃，而低位补入的数值为 0。将 a 左移 1 位，即等于将 a 乘以 2；将 a 左移 n 位，即等于将 a 乘以 2 的 n 次方。  
●  由于结构体数据类型的名字由标识符和结构体名两部分组成，书写起来名字较长，因此常常使用 typedef 来简化其数据类型名字，如下所示。一般的系统中，为了寻址的方便，数据在内存中存储时一般以其本身数据类型的字节长度为基本单位对齐。例如，int 型在内存中是以 sizeof(int)个字节对齐的，char 型在内存中是以 sizeof(char)个字节对齐的。而结构体数据在存储时是以其中字节长度最大的成员的字节数为基本单位对齐的。所谓对齐，是指将内存以一个固定的字节数作为最小单位分块，分配内存时只能一块一块地分配。例如，以 4 字节对齐，就是将整个内存 4 个字节 4 个字节地分块，分配内存时只能 4 个字节 4 个字节地分。可以使用#pragma pack 预处理命令来改变对齐规则。由于共用体实际上只有一个有效成员，因此无法像初始化结构体那样使用多个值的序列对共用体进行初始化，只能使用含一个值的序列对其初始化，由于共用体中的所有成员共享一块空间，因此对任意成员的赋值都会影响其他成员的值。由于宏函数体只能是与宏函数同一行的后面的内容，因此当函数体内容较多时，如果都写在一行中，会影响程序的可读性。为使程序逻辑清晰，可以使用分行符“\\”将宏函数体分拆为多行。如果一个宏函数需要返回一个值，即可能作为一个语句的子表达式，应该将整个宏函数体放在一个括号内；如果宏函数只执行一些操作，不返回值，则应该将整个函数体放在花括号内，形成单独一个程序块。  
●  结构体和类唯一区别, 结构体成员默认情况下是公有的(public),C 语言中结构体不能写函数  
●  一般情况下，C 程序的所有代码都会被编译器编译。C 语言提供了条件编译使程序员能设定需要被编译的代码。条件编译通过条件判断的方法来实现编译代码的选择。当满足一些条件时，编译才会编译其指定的代码。本节将讨论几种条件编译命令的使用。#ifdef 命令的形式有三种，如下所示：1．形式一#ifdef 标识符程序段 1#else 程序段 2#endif。#ifndef 命令和#ifdef 命令刚好相反。该形式的#ifndef 命令实现了程序段 1 和程序段 2 的选择编译功能，即：如果程序未宏定义标识符，则编译程序段 1；如果该标识符已被宏定义，则对程序段 2 进行编译。  
  
2.  ●  ●  ●  ●  Windows 程序设计●  ●  ●  ●    
  
●  表头文件  
WINDOWS.H 是主要的引入文件，它包含了其它 Windows 表头文件，这些表头文件的某些也包含了其它表头文件。这些表头文件中最重要的和最基本的是：  
WINDEF.H    基本型态定义。  
WINNT.H     支持 Unicode 的型态定义。  
WINBASE.H   系统核心函数。  
WINUSER.H   用户接口函数。  
WINGDI.H    图形设备接口函数。  
int WINAPI WinMain(  
  HINSTANCE hInstance,  // 当前应用程序实例句柄,由系统提供  
  HINSTANCE hPrevInstance,  // 上一个实例句柄 ,CreateMutex 可创建实例互斥体  
  LPSTR lpCmdLine,      // cmd 命令行,可以使用 GetCommandLine 获得完整的命令行参数  
  int nCmdShow          // 窗口显示方式,以 SW\_XXXX 开始,例:SW\_NORMAL 默认显示方式  
);  
  WinMain 中第三个参数在 WINBASE.H 中定义为 LPSTR，我将它改为 PSTR。这两种数据型态都定义在 WINNT.H 中，作为指向字符串的指针。LP 前缀代表「长指针」，这是 16 位 Windows 下的产物。在 WinMain 声明中改变了两个参数的名称。许 Windows 程序中的变量名使用一种称作「匈牙利表示法」的命名系统，该系统在变量名称前面增加了表示变量数据型态的短前缀，现在仅需记住前缀 i 表示 int、sz 表示 String of Zero「以零结束的字符串」。  
\-----------------------------------------------------------------------------------------------  
●  //-Unicode 的简介  
\*\*\*宽字符和 C 语言  
C 中的宽字符基于 wchar\_t 数据型态，它在几个表头文件包括 WCHAR.H 中都有定义，像这样：typedef unsigned short wchar\_t ;因此，wchar\_t 数据型态与无符号短整数型态相同，都是 16 位宽。  
 strlen 接受的是 char 类型字符,如果接受 Unicode 字符,编译器会把每个字符都分配为两个字节,如果第一个字符为字母,则被分配为这个字母的 ASCII 字符和 0，如果此时再次用 strlen()命令将返回为 1\[第二个字符为 0 则结束\]，并不会返回定义的 Unicode 字符的长度.strlen 函数的宽字符版是 wcslen（wide-character string length：宽字符串长度），而 wcslen 函数则说明如下：size\_t \_\_cdecl wcslen (const wchar\_t \*) ;size\_t 在 stddef.h 中定义:typedef unsigned int size\_t;TCHAR.H 还用一个新的数据型态 TCHAR 来解决两种字符数据型态的问题.  
if ANSI:typedef char CHAR ;          
if UNICODE:typedef wchar\_t WCHAR ;   
\---------------------------------------------------------------------------  
#ifdef ANSI  
typedef CHAR \* PCHAR, \* LPCH, \* PCH, \* NPSTR, \* LPSTR, \* PSTR ;          
typedef CONST CHAR \* LPCCH, \* PCCH, \* LPCSTR, \* PCSTR ;     ANSI 字符定义,8\_bit  
#endif  
前缀 N 和 L 表示「near」和「long」，指的是 16 位 Windows 中两种大小不同的指标。在 Win32 中 near 和 long 指标没有区别  
\------------------------------------------------------------------------------------  
#IFDEF UNICODE  
typedef WCHAR \* PWCHAR, \* LPWCH, \* PWCH, \* NWPSTR, \* LPWSTR, \* PWSTR ;          
typedef CONST WCHAR \* LPCWCH, \* PCWCH, \* LPCWSTR, \* PCWSTR ;   UNCODE 字符定义 16\_bit  
#ENDIF  
\-----------------------------------------------------------------------------------------------  
●  //-窗口和消息  
\*\*\*Windows-Hello World 函数  
HICON LoadIcon(HINSTANCE hInstance,LPCTSTR lpIconName) 加载图标供程序使用。  
HCURSOR LoadCursor(HINSTANCE hInstance,LPCTSTR lpCursorName)  加载鼠标光标供程序使用。  
HGDIOBJ GetStockObject(int fnObject) 取得一个图形对象（在这个例子中，是取得绘制窗口背景的画刷对象）。  
ATOM RegisterClass (CONST WNDCLASS \*lpWndClass)为程序窗口注册窗口类别。  
int MessageBox(HWND,LPCTSTR,LPCTSTR,UINT) 显示消息框。  
HWND CreateWindow(   //根据窗口类别建立一个窗口。  
  LPCTSTR lpClassName,  // 窗口类名  
  LPCTSTR lpWindowName, // 窗口标题  
  DWORD dwStyle,        // 窗口风格  
  int x,                // X 坐标  
  int y,                // Y 坐标  
  int nWidth,           // 宽度  
  int nHeight,          // 高度  
  HWND hWndParent,      // 父窗口句柄  
  HMENU hMenu,          // 菜单句柄  
  HANDLE hInstance,     // 单签实例句柄  
  LPVOID lpParam        // 指向 CREATESTRUCT 结构  
);  
ShowWindow 在屏幕上显示窗口,第二个参数是作为参数传给 WinMain 的 iCmdShow。它确定最初如何在屏幕上显示窗口，是一般大小、最小化还是最大化。  
UpdateWindow 指示窗口自我更新。  
GetMessage 从消息队列中取得消息,第二、第三和第四个参数设定为 NULL 或者 0，表示程序接收它自己建立的所有窗口的所有消息。  
TranslateMessage 转译某些键盘消息。  
DispatchMessage 将消息发送给窗口消息处理程序。　   
PlaySound 播放一个声音文件。  
BeginPaint 开始绘制窗口。  
GetClientRect 取得窗口显示区域的大小。　   
DrawText 显示字符串。　   
EndPaint 结束绘制窗口。  
PostQuitMessage 在消息队列中插入一个「退出程序」消息。  
DefWindowProc 执行内定的消息处理。  
  窗口消息处理程序总是定义为如下形式：LRESULT CALLBACK WndProc (HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam);  
LRESULT 在 winuser.h 定义为:typedef LRESULT (CALLBACK\* WNDPROC)(HWND, UINT, WPARAM, LPARAM);  
这里 typedef 的用法是:把 LRESULT 作为 WNDPROC 的函数指针  
  窗口消息处理程序是命名为 WndProc 的函数。窗口消息处理程序可任意命名（只要求不和其它名字发生冲突）。一个 Windows 程序可以包含多个窗口消息处理程序。一个窗口消息处理程序总是与呼叫 RegisterClass 注册的特定窗口类别相关联  
  一般来说，Windows 程序写作者使用 switch 和 case 结构来确定窗口消息处理程序接收的是什么消息，以及如何适当地处理它。窗口消息处理程序在处理消息时，必须传回 0。窗口消息处理程序不予处理的所有消息应该被传给名为 DefWindowProc 的 Windows 函数。从 DefWindowProc 传回的值必须由窗口消息处理程序传回。  
  对 WM\_PAINT 的处理几乎总是从一个 BeginPaint 呼叫开始,而以一个 EndPaint 呼叫结束.  
\*\*\*Windows 中使用的值常数在表头文件中均有相应的标识符定义。  
CS 窗口类别样式\[ClassStyle\]  
CW 建立窗口\[CreateWindow\]  
DT 绘制文字\[DrawText\]  
IDI 图示 ID\[IDIcon\]  
IDC 游标 ID\[IDCursor\]  
MB 消息框\[MessageBox\]  
SND 声音\[Sound\]  
WM 窗口消息\[WindowMessage\]  
WS 窗口样式\[WindowStyle\]  
●  WNDCLASS\[窗口类\],表头文件 WINUSER.H 中定义  
typedef struct \_WNDCLASS {   
    UINT    style;    //显示风格,以 CS\_XXXX 形式  
    WNDPROC lpfnWndProc;  //WndProc 函数子程  
    int     cbClsExtra;  //初始化为 0  
    int     cbWndExtra;  //初始化为 0  
    HANDLE  hInstance;   //当前实例句柄  
    HICON   hIcon; //图标句柄  
    HCURSOR hCursor; //光标句柄  
    HBRUSH  hbrBackground; //背景颜色,COLOR\_XXXXX 形式  
    LPCTSTR lpszMenuName;   //菜单名称,填 NULL,则使用默认名称  
    LPCTSTR lpszClassName; //窗口类名  
} WNDCLASS;  
  其中的 lpfn 前缀代表「指向函数的指标」。（在 Win32 API 中，长指标和短指标（或者近程指标）没有区别。这只是 16 位 Windows 的遗物。）cb 前缀代表「字节数 CountOfByte」而且通常作为一个常数来表示一个字节的大小。h 前缀是一个句柄，而 hbr 前缀代表「一个画刷的代号」。lpsz 前缀代表「指向以 0 结尾字符串的指针」。  
●  普通的迭代窗口:各窗口左上角的垂直和水平距离在屏幕上按一定的大小递增  
●  消息类型  
typedef struct tagMSG {     // 在 WinUser.h 中定义  
    HWND   hwnd;     //接受消息的窗口句柄  
    UINT   message;  //消息类型,一般以 WM\_XXXX 形式  
    WPARAM wParam;   //消息参数值,据消息而定  
    LPARAM lParam;   //消息参数值,据消息而定  
    DWORD  time;     //消息放入消息队列中的时间  
    POINT  pt;       //消息发布时,光标的坐标  
} MSG;  
●  //-输出文字  
要得到窗口显示区域的设备内容句柄，可以呼叫 HDC GetDC(HWND hWnd)来取得句柄，在使用完后呼叫 ReleaseDC  
GetSystemMetrics 函数以取使用者接口上各类视觉组件大小的信息,取得与系统有关的度量信息  
BOOL GetTextMetrics(   //取得字体大小  
  HDC hdc,            // 指向设备上下文句柄  
  LPTEXTMETRIC lptm   // 指向 TEXTMETRIC 结构  
);  
TEXTMETRIC 结构定义如下:  
typedef struct tagTEXTMETRIC { // tm   
    LONG tmHeight;   
    LONG tmAscent;   
    LONG tmDescent;   
    LONG tmInternalLeading;   
    LONG tmExternalLeading;   
    LONG tmAveCharWidth;   
    LONG tmMaxCharWidth;   
    LONG tmWeight;   
    LONG tmOverhang;   
    LONG tmDigitizedAspectX;   
    LONG tmDigitizedAspectY;   
    BCHAR tmFirstChar;   
    BCHAR tmLastChar;   
    BCHAR tmDefaultChar;   
    BCHAR tmBreakChar;   
    BYTE tmItalic;   
    BYTE tmUnderlined;   
    BYTE tmStruckOut;   
    BYTE tmPitchAndFamily;   
    BYTE tmCharSet;   
} TEXTMETRIC;  
●  滚动条的范围和位置:BOOL SetScrollRange (hwnd, iBar, iMin, iMax, bRedraw) ;参数 iBar 为 SB\_VERT 或者 SB\_HORZ，iMin 和 iMax 分别是范围的最小值和最大值。如果想要 Windows 根据新范围重画滚动条，则设置 bRedraw 为 TRUE（如果在呼叫 SetScrollRange 后，呼叫了影响滚动条位置的其它函数，则应该将 bRedraw 设定为 FALSE 以避免过多的重画）。Windows 提供了类似的函数（GetScrollRange 和 GetScrollPos/ShowScrollBar）来取得滚动条的目前范围和位置以及显示滚动条.  
●  //-图形基础  
BeginPaint、GetDC 和 GetWindowDC 获得的设备内容都与视讯显示器上的某个特定窗口相关。取得设备内容句柄的另一个更通用的函数是 CreateDC：hdc = CreateDC (pszDriver, pszDevice, pszOutput, pData) ;可以通过下面的呼叫来取得整个屏幕的设备内容句柄：hdc = CreateDC (TEXT ("DISPLAY"), NULL, NULL, NULL) ;  
metafile 是一些 GDI 呼叫的集合，以二进制形式编码。您可以通过取得 metafile 设备内容来建立 metafile,hdcMeta = CreateMetaFile (pszFilename) ;  
●  //-键盘  
\*\*\*按键消息  
\----------------------------------------  
|        |  键按下       |     键释放    |  
\-----------------------------------     |  
|非系统键 | WM\_KEYDOWN    |  WM\_KEYUP    |  
\--------------------------------------- |  
|系统键   | WM\_SYSKEYDOWN |  WM\_SYSKEYUP |  
\---------------------------------------   
在四个按键消息（WM\_KEYDOWN、WM\_KEYUP、WM\_SYSKEYDOWN 和 WM\_SYSKEYUP）中，wParam 消息参数含有上面所讨论的虚拟键码，而 lParam 消息参数则含有对了解按键非常有用的其它信息。lParam 的 32 位分为 6 个字段,0-15 位记录重复计数,16-23 位指定 OEM 扫描码(键盘硬件产生的代码),第 24 为保存扩充键旗标,第 29 位保存内容代码,第 30 位保存此按键的先前状态,第 31 为保存转换状态.  
  
重复计数:重复计数是该消息所表示的按键次数，大多数情况下，重复计数设定为 1。不过，如果按下一个键之后，您的窗口消息处理程序不够快，以致不能处理自动重复速率（您可以在「控制台」的「键盘」中进行设定）下的按键消息，Windows 就把几个 WM\_KEYDOWN 或者 WM\_SYSKEYDOWN 消息组合到单个消息中，并相应地增加重复计数。WM\_KEYUP 或 WM\_SYSKEYUP 消息的重复计数总是为 1。因为重复计数大于 1 指示按键速率大于您程序的处理能力，所以您也可能想在处理键盘消息时忽略重复计数。几乎每个人都有文书处理或执行电子表格时画面卷过头的经验，因为多余的按键堆满了键盘缓冲区，所以当程序用一些时间来处理每一次按键时，如果忽略您程序中的重复计数，就能够解决此问题。不过，有时可能也会用到重复计数，您应该尝试使用两种方法执行程序，并从中找出一种较好的方法。  
  
OEM 扫描码:OEM 扫描码是由硬件（键盘）产生的代码。这对中古时代的汇编程序写作者来说应该很熟悉，它是从 PC 相容机种的 ROM BIOS 服务中所获得的值（OEM 指的是 PC 的原始设备制造商（Original Equipment Manufacturer）及其与「IBM 标准」同步的内容）。在此我们不需要更多的信息。除非需要依赖实际键盘布局的样貌，不然 Windows 程序可以忽略掉几乎所有的 OEM 扫描码信息  
  
扩充键旗标:如果按键结果来自 IBM 增强键盘的附加键之一，那么扩充键旗标为 1（IBM 增强型键盘有 101 或 102 个键。功能键在键盘顶端，光标移动键从数字键盘中分离出来，但在数字键盘上还保留有光标移动键的功能）。对键盘右端的 Alt 和 Ctrl 键，以及不是数字键盘那部分的光标移动键（包括 Insert 和 Delete 键）、数字键盘上的斜线（／）和 Enter 键以及 Num Lock 键等，此旗标均被设定为 1。Windows 程序通常忽略扩充键旗标。  
  
内容代码:右按键时，假如同时压下 ALT 键，那么内容代码为 1。对 WM\_SYSKEYUP 与 WM\_SYSKEYDOWN 而言，此位总视为 1；而对 WM\_SYSKEYUP 与 WM\_KEYDOW 消息而言，此位为 0。除了两个之外：如果活动窗口最小化了，则它没有输入焦点。这时候所有的按键都会产生 WM\_SYSKEYUP 和 WM\_SYSKEYDOWN 消息。如果 Alt 键未被按下，则内容代码字段被设定为 0。Windows 使用 WM\_SYSKEYUP 和 WM\_SYSKEYDOWN 消息，从而使最小化了的活动窗口不处理这些按键。对于一些外国语文（非英文）键盘，有些字符是通过 Shift、Ctrl 或者 Alt 键与其它键相组合而产生的。这时内容代码为 1，但是此消息并非系统按键消息。  
  
键的先前状态:如果在此之前键是释放的，则键的先前状态为 0，否则为 1。对 WM\_KEYUP 或者 WM\_SYSKEYUP 消息，它总是设定为 1；但是对 WM\_KEYDOWN 或者 WM\_SYSKEYDOWN 消息，此位可以为 0，也可以为 1。如果为 1，则表示该键是自动重复功能所产生的第二个或者后续消息。  
  
转换状态:如果键正被按下，则转换状态为 0；如果键正被释放，则转换状态为 1。对 WM\_KEYDOWN 或者 WM\_SYSKEYDOWN 消息，此字段为 0；对 WM\_KEYUP 或者 WM\_SYSKEYUP 消息，此字段为 1。  
\*\*\*虚拟键码  
虚拟键码保存在 WM\_KEYDOWN、WM\_KEYUP、WM\_SYSKEYDOWN 和 WM\_SYSKEYUP 消息的 wParam 参数中,此代码标识按下或释放的键,以 VK\_XXX 形式存定义在 WinUser.h 中,其中 VK\_RETURN 为 Enter 键,VK\_MENU 为 Alt 键,VK\_PRIOR 为 Page Up 键,VK\_NEXT 为 Page Down 键,VK\_SNAPSHOT 为 Print Screen 键,VK\_CLEAR 为 Num Lock 关闭时的数字键盘 5  
\*\*\*位移状态  
在处理按键消息时，您可能需要知道是否按下了位移键（Shift、Ctrl 和 Alt）或开关键（Caps Lock、Num Lock 和 Scroll Lock）。通过呼叫 GetKeyState 函数，您就能获得此信息。例如：iState = GetKeyState (VK\_SHIFT) ;如果按下了 Shift，则 iState 值为负（即设定了最高位置位）,SHORT GetKeyState(int nVirtKey)若返回 1，则表示此按键被按下,若是其他,则是没有被按下  
SHORT GetAsyncKeyState(  
  int vKey   // virtual-key code  
);//判断键盘上任何按键的状态  
BOOL GetKeyboardState(  
  PBYTE lpKeyState   // 指定 256 字节数组接受键状态  
);  //获取全部按键状态  
\*\*\*字符消息  
\---------------------------------------------  
|         |     字符       |     死字符      |  
\--------------------------------------------|  
|非系统字符|    WM\_CHAR    |  WM\_DEADCHAR    |  
\------------------------------------------  |  
|系统字符  |  WM\_SYSCHAR   |  WM\_SYSDEADCHAR |  
\--------------------------------------- ----  
参数 wParam 是 ANSI 或 Unicode 字符代码,lParam 参数与按键消息的 lParam 参数相同  
int GetKeyNameText(  
  LONG lParam,      // 消息的第二个参数  
  LPTSTR lpString,  // 指定一个缓冲区,存放字符串  
  int nSize         // 键名字的最大长度  
);  
\*\*\*消息顺序  
因为 TranslateMessage 函数从 WM\_KEYDOWN 和 WM\_SYSKEYDOWN 消息产生了字符消息，所以字符消息是夹在按键消息之间传递给窗口消息处理程序的,WM\_KEYDOWN->WM\_CHAR->WM\_KEYUP.如果您按下 Shift 键，再按下 A 键，然后释放 A 键，再释放 Shift 键，就会输入大写的 A，而窗口消息处理程序会接收到五个消息，依次:WM\_KEYDOWN\[虚拟键码 VK\_SHIFT （0x10）\]->WM\_KEYDOWN\[「A」的虚拟键码（0x41）\]->WM\_CHAR\[「A」的字符代码（0x41）\]->WM\_KEYUP\[「A」的虚拟键码（0x41）\]->WM\_KEYUP\[虚拟键码 VK\_SHIFT（0x10）\],Shift 键本身不产生字符消息。  
\*\*\*处理控制字符  
处理按键和字符消息的基本规则是：如果需要读取输入到窗口的键盘字符，那么您可以处理 WM\_CHAR 消息。如果需要读取光标键、功能键、Delete、Insert、Shift、Ctrl 以及 Alt 键，那么您可以处理 WM\_KEYDOWN 消息。  
\*\*\*死字符消息  
在某些非 U.S.英语键盘上，有些键用于给字母加上音调。因为它们本身不产生字符，所以称之为「死键」。例如，使用德语键盘时，对于 U.S.键盘上的+/=键，德语键盘的对应位置就是一个死键，未按下 Shift 键时它用于标识锐音，按下 Shift 键时则用于标识抑音.  
\*\*\*插入符号函数  
CreateCaret　建立与窗口有关的插入符  
SetCaretPos　在窗口中设定插入符的位置  
ShowCaret　显示插入符  
HideCaret　隐藏插入符  
DestroyCaret　撤消插入符  
GetCaretPos  取得插入符目前位置  
GetCaretBlinkTime  取得插入符闪烁时间 //Blink 闪烁  
SetCaretBlinkTime  设定插入符闪烁时间  
\------------------------------------------------------------------------------------------------  
●  //-鼠标  
\*\*\*鼠标基础  
理论上，可以用 GetSystemMetrics 函数来确认鼠标是否存在：fMouse = GetSystemMetrics (SM\_MOUSEPRESENT) ;如果已经安装了鼠标，fMouse 将传回 TRUE（非 0）；如果没有安装，则传回 0。然而，在 Windows 98 中，不论鼠标是否安装，此函数都将传回 TRUE 。在 Microsoft Windows NT 中，它可以正常工作。要确定所安装鼠标其上按键的个数，可使用 cButtons = GetSystemMetrics (SM\_CMOUSEBUTTONS) ; 如果没有安装鼠标，那么函数将传回 0。  
对于所有鼠标消息,wParam 的值指示鼠标按键以及 Shift 和 Ctrl 键的状态,lParam 值均含有鼠标的位置：低字组为 x 坐标，高字组为 y 坐标，这两个坐标是相对于窗口显示区域左上角的位置。  
\*\*\*非显示区域鼠标消息  
可以用两个 Windows 函数将屏幕坐标转换为显示区域坐标或者反之：ScreenToClient (hwnd, &pt);ClientToScreen (hwnd, &pt) ;  
ShowCursor (TRUE) 显示光标 同理的↓  
HideCuesor()  隐藏光标  
\*\*\*拦截鼠标  
只要呼叫：SetCapture (hwnd) ;在这个函数呼叫之后，Windows 将所有鼠标消息发给窗口句柄为 hwnd 的窗口消息处理程序。之后收到鼠标消息都是以显示区域消息的型态出现，即使鼠标正在窗口的非显示区域。lParam 参数将指示鼠标在显示区域坐标中的位置。不过，当鼠标位于显示区域的左边或者上方时，这些 x 和 y 坐标可以是负的。当您想释放鼠标时，呼叫：ReleaseCapture () ;  
\------------------------------------------------------------------------------------------------  
●  //-定时器  
  呼叫 SetTimer(HWND hWnd , UINT nIDEvent, UINT uElapse, TIMERPROC lpTimerFunc),第一个参数是其窗口消息处理程序将接收 WM\_TIMER 消息的窗口句柄。第二个参数是定时器 ID，它是一个非 0 数值，在整个例子中假定为 1。第三个参数是一个 32 位无正负号整数，以毫秒为单位指定一个时间间隔，一个 60,000 的值将使 Windows 每分钟发送一次 WM\_TIMER 消息。函数为您的 Windows 程序分配一个定时器。SetTimer 有一个时间间隔范围为 1 毫秒 4,294,967,295 毫秒（将近 50 天）的整数型态参数，这个值指示 Windows 每隔多久时间给您的程序发送 WM\_TIMER 消息。KillTimer(HWND,nIDEvent),撤销定时器,第一个参数为窗口的句柄,第二个为时钟的 ID.  
\*\*\*时间日期结构 SYSTEMTIME  
typedef struct \_SYSTEMTIME {  
    WORD wYear;  //年  
    WORD wMonth; //月  
    WORD wDayOfWeek; //星期,从 0 开始  
    WORD wDay;  //天  
    WORD wHour; //小时  
    WORD wMinute;  //分钟  
    WORD wSecond;  //秒  
    WORD wMilliseconds; //微秒  
} SYSTEMTIME;  
SYSTEMTIME 主要用于 GetLocalTime 和 GetSystemTime 函数。GetSystemTime 函数传回目前的世界时间（Coordinated Universal Time，UTC），大概与英国格林威治时间相同。GetLocalTime 函数传回当地时间以及 GetCurrentTime 返回当前时间，依据计算机所在的时区。这些值的精确度完全决定于使用者所调整的时间精确度以及是否指定了正确的时区。  
\------------------------------------------------------------------------------------------------  
●  //-子窗口控件  
GetWindowLong 该函数获得有关指定窗口的信息，函数也获得在额外窗口内存中指定偏移位地址的 32 位度整型值。   
LONG GetWindowLong(  
  HWND hWnd,  // 窗口句柄  
  int nIndex  // 欲取回的信息,以 GWL\_XXX 或者 DWL\_XXX 形式,在 WinUser.h 中,有以下数值  
                 GWL=GetWindowLong    DWL=  
　             　GWL\_EXSTYLE= (-20) 扩展窗口样式 　　  
                 GWL\_STYLE=(-16) 窗口样式 　　  
                 GWL\_WNDPROC= (-4) 该窗口的窗口函数的地址 　　  
                 GWL\_HINSTANCE= (-6) 拥有窗口的实例的句柄 　　  
                 GWL\_HWNDPARENT= (-8) 该窗口之父的句柄。不要用 SetWindowWord 来改变这个值 　　  
                 GWL\_ID= (-12) 对话框中一个子窗口的标识符 　　  
                 GWL\_USERDATA = (-21) 含义由应用程序规定 　　  
                 DWL\_DLGPROC = 4 这个窗口的对话框函数地址 　　  
                 DWL\_MSGRESULT = 0 在对话框函数中处理的一条消息返回的值 　　  
                 DWL\_USER = 8 含义由应用程序规定   
);  
  
BOOL IsWindowVisible(  //窗口是否可见  
  HWND hWnd   // handle to window  
);  
HBRUSH CreateSolidBrush( //创建一个具有指定颜色的逻辑刷子  
  COLORREF crColor   // brush color value Specifies the color of the brush. To create a COLORREF color value, use the RGB macro  
);  
\------------------------------------------------------------------------------------------------  
●  //-菜单及其他资源  
HMENU GetSystemMenu( //得到系统菜单句柄  
  HWND hWnd,    // handle to window  
  BOOL bRevert  // reset option 如果参数 bRevert 为 FALSE，返回值是窗口菜单的拷贝的句柄：如果参数 bRevert 为 TRUE，返回值是 NULL。   
);  
\*\*\*改变菜单  
AppendMenu  在菜单尾部添加一个新的菜单项目  
DeleteMenu  删除菜单中一个现有的菜单项并清除该项目  
InsertMenu  在菜单中插入一个新项目  
ModifyMenu  修改一个现有的菜单项目  
RemoveMenu  从菜单中移走某一项目  
\------------------------------------------------------------------------------------------------  
●  //-多任务和多线程  
建立新的线程的 API 函数是  HANDLE CreateThread(  //创建线程  
  LPSECURITY\_ATTRIBUTES lpThreadAttributes, // 指向 SECURITY\_ATTRIBUTES 结构的指针  
  DWORD dwStackSize,                        // 初始线程堆栈大小,默认 0  
  LPTHREAD\_START\_ROUTINE lpStartAddress,    // 线程函数  
  LPVOID lpParameter,                       // 指向线程的参数值  
  DWORD dwCreationFlags,                    // 额外标志,通常 0  
  \[out\]LPDWORD lpThreadId                   // 返回线程句柄  
);  
HANDLE CreateEvent( //创建事件对象  
  LPSECURITY\_ATTRIBUTES lpEventAttributes, // 指向 SECURITY\_ATTRIBUTES 结构的指针  
  BOOL bManualReset,                       // 复位方式  
  BOOL bInitialState,                      // 初始状态的事件对象  
  LPCTSTR lpName                           // 事件对象名称  
);  
要设立一个现存的事件对象，呼叫 SetEvent (hEvent) ;要重置一个事件对象，呼叫 ResetEvent (hEvent) ;一个程序通常呼叫：DWORD WaitForSingleObject(  
  HANDLE hHandle,        // 事件句柄  
  DWORD dwMilliseconds   // 时间间隔  
);  
\------------------------------------------------------------------------------------------------  
●  //动态链接库  
  尽管一个动态链接库模块可能有其它扩展名（如.EXE 或.FON），但标准扩展名是.DLL。只有带.DLL 扩展名的动态链接库才能被 Windows 自动加载。如果文件有其它扩展名，则程序必须另外使用 LoadLibrary 或者 LoadLibraryEx 函数加载该模块.动态链接库有着令人困惑的印象，部分原因是由于「链接库」这个词被放在几种不同的用语之后。除了动态链接库之外，我们也用它来称呼「目的码链接库」或「引用链接库」。目的码链接库是带.LIB 扩展名的文件。在使用连结程序进行静态连结时，它的程序代码就会加到程序的.EXE 文件中。  
 DLL 入口函数:int WINAPI DllMain (   //函数入口  
     HINSTANCE hInstance,  //模块实例句柄  
     DWORD fdwReason,      //呼叫函数的理由  
     PVOID pvReserved      //DLL 的初始化  
);  
HMODULE LoadLibrary(      //加载函数库  
     LPCTSTR lpFileName   // 文件名,返回一个模块实例句柄  
);  
BOOL FreeLibrary(        //卸载函数库  
     HMODULE hModule   // 模块句柄,逻辑型  
);  
HMODULE GetModuleHandle(   //同样返回一个模块的实例句柄  
  LPCTSTR lpModuleName   // 模块名字  
);  
  
  
  
3. ●  ●●●●●●汇编语言程序设计●●●●  
 ●  80386 处理器有 3 种工作模式：实模式、保护模式和虚拟 86 模式。实模式和虚拟 86 模式是为了和 8086 处理器兼容而设置的。在实模式下，80386 处理器就相当于一个快速的 8086 处理器。保护模式是 80386 处理器的主要工作模式。在此方式下，80386 可以寻址 4 GB 的地址空间，同时，保护模式提供了 80386 先进的多任务、内存分页管理和优先级保护等机制。80386 处理器被复位或加电的时候以实模式启动。虚拟 86 模式以保护模式为基础，它的工作方式实际上是实模式和保护模式的混合。从实模式切换到保护模式是通过修改控制寄存器 CR0 的控制位 PE（位 0）来实现的 PE 为 1 则进入保护模式,分页机制是通过 PG 位来实现 PG 为 1 则启用分页,只有在 PE 位是 1 的时候,PG 位才有可能是 1,分页机制把线性地址空间和物理地址空间分别划分为大小相同的块。这样的块称为页。通过在线性地址空间的页与物理地址空间的页之间建立映射,分页机制可以实现线性地址到物理地址的转换。  
●  保护模式下的分段模式:  
 在 32 位机器中,CPU 中 CS,DS,ES,SS,4 个 16 位的段寄存器存放的的是选择符,共 16\*4=64 位  
 各项任务共享的内存空间由全局选择符索引,而某个独立使用的内存空间由局部选择符来索引,以选择符的高 13 位左右偏移量,再以 CPU 内部事先初始好的 GDTR(全局描述符表寄存器)中的 32 位作为基地址  
,可以获得相应的描述符,由描述符中的线性地址决定段的基地址,再利用指令(或其他方式)给出的偏移量,便可得到线性地址.物理地址=线性基地址+偏移量  
4KB 页面的线性地址,如何映射到物理内存:  
  4KB=1024 个页映射表                  (每个映射表 4 字节)  
  4 字节=4\*8 位=32 位线性地址  
  线性地址的高 10 位->(物理页中的页目录表)1024 个页目录项 (每个页目录项 4 字节\*1024=4096 字节=4KB)  
  线性地址的低 22 位->高 10 位+低 12 位  
  高 10 位->(物理页中页表)1024 个页表项   (每个页表项 4 字节\*1024=4096 字节=4KB)  
  低 12 位->存放诸如“页是否存在于内存”或“页的权限”等信息  
举例说:00402080，首先转换为 32 位地址=0000 0000 0100 0000 0010 0000 1000 0000,那么高 10 位 0010000000，这个是页目录,系统根据 CR3 寄存器显示的页目录表的基址,找到页目录,然后找索引为 10000000 的目录,每个页目录对应 1024 个页表,找到后,在根据地址中间 10 位 1000,从页表中找到索引为 1000 的地址,这样就得到了目标页面的物理地址,最后加上低 12 位 1，就得到数据存放的真正物理地址  
一个线性地址大小为 4 个字节(32bit),包含着找到物理地址的信息,分为 3 个部分:第 22 位到第 31 位这 10 位(最高 10 位)是页目录中的索引,第 12 位到第 21 位这 10 位是页表中的索引,第 0 位到第 11 位这 12 位(低 12 位)是页内偏移。在把一个线性地址转换成物理地址时,CPU 首先根据 CR3 中的值,找到页目录所在的物理页。然后根据线性地址的第 22 位到第 31 位这 10 位(最高的 10bit)的值作为索引,找到相应的 PDE,其中含有这个虚拟地址所对应页表的物理地址。有了页表的物理地址,再把虚拟地址的第 12 位到第 21 位这 10 位的值作为索引,找到该页表中相应的 PTE,其中就有这个虚拟地址所对应物理页的物理地址。最后用线性地址的最低 12 位,也就是页内偏移,加上这个物理页的物理地址,就得到了该线性地址所对应的物理地址。  
控制寄存器  :CR0  
屏蔽中断   :cli  
中断返回指令:  iret  
●  实模式下，一个完整的地址由段地址和偏移地址两部分组成。段地址放在 16 位的段寄存器中，然后在指令中用 16 位的偏移地址寻址。处理器换算时先将段地址乘以 10h，得到段在物理内存中的起始地址；然后加上 16 位的偏移地址得到实际的物理地址。如 xxxx:yyyy 格式的虚拟地址在内存中的实际位置是 xxxx×10h＋yyyy。  
●  80386 所有的通用寄存器都是 32 位的，2 的 32 次方相当于 4G，所以用任何一个通用寄存器来间接寻址，不必分段就已经可以访问到所有的内存地址。虽然在寻址上不再有分段的限制问题，但在保护模式下，一个地址空间是否可以被写入，可以被多少优先级的代码写入，是不是允许执行等涉及保护的问题就出来了。要解决这些问题，必须对一个地址空间定义一些安全上的属性。段寄存器这时就派上了用途。但是涉及属性和保护模式下段的其他参数，要表示的信息太多了，要用 64 位长的数据才能表示。我们把这 64 位的属性数据叫做段描述符（Segment Descriptor）.  
●  80386 的段寄存器是 16 位的，无法放下保护模式下 64 位的段描述符。如何解决这个新的问题呢？解决办法是把所有段的段描述符顺序放在内存中的指定位置，组成一个段描述符表（Descriptor Table）；而段寄存器中的 16 位用来做索引信息，指定这个段的属性用段描述符表中的第几个描述符来表示。这时，段寄存器中的信息不再是段地址了，而是段选择器（Segment Selector）。  
●  80386 中引入了两个新的寄存器来管理段描述符表。一个是 48 位的全局描述符表寄存器 GDTR，一个是 16 位的局部描述符表寄存器 LDTR。GDTR 指向的描述符表为全局描述符表 GDT（Global Descriptor Table）。它包含系统中所有任务都可用的段描述符，通常包含描述操作系统所使用的代码段、数据段和堆栈段的描述符及各任务的 LDT 段等；全局描述符表只有一个。  
LDTR 则指向局部描述符表 LDT（Local Descriptor Table）。80386 处理器设计成每个任务都有一个独立的 LDT。它包含有每个任务私有的代码段、数据段和堆栈段的描述符，也包含该任务所使用的一些门描述符，如任务门和调用门描述符等。  
●  80386 中可以使用下述指令进行数组访问：mov cx,\[eax + ebx \* 2 + 数组基地址\],这相当于把数组中下标为 eax 和 ebx 的项目放入 cx 中；ebx \* 2 中的 2 可以是 1，2，4 或 8，这样就可以支持 8 位到 64 位的数组。  
●  在保护地址模式下,经常遇到三种地址:逻辑地址(ogica Address)、线性地址(inear Address)和物理地址(Physica Address)。CPU 通过分段机制将逻辑地址转换为线性地址,再通过分页机制将线性地址转换为物理地址。(1)逻辑地址这是内存地址的精确描述,通常表示为十六进制:xxxx:YYYYYYYY,这里 xxxx 为 seector(选择器),而 YYYYYYYY 是针对 seector 所选择的段地址的线性偏移量。除了指定 xxxx 的具体数值外,还可使用具体的段寄存器的名字来替代,如 CS(代码段),DS(数据段),ES(扩展段),FS(附加数据段#1),GS(附加数据段#2)和 SS(堆栈段)。这些符号都来自旧的“段:偏移量”风格,在 8086 实模式下使用此种方式来指定“far pointers”(远指针)。(2)线性地址线性地址是逻辑地址到物理地址变换之间的中间层,是处理器可寻址的内存空间(称为线性地址空间)中的地址。程序代码会产生逻辑地址,或者说是段中的偏移地址,加上相应段的基地址就生成了一个线性地址。如果启用了分页机制,那么线性地址可以再经变换以产生一个物理地址。若没有启用分页机制,那么线性地址直接就是物理地址。不过,在开启分页功能之后,一个线性地址可能没有相对映的物理地址,因为它所对应的内存可能被交换到硬盘中。32 位线性地址可用于定位 4GB 存储单元。(3)物理地址所谓物理地址,就是指系统内存的真正地址。对于 32 位的操作系统,它的范围为 0x00000000~0xFFFFFFFF,共有 4GB。只有当 CPU 工作于分页模式时,此种类型的地址才会变得非常“有趣”。  
●  80386 处理器把 4 KB 大小的一块内存当做一“页”内存，每页物理内存可以根据“页目录”和“页表”，随意映射到不同的线性地址上。页表规定的不仅是地址的映射，同时还规定了页的访问属性，如是否可写、可读和可执行等。是否启用内存分页机制是由 80386 处理器新增的 CR0 寄存器中的位 31（PG 位）决定的。如果 PG＝0，则分页机制不启用，这时所有指令寻址的地址（线性地址）就是系统中实际的物理地址；当 PG＝1 的时候，80386 处理器进入内存分页管理模式，所有的线性地址要经过页表的映射才得到最后的物理地址。在实模式下寻址的时候，段寄存器＋偏移地址经过转换计算以后得到的地址是“物理地址”，而保护模式下，段选择器＋偏移地址转换后的地址被称为“线性地址”而不是“物理地址”。  
●   每个应用程序都有自己的 4 GB 的寻址空间。该空间可存放操作系统、系统 DLL 和用户 DLL 的代码，它们之中有各种函数供应用程序调用。再除去其他的一些空间，余下的是应用程序的代码、数据和可以分配的地址空间。  
●   不同应用程序的线性地址空间是隔离的。虽然它们在物理内存中同时存在，但在某个程序所属的时间片中，其他应用程序的代码和数据没有被映射到可寻址的线性地址中，所以是不可访问的。从编程的角度看，程序可以使用 4 GB 的寻址空间，而且这个空间是“私有”的。  
●   DLL 程序没有自己“私有”的空间。它们总是被映射到其他应用程序的地址空间中，当做其他应用程的一部分运行。原因很简单，如果它不和其他程序同属一个地址空间，应用程序该如何调用它呢？  
●  并不是 Win32 汇编源代码用不到段寄存器，而是用户在使用中不必去关心段寄存器！  
●  为了使高优先级的代码能够安全地被低优先级的代码调用，保护模式下增加了“门”的概念。“门”指向某个优先级高的程序所规定的入口点，所有优先级低的程序调用优先级高的程序只能通过门重定向，进入门所规定的入口点。这样可以避免低级别的程序代码从任意位置进入优先级高的程序的问题。保护模式下的中断和异常等服务程序也要从“门”进入，80386 的门分为中断门、自陷门和任务门几种。  
●  在保护模式下要表示一个中断或异常服务程序的信息需要用 8 个字节，包括门的种类以及 xxxx:yyyyyyyy 格式的入口地址等。这组信息叫做“中断描述符”。保护模式下把所有的中断描述符放在一起组成“中断描述符表”IDT（Interrupt Descriptor Table）。  
●  实模式下的中断和异常服务程序地址存放在中断向量表中。中断向量表位于物理内存 00000h 开始的 400h 字节中，共支持 100h 个中断向量；每个中断向量是一个 xxxx:yyyy 格式的地址，占用 4 字节。当发生 n 号异常或 n 号中断，或者执行到 int n 指令的时候，CPU 首先到内存 n×4 的地方取出服务程序的地址 aaaa:bbbb；然后将标志寄存器、中断时的 CS 和 IP 压入堆栈，接着转移到 aaaa:bbbb 处执行；在服务程序最后遇到 iret 的时候，CPU 从堆栈中恢复标志寄存器，然后取出 CS 和 IP 并返回。  
●  在 Windows 中，操作系统使用动态链接库来代替中断服务程序提供系统功能，所以 Win32 汇编中 int 指令也就失去了存在的意义。这就是在 Win32 汇编源代码中看不到 int 指令的原因。其实那些调用 API 的指令原本是用 int 指令实现的。  
●  16 位的段选择器中只有高 13 位表示索引值。剩下的 3 个数据位中，第 0，1 位表示程序的当前优先级 RPL；第 2 位 TI 位用来表示在段描述符的位置；TI＝0 表示在 GDT 中，TI＝1 表示在 LDT 中。  
●   保护机制主要由下列几方面组成：  
段的类型检查段的类型是由段描述符指定的，主要属性有是否可执行，是否可读和是否可写等。而 CS，DS 和 SS 等段选择器是否能装入某种类型的段描述符是有限制的。如不可执行的段不能装入 CS；不可读的段不能装入 DS 与 ES 等数据段寄存器；不可写的段不能装入 SS 等。如果段类型检查通不过，则处理器会产生一般性保护异常或堆栈异常。  
●   页的类型检查除了可以在段级别上指定整个段是否可读写外，在页表中也可以为每个页指定是否可写。对于特权级下的执行代码，所有的页都是可写的。但对于 1，2 和 3 级的代码，还要根据页表中的 R/W 项决定是否可写，企图对只读的页进行写操作会产生页异常。  
●   访问数据时的级别检查优先级低的代码不能访问优先级高的数据段。80386 的段描述符中有一个 DPL 域（描述符优先级），表示这个段可以被访问的最低优先级。而段选择器中含有 RPL 域（请求优先级），表示当前执行代码的优先级。只有 DPL 在数值上大于或等于 RPL 值的时候，该段才是可以访问的，否则会产生一般性保护异常。  
●   控制转移的检查在处理器中，有很多指令可以实现控制转移，如 jmp，call，ret，int 和 iret 等指令。但优先级低的代码不能随意转移到优先级高的代码中，所以遇到这些指令的时候，处理器要检查转移的目的位置是否合法。  
●   指令集的检查有两类指令可以影响保护机制。第一类是改变 GDT，LDT，IDT 以及控制寄存器等关键寄存器的指令，称为特权指令；第二类是操作 I/O 端口的指令以及 cli 和 sti 等改变中断允许的指令，称为敏感指令。试想一下，如果用户级程序可以用 sti 禁止一切中断（包括时钟中断），那么整个系统就无法正常运行，所以这些指令的运行要受到限制。特权指令只能在优先级 0 上才能运行，而敏感指令取决于 eflags 寄存器中的 IOPL 位。只有 IOPL 位表示的优先级高于等于当前代码段的优先级时，指令才能执行。  
●   I/O 操作的保护 I/O 地址也是受保护的对象。因为通过 I/O 操作可以绕过系统对很多硬件进行控制。80386 可以单独为 I/O 空间提供保护，每个任务有个 TSS（任务状态段）来记录任务切换的信息。TSS 中有个 I/O 允许位图，用来表示对应的 I/O 端口是否可以操作。某个 I/O 地址在位图中的对应数据位为 0 则表示可以操作；如果为 1 则还要看 eflags 中的 IPOL 位，这时只有 IOPL 位表示的优先级高于等于当前代码段的优先级，才允许访问该 I/O 端口。  
●●●●●●●●PE 文件格式学习●●●●●●●●●  
●   在 PE 文件中，代码、已初始化的数据、资源和重定位信息等数据被按照属性分类放到不同的节（Section）中，而每个节的属性和位置等信息用一个 IMAGE\_SECTION\_HEADER 结构来描述，所有的 IMAGE\_SECTION\_HEADER 结构组成一个节表(Section Table)，节表数据在 PE 文件中被放在所有节数据的前面。  
●IMAGE\_SECTION\_HEADER 结构在 winnt.h 定义如下:  
typedef struct \_IMAGE\_SECTION\_HEADER   
{  
   BYTE Name\[IMAGE\_SIZEOF\_SHORT\_NAME\];//节表名称,如“.text”IMAGE\_SIZEOF\_SHORT\_NAME=8  
union   
 {  
    DWORD PhysicalAddress;//物理地址  
    DWORD VirtualSize;//真实长度，可以使用其中的任何一个，一般是节的数据大小  
 } Misc;  
  DWORD VirtualAddress;//相对虚拟地址  
  DWORD SizeOfRawData;//物理长度  
  DWORD PointerToRawData;//节基于文件的偏移量  
  DWORD PointerToRelocations;//重定位的偏移  
  DWORD PointerToLinenumbers;//行号表的偏移  
  WORD NumberOfRelocations;//重定位项数目  
  WORD NumberOfLinenumbers;//行号表的数目  
  DWORD Characteristics;//节属性 如可读，可写，可执行等  
} IMAGE\_SECTION\_HEADER, \*PIMAGE\_SECTION\_HEADER;   
  
●IMAGE\_IMPORT\_DESCRIPTOR 是导入表结构数组，数组以全 0 作为结束标记，该结构定义如下：  
typedef struct \_IMAGE\_IMPORT\_DESCRIPTOR  
{  
  union   
   {  
      DWORD Characteristics;  //0  
      DWORD OriginalFirstThunk;// 指向一个 IMAGE\_THUNK\_DATA 结构数组的 RVA  
}  
  DWORD TimeDateStamp;// 文件生成的时间  
  DWORD ForwarderChain;// 这个数据一般为 0，可以不关心  
  DWORD Name1;  // RVA，指向 DLL 名字的指针，ASCII 字符串  
  DWORD FirstThunk; //指向一个 IMAGE\_THUNK\_DATA 结构数组的 RVA,这个数据与 IAT 所指向的地址一致  
}IMAGE\_IMPORT\_DESCRIPTOR,\*PIMAGE\_IMPORT\_DESCRIPTOR  
●IMAGE\_EXPORT\_DIRECTORY 是导出表结构数组，数组以全 0 作为结束标记，该结构定义如下：  
typedef struct \_IMAGE\_EXPORT\_DIRECTORY {  
    DWORD   Characteristics;  
    DWORD   TimeDateStamp;  
    WORD    MajorVersion;  
    WORD    MinorVersion;  
    DWORD   Name;  
    DWORD   Base;  
    DWORD   NumberOfFunctions;  
    DWORD   NumberOfNames;  
    DWORD   AddressOfFunctions;     // RVA from base of image  
    DWORD   AddressOfNames;         // RVA from base of image  
    DWORD   AddressOfNameOrdinals;  // RVA from base of image  
} IMAGE\_EXPORT\_DIRECTORY, \*PIMAGE\_EXPORT\_DIRECTORY;  
●IMAGE\_THUNK\_DATA 这是一个 DWORD 类型的集合。通常我们将其解释为指向一个 IMAGE\_IMPORT\_BY\_NAME 结构的指针,其定义如下：  
IMAGE\_THUNK\_DATA  
{  
 union   
  {  
    PBYTE ForwarderString;  
    PDWORD Function;  
    DWORD Ordinal;//判定当前结构数据是不是以序号为输出的，如果是的话该值为 0x800000000,此时                                                                                                                                                                                      ....PIMAGE\_IMPORT\_BY\_NAME 不可做为名称使用  
PIMAGE\_IMPORT\_BY\_NAME AddressOfData;  
}u1;  
} IMAGE\_THUNK\_DATA,\*PIMAGE\_THUNK\_DATA;  
●typedef struct \_IMAGE\_IMPORT\_BY\_NAME  
{  
   WORD Hint;// 函数输出序号  
   BYTE Name1\[1\];//输出函数名称  
} IMAGE\_IMPORT\_BY\_NAME,\*PIMAGE\_IMPORT\_BY\_NAME  
  
我们知道，Win32 中可以对每个内存页分别指定可执行、可读写等属性，PE 文件将同样属性的数据分类放在一起是为了统一描述这些数据装入内存后的页面属性。由于数据是按照属性在节中放置的，不同用途但是属性相同的数据（如导入表、导出表以及.const 段指定的只读数据）可能被放在同一个节中，所以 PE 文件中还用一系列的数据目录结构 IMAGE\_DATA\_DIRECTORY 来分别指明这些数据的位置，数据目录表和其他描述文件属性的数据合在一起称为 PE 文件头，PE 文件头被放置在节和节表的前面。  
●  操作系统识别可执行文件的方法是按照文件格式而不是按照扩展名,如果文件头中的数据格式不符合任何已经定义的格式，那么系统按照 COM 文件的格式装入文件，也就是说将整个文件的数据全部当做代码装入执行。这个规则说明了为什么很多非.exe 扩展名的可执行文件（如 LE 格式的 VxD 文件、PE 格式的.dll，.scr 文件等等）也能够被装入并正确运行，也说明了为什么把可执行文件的扩展名随意修改为.exe、.com 或者.bat（甚至是.pif，.scr 或者.bat），系统也能正确识别并执行的原因。  
●  PE 文件中的 DOS 部分由 MZ 格式的文件头和可执行代码部分组成，可执行代码被称为“DOS 块”（DOS stub）。MZ 格式的文件头由 IMAGE\_DOS\_HEADER 结构定义：以下是 cmd.exe 文件解析来的  
    IMAGE\_DOS\_HEADER STRUCT  
  {  
   WORD     e\_magic     Magic DOS  signature MZ(4Dh 5Ah)   //DOS 可执行文件标记  
   WORD     e\_cblp      Bytes on last page of file       
   WORD     e\_cp        Pages in file                        
   WORD     e\_crlc      Relocations                          
   WORD     e\_cparhdr   Size of header in paragraphs         
   WORD     e\_minalloc  Minimun extra paragraphs needs       
   WORD     e\_maxalloc  Maximun extra paragraphs needs       
   WORD     e\_ss        intial(relative)SS value           //DOS 代码的初始化堆栈 SS  
   WORD     e\_sp        intial SP value                    //DOS 代码的初始化堆栈指针 SP  
   WORD     e\_csum      Checksum                             
   WORD     e\_ip        intial IP value              //DOS 代码的初始化指令入口\[指针 IP\]  
   WORD     e\_cs        intial(relative)CS value           //  DOS 代码的初始堆栈入口  
   WORD     e\_lfarlc    File Address of relocation table     
   WORD     e\_ovno      Overlay number                       
   WORD     e\_res\[4\]    Reserved words                       
   WORD     e\_oemid     OEM identifier(for e\_oeminfo)        
   WORD     e\_oeminfo   OEM information;e\_oemid specific   0000  
   WORD     e\_res2\[10\]  Reserved words                      
   DWORD    e\_lfanew    Offset to start of PE header       //指向真正 PE 文件头  
  }IMAGE\_DOS\_HEADER ENDS  
●  PE 文件头（NT 文件头）  
从 DOS 文件头的 e\_lfanew 字段（文件头偏移 003ch）得到真正的 PE 文件头位置后，现在来看看它的定义，PE 文件头是由 IMAGE\_NT\_HEADERS 结构定义的：  
●typedef struct \_IMAGE\_NT\_HEADERS {  
    DWORD Signature;  //00004550h  
    IMAGE\_FILE\_HEADER FileHeader;  //结构体  
    IMAGE\_OPTIONAL\_HEADER32 OptionalHeader;//结构体  
}  
●  PE 文件头的第一个双字是一个标志，它被定义为 00004550h，也就是字符“P”，“E”加上两个 0，这也是“PE”这个称呼的由来，大部分的文件属性由标志后面的 IMAGE\_FILE\_HEADER 和 IMAGE\_OPTIONAL\_HEADER32 结构来定义，从名称看，似乎后面的这个 PE 文件表头结构是可选的（Optional），但实际上这个名称是名不符实的，因为它总是存在于每个 PE 文件中。  
IMAGE\_FILE\_HEADER 结构的定义如下所示，字段后面的注释中标出了字段相对于 PE 文件头的偏移量，以供读者快速参考：  
●typedef struct \_IMAGE\_FILE\_HEADER {  
    WORD    Machine;   //运行平台 intel i386 一般为 14Ch  
    WORD    NumberOfSections;    //块(section)数目   
    DWORD   TimeDateStamp;    //时间日期标记  
    DWORD   PointerToSymbolTable;  //COFF 符号指针，这是程序调试信息  
    DWORD   NumberOfSymbols;  //符号数  
    WORD    SizeOfOptionalHeader;  //可选部首长度，是 IMAGE\_OPTIONAL\_HEADER 的长度  
    WORD    Characteristics;  //文件属性  
}   
●  定义 IMAGE\_OPTIONAL\_HEADER32 结构的本意在于让不同的开发者能够在 PE 文件头中使用自定义的数据，这就是结构名称中“Optional”一词的由来，但实际上 IMAGE\_FILE\_HEADER 结构不足以用来定义 PE 文件的属性，反而在这个“可选”的部分中有着更多的定义数据，对于读者来说，可以完全不必考虑这两个结构的区别在哪里，只要把它们当成是连在一起的“PE 文件头结构”就可以了。IMAGE\_OPTIONAL\_HEADER32 结构的定义如下，同样，字段后面的注释中标出了字段本身相对于 PE 文件头的偏移量：  
IMAGE\_OPTIONAL\_HEADER32 STRUCT  
typedef struct \_IMAGE\_OPTIONAL\_HEADER   
{  
    WORD    Magic;  //幻数，一般为 10BH    
    BYTE    MajorLinkerVersion;  //链接程序的主版本号  
    BYTE    MinorLinkerVersion;  //链接程序的次版本号  
    DWORD   SizeOfCode;  //代码段大小  
    DWORD   SizeOfInitializedData;  //已初始化数据块的大小  
    DWORD   SizeOfUninitializedData;  //未初始化数据库的大小  
    DWORD   AddressOfEntryPoint;  //程序开始执行的入口地址,这是一个 RVA（相对虚拟地址）  
    DWORD   BaseOfCode; //代码段的起始 RVA  
    DWORD   BaseOfData; //数据段的起始 RVA  
    DWORD   ImageBase; //可执行文件默认装入的基地址  
    DWORD   SectionAlignment; //内存中块的对齐值（默认的块对齐值为 1000H，4KB 个字节）  
    DWORD   FileAlignment;//文件中块的对齐值（默认值为 200H 字节，为了保证块总是从磁盘的扇区开始的）  
    WORD    MajorOperatingSystemVersion;//要求操作系统的最低版本号的主版本号  
    WORD    MinorOperatingSystemVersion;//要求操作系统的最低版本号的次版本号  
    WORD    MajorImageVersion;//该可执行文件的主版本号  
    WORD    MinorImageVersion;//该可执行文件的次版本号  
    WORD    MajorSubsystemVersion;//要求最低之子系统版本的主版本号  
    WORD    MinorSubsystemVersion;//要求最低之子系统版本的次版本号  
    DWORD   Win32VersionValue;//保留字  
    DWORD   SizeOfImage;//映像装入内存后的总尺寸  
    DWORD   SizeOfHeaders;//部首及块表的大小  
    DWORD   CheckSum;//CRC 检验和  
    WORD    Subsystem;//程序使用的用户接口子系统  
    WORD    DllCharacteristics;//DLLmain 函数何时被调用，默认为 0  
    DWORD   SizeOfStackReserve;//初始化时堆栈大小  
    DWORD   SizeOfStackCommit;//初始化时实际提交的堆栈大小  
    DWORD   SizeOfHeapReserve;//初始化时保留的堆大小  
    DWORD   SizeOfHeapCommit;//初始化时实际提交的对大小  
    DWORD   LoaderFlags;//与调试有关，默认为 0  
    DWORD   NumberOfRvaAndSizes;//数据目录结构的数目  
    IMAGE\_DATA\_DIRECTORY DataDirectory\[IMAGE\_NUMBEROF\_DIRECTORY\_ENTRIES\];//数据目录表  
}   
●IMAGE\_DATA\_DIRECTORY 数据目录表,结构定义如下:  
typedef struct \_IMAGE\_DATA\_DIRECTORY {  
    DWORD   VirtualAddress;//数据块的起始 RVA  
    DWORD   Size;//数据块的长度  
}  
●IMAGE\_RESOURCE\_DIRECTORY 自愿结构表,结构定义如下:  
typedef struct \_IMAGE\_RESOURCE\_DIRECTORY {  
    DWORD   Characteristics;//理论上为资源的属性，不过事实上总是 0  
    DWORD   TimeDateStamp;//资源的产生时刻  
    WORD    MajorVersion;//理论上为资源的版本，不过事实上总是 0  
    WORD    MinorVersion;  
    WORD    NumberOfNamedEntries;//以名称命名的入口数量  
    WORD    NumberOfIdEntries;//以 ID 命名的入口数量  
//  IMAGE\_RESOURCE\_DIRECTORY\_ENTRY DirectoryEntries\[\];  
} IMAGE\_RESOURCE\_DIRECTORY, \*PIMAGE\_RESOURCE\_DIRECTORY;  
●IMAGE\_BASE\_RELOCATION 重定位表,结构:  
typedef struct \_IMAGE\_BASE\_RELOCATION {  
    DWORD   VirtualAddress;//重定位数据 RVA  
    DWORD   SizeOfBlock;//重定位数据大小  
    WORD    TypeOffset\[1\];//重定位数据项数则  
} IMAGE\_BASE\_RELOCATION;  
typedef IMAGE\_BASE\_RELOCATION UNALIGNED \* PIMAGE\_BASE\_RELOCATION;  
●IMAGE\_BOUND\_IMPORT\_DESCRIPTOR 绑定输入表表,结构:  
  
typedef struct \_IMAGE\_BOUND\_IMPORT\_DESCRIPTOR {  
    DWORD   TimeDateStamp;  
    WORD    OffsetModuleName;  
    WORD    NumberOfModuleForwarderRefs;  
// Array of zero or more IMAGE\_BOUND\_FORWARDER\_REF follows  
} IMAGE\_BOUND\_IMPORT\_DESCRIPTOR,  \*PIMAGE\_BOUND\_IMPORT\_DESCRIPTOR;  
●获取 PE 文件的结构信息有两种方法,①通过打开文件直接读取②是通过文件映射读取  
①:首先通过 CreateFile()来打开一个文件,获得文件的句柄 hFile  
  然后用  
BOOL ReadFile(  
  HANDLE hFile,                // handle of file to read  
  LPVOID lpBuffer,             // pointer to buffer that receives data  
  DWORD nNumberOfBytesToRead,  // number of bytes to read  
  LPDWORD lpNumberOfBytesRead, // pointer to number of bytes read  
  LPOVERLAPPED lpOverlapped    // pointer to structure for data  
);  第二个参数指向一个缓冲区,接收文件的指针  
最后把这个指针赋值给 IMAGE\_DOS\_HEADER,IMAGE\_DOS\_HEADER 是文件的 DOS 头,这一步是把 DOS 头结构的地址与文件的地址关联,之后就可以对 PE 中的数据进行访问  
②:首先  
HANDLE CreateFile(  
  LPCTSTR lpFileName,          // pointer to name of the file  
  DWORD dwDesiredAccess,       // access (read-write) mode  
  DWORD dwShareMode,           // share mode  
  LPSECURITY\_ATTRIBUTES lpSecurityAttributes,  
                               // pointer to security attributes  
  DWORD dwCreationDisposition,  // how to create  
  DWORD dwFlagsAndAttributes,  // file attributes  
  HANDLE hTemplateFile         // handle to file with attributes to   
                               // copy  
);返回文件的的句柄 hFile  
然后依据此句柄创建文件映射内核对象  
HANDLE CreateFileMapping(  
  HANDLE hFile,              // handle to file to map  
  LPSECURITY\_ATTRIBUTES lpFileMappingAttributes,  
                             // optional security attributes  
  DWORD flProtect,           // protection for mapping object  
  DWORD dwMaximumSizeHigh,   // high-order 32 bits of object size  
  DWORD dwMaximumSizeLow,    // low-order 32 bits of object size  
  LPCTSTR lpName             // name of file-mapping object  
);创建一个文件映射内核对象,返回文件映射句柄 hMapping  
之后  
LPVOID MapViewOfFile(  
  HANDLE hFileMappingObject,  // file-mapping object to map into   
                              // address space  
  DWORD dwDesiredAccess,      // access mode  
  DWORD dwFileOffsetHigh,     // high-order 32 bits of file offset  
  DWORD dwFileOffsetLow,      // low-order 32 bits of file offset  
  DWORD dwNumberOfBytesToMap  // number of bytes to map  
);把文件装入内存,返回得到文件起始地址,一个指针  
最后就是把这个指针转换成 IMAGE\_DOS\_HEADER 类型,赋值给 IMAGE\_DOS\_HEADER 首址  
●●●●●●●●●●●●●●●●●●●●●●●  
EAX :累加器,用于算术,逻辑运算以及外设传送信息  
EBX :基址寄存器,存放存储器的地址  
ECX :计数器,做循环或者串指令运算中的隐含计数器  
EDX :数据寄存器,存放数据的高位,或者存放外设端口地址  
  
ESI :源变址寄存器,若在串指令中有特殊用法  
EDI :目的变址寄存器,若在串指令中有特殊用法  
ESP :堆栈指针寄存器,指示栈顶的便宜地址,不能再用于其它用途  
EBP :基址指针寄存器,说明数据在堆栈段的基地址  
  
CF (carry flag)进位标志  
ZF (zero flag)零标志  
SF (sign flag)  
能够改变 ECS,EIP 的指令统称为跳转指令,若要单一修改 EIP 的值,可以通过 jmp 合法寄存器 来实现,通用寄存器的值都可以用 mov 指令改写  
●  有符号数比较大小的结果指令依次为:G\[大于\]  E\[等于\]  L\[小于\]  
●  无符号数比较大小的结果指令依次为:A\[大于\]  E\[等于\]  B\[小于\]  
立即数不能直接送段地址  
   
1. 通用数据传送指令.   
         MOV     传送字或字节.   
         MOVSX   先符号扩展,再传送.   
         MOVZX   先零扩展,再传送.   
         PUSH     把字压入堆栈.   
         POP     把字弹出堆栈.   
         PUSHA   把 AX,CX,DX,BX,SP,BP,SI,DI 依次压入堆栈.   
         POPA     把 DI,SI,BP,SP,BX,DX,CX,AX 依次弹出堆栈.   
         PUSHAD   把 EAX,ECX,EDX,EBX,ESP,EBP,ESI,EDI 依次压入堆栈.   
         POPAD   把 EDI,ESI,EBP,ESP,EBX,EDX,ECX,EAX 依次弹出堆栈.   
         BSWAP   交换 32 位寄存器里字节的顺序   
         XCHG     交换字或字节.( 至少有一个操作数为寄存器,段寄存器不可作为操作数)   
         CMPXCHG 比较并交换操作数.( 第二个操作数必须为累加器 AL/AX/EAX )   
         XADD     先交换再累加.( 结果在第一个操作数里 )   
         XLAT     字节查表转换.   
                 ── BX 指向一张 256 字节的表的起点, AL 为表的索引值 (0-255,即   
                 0-FFH); 返回 AL 为查表结果. ( \[BX+AL\]->AL )   
     2. 输入输出端口传送指令.   
         IN       I/O 端口输入. ( 语法: IN 累加器, {端口号│DX} )   
         OUT     I/O 端口输出. ( 语法: OUT {端口号│DX},累加器 )   
           输入输出端口由立即方式指定时, 其范围是 0-255; 由寄存器 DX 指定时,   
           其范围是 0-65535.   
     3. 目的地址传送指令.   
         LEA     装入有效地址.   
           例: LEA DX,string   ;把偏移地址存到 DX.   
         LDS     传送目标指针,把指针内容装入 DS.   
           例: LDS SI,string   ;把段地址:偏移地址存到 DS:SI.   
         LES     传送目标指针,把指针内容装入 ES.   
           例: LES DI,string   ;把段地址:偏移地址存到 ES:DI.   
         LFS     传送目标指针,把指针内容装入 FS.   
           例: LFS DI,string   ;把段地址:偏移地址存到 FS:DI.   
         LGS     传送目标指针,把指针内容装入 GS.   
           例: LGS DI,string   ;把段地址:偏移地址存到 GS:DI.   
         LSS     传送目标指针,把指针内容装入 SS.   
           例: LSS DI,string   ;把段地址:偏移地址存到 SS:DI.   
     4. 标志传送指令.   
         LAHF     标志寄存器传送,把标志装入 AH.   
         SAHF     标志寄存器传送,把 AH 内容装入标志寄存器.   
         PUSHF   标志入栈.   
         POPF     标志出栈.   
         PUSHD   32 位标志入栈.   
         POPD     32 位标志出栈.   
  
二、算术运算指令   
───────────────────────────────────────   
         ADD     加法.   
         ADC     带进位加法.   
         INC     加 1.   
         AAA     加法的 ASCII 码调整.   
         DAA     加法的十进制调整.   
         SUB     减法.   
         SBB     带借位减法.   
         DEC     减 1.   
         NEC     求反(以 0 减之).   
         CMP     比较.(两操作数作减法,仅修改标志位,不回送结果).   
         AAS     减法的 ASCII 码调整.   
         DAS     减法的十进制调整.   
         MUL     无符号乘法.   
         IMUL     整数乘法.   
           以上两条,结果回送 AH 和 AL(字节运算),或 DX 和 AX(字运算),   
         AAM     乘法的 ASCII 码调整.   
         DIV     无符号除法.   
         IDIV     整数除法.   
           以上两条,结果回送:   
               商回送 AL,余数回送 AH, (字节运算);   
           或   商回送 AX,余数回送 DX, (字运算).   
         AAD     除法的 ASCII 码调整.   
         CBW     字节转换为字. (把 AL 中字节的符号扩展到 AH 中去)   
         CWD     字转换为双字. (把 AX 中的字的符号扩展到 DX 中去)   
         CWDE     字转换为双字. (把 AX 中的字符号扩展到 EAX 中去)   
         CDQ     双字扩展.     (把 EAX 中的字的符号扩展到 EDX 中去)   
  
三、逻辑运算指令   
───────────────────────────────────────   
         AND     与运算.   
         OR       或运算.   
         XOR     异或运算.   
         NOT     取反.   
         TEST     测试.(两操作数作与运算,仅修改标志位,不回送结果).   
         SHL     逻辑左移.   
         SAL     算术左移.(=SHL)   
         SHR     逻辑右移.   
         SAR     算术右移.(=SHR)   
         ROL     循环左移.   
         ROR     循环右移.   
         RCL     通过进位的循环左移.   
         RCR     通过进位的循环右移.   
           以上八种移位指令,其移位次数可达 255 次.   
               移位一次时, 可直接用操作码.   如 SHL AX,1.   
               移位>1 次时, 则由寄存器 CL 给出移位次数.   
                 如   MOV CL,04   
                     SHL AX,CL   
  
四、串指令   
───────────────────────────────────────   
             DS:SI   源串段寄存器   :源串变址.   
             ES:DI   目标串段寄存器:目标串变址.   
             CX       重复次数计数器.   
             AL/AX   扫描值.   
             D 标志   0 表示重复操作中 SI 和 DI 应自动增量; 1 表示应自动减量.   
             Z 标志   用来控制扫描或比较操作的结束.   
         MOVS     串传送.   
             ( MOVSB   传送字符.     MOVSW   传送字.     MOVSD   传送双字. )   
         CMPS     串比较.   
             ( CMPSB   比较字符.     CMPSW   比较字. )   
         SCAS     串扫描.   
             把 AL 或 AX 的内容与目标串作比较,比较结果反映在标志位.   
         LODS     装入串.   
             把源串中的元素(字或字节)逐一装入 AL 或 AX 中.   
             ( LODSB   传送字符.     LODSW   传送字.     LODSD   传送双字. )   
         STOS     保存串.   
             是 LODS 的逆过程.   
         REP             当 CX/ECX<>0 时重复.   
         REPE/REPZ       当 ZF=1 或比较结果相等,且 CX/ECX<>0 时重复.   
         REPNE/REPNZ     当 ZF=0 或比较结果不相等,且 CX/ECX<>0 时重复.   
         REPC           当 CF=1 且 CX/ECX<>0 时重复.   
         REPNC           当 CF=0 且 CX/ECX<>0 时重复.   
  
五、程序转移指令   
───────────────────────────────────────   
     1>无条件转移指令 (长转移)   
         JMP     无条件转移指令   
         CALL     过程调用   
         RET/RETF 过程返回.   
     2>条件转移指令 (短转移,-128 到+127 的距离内)   
         ( 当且仅当(SF XOR OF)=1 时,OP1<OP2 )   
         JA/JNBE 不小于或不等于时转移.   
         JAE/JNB 大于或等于转移.   
         JB/JNAE 小于转移.   
         JBE/JNA 小于或等于转移.   
           以上四条,测试无符号整数运算的结果(标志 C 和 Z).   
         JG/JNLE 大于转移.   
         JGE/JNL 大于或等于转移.   
         JL/JNGE 小于转移.   
         JLE/JNG 小于或等于转移.   
           以上四条,测试带符号整数运算的结果(标志 S,O 和 Z).   
         JE/JZ   等于转移.   
         JNE/JNZ 不等于时转移.   
         JC       有进位时转移.   
         JNC     无进位时转移.   
         JNO     不溢出时转移.   
         JNP/JPO 奇偶性为奇数时转移.   
         JNS     符号位为 "0" 时转移.   
         JO       溢出转移.   
         JP/JPE   奇偶性为偶数时转移.   
         JS       符号位为 "1" 时转移.   
     3>循环控制指令(短转移)   
         LOOP             CX 不为零时循环.   
         LOOPE/LOOPZ     CX 不为零且标志 Z=1 时循环.   
         LOOPNE/LOOPNZ   CX 不为零且标志 Z=0 时循环.   
         JCXZ             CX 为零时转移.   
         JECXZ           ECX 为零时转移.   
     4>中断指令   
         INT     中断指令   
         INTO     溢出中断   
         IRET     中断返回   
     5>处理器控制指令   
         HLT     处理器暂停, 直到出现中断或复位信号才继续.   
         WAIT     当芯片引线 TEST 为高电平时使 CPU 进入等待状态.   
         ESC     转换到外处理器.   
         LOCK     封锁总线.   
         NOP     空操作.   
         STC     置进位标志位.   
         CLC     清进位标志位.   
         CMC     进位标志取反.   
         STD     置方向标志位.   
         CLD     清方向标志位.   
         STI     置中断允许位.   
         CLI     清中断允许位.   
  
六、伪指令   
───────────────────────────────────────   
         DB       定义字节.   
         DW       定义字(2 字节).   
         PROC     定义过程.   
         ENDP     过程结束.   
         SEGMENT 定义段.   
         ASSUME   建立段寄存器寻址.   
         ENDS     段结束.   
         END     程序结束.