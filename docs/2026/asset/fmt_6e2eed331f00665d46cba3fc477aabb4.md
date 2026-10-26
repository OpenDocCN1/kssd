
- 一堆加密算法, 就不一一截图了.

  ![image-20200922223914801](https://bbs.kanxue.com/upload/attach/202009/902265_4KH7K5YU5NQRWPG.jpg)

- 可以看出有很多`jxx`跳转到`001BAD83`的`xor eax, eax`然后调用`分析3(0024B238)`处, 上文已分析此函数对`eax`不影响. 所以关键点就是

  - `001BAD7C: mov eax, 1`
  - `001BAD83: xor eax, eax`

- 为了验证, 将`jxx` `001BAD83`的代码全部干掉. 一共三处

  ![image-20200922224605136](https://bbs.kanxue.com/upload/attach/202009/902265_JFCS5E7P7VFC7RU.jpg)

8. 注册激活分析就此告一段落

   ![image-20200922224734720](https://bbs.kanxue.com/upload/attach/202009/902265_WC8YHTK8BRXG3WJ.jpg)

总结:

- 函数(`001BAC60`) 是校验函数, 程序内部的多次校验都是调用的此函数以次函数的`eax`结果识别身份是否有效. 很容易绕过.

### 3.2 身份识别

1. 先监视注册表相关API, 在以下函数下断, 然后程序跑起来

   - `RegOpenKeyW`
   - `RegOpenKeyExA`
   - `RegOpenKeyExW`

   ![image-20200922230207411](https://bbs.kanxue.com/upload/attach/202009/902265_4MADN52M56FHEHU.jpg)

2. `013F7DFF` 处为系统注册表, 不理会.

   ![image-20200922231507221](https://bbs.kanxue.com/upload/attach/202009/902265_FEMKG8XESMXY93D.jpg)

3. `012B6F25` 打开失败, 也不理会

   ![image-20200922231719345](https://bbs.kanxue.com/upload/attach/202009/902265_TVT47ZU5NQ3VPR2.jpg)

4. 继续排出不关注的注册表断点, 此处就比较可疑.

   ![image-20200922232807968](https://bbs.kanxue.com/upload/attach/202009/902265_UHXCUVCNE9TGPB9.jpg)

5. 后面紧接着就是注册表的查询, 可疑

   ![image-20200922232907414](https://bbs.kanxue.com/upload/attach/202009/902265_AB8QB3C7A5PM3C7.jpg)

6. 继续运行发现另一处也断下来了, 而且查询了相同的`ValueName = License 5`, 但跟结点不同.

   ![image-20200922233856347](https://bbs.kanxue.com/upload/attach/202009/902265_P34R3UGTRPBAF9R.jpg)

7. 基于此搜索注册表 `ES-Computing`, 找到三处 `?\Software\ES-Computing\EditPlus\Install\License xx`, 全部删除测试

   1. `HKEY_LOCAL_MACHINE`
   2. `HKEY_CURRENT_USER`
   3. `HKEY_USERS`

8. 分析完毕, 身份是以加密形式存放在注册表中.

   ![image-20200923000153850](https://bbs.kanxue.com/upload/attach/202009/902265_3E9QD6Y5SZEYYNE.jpg)

总结:

- 软件怎么识别身份这里, 运气成分较大所以到这里好像没什么可以总结的.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2020-9-23 12:38

被scanxx编辑

，原因：
