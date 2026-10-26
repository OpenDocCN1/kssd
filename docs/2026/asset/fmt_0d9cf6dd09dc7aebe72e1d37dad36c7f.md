
[1b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5$3S2I4i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6o6P5h3u0W2M7V1y4Z5k6h3k6Q4x3V1j5`.](https://bbs.kanxue.com/elink@29eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5$3S2I4i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6o6P5h3u0W2M7V1y4Z5k6h3k6Q4x3V1j5`.)

把响应数据`HEx`拿到这个网站解析

![抓包rsa](https://bbs.kanxue.com/upload/attach/202309/895862_T2RMBA2WEAZKRVE.png)

成功获取解析出数据,左边可以边写`.proto`边查看右边解析结果,

ok第一个包完成文件编写

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 | `syntax` `=` `"proto3"` `;` `package Protoc;` `option go_package` `=` `"."` `;` `//响应` `message Rsa_Response{` `string PublicKey` `=` `1` `;` `//公钥` `string MoShu` `=` `2` `;` `//模数` `int64 SuiJiShu` `=` `3` `;` `//随机数` `}` |

运行 `protoc --go_out=./__/ *.proto` 生成`go`代码

# 第一个包取RSA公钥

顺便看一下`url`内的两个参数,第一个固定,第二个是`base64`编码后的用户名,就不细说了

然后编写调用测试一下

|  |  |
| --- | --- |
