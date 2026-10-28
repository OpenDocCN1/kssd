
![](https://bbs.kanxue.com/upload/attach/202201/777502_T4GDUVP2AXH44YJ.jpg)![](https://bbs.kanxue.com/upload/attach/202201/777502_3FWVTZ6KJSG34MH.jpg)
```
# 应用对数变换

df_train['SalePrice'] = np.log(df_train['SalePrice'])



# 变换后的直方图和正态概率图

sns.distplot(df_train['SalePrice'], fit=norm);

fig = plt.figure()

res = stats.probplot(df_train['SalePrice'], plot=plt)
```
![](https://bbs.kanxue.com/upload/attach/202201/777502_3CBT2NMNN2MT3XS.jpg)![](https://bbs.kanxue.com/upload/attach/202201/777502_KMTQ8X6YZCVMMJC.jpg)
```
# grlivarea在对数变换前的直方图和正态概率图

sns.distplot(df_train['GrLivArea'], fit=norm);

fig = plt.figure()

res = stats.probplot(df_train['GrLivArea'], plot=plt)
```
![](https://bbs.kanxue.com/upload/attach/202201/777502_Z6CR4SV6WXXKVCJ.jpg)![](https://bbs.kanxue.com/upload/attach/202201/777502_38BC9AATUJ3X4E3.jpg)
```
# 应用对数变换

df_train['GrLivArea'] = np.log(df_train['GrLivArea'])



# grlivarea做了对数变换后的直方图和正态概率图

sns.distplot(df_train['GrLivArea'], fit=norm);

fig = plt.figure()

res = stats.probplot(df_train['GrLivArea'], plot=plt)
```
![](https://bbs.kanxue.com/upload/attach/202201/777502_GPYWSCN4UU6FJ4W.jpg)![](https://bbs.kanxue.com/upload/attach/202201/777502_VTDWC4MKGNTBQ2K.jpg)
```
# totalbsmtsf在做对数转换前的直方图和正态概率图

sns.distplot(df_train['TotalBsmtSF'], fit=norm);

fig = plt.figure()

res = stats.probplot(df_train['TotalBsmtSF'], plot=plt)
```
![](https://bbs.kanxue.com/upload/attach/202201/777502_8K752PJSYAGNKG4.jpg)![](https://bbs.kanxue.com/upload/attach/202201/777502_WVMAZB6N3NBXGUQ.jpg)
```
# totalbsmtsf这项数据有些特殊，因为有很多0值，而0是没有对数的，所以不能进行对数转换

# 但是有许多非零的值是可以转换的，所以我们需要暂时忽略零值

df_train['HasBsmt'] = pd.Series(len(df_train['TotalBsmtSF']), index=df_train.index)

df_train['HasBsmt'] = 0

df_train.loc[df_train['TotalBsmtSF']>0,'HasBsmt'] = 1



# 应用对数变换

df_train.loc[df_train['HasBsmt']==1,'TotalBsmtSF'] = np.log(df_train['TotalBsmtSF'])



# 转换后的方直图和正态概率图

sns.distplot(df_train[df_train['TotalBsmtSF']>0]['TotalBsmtSF'], fit=norm);

fig = plt.figure()

res = stats.probplot(df_train[df_train['TotalBsmtSF']>0]['TotalBsmtSF'], plot=plt)
```
![](https://bbs.kanxue.com/upload/attach/202201/777502_EYRTATYGY6GZSQR.jpg)![](https://bbs.kanxue.com/upload/attach/202201/777502_PBH86QEJ38ZMTC3.jpg)
```
# 之前的grlivarea与saleprice的图是类似于圆锥的形状（同质问题严重）

# 做了正态处理后，散点图就不再有圆锥型状了（解决了同质问题）

plt.scatter(df_train['GrLivArea'], df_train['SalePrice']);
```
![](https://bbs.kanxue.com/upload/attach/202201/777502_A9DMJD2XHNKYMN8.jpg)
```
# 对totalbsmtsf做正态处理后，同质问题也得到了解决

plt.scatter(df_train[df_train['TotalBsmtSF']>0]['TotalBsmtSF'], df_train[df_train['TotalBsmtSF']>0]['SalePrice']);
```
![](https://bbs.kanxue.com/upload/attach/202201/777502_PQQA92M8T3HRJWN.jpg)
```
# 将分类变量转换为虚拟变量

df_train = pd.get_dummies(df_train)
```
剩下的就和我自己的代码一样了，就是用随机森林做二分类判断，然后将训练好的模型直接对`test.csv`进行预测，然后将预测好的结果输出到`submission.csv`，这里不知道为什么没有写下去

# 结尾

诸如此类二分类任务都是先分析每个特征和要预测的值之间的关系，然后对特征进行增强，需要注意的是泰坦尼克和本次的房价预测用的都是随机森林（线性），还有用`tanh`或者`relu`（逻辑）的情况同样值得学习

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

[#开发技巧](https://bbs.kanxue.com/forum-41-1-135.htm)

上传的附件：

- [`comprehensive-data-exploration-with-python.ipynb`](javascript:void(0);)

  （1.13MB，18次下载）

- [`my-house-price.ipynb`](javascript:void(0);)

  （13.61kb，15次下载）

- [`house-prices-advanced-regression-techniques.zip`](javascript:void(0);)

  （199.03kb，17次下载）
