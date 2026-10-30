
```asm
426 JUMPDEST
427 DUP3
428 DUP2
429 DUP4
430 CALLDATACOPY    ;CALLDATACOPY(80, 0, 4)
431 PUSH1 00
433 DUP4
434 DUP4
435 ADD             ;80h+4
436 MSTORE          ;MSTORE(84h,0)
437 POP
438 POP
439 POP
440 JUMP
301 JUMPDEST
302 DUP3
303 DUP5
304 ADD
305 SWAP1
306 POP
307 SWAP4
308 SWAP3
309 POP
310 POP
311 POP
312 JUMP
326 JUMPDEST
327 SWAP2
328 POP
329 DUP2
330 SWAP1
331 POP
332 SWAP4
333 SWAP3
334 POP
335 POP
336 POP
337 JUMP
```

上述指令段中将`calldata`的数据根据长度拷贝至了`Memory`中，随后对栈进行了清理，至此，栈终于干净了

```asm
0: 0x0000000000000000000000000000000000000000000000000000000000000084
1: 0x000000000000000000000000358aa13c52544eccef6b0add0f801012adad5ee3
2: 0x0000000000000000000000000000000000000000000000000000000000000000
3: 0x00000000000000000000000000000000000000000000000000000000dd365b8b
```

#### 3.2.2 DelegateCall

```asm
124 JUMPDEST
125 PUSH1 00
127 PUSH1 40
129 MLOAD        ;MLOAD(40h)加载预留指针
130 DUP1         ;push 80h
131 DUP4         ;push 84h
132 SUB          ;计算calldata长度
133 DUP2         ;push 80h
134 DUP6         ;push Delegate.address
135 GAS          ;push gas
136 DELEGATECALL ;DelegateCall(gas,addr,argOst,argLen,retOst,retLen)
```

至此就正式进入了委托调用流程，参数分别如下

- 剩余Gas：2497
- 合约地址：`Delegate.address`
- 参数存储位置偏移：80h
- 参数长度：4
- 返回值存储位置偏移：0
- 返回值长度：84h

随后我们继续步入，首先进入到了`Non Payable Check`，接着直接来到了`Calldata Prepared & Compared`，最后根据我们传入的`Calldata`找到`pwn()`函数

```asm
135 JUMPDEST LINE 12
136 CALLER               ;push msg.sender
137 PUSH1 00
139 DUP1
140 PUSH2 0100
143 EXP
144 DUP2
145 SLOAD                ;SLOAD(0)加载Delegate的Owner成员
146 DUP2
147 PUSH20 ffffffffffffffffffffffffffffffffffffffff
168 MUL
169 NOT
170 AND
171 SWAP1
172 DUP4
173 PUSH20 ffffffffffffffffffffffffffffffffffffffff
194 AND
195 MUL
196 OR
197 SWAP1
198 SSTORE               ;owner = msg.sender
199 POP
200 JUMP
097 JUMPDEST
098 STOP
```

在委托函数`STOP`后便回到了`DelegateCall`下一条指令处

```asm
137 SWAP2
138 POP
139 POP
140 RETURNDATASIZE
141 DUP1
142 PUSH1 00
144 DUP2
145 EQ
146 PUSH2 00b7
149 JUMPI              ;返回值判断，若返回值长度为0，跳转
194 JUMPDEST
195 PUSH1 60
197 SWAP2
198 POP
199 JUMPDEST
200 POP
201 POP
202 SWAP1
203 POP
204 POP
205 STOP                ;结束
```

可以看到在`DelegateCall`之后跟合约源代码一样，就没有其它的操作，直接进入结束流程了，但是结果是，`Storage[0]`的位置，被写入了此刻的`msg.sender`

由此可见，`DelegateCall`，虽然叫做委托调用，但实际上只是引用了外部的指令，并不开辟或引用新内存，在此情况下，若被调用函数的签名可被随意操控，也就意味着攻击者可以编写任意的shellcode在你的合约中执行，篡改你的内存数据

最后把合约代码贴一下子，差点忘了

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Delegate {
    address public owner;

    constructor(address _owner) {
        owner = _owner;
    }

    function pwn() public {
        owner = msg.sender;
    }
}

contract Delegation {
    address public owner;
    Delegate delegate;

    constructor(address _delegateAddress) {
        delegate = Delegate(_delegateAddress);
        owner = msg.sender;
    }

    fallback() external {
        (bool result,) = address(delegate).delegatecall(msg.data);
    }
}
```

合约来源于：[55dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6r3S2W2M7X3&6S2N6i4c8Q4x3X3g2G2M7r3g2F1P5X3g2H3M7r3g2D9K9h3&6Q4x3X3g2U0L8$3#2Q4x3V1j5`.](https://bbs.kanxue.com/elink@6baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6r3S2W2M7X3&6S2N6i4c8Q4x3X3g2G2M7r3g2F1P5X3g2H3M7r3g2D9K9h3&6Q4x3X3g2U0L8$3#2Q4x3V1j5`.)

## 4.写在最后

区块链萌新，关于本文中所列指令内容，我也依然有很多不理解的地方，如有错误，欢迎指出，感谢

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2023-12-15 11:28

被LeaMov编辑

，原因： 修改标题

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
