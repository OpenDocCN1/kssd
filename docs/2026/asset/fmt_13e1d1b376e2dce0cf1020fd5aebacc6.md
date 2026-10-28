
至此病毒程序的运行流程分析完毕，加密数据部分没有详细分析，自己关于算法方面的逆向能力不太足，怕分析起来浪费时间，得不偿失

---

# 五、病毒行为流程总结：

![](https://bbs.kanxue.com/upload/attach/202001/774842_CVQ3WAJP3U9GFFB.jpg)

# 六、技术思考总结：

1. 通过PEID查壳发现程序带壳的话不要慌，判断是已知壳还是未知壳，已知壳的话查阅网上相关资料手工或者寻找脱壳机帮助自己脱壳，未知的话单步分析，合理下断，注意观察内存变化

2. 对于某些影响我们分析运行的函数，要大胆的`NOP`和跳过

3. 多熟悉不同病毒种类，远控类，勒索类，挖矿类等等，下次遇到时，分析起来速度会变快，效率提高

4. 算法加密部分不仅要熟悉常见的Windows系统函数，更要知道相关原理，自己能力有待提高

# 七、防护建议：

1. 个人电脑、服务器应及时打补丁，修复漏洞

2. 对重要的数据文件定期进行非本地备份

3. 不要点击来源不明的邮件附件，不从不明网站下载软件

4. 尽量关闭不必要的文件共享权限

5. 更改账户密码，设置强密码，避免使用统一的密码

6. GandCrab勒索软件会利用RDP(远程桌面协议），如果业务上无需使用RDP的，建议关闭RDP

# 八、相关IOC：

- http[:]//drlarrybenovitz.com/qhcka/templates/binarystings.php
- http[:]//holishit.in/wp-content/plugins/wpclef/assets/src/sass/neat/grid/binarystings.php
- http[:]//starsoftheworld.org/cgi-bin/binarystings.php
- http[:]//hknsjsu.org/wp-content/plugins/profile-builder/assets/lib/wck-api/assets/js/tiny\_mce/utils/binarystings.php
- http[:]//vtc360.com/wp-content/themes/vtc360\_maxf3d/ReduxFramework/ReduxCore/inc/extensions/wbc\_importer/demo-data/Demo2/binarystings.php
- http[:]//grosirkecantikan.com/wp-content/plugins/contact-form-7/includes/js/jquery-ui/themes/smoothness/images/binarystings.php
- http[:]//88fga.ketteaero.com/FE4A8D7E1E829B86
- http[:]//2bdfb.spinakrosa.at/FE4A8D7E1E829B86
- http[:]//uj5nj.onanwhit.com/FE4A8D7E1E829B86

MD5：9d3dbe44445624abb04bdb3bdace4fc2

SHA-256：7829ae7240eb4c6f5fe33c5af43e6d388f0439011c27cf609cc4d1a8b6092ec1

SHA-1：0ffc994e247d2b108910e8bb6de3ef57439c51af

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-5-17 16:50 被kanxue编辑，原因：

上传的附件：

- [样本分析.zip](javascript:void(0);)（3.48MB，75次下载）
