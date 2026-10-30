
![图片描述](https://bbs.kanxue.com/upload/attach/202606/943869_TNTJ7UMRXCW9E5.webp)

`EvilAgent.exe` 是 [SystemGap](https://bbs.kanxue.com/elink@4d4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9N6X3^5H3P5s2y4&6i4K6u0r3f1%4W2K6N6r3g2E0c8$3q4H3) 项目里的 `SystemGapAll`。其同样也借助命名管道实现高低权限进程间的通信，低权限向高权限发送要执行的命令，高权限执行并把结果返回给低权限。

![图片描述](https://bbs.kanxue.com/upload/attach/202606/943869_CK2RCQ8PH3DXBYY.webp)

## 总结

本提权利用链的成功构建，关键在于命名管道访问控制配置不当与服务可执行文件权限配置错误这两个缺陷的组合利用，在实战过程中发现类似的问题很多，是个值得关注的攻击面。最后，在此特别感谢 [@倾旋](https://bbs.kanxue.com/elink@c1dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4W2D9L8$3q4V1M7#2)9J5k6h3!0F1L8r3W2F1k6g2)9J5c8R3`.`.) 在漏洞挖掘过程中提供的协助。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
