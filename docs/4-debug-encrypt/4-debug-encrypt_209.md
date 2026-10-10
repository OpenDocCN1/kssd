

# AZR

> 原文：[`www.pediy.com/kssd/pediy04/kanxue208.htm`](https://www.pediy.com/kssd/pediy04/kanxue208.htm)

AZR 注册流程分析及疑问：   
    1.注册号形式为：ZR-765432-987654-123456   
    2.由注册号得到字符 String1：654329875412345   
    3.String1 每一位和 0x43 异或，得到字符串 String2：uvwpqz{tvwrqpwv   
    4.BlowFish\_En(String1,key=String1)，输出级连设为 String3   
    5.软件内部自定义字串 String4：0xC000C5FD0F4F7BE05BF868FCFF9B12   
    6.可以看出，String3 的长度为 16 个字节，而 String4 的长度只有 15 个字节，   
      该软件只要求 String3 前面的 15 个字节和 String4 完全一致就可，最后 1 个   
      字节不参与比较。   
    现在的问题是：如何找出这个算法的逆算法。   
    我觉得软件只比较前 15 个字节，可能会减小穷举的难度，但是我还未编程穷举。   
    哪位大虾给个思路，如何求出满足要求的序列号。   
    BTW：就我的跟踪过程来看，序列号和名字无关  4dd466d0e1e0cc123afbe504da2462ca