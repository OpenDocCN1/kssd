
# [原创]ida81输入密码验证算法分析以及破解思路

本文分析了`ida81`对输入密码的验证流程，分别对输入密码到生成解密密钥、密码素材的生成过程以及文件数据的加密过程这三个流程进行分析，并尝试找一些可利用的破绽。很遗憾，由于水平有限，目前也只是有个思路未能完全实现，故分享出来抛砖引玉。

## 1. 密码验证算法

```
proc m4ZQu {password LZQjH} {
set ::maui::util::H6VeX ""
set ::maui::util::EL0BW [string repeat \\0 32]
package require tcltwofish
package require sha256
if {[string length $LZQjH] == 0} {
error "Unable to initialize encryption - missing essential data"
}
//1.展开payload 获取相关参数
if {[binary scan $LZQjH Ia16a32a64a32a\* times iv passwordKey encryptedKey payloadIVsHash encryptedPayloadIVs] < 6} {
error "Unable to initialize encryption - header missing or corrupt"
}
//2.使用输入的密码计算sha256，然后根据迭代次数times加密passwordkey，最后计算结果的sha256
set iHcWR [a64bL $password $passwordKey $iv $times]
//3.取上一步中的后32字节
set BJvoG [string range [tcltwofish::decrypt $iHcWR $encryptedKey] 32 63]
//4.使用上一步中的结果作为key 循环times/64次 解密encryptedPayloadIVs
//这一步没有指定iv 实际过程为16个字节的0x00
for {set i 0} {$i < $times} {incr i 64} {
set encryptedPayloadIVs [tcltwofish::decrypt $BJvoG $encryptedPayloadIVs]
}
//5.获取32字节后的数据
set ZQpQw [string range $encryptedPayloadIVs 32 end]
//6.计算上一步结果的sha256，和预置的hash比对，一致则密码正确
if {[sha2::sha256 -bin $ZQpQw] != $payloadIVsHash} {
maui::d\_RUj "Invalid payload password"
}
//7.使用第五步中的结果作为后续选取iv的数据
set ::maui::util::H6VeX $ZQpQw
//8.使用第三步中的结果作为密钥
set ::maui::util::EL0BW $BJvoG
}
```

在这个过程中，第一步需要的`payload`为`4020`字节，这是已知的。意味着可以获取到第六步中正确的`hash`结果。

然而，根据下面的帖子总结，`ida`的密码选取字符为`58`个，密码长度为`14`个字符，这个测试结果是非常大的。

<mark class="encrypted">f53K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6U0L8#2)9J5k6i4u0W2i4K6u0r3j5X3I4G2k6#2)9J5c8U0t1H3x3e0W2Q4x3V1j5H3y4W2)9J5c8U0t1I4i4K6u0r3L8%4m8W2M7X3q4@1K9h3!0F1i4K6u0V1j5%4u0S2j5$3E0Q4x3X3c8Z5j5h3y4C8K9h3&6Y4i4K6u0V1d9f1c8m8i4K6u0V1f1s2u0G2i4K6u0V1K9h3&6K6N6r3q4D9L8r3g2J5i4K6u0V1f1q4u0z5c8#2)9J5k6r3k6J5L8$3#2Q4x3X3c8S2L8W2)9J5k6s2g2F1N6i4y4#2j5h3I4Q4x3X3c8%4j5i4W2Q4x3V1j5`.</mark>

https://bbs.kanxue.com/thread-277712-1.htm

而我在`i9-13900K`测试从随机生成输入密码到第六步验证`hash`整个过程，测试一个密码就需要`0.045`秒。所以手里有超算也不建议直接爆破。

## 2. 密码素材的生成过程

```
//根据输入密码生成payload
proc yOZ7q {password {times 16000}} {
set ::maui::util::H6VeX ""
set ::maui::util::EL0BW [string repeat \\0 32]
package require tcltwofish
package require sha256
set BJvoG [qA4kM 32]
set passwordKey [qA4kM 32]
set ZQpQw [qA4kM [expr {16 \* 256}]]
set payloadIVsHash [sha2::sha256 -bin $ZQpQw]
set encryptedPayloadIVs "[qA4kM 32]$ZQpQw"
for {set i 0} {$i < $times} {incr i 64} {
set encryptedPayloadIVs [tcltwofish::encrypt $BJvoG $encryptedPayloadIVs]
}
set iv [qA4kM 16]
set iHcWR [a64bL $password $passwordKey $iv $times]
set encryptedKey [tcltwofish::encrypt $iHcWR "[qA4kM 32]$BJvoG"]
set LZQjH [binary format Ia16a32a64a32a\* $times $iv $passwordKey $encryptedKey $payloadIVsHash $encryptedPayloadIVs]
set ::maui::util::H6VeX $ZQpQw
set ::maui::util::EL0BW $BJvoG
return $LZQjH
}
//生成随机数据
proc qA4kM {length} {
set YKL8w ""
if {[RLPut unix]} {
foreach xML4B {/dev/urandom /dev/random} {
if {[catch {
set WyLEv [open $xML4B r]
fconfigure $WyLEv -translation binary
set YKL8w [read $WyLEv $length]
close $WyLEv
}]} {
catch {close $WyLEv}
} else {
break
}
}
} else {
}
if {[string length $YKL8w] != $length} {
incr length -[string length $YKL8w]
while {$length >= 2} {
append YKL8w [binary format S [expr {int([maui::util::rand] \* 0x10000)}]]
incr length -2
}
if {$length > 0} {
append YKL8w [binary format c [expr {int([maui::util::rand] \* 0x100)}]]
}
}
return $YKL8w
}
proc rand {} {
if {[maui::util::RLPut "windows"] && [info commands ::bitrock::secure::rand\_s] != ""} {
if {[catch {
set r [::bitrock::secure::rand\_s]
} V77Ls]} {
maui::util::debug "failed to call rand\_s: $V77Ls"
set r [expr {rand()}]
}
} else {
set r [expr {rand()}]
}
return $r
}
```

继续分析输入密码的时生成`payload`的算法，根据安装密码生成`payload`的过程中，很多数据包括最后解密数据用的密钥`BJvoG`也是随机生成的，如果这部分随机数生成流程能复现那也是可以破解的。仔细看随机数的生成，已经排除了以前的`LCG`随机数生成算法，换成了更安全的，前后状态无影响的方式，`linux`或`unix`下用`/dev/random`或者`/dev/urandom`，`windows`下调用`rand_s`。

<mark class="encrypted">187K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8Y4A6Z5i4K6u0V1j5$3&6Q4x3V1k6U0M7s2m8Q4x3V1k6U0i4K6u0V1M7Y4g2F1N6r3W2E0k6g2)9J5k6r3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6J5k6h3k6W2M7X3g2F1j5$3g2Q4x3V1k6J5j5h3&6V1i4K6u0V1M7#2)9K6c8Y4k6A6k6i4N6Q4x3@1c8E0M7%4k6U0i4K6u0V1x3e0M7H3</mark>

## 3. 文件数据加密流程

```
proc MmyBM {compressionAlgorithm command data} {
if {[catch {
//1.压缩数据
if {($compressionAlgorithm == "lzma") || ($compressionAlgorithm == "lzma-ultra") || ($compressionAlgorithm == "lzham") || ($compressionAlgorithm == "lzham-ultra")} {
set data [eval [concat $command [list $data]]]
} else {
set data [vfs::zip -mode compress -nowrap 1 $data]
}
//2.1 前面添加四个字节的长度，然后末尾补全32的倍数
//2.2 前面添加32字节的随机数据
set data [qA4kM 32][UOijU $data 32]
//3.在H6VeX中截取16字节的iv
set PG2JA [expr {int([maui::util::rand] \* 0x100)}]
set iv [string range $::maui::util::H6VeX [expr {$PG2JA \* 16}] [expr {$PG2JA \* 16 + 15}]]
//4.计算crc32
set hbA8n [format 0x%08x [expr {[zlib crc32 $data] & 0xffffffff}]]
//5.使用EL0BW中的数据作为key 加密
set data [tcltwofish::encrypt $::maui::util::EL0BW $data iv]
//6. 前面填充四个字节的crc32以及选取iv的位置，共加了五个字节
set YKL8w [binary format Ica\* $hbA8n $PG2JA $data]
} V77Ls]} {
set ykaki $::errorCode
set qXoNi $::errorInfo
maui::util::debug "encryptPayload: error encrypting: $V77Ls"
error $V77Ls $qXoNi $ykaki
}
return $YKL8w
}
```

```
proc UOijU {data DeSRF} {
//1. 将四字节的长度添加在数据前
set data [binary format Ia\* [string length $data] $data]
//2. 计算末尾添加padding的长度
set add [expr {($DeSRF - ([string length $data] % $DeSRF)) % $DeSRF}]
//3. 生成随机padding并附加在末尾
append data [qA4kM $add]
return $data
}
```

再看看文件数据的加密流程。从上面的脚本中，可以看到加密前的数据结果为 `32字节随机数据+4字节数据+$data+最后补全32字节的padding` 然后使用`twofish`进行加密。而`twofish`是分组加密算法，所以加密过程就是下面这样的

![](upload/attach/202310/980588\_MHT9QNMM83XZMPQ.webp)

通过在前面添加`32`字节的随机数据，猜测是为了避免明文攻击。因为很多文件前面都有固定字节的数据

> 比如`sqlite3`数据库的前16字节固定为`SQLite format 3\x00`，在`ECB`模式下，相同密钥加密任何`sqlite3`数据库的前16个字节都是一样的。所以衍生出了`CBC`模式，在每个分组加密之前用`iv`或者上一个密文分组异或明文以产生不一样的数据。

然而，这个例子中前面`32`个字节的随机数的使用却颇有点画蛇添足的味道。

首先，它不能掩盖明文数据或者阻止明文攻击（稍后提到），其次因为这`32`字节的加密数据，我们在后面进行密码分析的时候可以忽略了原本的初始`iv`，初始`iv`来自哪里，来自输入密码后进行的`16000`次`twofish`加密和`256`次`twofish`解密后生成的`0x1000`字节的`encryptedPayloadIVs`，而现在却可以跳过了，从而暴露了它的阿克琉斯之踵。根据密码算法，现在解密每一段文件数据时只需要从第三个分组的数据开始解密即可，输入`iv`来源于上一个分组的的密文（即第二个分组），输入的`key`对所有文件都是一样的（也就是本次要分析出来的目标）。

再回头说说有哪些明文，从脚本可知加密的第三个分组前四个字节为加密数据的长度。在安装过程中会生成`installbuilder_installer_{random}.log`文件，统计生成的文件数。

![](upload/attach/202310/980588\_EFP7SGFY3C7DHFY.webp)

在`hook`解密函数的过程中也发现了同样数量的加密数据，而这些数据中最小长度为`0x000012a0`，最大为`0x00040060`。所以这`1212`个文件在相同的密钥解密之后的前四个字节肯定是

`00 0x xx xx`。如果能将每段加密数据对应到具体的文件，可能还会有更多的明文信息。

综上所述，根据解密的`tcl`安装脚本共分析了三种破解思路：

第一种，根据输入密码爆破，工作量太大，`PC`上几乎不可能成功。

第二种，随机数攻击。新版本的安装程序摒弃了`LCG`算法的使用，使用了更加安全的随机数生成方式。

第三种，根据部分明文信息和`1212`个样本数据爆破`32`字节密钥，爆破的过程中每次验证只需要一次`twofish`解密16个字节即可。再配合密码差分分析，可以减少一定的尝试次数。有熟悉`angr`的大佬也可以尝试一下形式化验证的方式去破解。

另外，`ida`安装脚本中的`twofish`算法来源于<mark class="encrypted">3dcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5j5h3N6W2N6$3!0J5P5q4)9J5c8X3I4A6j5Y4c8%4L8$3k6A6M7$3R3`.</mark>

，但是作了一些修改。文末附带校正后的版本，可以直接用。另外还有调试时用的`frida`脚本。

本文分析了`ida81`对输入密码的验证流程，分别对输入密码到生成解密密钥、密码素材的生成过程以及文件数据的加密过程这三个流程进行分析，并尝试找一些可利用的破绽。很遗憾，由于水平有限，目前也只是有个思路未能完全实现，故分享出来抛砖引玉。

## 1. 密码验证算法

|  |  |

| --- | --- |
