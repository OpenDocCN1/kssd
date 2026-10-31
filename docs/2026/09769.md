
# Math ActivityMaker Skills v2.20注册算法（简单）

## 软件名称
Math ActivityMaker Skills v2.20

## 主要功能
MATH ACTIVITYMAKER FRACTIONS(分数数学出题软体)是　GEPETO SOFTWARE公司出品的教学软体之一，该公司出版的一系列软体，都很适用中小学教师在教学中使用，或学生自行操作学习。程序包括`addition/subtractions skills`(分数加减运算),`multiplication/division skills`(分数乘除运算),`GCF worksheet`(最大公因数),`LCM worksheet`(最小公倍数),`reducing fractions`(约分),`greater/less than`(比较分数大小),`value-matching`(数值配对连连看),`fraction fill-in`(扩分)等功能!非常容易使用!

## 破解工具
`SmartCheck`

## 破解过程
用`SmartCheck`调用程序，输入姓名`coldeye`、注册码`787878`，提示密码错误，退出。查看记录：

`Len(String:"coldeye")` returns `LONG:7` 姓名长度

Arguments

--------------------

String  `string1 = 0013474C`

      = `"coldeye"`

Long `(7) --> Integer (7)`

Long `7`

Integer `7`

`Mid(VARIANT:ByRef String:"coldeye", long:1, VARIANT:Integer:1)`

Arguments

--------------------

string `(variant)`

    unsigned short \* \* .pbstrVal = `00154FA4`

        String   = `0013474C`

              = `"coldeye"`

Long  length = `1` `0x00000001`

start `(variant)`

    Integer  .iVal = `1` `0x0001`

`Asc(String:"c")` returns `Integer:99` 取第一位字母的ASCII值

Arguments

--------------------

String  `string = 0014E28C`

      = `"c"`

`Mid(VARIANT:ByRef String:"coldeye", long:2, VARIANT:Integer:1)`

Arguments

--------------------

string `(variant)`

    unsigned short \* \* .pbstrVal = `00154FA4`

        String   = `0013474C`

              = `"coldeye"`

Long  length = `2` `0x00000002`

start `(variant)`

    Integer  .iVal = `1` `0x0001`

`Asc(String:"o")` returns `Integer:111` 取第二位字母的ASCII值

Arguments

--------------------

String  `string = 0014E28C`

      = `"o"`

`Mid(VARIANT:ByRef String:"coldeye", long:3, VARIANT:Integer:1)`

Arguments

--------------------

string `(variant)`

    unsigned short \* \* .pbstrVal = `00154FA4`

        String   = `0013474C`

              = `"coldeye"`

Long  length = `3` `0x00000003`

start `(variant)`

    Integer  .iVal = `1` `0x0001`

`Asc(String:"l")` returns `Integer:108` 取第三位字母的ASCII值

Arguments

--------------------

String  `string = 0014E28C`

      = `"l"`

`Mid(VARIANT:ByRef String:"coldeye", long:4, VARIANT:Integer:1)`

Arguments

--------------------

string `(variant)`

    unsigned short \* \* .pbstrVal = `00154FA4`

        String   = `0013474C`

              = `"coldeye"`

Long  length = `4` `0x00000004`

start `(variant)`

    Integer  .iVal = `1` `0x0001`

`Asc(String:"d")` returns `Integer:100` 取第四位字母的ASCII值

Arguments

--------------------

String  `string = 0014E28C`

      = `"d"`

`Mid(VARIANT:ByRef String:"coldeye", long:5, VARIANT:Integer:1)`

Arguments

--------------------

string `(variant)`

    unsigned short \* \* .pbstrVal = `00154FA4`

        String   = `0013474C`

              = `"coldeye"`

Long  length = `5` `0x00000005`

start `(variant)`

    Integer  .iVal = `1` `0x0001`

`Asc(String:"e")` returns `Integer:101` 取第五位字母的ASCII值

Arguments

--------------------

String  `string = 0014E28C`

      = `"e"`

`Mid(VARIANT:ByRef String:"coldeye", long:6, VARIANT:Integer:1)`

Arguments

--------------------

string `(variant)`

    unsigned short \* \* .pbstrVal = `00154FA4`

        String   = `0013474C`

              = `"coldeye"`

Long  length = `6` `0x00000006`

start `(variant)`

    Integer  .iVal = `1` `0x0001`

`Asc(String:"y")` returns `Integer:121` 取第六位字母的ASCII值

Arguments

--------------------

String  `string = 0014E28C`

      = `"y"`

`Mid(VARIANT:ByRef String:"coldeye", long:7, VARIANT:Integer:1)`

Arguments

--------------------

string `(variant)`

    unsigned short \* \* .pbstrVal = `00154FA4`

        String   = `0013474C`

              = `"coldeye"`

Long  length = `7` `0x00000007`

start `(variant)`

    Integer  .iVal = `1` `0x0001`

`Asc(String:"e")` returns `Integer:101` 取第七位字母的ASCII值

Arguments

--------------------

String  `string = 0014E28C`

      = `"e"`

`Str(VARIANT:Integer:963)` ASCII值总和

Arguments

--------------------

number `(variant)`

    Integer  .iVal = `963` `0x03C3`

`Mid(VARIANT:ByRef String:" 963", long:1, VARIANT:Integer:1)`

Arguments

--------------------

string `(variant)`

    unsigned short \* \* .pbstrVal = `00154FA4`

        String   = `0014E93C`

              = `" 963"`

Long  length = `1` `0x00000001`

start `(variant)`

    Integer  .iVal = `1` `0x0001`

`Len(String:" 963")` returns `LONG:4`

Arguments

--------------------

String  `string1 = 0014E93C`

      = `" 963"`

`Right(VARIANT:ByRef String:" 963", long:3)`

Arguments

--------------------

string `(variant)`

    unsigned short \* \* .pbstrVal = `00154FA4`

        String   = `0014E93C`

              = `" 963"`

Long  length = `3` `0x00000003`

`Len(String:"78787878")` returns `LONG:8` 注册码长度

Arguments

--------------------

String  `string1 = 0013474C`

      = `"78787878"`

`Right(VARIANT:String:"78787878", long:6)` 取后6位

Arguments

--------------------

string `(variant)`

    String  .bstrVal = `0014E93C`

          = `"78787878"`

Long  length = `6` `0x00000006`

`MsgBox(VARIANT:String:"Invalid ...", Integer:0, VARIANT:Missing, VARIANT:Missing, VARIANT:Missing)` returns `Integer:1`

Arguments

--------------------

prompt `(variant)` 密码错误

    String  .bstrVal = `0017778C`

          = `"Invalid Registration Code. Please re-enter the registration info"`

Long  buttons = `0` `0x00000000`

title `(variant)`

    Long  .scode = `-2147352572` `0x80020004`

helpfile `(variant)`

    Long  .scode = `-2147352572` `0x80020004`

context `(variant)`

    Long  .scode = `-2147352572` `0x80020004`

## 总结
注册算法：将注册名的各位ASCII值加起来，与注册码的第三位开始进行比较。注册码首二位可以是任何值。

Name:`coldeye`

Password:`cn963`
