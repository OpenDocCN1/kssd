
如今，移动技术的使用在呈指数增长。由于大多数应用程序都需要与外部服务进行通信，移动应用程序市场正在产生新的安全威胁。实际上，不安全的通信在 OWASP Top 10 中排名第3，因此这也是一个非常重要的课题。当前，应用程序的使用至少与访问互联网一样普遍，保护这些应用程序以确保信息的机密性和完整性至关重要。在这一方面，我们向应用程序开发人员和渗透测试人员提供了在 Android 应用程序通信中加强安全性的指南。

在这篇论文中，我们展示了即使是使用了 SSL pinning 技术，SSL/TLS 的实现也是容易遭受攻击的。这样，必须采取一些措施来保护应用程序免受 SSL pinning 绕过。在应用程序的实现阶段，必须实现 root 检测、调试检测和防篡改措施，最后代码也必须要被混淆。

我们还研究了绕过这些对策的方法。首先，我们研究了现有的绕过 SSL pinning 的技术。作为安全审计人员和顾问的一条业务线，绕过技术在专业领域内是众所周知的，但在学术界很少看到它们。因此，我们试图填补这一研究的空白。这篇论文主要的贡献和价值在于介绍了五种已知的方法，并增加了两种新的绕过 SSL pinning 的方法，提出了一些开发人员可以用来避免这种情况的解决方案，并开发了一个适用性的框架。这个框架对开发人员和安全审计人员来说像指南一样有用。

一共介绍了5种方法：更改证书中的 CM、在应用程序中嵌入 CA、使用 SSL Unpinning、调试应用程序和修改应用程序的可执行文件。然后，我们提出了两种新的方法来来绕过 Android 设备中的SSL/TLS验证。第一种方法用在 rooted 设备上，第二种方法用在 non-rooted 设备上。最后，我们设计了一组应用程序的实验来评估我们的建议。实验分为两个阶段：（1）检查 app 是否有 SSL pinning 和是否包含任何安全控制；（2）检查本文提出的七种方法。我们使用的应用程序包括十个广泛应用于不同领域的应用程序。

结果显示了 Android 版本的应用程序是否实现了与 iOS 版本一样的安全控制。此外，新的绕过方法可以作为旧绕过方法的补充。

最后，我们提出了一个适用性框架作为我们研究的一个重要贡献。该框架旨在帮助开发人员和审计人员正确开发和检查安全通信，作为 OWASP 指南一个有用的补充。我们的框架有助于简化 Android 移动应用程序的开发和审计过程。适用性框架提供了一个具有两个主要目标的指南：（1）学习 SSL pinning 的绕过方法来检查安全控制应该在什么时候实现；（2）学习实现安全控制来对抗 SSL pinning 绕过方法。从开发人员和安全审计人员的角度来看，这个框架都很很有帮助。此外，适用性框架表明需要实现所有建议的安全控制来停止所有的 SSL pinning 绕过的方法。适用性框架的评估将是我们以后研究的一部分。

现在已经完成了演示如何绕过 SSL pinning来攻击通信的完整性，我们也提供了一些机制来防止这些攻击。

# 作者贡献

F.J.R.-L. 在论文的所有阶段都做出了贡献；A.J.V.-V. 构思和设计了实验，分析看结果并撰写论文； J.R., J.L., 和 A.C. 分析了数据和结果并撰写了论文。

# 赞助

这项工作由塞维利亚大学"红色情报中心"资助，部分资金由西班牙科学和技术部通过ECLIPSE（RTI2018-094283-B-C33）提供，安达卢西亚委员会通过 METAMORFOSIS 项目和欧洲区域发展基金（ERDF/FEDER）提供。

# 利益冲突

所有作者都同意了手稿的最后内容，作者未报告潜在的利益冲突。

# 免责声明

在这项工作中进行的实验是严格为学术目的进行的。

# 引用

1. Li, D.; Guo, B.; Shen, Y.; Li, J.; Huang, Y. The evolution of open-source mobile applications: An empirical study. J. Softw. Evol. Process. 2017, 29, e1855. [Google Scholar] [CrossRef]
2. Unal, P.; Temizel, T.T.; Eren, P.E. What installed mobile applications tell about their owners and how they affect users’ download behavior. Telemat. Inform. 2017, 34, 1153–1165. [Google Scholar] [CrossRef]
3. Kumar, R.; Perti, A. Security issues with self-signed SSL certificates. Int. J. Innov. Technol. Explor. Eng. (IJITEE) 2019, 8, 7S2. [Google Scholar]
4. Lindgren, A.; Lindoff, B. On Estimating the Number of Worldwide LTE Cell-IDs and WiFi Aps. 2018. Available online: <mark class="encrypted">7bfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3#2T1j5h3W2F1i4K6u0W2j5$3!0E0i4K6u0r3N6i4m8D9L8$3q4V1M7#2)9J5c8W2N6Z5K9i4c8W2M7r3q4H3k6i4u0Q4y4h3k6i4L8%4u0D9k6q4N6A6k6r3g2Q4y4h3k6x3g2p5g2Q4y4h3k6o6k6h3I4D9d9f1c8Q4y4h3k6S2L8X3c8Q4y4h3k6i4K9f1k6A6i4K6g2X3b7g2m8K6i4K6g2X3b7g2)9J5k6i4m8V1k6W2!0o6x3W2!0m8x3q4)9J5z5r3q4U0j5$3g2K6M7$3g2V1</mark> on 3 September 2019).
5. Anthi, E.; Theodorakopoulos, G. Sensitive data in Smartphone Applications: Where does it go? Can it be intercepted? In International Conference on Security and Privacy in Communication Systems; Springer: Berlin/Heidelberg, Germany, 2017; pp. 301–319. [Google Scholar]
6. Khan, J.; Abbas, H.; Al-Muhtadi, J. Survey on mobile user’s data privacy threats and defense mechanisms. In Proceedings of the 12th Iberian Conference on Information Systems Technolo-Gies (CISTI), Lisbon, Portugal, 14–17 June 2017. No. 7975981. [Google Scholar]
7. D’Orazio, C.J.; Choo, K.-K.R. A technique to circumvent SSL/TLS validations on iOS devices. Future Gener. Comput. Syst. 2017, 74, 366–374. [Google Scholar]
8. Razaghpanah, A.; Sundaresan, S.; Niaki, A.A.; Amann, J.; Vallina-Rodriguez, N.; Gill, P. Studying TLS usage in Android apps. In Proceedings of the 13th International Conference on Emerging Technologies (CoNEXT 2017), Ingeon, Korea, 12–15 December 2017; pp. 350–362. [Google Scholar]
9. Fahl, S.; Harbach, M.; Perl, H.; Koetter, M.; Smith, M. Rethinking SSL development in an appified world. In Proceedings of the ACM SIGSAG Conference on Computer & Communications Security (CCS 2013), Berlin, Germany, 4–8 November 2013; pp. 49–60. [Google Scholar]
10. De los Santos, S.; Torres, J. Analysing HSTS and HPKP implementation in both browsers and servers. IET Inf. Secur. 2017, 12, 275–284. [Google Scholar] [CrossRef]
11. Mueller, B.; Schleier, S. OWASP Mobile Application Security Verification Standard v 1.1.4. Available online: <mark class="encrypted">9b3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2G2N6$3q4K6M7q4)9J5k6h3!0J5k6#2)9J5c8X3W2F1k6r3g2^5i4K6u0W2M7r3S2H3i4K6u0r3e0#2N6m8f1#2m8Q4y4h3k6y4L8$3u0A6L8r3g2Q4y4h3k6e0k6h3y4#2M7X3W2@1P5g2)9#2k6W2c8W2M7%4c8A6L8X3N6Q4y4h3k6s2N6h3W2V1k6g2!0o6x3W2!0m8x3q4)9J5z5r3q4U0j5$3g2K6M7$3g2V1</mark> on 21 November 2019).
12. Dhawale, C.A.; Misra, S.; Jambhekar, N.D.; Thakur, S.U. Mobile computing security threats and solution. Int. J. Pharm. Technol. 2016, 8, 23075–23086. [Google Scholar]
13. OWASP Mobile Top 10. 2016. Available online: <mark class="encrypted">0caK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2G2N6$3q4K6M7q4)9J5k6h3!0J5k6#2)9J5c8X3W2F1k6r3g2^5i4K6u0W2M7r3S2H3i4K6u0r3e0h3!0T1K9h3I4W2i4K6g2X3g2r3!0H3i4K6g2X3x3e0m8Q4y4h3j5J5x3o6p5$3i4K6u0V1g2r3!0H3i4K6g2X3x3e0m8Q4b7K6u0Q4b7e0m8Q4x3U0S2S2j5$3y4W2M7%4y4W2k6l9`.`.</mark> on 22 February 2017).
14. Kim, S.; Han, H.; Shin, D.; Jeun, I.; Jeong, H. A study of International Trend Analysis on Web Service Vulnerabilities in OWASP and WASC. In Proceedings of the 3rd International Conference on Information Security and Assurance (ISA 2009), Seoul, Korea, 25–27 June 2009; Springer: Heidelberg, Germany; Volume 5576, pp. 788–796. [Google Scholar]
15. Szczepanik, M.; Jozwiak, I. Security of mobile banking applications. Adv. Intell. Syst. Comput. 2018, 635, 412–419. [Google Scholar]
16. Hickman, K. The SSL Protocol; Netscape Communications Corp: Mountain View, CA, USA, 1995. [Google Scholar]
17. Dierks, T.; Rescorla, E. The TLS Protocol Version 1.2. RFC 5246. Available online: <mark class="encrypted">4bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3M7X3k6U0y4e0t1@1y4W2!0o6x3W2!0m8x3q4)9J5z5r3q4U0j5$3g2K6M7$3g2V1</mark> on 21 November 2019).
18. Gu, X.; Gu, X. On the detection of fake certificates via attribute correlation. Entropy 2015, 17, 3806–3837. [Google Scholar] [CrossRef]
19. Varela-Vaca, A.J.; Gasca, R.M. Towards the automatic and optimal selection of risk treatments for business processes using a constraint programming approach. Inf. Softw. Technol. 2013, 55, 1948–1973. [Google Scholar] [CrossRef]
20. Oracle—Java Secure Socket Extension (JSSE) Reference Guide. 2018. Available online: <mark class="encrypted">256K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8%4u0S2j5$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3K9X3q4$3j5i4y4W2i4K6u0r3z5q4)9J5c8X3c8G2j5%4y4Q4x3V1k6@1k6h3y4Z5L8X3!0@1k6i4y4Q4x3V1k6Y4N6h3W2V1k6i4y4Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5c8X3A6K6M7$3g2Q4x3V1k6v1f1#2y4q4f1X3g2X3c8%4g2A6k6r3g2Q4x3X3g2Z5N6r3#2D9i4@1x3J5i4@1p5H3i4K6t1^5j5h3y4U0k6i4y4K6k6h3b7`.</mark> on 3 September 2019).
21. OpenSSL. Available online: <mark class="encrypted">197K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2G2M7r3g2F1M7%4y4D9i4K6u0W2L8%4u0Y4i4K6u0r3i4@1x3J5i4@1p5H3i4K6t1^5j5h3y4U0k6i4y4K6k6h3b7`.</mark> on 3 September 2019).
22. LibreSSL. Available online: <mark class="encrypted">fc5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3I4A6j5Y4u0W2M7%4y4D9i4K6u0W2L8%4u0Y4i4K6u0r3i4@1x3J5i4@1p5H3i4K6t1^5j5h3y4U0k6i4y4K6k6h3b7`.</mark> on 3 September 2019).
23. GNUTLS. Available online: <mark class="encrypted">e7fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Y4L8Y4g2@1L8s2y4Q4x3X3g2G2M7X3N6Q4x3V1k6Q4b7K6u0Q4b7e0m8Q4x3U0S2S2j5$3y4W2M7%4y4W2k6l9`.`.</mark> on 3 September 2019).
24. Al-Qershi, F.; Al-Qurishi, M.; Md Mizanur Rahman, S.; Al-Amri, A. Android vs. iOS: The security battle. In Proceedings of the 2014 World Congress on Computer Applications and Information Systems (WCCAIS), Hammamet, Tunisia, 17–19 January 2014; pp. 1–8. [Google Scholar] [CrossRef]
