
```text
################1################
# 编译Android7 报错：
# FAILED: /bin/bash -c "prebuilts/misc/linux-x86/flex/flex-2.5.39 -oout/host/linux-x86/obj/STATIC_LIBRARIES/libaidl-common_intermediates/aidl_language_l.cpp system/tools/aidl/aidl_language_l.ll"
# 解决方法： 在android目录下输入命令：
export LC_ALL=C
# 再重新make
#用 export LC_ALL=C 来去除所有本地化的设置，让命令能正确执行
################2#################
# 下载源码报错
# curl 92 HTTP/2 stream 3 was not closed cleanly before end of the underlying
git config --global http.version HTTP/1.1
git config --global http.postBuffer 524288000
# LibreSSL SSL_connect: SSL_ERROR_SYSCALL 报错
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --global http.sslBackend "openssl"
git config --global http.sslVerify false
# fatal: unable to access '95bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8%4y4H3i4K6u0W2N6s2g2F1j5g2)9J5k6i4c8K6K9h3&6Y4K9s2g2S2i4K6u0W2k6h3c8#2i4K6u0W2j5$3&6Q4x3V1k6H3L8r3q4@1k6X3!0J5L8g2)9J5c8X3#2S2L8X3W2X3k6i4y4@1i4K6u0r3i4K6t1%4i4K6y4m8 server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none
git config --global http.sslverify false
###################################
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs
/Library/Developer/CommandLineTools/SDKs
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2022-10-26 19:27 被iyue\_t编辑，原因：

- [#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)
- [#系统相关](https://bbs.kanxue.com/forum-161-1-126.htm)
- [#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)
