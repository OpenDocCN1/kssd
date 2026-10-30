
```cpp
ClearList();

DeleteCriticalSection(&cs);
}

//解码函数

void decode(BYTE* addr,int len)

{

        for(int i = 0; i < len; i++)

        {

                if( method == 0 )

                        addr[i] -= decodekey;

                else

                        addr[i] ^= decodekey;

                if( UseAdd )

                        decodekey += AddNum;

        }

}
```

这是源代码

[Vx\_AntiVirus.rar](https://bbs.kanxue.com/attach-download-8815.htm)

再放个ccfer拿来测试的样本

[virus.rar](https://bbs.kanxue.com/attach-download-8818.htm)

密码是vir

谢谢观看

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [Vx\_AntiVirus.rar](javascript:void(0);)
  （821.14kb，86次下载）
- [virus.rar](javascript:void(0);)
  （668.91kb，52次下载）
