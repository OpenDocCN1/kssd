
`0x7eae056e34`: `add w20, w23, w20` ------ `w20=0x4c` `w23=0x774` //`w20=0x7c0`

`0x7eae056e34`: `add w20, w23, w20` ------ `w20=0x30` `w23=0x7c0` //`w20=0x7f0`

`0x7eae056e34`: `add w20, w23, w20` ------ `w20=0x66` `w23=0x7f0` //`w20=0x856`

`0x7eae056e34`: `add w20, w23, w20` ------ `w20=0x57` `w23=0x856` //`w20=0x8ad`

`0x7eae056e34`: `add w20, w23, w20` ------ `w20=0x55` `w23=0x8ad` //`w20=0x902`

`0x7eae056e34`: `add w20, w23, w20` ------ `w20=0x47` `w23=0x902` //`w20=0x949`

`0x7eae056e34`: `add w20, w23, w20` ------ `w20=0x43` `w23=0x949` //`w20=0x98c`

`0x7eae056e34`: `add w20, w23, w20` ------ `w20=0x4f` `w23=0x98c` //`w20=0x9db`

到这里就知道了。这个`0x9db`是`input`的累加。不过并不是完整的累加。所以这里稍微留意下。就能得出结果了。

最后`0x23`位置的字节处理和`0x22`位置的基本雷同。我就不重复讲了。直接贴上解密后的代码

```c++
#include <iostream>

# include <stdlib.h>

#include <stdio.h>

char getAscii(int num){

char tmp[3];

snprintf(tmp,3,"%x",num);

return tmp[0];

}

int main() {

char input[]="L8Cs5ntgNWiPMQRUKYXva0juFw9JqI4ApyZT";

char output[strlen(input)];

int sum=0;

int eorsum=0xff;

for(int i=0;i<strlen(input);i++){

output[i]=input[i]^0x1;

if(i==0x8||i==0xd||i==0x12||i==0x18){

output[i]=0x2d;

continue;

}else if(i==0xe){

output[i]=0x34;

continue;

}else if(i==0x23){

output[i]=input[0x22];

continue;

}else if(i==0x22){

output[i]=input[0x9];

continue;

}else if(i==0x17){

output[i]=input[0x18]^0x1;

}

if(i<0x22){

int addvalue=input[i];

if(i==0x17){

addvalue=input[0x18];

}

sum+=addvalue;

eorsum^=addvalue;

}

}

int data22=sum-(sum&0xfffffff0);

char tmp22=getAscii(data22);

int data23=eorsum&0xf;

char tmp23=getAscii(data23);

output[0x22]=tmp22;

output[0x23]=tmp23;

output[strlen(input)]=0x0;

printf("input:%s output:%s\n",input,output);

return 0;

}
```

最后输出结果

```
input:L8Cs5ntgNWiPMQRUKYXva0juFw9JqI4ApyZT output:M9Br4ouf-VhQL-4TJX-w`1kG-v8KpH5@qx7c
```

题目出自3W班9月的题目还原`ollvm`混淆的自定义算法

`ollvm`的自定义算法的还原一般就是意味着非常多的分支。

我们可以`F5`打开后。逐步根据参数的传递。逐步的进行追踪。又或者是根据返回值。来逐层向来源追踪。如果这个算法是一个比较纯粹的算法。那么我们可以用`unicorn`来模拟执行这个代码段。并且打印所有的汇编指令每一行的变化。最后还原出这个算法。这个题目就是一个比较纯粹的算法。这里我采用了`unicorn`来进行算法的恢复。

做这个题目。我定制了一个专门用来`trace`指令详细变化的回调。可以直观的看到每行汇编指令当前的寄存器的数值和指令执行后的数值。

`unicorn`相当于是一个`cpu`的模拟器，可以用来执行`so`中的代码段，一般不要直接使用`apk`中的`so`文件，直接使用是需要修复上下文的，这样会比较复杂。最好是直接从内存中直接`dump`一个`so`出来。然后就可以直接执行`so`里面的代码段。然后看一个例子`ollvm9.apk`

解密案例:`95cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1M7i4A6Y4x3e0t1K6x3o6m8Q4x3V1k6#2L8X3W2U0L8%4u0F1c8r3g2E0L8H3`.`.

首先用`jadx`打开这个`apk`。看看里面的按钮的功能

![](https://bbs.kanxue.com/upload/attach/202010/659397_6BHURJ9VPSTVT22.png)

然后看到这里主要用到了`UUIDCheckSum`这个函数来进行加密

解压这个`apk`。用`ida`打开`libnative-lib.so`找到`UUIDCheckSum`函数。发现这个函数是`ollvm`混淆的。然后找入参的使用，发现了一个关键函数。

![](https://bbs.kanxue.com/upload/attach/202010/659397_6V3QXQKTCXKZ5UN.png)

然后我们想要用`unicorn`来执行这个函数。首先找到函数的起始和终止位置，这个函数我找到是`start:0xfcb4` `end:0xff2c`

直接使用我们这个`ida`查看的`so`文件是不行了。最好是在真机执行时从内存中`dump`这个`so`出来。就是拥有完整上下文信息的。这里我使用了大佬的工具来`dump`

`6e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9j5i4y4@1K9h3&6Y4i4K6u0V1P5h3q4F1k6#2)9J5c8X3k6J5K9h3c8S2i4K6g2X3k6s2g2E0M7l9`.`.

使用起来也是非常简单

`frida -U com.kanxue.ollvm_ndk_9 -l dump_so.js`

`dump_so("libnative-lib.so")`

执行后生成了一个`so`文件`libnative-lib.so_0x7eae047000_0x38000.so`。将这个文件拷贝到`py`项目下。

接着梳理一下想要写一个使用`unicorn`来执行的流程。

1、创建一个`unicorn`对象
2、使用`unicorn`创建一块内存，用来存放这个`so`代码
3、使用`unicorn`创建一块内存，用来存放栈空间
4、使用`unicorn`创建一块内存，用来存放要执行函数的参数
5、读取`so`，将`so`写入到上面预先创建的内存中。
6、给参数的那块内存赋值
7、给寄存器赋值（`x0,x1,sp`），`X0`就是`ida`函数中看到的第一个参数，`X1`就是第二个参数
8、`unicorn`启动执行指定片段的代码
9、读取执行结果
10、释放创建的内存

然后下面看一下实现的代码

```python
if __name__ == '__main__':
    initGlobalData()
    #创建uc对象
    uc=unicorn.Uc(unicorn.UC_ARCH_ARM64,unicorn.UC_MODE_ARM)
    #从内存中dump下来so的基址
    code_addr=0x7eae047000
    #用来存放so代码的大小，尽量大一点。内存不值钱
    code_size=8*0x1000*0x1000
    #创建一块内存
    uc.mem_map(code_addr,code_size)
    #在上面那块内存后面继续划一片内存来当做栈空间
    stack_addr=code_addr+code_size
    stack_size=0x1000
    #栈顶的位置，这里是64位的，所以偏移8个字节
    stack_top=stack_addr+stack_size-0x8
    #申请一块栈空间
    uc.mem_map(stack_addr,stack_size)
    #栈空间往后继续划一块空间用来存放参数
    args_addr=stack_addr+stack_size
    args_size=0x1000
    uc.mem_map(args_addr, args_size)
    #设置每句汇编执行都会调用hook_code
    #uc.hook_add(unicorn.UC_HOOK_CODE,hook_code)
    #读取so
    with open("./libnative-lib.so_0x7eae047000_0x38000.so","rb") as f:
        sodata=f.read()
    #给前面创建的空间写入so的数据
    uc.mem_write(code_addr,sodata)
    #要执行的代码开始位置
    start_addr=code_addr+0xFCB4
    #要执行的代码结束位置
    end_addr=code_addr+0xFF2C
    #随机生成一个入参
    input_str = ranstr(36)
    print("input:%s input_addr:0x%x" % (input_str,args_addr))
    input_byte=str.encode(input_str)
    #将生成的入参写入前面创建的内存空间
    uc.mem_write(args_addr,input_byte)
    #ida中看到的函数有参数1、2，然后分别对应X0和X1，写入对应数据，栈寄存器给一个栈顶的地址
    uc.reg_write(unicorn.arm64_const.UC_ARM64_REG_X0,args_addr)
    uc.reg_write(unicorn.arm64_const.UC_ARM64_REG_X1,len(input_str))
    uc.reg_write(unicorn.arm64_const.UC_ARM64_REG_SP,stack_top)
    #开始执行代码段
    uc.emu_start(start_addr,end_addr)
    #ida中看到返回值是直接写在入参中，所以结果我们直接从入参的内存中读取
    result=uc.mem_read(args_addr,args_size)
    print("result:",result.decode(encoding="utf-8"))
    #最后释放创建的内存
    uc.mem_unmap(args_addr, args_size)
    uc.mem_unmap(stack_addr,stack_size)
    uc.mem_unmap(code_addr,code_size)
```

执行后，得到下面的结果

```
input:nxRH3WwuTJgUfqcOS94CM5QEkoPeF0sZ8mGj input_addr:0x7eb6048000
result: oySI2Vvt-KfTg-4NR8-BL4Pj-nQdG1r[9laf
```

执行这个代码段成功后，我们想的是能够将每行执行的汇编代码都打印一下。可以使用`hook_add`来添加一个指令执行时的回调函数，我们解开上面代码的这句注释

```python
uc.hook_add(unicorn.UC_HOOK_CODE,hook_code)
```

然后先设置一个最简单的打印看看

```python
def hook_code(uc: unicorn.Uc, address, size, user_data):
    inst_code=uc.mem_read(address,size)
    for inst in cs.disasm(inst_code,size):
        print("0x%x:\t%s\t%s" % (address, inst.mnemonic, inst.op_str))
```

设置之后的打印效果是

```
0x7eae056f08:    ldr    x25, [sp,#0x10]
0x7eae056f0c:    sub    w9, w10, w9
0x7eae056f10:    strb    w8, [x0,#0x23]
0x7eae056f14:    ldrb    w8, [x11, w9, uxtw]
0x7eae056f18:    strb    w8, [x0,#0x22]
0x7eae056f1c:    ldp    x20, x19, [sp,#0x40]
0x7eae056f20:    ldp    x22, x21, [sp,#0x30]
0x7eae056f24:    ldp    x24, x23, [sp,#0x20]
0x7eae056f28:    add    sp, sp,#0x50
```

但是仅仅是这样的效果还是没办法拿来分析算法，必须将每个寄存器的结果在后面打印出来，所以要改造下,下面是我根据自己的需求定制的一个`trace`打印

1、在每行的汇编后面打印该行中所有使用到的寄存器数据
2、在每行的最后再打印这个寄存器在计算后的结果值
3、监控指定地址的内存变动。如果数据发生改变。则打印这块内存的数据

下面看具体实现。

首先创建一个全局文件`globalData.py`用来存放全局变量

```python
#上一次汇编指令
global pre_codestr
#上一次汇编的第一个寄存器名称
global pre_regname
#是否有记录上一次的数据
global has_pre
#监控的地址
global watch_addrs
```

下面贴上完整例子
