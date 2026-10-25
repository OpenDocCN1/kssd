
```python
import angr
import claripy
import sys

def main(argv):
    path_to_binary = argv[1]
    project = angr.Project(path_to_binary)

    # 这里我们指定符号执行引擎开始的地方
    # 注意，这里state的构造函数用的是blank_state，构造了一个空白state，不再是entry_state了
    start_address = 0x80488d1
    # :integer (probably hexadecimal)
    initial_state = project.factory.blank_state(addr=start_address)

    # 创建一个符号位向量（angr用于给二进制文件注入符号值的数据类型）
    password0_size_in_bits = 32
    # :integer
    password0 = claripy.BVS('password0', password0_size_in_bits)

    password1_size_in_bits = 32
    # :integer
    password1 = claripy.BVS('password1', password1_size_in_bits)

    password2_size_in_bits = 32
    # :integer
    password2 = claripy.BVS('password2', password2_size_in_bits)

    # 将寄存器设置为符号值。 这是向程序中注入符号的一种方法。
    # 将不同寄存器设置为不同的符号值
    initial_state.regs.eax = password0
    initial_state.regs.ebx = password1
    initial_state.regs.edx = password2

    simulation = project.factory.simgr(initial_state)

    def is_successful(state):
        stdout_output = state.posix.dumps(sys.stdout.fileno())
        return 'Good Job.' in str(stdout_output)

    def should_abort(state):
        stdout_output = state.posix.dumps(sys.stdout.fileno())
        return 'Try again.' in str(stdout_output)

    simulation.explore(find=is_successful, avoid=should_abort)

    if simulation.found:
        solution_state = simulation.found[0]

        # 使用eval来得到求解值
        # 原解题脚本的state.se已经被废弃，根据提示使用state.solver
        solution0 = solution_state.solver.eval(password0)
        solution1 = solution_state.solver.eval(password1)
        solution2 = solution_state.solver.eval(password2)

        # 汇总并格式化上面的结果，最后打印出来
        solution = ' '.join(map('{:x}'.format, [ solution0, solution1, solution2 ]))
        # :string
        print(solution)
    else:
        raise Exception('Could not find the solution')

if __name__ == '__main__':
    main(sys.argv)
```

运行结果：

![image-20210102220626889](https://bbs.kanxue.com/upload/attach/202107/830671_7SARDTPQA8X77YR.jpg)

> - Bitvectors：可以看angrctf里的ppt，很清楚，这里就不再贴了。
>
> - eval
>
>   - `solver.eval(expression)` 将会解出一个可行解
>   - `solver.eval_one(expression)`将会给出一个表达式的可行解，若有多个可行解，则抛出异常。
>   - `solver.eval_upto(expression, n)`将会给出最多n个可行解，如果不足n个就给出所有的可行解。
>   - `solver.eval_exact(expression, n)`将会给出n个可行解，如果解的个数不等于n个，将会抛出异常。
>   - `solver.min(expression)`将会给出最小可行解
>   - `solver.max(expression)`将会给出最大可行解

## 04_angr_symbolic_stack

### 步骤一：查看文件属性和检查安全机制

- 32位程序
- 关闭了Canary和PIE

![image-20210102215817440](https://bbs.kanxue.com/upload/attach/202107/830671_U4MVPDAC3R5BRN7.jpg)

### 步骤二：IDA静态分析

![image-20210629174704700](https://bbs.kanxue.com/upload/attach/202107/830671_TQUY4DXKQJWG3A5.jpg)

这里同样的，`scanf`的格式化字符串有两个参数，相对“复杂”。因为对于`scanf`函数，`angr`只能自动注入一个参数，所以需要我们手动进行注入。同时在对输入符号化后还需找到启动程序的地方。

![image-20210629194135931](https://bbs.kanxue.com/upload/attach/202107/830671_6KC3756WM6ZPH2F.jpg)

调用函数`scanf`对应的汇编代码如下所示，红框中的代码为调用`scanf`函数的过程。

![image-20210629194026620](https://bbs.kanxue.com/upload/attach/202107/830671_J8E56YBVGKN5VA9.jpg)

`v1`的地址是`ebp-0x10`，`v2`的地址是`ebp-0xc`，函数`handle_user`调用`scanf`函数前后栈的结构如下图所示。所以，很明显，`add esp,10h`指令才是调用`scanf`函数的最后一条指令。因此，`angr`启动该程序的地址应该设置为`0x08048697`。

![image-20210629204838623](https://bbs.kanxue.com/upload/attach/202107/830671_ANWTCJPD2T8KWYV.jpg)

上一题只需要把符号值注入给寄存器，然后从指定地址启动程序就可以了，但是这一题不一样，要符号化的输入在栈里，所以我们需要在启动该程序之前自己构造相应的栈结构。

方法是通过`state.stack_push(my_bitvector)`来将值`my_bitvector` push到栈顶。另外，如果需要push一些无用的数据，则可以使用类似`state.regs.esp-=4`的代码来达到目的，这行代码实现的就是填充4字节的padding。

因为这一题关闭了Canary，所以`v2`和`ebp`之间只填充无用数据就可以。

### 步骤三：使用`angr`解题

```python
import angr
import claripy
import sys

def main(argv):
    path_to_binary = './04_angr_symbolic_stack'
    project = angr.Project(path_to_binary)

    # 启动地址设置为 add esp, 10h的下一条指令的地址
    start_address = 0x8048697
    initial_state = project.factory.blank_state(addr=start_address)

    # 开始构造栈结构
    # 最初，ebp和esp的值相等
    initial_state.regs.ebp = initial_state.regs.esp

    # 两个位向量，符号化输入
    password0 = claripy.BVS('password0', 32)
    password1 = claripy.BVS('password1', 32)

    # 由上面的栈结构图可知ebp距离输入8字节，所以先填充8字节无用数据
    padding_length_in_bytes = 8
    # :integer
    initial_state.regs.esp -= padding_length_in_bytes

    # 然后填充两个位向量
    initial_state.stack_push(password0) # :bitvector (claripy.BVS, claripy.BVV, claripy.BV)
    initial_state.stack_push(password1)

    simulation = project.factory.simgr(initial_state)

    def is_successful(state):
        stdout_output = state.posix.dumps(sys.stdout.fileno())
        return 'Good Job.' in str(stdout_output)

    def should_abort(state):
        stdout_output = state.posix.dumps(sys.stdout.fileno())
        return 'Try again.' in str(stdout_output)

    simulation.explore(find=is_successful, avoid=should_abort)

    if simulation.found:
        solution_state = simulation.found[0]
        # 求解
        solution0 = solution_state.se.eval(password0)
        solution1 = solution_state.se.eval(password1)

        solution = ' '.join(map(str, [ solution0, solution1 ]))
        print(solution)
    else:
        raise Exception('Could not find the solution')

if __name__ == '__main__':
    main(sys.argv)
```

运行结果：

![image-20210102220436236](https://bbs.kanxue.com/upload/attach/202107/830671_B2ME5A8QF3VHJKK.jpg)

## 05_angr_symbolic_memory

### 步骤一：查看文件属性和检查安全机制

- 32位程序
- Canary和PIE关闭

![image-20210102220803629](https://bbs.kanxue.com/upload/attach/202107/830671_BM24EXE75S5D62G.jpg)

### 步骤二：IDA静态分析

![image-20210629214806006](https://bbs.kanxue.com/upload/attach/202107/830671_6HANCWJ6GY7CQ89.jpg)

解题思路和上一题是一样的，不过这里使用`state.memory.store(address, value)`将全局变量的值修改为符号值（操作一段连续的内存）。

![image-20210629211803353](https://bbs.kanxue.com/upload/attach/202107/830671_VPCKT566G3R85YU.jpg)

启动地址：`0x8048606`。

![image-20210629222225378](https://bbs.kanxue.com/upload/attach/202107/830671_52X6T7YKYKY22ZF.jpg)

### 步骤三：使用`angr`解题
