
以上代码中，`callback_modify_param`被填充为以下函数：

```
void modify_param(inline_item_t * p_item ){
  int n;
  int index;
  address_t data;
  list_node_t * param_list ;
  if (!p_item){
    return ;
  }
  param_list = p_item->param_list; // 参数存储在结构体的一个链表中
  while (param_list != 0){
    // 遍历整个链表
    index = ((param_t *)param_list->data)->index;
    data = ((param_t *)param_list->data)->data;
    if ( index <=3 ){
      // 如果参数小于3个，修改寄存器的值
      memcpy(p_item->reg + sizeof(address_t)*index, &data, sizeof(address_t) );
    }
    else {
      // 参数大于3个，修改栈顶值
      memcpy(p_item->stack + sizeof(address_t)*(index-4), &data, sizeof(address_t));
    }
    param_list = param_list->next;
  }
}
```

`callback_modify_return`的值被填充为以下函数的地址：

```
void modify_return(inline_item_t *p_item){
  if (!p_item)    {
    return ;
  }
  // 覆盖r0的值修改返回值
  memcpy(p_item->reg, p_item->ret_value, sizeof(address_t));
}
```

指令修复部分还没完成。

最后附上Makefile:

```
NDK_ROOT=~/Android/Sdk/ndk/21.0.6113669
TOOL_CHAIN=$(NDK_ROOT)/lone_toolchain_23
CC=$(TOOL_CHAIN)/bin/armv7a-linux-androideabi23-clang
CFLAG=-g -m32
TARGET=sniper
RM=rm -f
DIR=.
OBJ_DIR=$(DIR)/obj
SRC_DIR=$(DIR)/src
SRC=$(wildcard $(DIR)/*.c)
SRC+=$(wildcard $(DIR)/*.asm)
OBJ=$(patsubst %.asm,%.o,$(patsubst %.c,%.o,$(SRC)))
all:$(TARGET)
$(TARGET):$(OBJ)
$(CC) $(CFLAG) -o $@ $^
$(DIR)/%.o:$(DIR)/%.c
$(CC) $(CFLAG) -c $<
$(DIR)/%.o:$(DIR)/%.asm
$(CC) $(CFLAG) -c $<
clean:
$(RM) *.o
$(RM) *.s
install:
adb push $(TARGET) /sdcard
```

### 4.结语

总体来讲arm上的inline hook框架要比x86难实现一点，传参方式使用寄存器加栈顶空间，还需要对两种不同类型的指令进行判断和修复。目前的代码只能说是写了一个开头，要想做一个完整的框架，还是需要一点工作量的。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-5-23 14:53

被某警官编辑

，原因：

[#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)
