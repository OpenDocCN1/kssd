
- containerd

<mark class="encrypted">88eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3&6@1j5h3W2F1k6i4u0V1i4K6u0r3j5$3!0F1N6r3q4A6L8X3g2J5k6q4)9J5c8X3u0D9L8$3u0Q4x3V1j5I4x3h3c8W2x3e0W2S2k6U0j5^5j5K6N6V1x3U0q4U0z5r3k6W2x3o6p5H3y4e0R3H3x3U0j5J5y4e0N6W2j5$3b7#2k6o6k6W2k6o6p5K6i4K6u0r3j5$3!0F1N6s2u0A6j5W2)9J5c8X3k6#2P5Y4A6Q4x3V1k6G2M7%4y4Q4y4h3k6X3N6i4A6\*7i4K6g2X3j5Y4g2A6L8r3c8Q4x3X3g2K6K9l9`.`.</mark>

本文介绍了如何使用OSS-fuzz对一些go项目进行模糊测试，oss-fuzz是谷歌提出的一款多引擎的模糊测试平台，该平台以docker为基础，能够实现多种语言的持续模糊测试。Google希望通过“模糊测试（fuzz testing，fuzzing）”为程序提供随机数据输入，作为开源开发的标准部分，oss-fuzz能够针对开源软件进行持续的模糊测试，其测试开发团队也提到“OSS-Fuzz的目的是利用更新的模糊测试技术与可拓展的分布式执行相结合，提高一般软件基础架构的安全性与稳定性。

OSS-Fuzz结合了多种模糊测试技术/漏洞捕捉技术（即原来的libfuzzer）与清洗技术（即原来的AddressSanitizer），并且通过ClusterFuzz为大规模可分布式执行提供了测试环境，当然fuzzer也可以选择AFL,libfuzzer。在oss-fuzz中，go语言由于其运行环境，如网络问题的联通性等问题，相比于C与C++编写的项目，环境搭建较为复杂，本文以容器运行时项目containerd为例，演示如何使用oss-fuzz构建模糊测试句号并介绍了go语言模糊测试中的一个开源项目`go-fuzz-header`。

## OSS-fuzz

**项目地址**：`d43K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2Q4x3V1k6G2M7%4y4Q4x3X3c8X3N6i4A6\*7`

先将oss-fuzz下载到本地，`project`文件夹下列出了集成到oss-fuzz中进行持续模糊测试的项目，目前有将近600个项目，如下图所示：

![1653549118895.png](https://bbs.kanxue.com/upload/attach/202205/921173_DJ5DFFQHHXRN2TJ.jpg)

其中每个项目下有3个主要的文件，分别为`project.yaml`、`Dockerfile`和`build.sh`。

### project.yaml

该文件记录了项目的基本信息，以`containerd`为例，该文件夹下的`project.yaml`如下：

```
homepage: "f9cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3&6@1j5h3W2F1k6i4u0V1i4K6u0r3j5$3!0F1N6r3q4A6L8X3g2J5k6l9."
main_repo: "745K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3&6@1j5h3W2F1k6i4u0V1i4K6u0r3j5$3!0F1N6r3q4A6L8X3g2J5k6l9."
primary_contact: "security@containerd.io"
auto_ccs:
- "adam@adalogics.com"
language: go
fuzzing_engines:
- libfuzzer
sanitizers:
- address
```

- `homepage`：项目地址
- `main_repo`：托管代码的源代码存储库的路径
- `language`：项目编写的编程语言
- `primary_contact`、`auto_ccs`：联系人列表
- `fuzzing_engines`：模糊测试所使用的引擎，如`afl`、`libfuzzer`
- `sanitizer`：消毒剂，支持`ASAN`和`MSAN`，可以有效的提高模糊测试发现crash的概率
- `architectures`：架构列表
- ...

### Dockerfile

`Dockerfile`为项目定义了docker镜像，`build.sh`也将在镜像中运行，`containerd`的`Dockerfile`如下：

```
FROM gcr.io/oss-fuzz-base/base-builder-go
RUN apt-get update && apt-get install -y btrfs-progs libc-dev pkg-config libseccomp-dev gcc wget libbtrfs-dev
RUN git clone --depth 1 https://github.com/containerd/containerd
RUN git clone --depth 1 https://github.com/cncf/cncf-fuzzing
COPY build.sh $SRC/
WORKDIR $SRC/containerd
```

- `FROM`：规定了项目的基本镜像
- `RUN`：构建镜像时执行的命令，首先下载了一些必须的软件和库，之后下载了项目源码
- `COPY`：将`build.sh`复制到镜像中
- `WORKDIR`：指定工作目录
- ...

在Dockerfile中可以看到`cncf-fuzzing`的项目，该项目致力于将CNCF中的开源项目集成到OSS-fuzz中进行持续的模糊测试，如kubernetes、cri-o、runc等。

### build.sh

构建脚本，用来编译项目的，生成的二进制文件应放在`$OUT`中，以示例，一般就是编译和复制语句，示例如下：

```
#!/bin/bash -eu

./buildconf.sh
# configure scripts usually use correct environment variables.
./configure

make clean
make -j$(nproc) all

$CXX $CXXFLAGS -std=c++11 -Ilib/ \
  $SRC/parse_fuzzer.cc -o $OUT/parse_fuzzer \
  $LIB_FUZZING_ENGINE .libs/libexpat.a

cp $SRC/*.dict $SRC/*.options $OUT/
```

以下位置对应的环境变量

- `$OUT` -> `/out`：用来存储构建好的文件
- `$SRC` -> `/src`：放源文件的位置
- `$WORK` -> `work`：存储中间文件的位置

更多的变量可以参考[官方文档](<https://bbs.kanxue.com/elink@58bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8X3!0K6M7#2)9J5k6r3k6#2P5Y4A6Q4x3V1j5>.)

## 环境准备

1. 确保Docker安装成功以及主机能够访问外网。
2. 克隆oss-fuzz项目代码到本地。
3. 设置Dockerd走代理以解决pull镜像时无法访问的问题。oss中用到的镜像都需要从谷歌拉取，有两种解决方法，给docker挂个代理然后直接pull；或者利用github+dockerhub的方法将所有的镜像先拖到dockerhub上，然后将源码中所有的`gcr.io/oss-fuzz-base/xxxx`改成对应dockerhub上的就行；这里使用直接给docker走代理的方式，
   - 首先创建`/etc/systemd/system/docker.service.d/proxy.conf`配置文件，添加以下内容设置代理：

      |  |  |
      | --- | --- |
