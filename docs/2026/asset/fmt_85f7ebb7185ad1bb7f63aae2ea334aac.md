
```
void Read_DexFieldId(char* fp)

{

    DexFile *dex = (DexFile*)&fp;

    printf("\n\nstruct field_id_list dex_field_ids\t%d\t\t\t\t\t\t\t\t\t\t\t\t0x%x", dex->pHeader->fieldIdsSize,(unsigned int) sizeof(DexHeader));

    printf("\t\t\t0x%x", (unsigned int) sizeof(DexFieldId));

    printf("\t\t\t数组,存储类的相关的信息");

    //转为DexFieldId结构体

    DexFieldId *dexFieldId = (DexFieldId*)(fp + dex->pHeader->fieldIdsOff);

    for (int i = 0; i < dex->pHeader->fieldIdsSize; ++i) {

        printf("\n\tstruct field_id_item field_id[%d]",i);



        /**

         * 重点:

         *      要想找到field_id的typeIdx对应位置的值 要先找到typeIdsOff偏移地址+typeIdx等于string_ids的索引,再通过string_ids来寻找值

         *

         *      [类型偏移首地址+typeIdx索引] = 字符串所在的索引位置

         *      字符串偏移首地址[字符串所在的索引位置] =  [类型偏移首地址+typeIdx索引]的具体值

         */

        //拿到类型的偏移首地址

        int *typeIdsOff = (int*)(fp + dex->pHeader->typeIdsOff);

        //拿到字符串的偏移首地址

        int *stringIdsOff = (int*)(fp + dex->pHeader->stringIdsOff);

        //类型偏移地址+索引=字符串索引

        //字符串偏移+字符串索引=typeIdx位置具体的值

        const u1* typeIdx_stringdata = (u1*)fp+stringIdsOff[(*(typeIdsOff+dexFieldId->typeIdx))];

        //解码

        readUnsignedLeb128(&typeIdx_stringdata);

        printf("\n\t\ttypeIdx --> ");

        while(*typeIdx_stringdata != '\0')

        {

            printf("%c",(*typeIdx_stringdata));

            typeIdx_stringdata++;

        }







        //u2  classIdx;           /* field所属的class类型，class_idx的值时type_ids的一个index，指向所属的类 */

        const u1* classIdx_stringdata = (u1*)fp+stringIdsOff[(*(typeIdsOff+dexFieldId->classIdx))];

        //解码

        readUnsignedLeb128(&classIdx_stringdata);

        printf("\n\t\tclassIdx--> ");

        while(*classIdx_stringdata != '\0')

        {

            printf("%c",(*classIdx_stringdata));

            classIdx_stringdata++;

        }





        //u4  nameIdx;            /* field的名称，它的值是string_ids的一个index */

        const u1* nameIdx_stringdata = (u1*)fp+stringIdsOff[(dexFieldId->nameIdx)];

        //解码

        readUnsignedLeb128(&nameIdx_stringdata);

        printf("\n\t\tnameIdx --> ");

        while(*nameIdx_stringdata != '\0')

        {

            printf("%c",(*nameIdx_stringdata));

            nameIdx_stringdata++;

        }

        dexFieldId++;

    }

}
```

输出

![](https://bbs.kanxue.com/upload/attach/201911/816237_MVXMGSXXN8WUC5X.png)

010

![](https://bbs.kanxue.com/upload/attach/201911/816237_JRSZGRKPD9VMYD7.png)

# 5、`DexMethodId`

## 5.0、理解`DexMethodId`

```
DexMethodId*  pMethodIds;     //数组,存储成员函数信息包括函数名 参数和返回值类型<br>
```

查看具体结构

描述的比较清晰

```
struct DexMethodId {

    u2  classIdx;           /* method所属的class类型，class_idx的值是type_ids的一个index，必须指向一个class类型 */

    u2  protoIdx;           /* method的原型，指向proto_ids的一个index */

    u4  nameIdx;            /* method的名称，值为string_ids的一个index */

};
```

可以自己尝试手动分析了,这里就不过多的演示了..之前演示很多次了

## 5.1、解析`DexMethodId`

又开始写代码了

```
void Read_DexMethodId(char* fp)

{



    DexFile *dex = (DexFile*)&fp;

    printf("\n\nstruct method_id_list dex_method_ids\t%d\t\t\t\t\t\t\t\t\t\t\t\t0x%x", dex->pHeader->methodIdsSize,(unsigned int) sizeof(DexHeader));

    printf("\t\t\t0x%x", (unsigned int) sizeof(DexMethodId));

    printf("\t\t\t数组,存储成员函数信息包括函数名 参数和返回值类型");

    //转为DexMethodId结构体

    DexMethodId *dexMethodId = (DexMethodId*)(fp + dex->pHeader->methodIdsOff);

    for (int i = 0; i < dex->pHeader->methodIdsSize; ++i) {

        printf("\n\tstruct method_id_item method_id[%d]", i);

        int *typeIdsOff = (int*)(fp + dex->pHeader->typeIdsOff);

        int *protoIdsOff = (int*)(fp + dex->pHeader->protoIdsOff);

        int *stringIdsOff = (int*)(fp + dex->pHeader->stringIdsOff);

        const u1* classIdx_stringdata = (u1*)fp+stringIdsOff[(*(typeIdsOff+dexMethodId->classIdx))];

        //解码

        readUnsignedLeb128(&classIdx_stringdata);

        printf("\n\t\tclassIdx--> ");

        while(*classIdx_stringdata != '\0')

        {

            printf("%c",(*classIdx_stringdata));

            classIdx_stringdata++;

        }



        //method的原型，指向proto_ids的一个index

        DexProtoId* dexProtoId = (DexProtoId*) protoIdsOff+dexMethodId->protoIdx;

        const u1* protoIdx_stringdata = (u1*)fp+stringIdsOff[(*(typeIdsOff+dexProtoId->returnTypeIdx))];

        readUnsignedLeb128(&protoIdx_stringdata);

        printf("\n\t\tprotoIdx--> ");

        while(*protoIdx_stringdata != '\0')

        {

            printf("%c",(*protoIdx_stringdata));

            protoIdx_stringdata++;

        }









        const u1* nameIdx_stringdata = (u1*)fp+stringIdsOff[dexMethodId->nameIdx];

        //解码

        readUnsignedLeb128(&nameIdx_stringdata);

        printf("\n\t\tnameIdx --> ");

        while(*nameIdx_stringdata != '\0')

        {

            printf("%c",(*nameIdx_stringdata));

            nameIdx_stringdata++;

        }





        dexMethodId++;

    }

}
```

输出:

![](https://bbs.kanxue.com/upload/attach/201911/816237_4F89XMFYR2KQZFC.png)

010

![](https://bbs.kanxue.com/upload/attach/201911/816237_5G3N65PJX9CYAXR.png)

# 6、`DexProtoId`

## 6.0、理解`DexProtoId`

```
DexProtoId*   pProtoIds;      //数组,函数原型数据索引，记录了方法声明的字符串，返回类型和参数列表
```

具体结构

```
struct DexProtoId {

    u4  shortyIdx;          /* 值为一个string_ids的index号，用来说明该method原型 */

    u4  returnTypeIdx;      /* 值为一个type_ids的index，表示该method原型的返回值类型 */

    u4  parametersOff;      /* 指定method原型的参数列表type_list，若method没有参数，则值为0. 参数的格式是type_list */

};
```

## 6.0、解析`DexProtoId`

```
void Read_DexProtoId(char* fp)

{

    

    DexFile *dex = (DexFile*)&fp;

    printf("\n\nstruct proto_id_list dex_proto_ids\t%d\t\t\t\t\t\t\t\t\t\t\t\t0x%x", dex->pHeader->protoIdsSize,(unsigned int) sizeof(DexHeader));

    printf("\t\t\t0x%x", (unsigned int) sizeof(DexMethodId));

    printf("\t\t\t数组,函数原型数据索引，记录了方法声明的字符串，返回类型和参数列表");

    //转为DexMethodId结构体

    DexProtoId *dexProtoId = (DexProtoId*)(fp + dex->pHeader->protoIdsOff);

    for (int i = 0; i < dex->pHeader->protoIdsSize; ++i) {

        printf("\n\tstruct proto_id_item proto_id[%d]", i);

        int *typeIdsOff = (int*)(fp + dex->pHeader->typeIdsOff);

        int *stringIdsOff = (int*)(fp + dex->pHeader->stringIdsOff);

        const u1* shortyIdx_stringdata = (u1*)fp+stringIdsOff[dexProtoId->shortyIdx];

        //解码

        readUnsignedLeb128(&shortyIdx_stringdata);

        printf("\n\t\tshortyIdx--> ");

        while(*shortyIdx_stringdata != '\0')

        {

            printf("%c",(*shortyIdx_stringdata));

            shortyIdx_stringdata++;

        }



        const u1* returnTypeIdx_stringdata = (u1*)fp+stringIdsOff[(*(typeIdsOff+dexProtoId->returnTypeIdx))];

        readUnsignedLeb128(&returnTypeIdx_stringdata);

        printf("\n\t\tprotoIdx--> ");

        while(*returnTypeIdx_stringdata != '\0')

        {

            printf("%c",(*returnTypeIdx_stringdata));

            returnTypeIdx_stringdata++;

        }



        printf("\n\t\tparametersOff--> %d",dexProtoId->parametersOff);



        dexProtoId++;

    }

}
```

输出:

![](https://bbs.kanxue.com/upload/attach/201911/816237_BHH28VGTWNDGAYU.png)

010:

![](https://bbs.kanxue.com/upload/attach/201911/816237_QN6WUNV4VDSGDWX.png)

# 7、`DexClassDef`

## 7.0、理解`DexClassDef`

```
DexClassDef*  pClassDefs;     //数组,存储类的信息
```

结构体
