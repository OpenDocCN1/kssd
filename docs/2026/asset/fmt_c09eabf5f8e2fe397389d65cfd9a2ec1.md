
```
                                         000000EF     74 - LOAD_GLOBAL         'CreateActivationRequest'

                                         000000F2     7C - LOAD_FAST           'lic'

                                         000000F5     83 - CALL_FUNCTION       

                                         000000F8     7D - STORE_FAST          'lichash'

                                         000000FB     7C - LOAD_FAST           'lic'

                                         000000FE     64 - LOAD_CONST          'activation'

                                         00000101     19 - BINARY_SUBSCR       

                                         00000102     7D - STORE_FAST          'act30'

                                         00000105     7C - LOAD_FAST           'act30'

                                         00000108     69 - LOAD_ATTR           'replace'

                                         0000010B     64 - LOAD_CONST          '-'

                                         0000010E     64 - LOAD_CONST          ''

                                         00000111     83 - CALL_FUNCTION       

                                         00000114     64 - LOAD_CONST          3

                                         00000117     1F - SLICE+1             

                                         00000118     7D - STORE_FAST          'act'

                                         0000011B     74 - LOAD_GLOBAL         'textutils'

                                         0000011E     69 - LOAD_ATTR           'BaseConvert'

                                         00000121     7C - LOAD_FAST           'act'

                                         00000124     74 - LOAD_GLOBAL         'textutils'

                                         00000127     69 - LOAD_ATTR           'BASE30'

                                         0000012A     74 - LOAD_GLOBAL         'textutils'

                                         0000012D     69 - LOAD_ATTR           'BASE16'

                                         00000130     83 - CALL_FUNCTION       

                                         00000133     7D - STORE_FAST          'hexact'

                                         00000136     78 - SETUP_LOOP          -> 0000015B

                                         00000139     74 - LOAD_GLOBAL         'len'

                                         0000013C     7C - LOAD_FAST           'hexact'

                                         0000013F     83 - CALL_FUNCTION       

                                         00000142     64 - LOAD_CONST          20

                                         00000145     6A - COMPARE_OP          "<"

                                         00000148     6F - JUMP_IF_FALSE       -> 00000159

                                         0000014B     01 - POP_TOP             

                                         0000014C     64 - LOAD_CONST          '0'

                                         0000014F     7C - LOAD_FAST           'hexact'

                                         00000152     17 - BINARY_ADD          

                                         00000153     7D - STORE_FAST          'hexact'

                                         00000156     71 - JUMP_ABSOLUTE       -> 00000139

                                         00000159     01 - POP_TOP             

                                         0000015A     57 - POP_BLOCK           

                                         0000015B     64 - LOAD_CONST          0

                                         0000015E     74 - LOAD_GLOBAL         'config'

                                         00000161     5F - STORE_ATTR          '_locale_valid'

                                         00000164     74 - LOAD_GLOBAL         'control'

                                         00000167     69 - LOAD_ATTR           'validate'

                                         0000016A     7C - LOAD_FAST           'lichash'

                                         0000016D     7C - LOAD_FAST           'lic'

                                         00000170     64 - LOAD_CONST          'os'

                                         00000173     19 - BINARY_SUBSCR       

                                         00000174     7C - LOAD_FAST           'lic'

                                         00000177     64 - LOAD_CONST          'version'

                                         0000017A     19 - BINARY_SUBSCR       

                                         0000017B     7C - LOAD_FAST           'lic'

                                         0000017E     64 - LOAD_CONST          'version'

                                         00000181     19 - BINARY_SUBSCR       

                                         00000182     69 - LOAD_ATTR           'find'

                                         00000185     64 - LOAD_CONST          '.'

                                         00000188     83 - CALL_FUNCTION       

                                         0000018B     20 - SLICE+2             

                                         0000018C     7C - LOAD_FAST           'hexact'

                                         0000018F     83 - CALL_FUNCTION       

                                         00000192     7D - STORE_FAST          'valid'

                                         00000195     74 - LOAD_GLOBAL         'config'

                                         00000198     69 - LOAD_ATTR           '_locale_valid'

                                         0000019B     7D - STORE_FAST          'valid'

                                         0000019E     57 - POP_BLOCK           

                                         0000019F     6E - JUMP_FORWARD        -> 000001AF

                                         000001A2     01 - POP_TOP             

                                         000001A3     01 - POP_TOP             

                                         000001A4     01 - POP_TOP             

                                         000001A5     64 - LOAD_CONST          0

                                         000001A8     7D - STORE_FAST          'valid'

                                         000001AB     6E - JUMP_FORWARD        -> 000001AF

                                         000001AE     58 - END_FINALLY         

                                         000001AF     7C - LOAD_FAST           'valid'

                                         000001B2     70 - JUMP_IF_TRUE        -> 000001C9

                                         000001B5     01 - POP_TOP             

                                         000001B6     74 - LOAD_GLOBAL         'kLicenseCorrupt'

                                         000001B9     74 - LOAD_GLOBAL         '_'

                                         000001BC     64 - LOAD_CONST          'Invalid license activation'

                                         000001BF     83 - CALL_FUNCTION       
```

简单说明一下这段代码

1. `CreateActivationRequest(lic)`创建一个request code，存入`lichash`，正版用户凭这个code去拿激活码

2. 去掉`lic.activation`中的`"-"`，存入`act30`

3. `textutils.BaseConvert(act30, textutils.BASE30, textutils.BASE16)`，转成16进制的字符串，存入`hexact`

4. `config._locale_valid`置0

5. 重点来了

   `control.validate(lichash, lic.os, lic.version[:lic.version.find('.')], hexact)`，把结果存入`config._locale_valid`，非真的话出报错信息。

   第一个参数是 request code，软件会显示出来

   第二个参数，操作系统，这里是"windows"

   第三个参数，版本号，这里是"4"

   第三个参数，是变形后的激活码

   返回结果为真，则注册成功，反之则失败。

`control`模块是在这里导入的

```
                 00000332     6B - IMPORT_NAME         'ctlutil'

                 00000335     5A - STORE_NAME          'control'
```

相当于python代码：

```
import ctlutil as control
```

## 曙光

在`process`目录下有`ctlutil.pyd`，这是用c编译的python模块，本质上就是一个dll。可能作者觉得编译成native code比较安全吧，真的是这样吗？

先在`test.py`中输入代码：

```
while True:

    abstract.control.validate("RW42X-XXXXX-XXXXX-XXXXX", "windows", "4", "11111111")
```
