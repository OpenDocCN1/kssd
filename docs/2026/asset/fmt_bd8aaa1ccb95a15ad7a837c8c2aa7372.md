
还原后加密数据与SDK内存中加密数据相同，如图6-1所示：

![](https://bbs.kanxue.com/upload/attach/202112/536985_N4D2EZ8J6UZTYKC.jpg)

图6-1

#### 6.2、签名算法

```c
sha256_init(&ctx);
sha256_update(&ctx, text, sizeof(text));
sha256_final(&ctx, buf);

ByteToHexStr(buf, strbuf, 32);
printf("sha256 :%s\n", strbuf);

int AliTigerTally_base64(BYTE* data, int len, BYTE* outdata, int mod) {
BYTE buf[1024];
size_t buf_len;
int pass = 1;
int idx;
if (mod == 1) {
buf_len = base64_encode(data, outdata, len, mod);
}
if (mod == 0) {
buf_len = base64_decode(data, outdata, len);
}

return buf_len;
}
```

### 七、总结

#### 技术角度：

对于这种高强度混淆的SDK逆向分析难度还是比较高的，主要分为三步，一是解密`appkey`，二是获测设备风险与获取设备指纹，三进行签名。

采集设备异常风险特征包括：

使用模拟器、使用代理、Root设备、调试模式、App被hook、App多开。

其中采集设备信息与加解密算法都是通过混淆的，如果对常见算法逻辑不熟悉，要完整还原算法是需要花费一定的时间。整体来讲安全度还是比较高。

但是强混淆与多重反射会影响效率、代码重复率高、体积大。

#### 业务角度：

我个人理解从老版本到新版本更新迭代这个业务比较成熟才是，但是从注册到获取SDK集成到配置策略发布大概需要2-3天的时间，其实整个流程下来完全可以实现自动化操作与自检验，节省时间的同时提升体验与更快的响应，但是期间我需要多次联系WAF技术支持人员获取App防护SDK包且反响时间较长。

版本变更内容未说明，黑盒状态，新增功能？功能调整？BUG修复？可能是因为整体架构都是全新开发的原因吧。

样本获取方式，关注公众号，公众号输入框回复“att” 获取下载链接。

![](https://bbs.kanxue.com/upload/attach/202112/536985_6FQVME6GX3FBZRU.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)
