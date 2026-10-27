
[MBAObfuscation.cpp](https://bbs.kanxue.com/elink@64fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8s2g2W2M7$3q4V1K9g2)9J5c8W2m8D9N6i4c8G2i4K6u0V1e0$3u0X3N6i4y4U0j5i4c8G2M7W2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5h3W2F1i4K6u0r3L8r3I4$3L8g2)9J5c8X3I4A6j5W2)9J5c8W2c8J5j5h3&6K6k6X3!0J5L8i4y4Q4x3V1k6a6j5X3k6#2M7$3y4S2N6r3W2G2L8W2)9J5c8V1#2n7b7f1!0T1k6Y4g2K6j5$3q4@1K9h3!0F1i4K6u0W2j5%4m8H3)

[MBAUtils.cpp](https://bbs.kanxue.com/elink@a4bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8s2g2W2M7$3q4V1K9g2)9J5c8W2m8D9N6i4c8G2i4K6u0V1e0$3u0X3N6i4y4U0j5i4c8G2M7W2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5h3W2F1i4K6u0r3L8r3I4$3L8g2)9J5c8X3I4A6j5W2)9J5c8W2c8J5j5h3&6K6k6X3!0J5L8i4y4Q4x3V1k6a6j5X3k6#2M7$3y4S2N6r3W2G2L8W2)9J5c8V1#2n7b7g2g2@1K9h3I4K6i4K6u0W2j5%4m8H3)

首先是把求逆多项式的代码移植一下，跟之前区别不大：

``` 
uint64_t inverse(uint64_t n, uint32_t bitWidth){
context c;
solver s(c);
expr a = c.bv_val(n, bitWidth);
expr a_inv = c.bv_const("a_inv", bitWidth);
s.add(a * a_inv == 1);
s.add(a_inv != 0);
s.check();
model m = s.get_model();
return m.eval(a_inv).get_numeral_uint64();
}
void generateUnivariatePoly(uint64_t *a, uint64_t *b, uint32_t bitWidth){
uint64_t a0, a1, b0, b1, a1_inv;
a0 = cryptoutils->get_uint64_t(), a1 = cryptoutils->get_uint64_t() | 1;
/* Calculate a1_inv */
a1_inv = inverse(a1, bitWidth);
/* Calculate b1 */
b1 = a1_inv;
/* Calculate b0 */
b0 = -(b1 * a0);
a[0] = a0, a[1] = a1, b[0] = b0, b[1] = b1;
}
```

然后按照上述混淆思路，在线性MBA表达式的基础上插入多项式MBA混淆，代码如下，还是比较好理解的：

```
Value* llvm::insertPolynomialMBA(Value *linearMBAExpr, BinaryOperator *insertBefore){
IRBuilder<> builder(insertBefore->getContext());
builder.SetInsertPoint(insertBefore);
Type *operandType = insertBefore->getOperand(0)->getType();
uint32_t bitWidth = operandType->getIntegerBitWidth();
uint64_t a[2], b[2];
generateUnivariatePoly(a, b, bitWidth);
Value *expr;
expr = builder.CreateMul(ConstantInt::get(operandType, b[1]), linearMBAExpr);
expr = builder.CreateAdd(expr, ConstantInt::get(operandType, b[0]));
expr = builder.CreateMul(ConstantInt::get(operandType, a[1]), expr);
expr = builder.CreateAdd(expr, ConstantInt::get(operandType, a[0]));
return expr;
}
```

对加法进行替换的代码修改如下，其他的运算同理：

```
void MBAObfuscation::substituteAdd(BinaryOperator *BI){
int64_t *terms = generateLinearMBA(TermsNumber);
terms[2] += 1;
terms[4] += 1;
Value *mbaExpr = insertPolynomialMBA(insertLinearMBA(terms, BI), BI);
BI->replaceAllUsesWith(mbaExpr);
}
```

# 0x04. 混淆效果

源代码：

```
#include <cstdio>
#include <cstring>
#include <cstdlib>

char *input;
char enc[100] = "\x86\x8a\x7d\x87\x93\x8b\x4d\x81\x80\x8a\
\x43\x7f\x49\x49\x86\x71\x7f\x62\x53\x69\x28\x9d";
void encrypt(unsigned char *dest, char *src){
int len = strlen(src);
for(int i = 0;i < len;i ++){
dest[i] = (src[i] + (32 - i)) ^ i;
}
}
//flag{s1mpl3_11vm_d3m0}
int main(int argc, char *argv[]){
printf("Welcome to LLVM world...\n");
if(argc <= 1){
printf("Input your flag as an argument.\n");
return 0;
}
input = argv[1];
printf("Your flag is: %s\n", input);
unsigned char dest[100] = {0};
encrypt(dest, input);
bool result = strlen(input) == 22 && !memcmp(dest, enc, 22);
if(result){
printf("Congratulations~\n");
}else{
printf("Sorry try again.\n");
}
}
```

混淆后：

![图片描述](https://bbs.kanxue.com/upload/attach/202203/910514_9BB5JXF9774Z8A5.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#软件保护](https://bbs.kanxue.com/forum-4-1-3.htm)
