
`ConstantInt` *eor_data = `ConstantInt::get`(Type::getInt8Ty(func.getContext()),y);

//申请一个int8的内存来存放数值y

`AllocaInst`* eor_alloc=new `AllocaInst`(Type::getInt8Ty(func.getContext()),0,nullptr,1,Twine(stripOp->getName()+".alloc.y"),&ins);

//将数值y赋值给申请的内存空间

`StoreInst`* store_eor=new `StoreInst`(eor_data,eor_alloc);

store_eor->insertAfter(eor_alloc);

//从内存空间中加载里面的数值y

`LoadInst`* eor_load=new `LoadInst`(eor_alloc,"");

eor_load->insertAfter(store_eor);

//如果是第一次异或

if(y==enclen-1){

//然后进行取反计算

`BinaryOperator`* binNotOp=`BinaryOperator::CreateNot`(eor_load);

binNotOp->insertAfter(eor_load);

//然后异或

`BinaryOperator`* binXorOp=`BinaryOperator::CreateXor`(enc_const,binNotOp);

binXorOp->insertAfter(binNotOp);

//将加密字节设置为上次异或的结果

`StoreInst`* store_eor_res=new `StoreInst`(binXorOp,eor_res);

store_eor_res->insertAfter(store_data);

}else{

//加载获取上次异或的结果

`LoadInst`* eor_load_res=new `LoadInst`(eor_res,stripOp->getName()+".load");

eor_load_res->insertAfter(store_eor);

//然后再进行异或计算

`BinaryOperator`* binXorOp=`BinaryOperator::CreateXor`(eor_load_res,eor_load);

binXorOp->insertAfter(eor_load);

//将计算后的结果存放回数组中

`StoreInst`* store_data=new `StoreInst`(binXorOp,eor_res);

store_data->insertAfter(binXorOp);

//当循环到最后一次时，获取一下最后一次赋值的指令块地址。方便后面接着往后插指令块

if(y==randkey){

last_store=store_data;

}

}

}

//读取这个字节经过多次异或后的最终结果

`LoadInst`* dec_res=new `LoadInst`(eor_res,stripOp->getName()+".dec.res");

dec_res->insertAfter(last_store);

//将这个结果写入到前面用来存放的解密结果bitcat处

`StoreInst`* store_data=new `StoreInst`(dec_res,element);

store_data->insertAfter(dec_res);

}

```
上面就是把c++的解密流程用插入指令块的方式实现的方式。流程比较繁琐，但是大概意思是差不多的。

最后这里完成后，我们就可以删除指令块中的字符串明文部分。然后只保留密文

```c++
 //将字符串操作数替换为我们准备好的解密结果的bitInst
 val->replaceAllUsesWith(bitInst);
 //然后再删掉之前我们获取到的字符串的明文部分。这样就只有密文数据和密文解密的流程，最后动态执行拿到解密字符串了
 GlobalVariable* globalVar= dyn_cast<GlobalVariable>(stripOp);
 globalVar->eraseFromParent();
```

到这里整个流程就完成了。这里还有一个点需要注意的是，由于字符串的特性，当使用了多个相同的字符串，实际在汇编层的代码中，会优化为一个字符串，所以在字符串加密的时候，我们要留意解密字符串的作用域。下面举一个例子

```c++
 int main(int argc, char** argv) {
 int a = argc;
 if(a == 0)
 printf("hello");
 else
 printf("hello");
 return 0;
 }
```

这个例子中使用了两个hello。如果我们在使用这个字符串时，调用的解密。那么下面else中的代码则会无法访问到bitcat。因为不在同一个作用域，所以为了防止出现这种情况，我在解密时再做一个特殊的处理，我们先获取第一个指令块的位置，然后所有的字符串解密指令块，都插入在最开始的位置，这样就不会出现作用域的问题了。最后贴上完整代码

```c++
// 
// Created by king on 2020/10/7.
// 
#include <kllvm/Transforms/Obfuscation/Utils.h>
#include "kllvm/Transforms/Obfuscation/KStringEncode.h"
#include <string>
using namespace llvm;
namespace {
const int defaultKStringSize = 0x3;
static cl::opt<int>
KStringSize("kstr_size", cl::desc("Choose the probability [%] each basic blocks will be obfuscated by the -kstr pass"), cl::value_desc("king string encode Encryption length"), cl::init(defaultKStringSize), cl::Optional);
struct KStringEncode: public FunctionPass{
static char ID; // Pass identification
bool flag;
KStringEncode() : FunctionPass(ID) {}
KStringEncode(bool flag) : FunctionPass(ID) {this->flag = flag; KStringEncode();}
virtual bool runOnFunction(Function &F){
if ( !((KStringSize > 0) && (KStringSize <= 0x20)) ) {
errs()<<"KStringEncode application basic blocks percentage -kstr_size=x must be 0 < x <= 100";
return false;
}
if(toObfuscate(flag,&F,"kstr")) {
kstr(F);
// printFunction(F);
return true;
}
return false;
}
//转换字符串
std::string ConvertOpToString(Value* op){
GlobalVariable* globalVar= dyn_cast<GlobalVariable>(op);
if(!globalVar){
errs()<<"dyn cast gloabl err";
return "";
}
ConstantDataSequential* cds=dyn_cast<ConstantDataSequential>(globalVar->getInitializer());
if(!cds){
errs()<<"dyn cast constant data err";
return "";
}
return cds->getRawDataValues();;
}
void kstr(Function& func){
Instruction* begin_ins=nullptr;
for(BasicBlock& bb:func){
for(Instruction& ins :bb){
if(begin_ins==nullptr){
begin_ins=&ins;
}
for(Value* val:ins.operands()){
Value* stripOp=val->stripPointerCasts();
if(stripOp->getName().contains(".str")){
std::string strdata=ConvertOpToString(stripOp);
errs()<<strdata<<"\n";
if(strdata.size()<=0){
continue;
}
//加密流程
uint8_t keys[strdata.size()];
char encres[strdata.size()];
int idx=0;
memset(encres,0,strdata.size());
for(int i=0;i<strdata.size();i++){
uint8_t randkey=llvm::cryptoutils->get_uint8_t();
keys[i]=randkey;
int enclen=randkey+defaultKStringSize;
for(int y=randkey;y<enclen;y++){
if(y==randkey){
encres[i]=strdata[i]^y;
}else if(y==enclen-1){
encres[i]=encres[i]^(~y);
}else{
encres[i]=encres[i]^y;
}
printf("%x ",encres[i]);
idx++;
}
}
ArrayType* arrType=ArrayType::get(Type::getInt8Ty(func.getContext()),strdata.size());
AllocaInst* arrayInst=new AllocaInst(arrType,0,nullptr,1,Twine(stripOp->getName()+".arr"),begin_ins);
BitCastInst* bitInst=new BitCastInst(arrayInst,Type::getInt8PtrTy(func.getParent()->getContext()),Twine(stripOp->getName()+".bitcast"),begin_ins);
//创建一个对象用来存放当前加密字节解密时每次异或的结果
AllocaInst* eor_res=new AllocaInst(Type::getInt8Ty(func.getContext()),0,nullptr,1,Twine(stripOp->getName()+".alloc.res"),begin_ins);
for(int i=0;i<strdata.size();i++){
uint8_t randkey=keys[i];
int enclen=randkey+defaultKStringSize;
ConstantInt* enc_const=ConstantInt::get(Type::getInt8Ty(func.getContext()),encres[i]);
//用来存放解密结果的bitcat
ConstantInt* i_const=ConstantInt::get(Type::getInt8Ty(func.getContext()),i);
GetElementPtrInst* element=GetElementPtrInst::CreateInBounds(bitInst,i_const);
element->insertBefore(begin_ins);
StoreInst* last_store=nullptr;
for(int y=enclen-1;y>=randkey;y--){
/*下面是获取y的指令块*/
//先是创建一个数值y
ConstantInt *eor_data = ConstantInt::get(Type::getInt8Ty(func.getContext()),y);
//申请一个int8的内存来存放数值y
AllocaInst* eor_alloc=new AllocaInst(Type::getInt8Ty(func.getContext()),0,nullptr,1,Twine(stripOp->getName()+".alloc.y"),begin_ins);
//将数值y赋值给申请的内存空间
StoreInst* store_eor=new StoreInst(eor_data,eor_alloc);
store_eor->insertAfter(eor_alloc);
//从内存空间中加载里面的数值y
LoadInst* eor_load=new LoadInst(eor_alloc,"");
eor_load->insertAfter(store_eor);
//
if(y==enclen-1){
//然后进行取反计算
BinaryOperator* binNotOp=BinaryOperator::CreateNot(eor_load);
binNotOp->insertAfter(eor_load);
//然后异或
BinaryOperator* binXorOp=BinaryOperator::CreateXor(enc_const,binNotOp);
binXorOp->insertAfter(binNotOp);
//将加密字节设置为上次异或的结果
StoreInst* store_eor_res=new StoreInst(binXorOp,eor_res);
store_eor_res->insertAfter(binXorOp);
}else{
LoadInst* eor_load_res=new LoadInst(eor_res,stripOp->getName()+".load");
eor_load_res->insertAfter(store_eor);
//然后进行异或计算
BinaryOperator* binXorOp=BinaryOperator::CreateXor(eor_load_res,eor_load);
binXorOp->insertAfter(eor_load);
//将计算后的结果存放回数组中
StoreInst* store_data=new StoreInst(binXorOp,eor_res);
store_data->insertAfter(binXorOp);
if(y==randkey){
last_store=store_data;
}
}
}
//读取这个字节经过多次异或后的最终结果
LoadInst* dec_res=new LoadInst(eor_res,stripOp->getName()+".dec.res");
dec_res->insertAfter(last_store);
//将这个结果写入到前面用来存放的解密结果bitcat处
StoreInst* store_data=new StoreInst(dec_res,element);
store_data->insertAfter(dec_res);
}
//将字符串操作数替换为我们准备好的解密结果的bitInst
val->replaceAllUsesWith(bitInst);
//然后再删掉之前我们获取到的字符串的明文部分。这样就只有密文数据和密文解密的流程，最后动态执行拿到解密字符串了
GlobalVariable* globalVar= dyn_cast<GlobalVariable>(stripOp);
globalVar->eraseFromParent();
}
}
}
}
};
}
char KStringEncode::ID = 0;
static RegisterPass<KStringEncode> X("kstr", "inserting bogus control flow");
Pass *llvm::createKStringEncode() {
return new KStringEncode();
}
Pass *llvm::createKStringEncode(bool flag) {
return new KStringEncode(flag);
}
```

题目出自3W班9月的题目用ollvm9.0实现字符串简单加密

题目主要是为了能熟练ollvm中如何进行一个简单的加密，以及c++部分怎么生成对应的IR指令来达到像c++函数效果。所以主要我们的思路可以切换成。

1、首先确定加密的核心逻辑并用C++实现

2、根据C++的算法。生成一份IR指令来为我们提供参考

3、ollvm里面如何用C++生成相应的IR指令

4、根据参考的IR指令来生成我们需要的加密和解密函数

做完这个题目，基本就可以对ollvm的工作原理有一定的了解，并且改造属于自己的加密或者混淆了。
