
# [原创]第二题 `ctf2017_Fpc.exe` 分析

1. 在输入框输入 `12345678000011A` 溢出返回地址到 `00413131`

   ![](https://bbs.kanxue.com/upload/attach/201710/728969_hqlnb2u9xp6vyiz.png)

2. 从 `00413131` 处开始跟，这里开始花指令，单步跟

   ![](https://bbs.kanxue.com/upload/attach/201710/728969_vhyajsqabl4qts7.png)

3. 扣出三处计算指令

   ![](https://bbs.kanxue.com/upload/attach/201710/728969_brfsenddzoibyeh.png)

4. 得出三元一次方程组，解出得前十二位字符 `Just0for0fun`

   ![](https://bbs.kanxue.com/upload/attach/201710/728969_iwvmzw40rj3r15g.png)

5. 加上 `11A`，最终答案：`Just0for0fun11A`

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
