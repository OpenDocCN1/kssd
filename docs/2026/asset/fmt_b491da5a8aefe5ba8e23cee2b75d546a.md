
`prop_area`映射的大小为`PA_SIZE = 128 * 1024`，每一个属性的内容区域都是这么大，因此`ls -l /dev/__properties__/`看到里边文件的大小都为128 * 1024。

生成出来的`prop_area`对象就存放在`ContextNode`对象的`pa_`成员中。

4. 接下来会创建打开`/dev/__properties__/properties_serial`文件，同样的用`fsetxattr`设置该文件的上下文，并且调用`mmap`将该文件映射为一个`prop_area`对象。该对象保存于`ContextsSerialized`类的`serial_prop_area_`成员变量中。

接下来会调用`property_info_area.LoadDefaultPath()`，和上面的第1步执行的操作一样，只不过操作对象是`init`进程中的`static PropertyInfoAreaFile property_info_area`；

至此`property_init()`函数结束，属性区域的初始化就完成了，此时所有属性对应的上下文已经确定，并且属性文件已经创建出来，只不过此时属性文件是空的。

接下来`init`进程就会调用`uint32_t (*property_set)(const std::string& name, const std::string& value)`函数来设置属性了，属性的来源主要有如下几方面：

1. 内核命令行
2. 设备树文件
3. 环境变量
4. 代码直接设置，如`ro.build.fingerprint`、`persist.sys.usb.config`
5. 系统文件，如`/system/etc/prop.default`、`/system/build.prop`、`/vendor/default.prop`等

其中系统文件占据属性绝大部分的来源，它由编译时根据编译选项和各个产品的配置mk文件生成。

`property_set`函数最终会调用到`property_service.cpp`文件的`PropertySet`函数：

1. 调用`__system_property_find()`得到属性对应的`prop_info*`
2. 如果`prop_info*`不为空且属性以`ro`开头表明调用函数想更改已经设置了的`ro`属性，这是不允许的，因为`ro`属性是"write-once"
3. 如果`prop_info*`不为空且属性不以`ro`开头，则调用`__system_property_update`函数更新属性的值
4. 如果`prop_info*`为空说明属性不存在，则调用`__system_property_add`函数增加属性的值
5. 如果属性以`persist`开头则调用`WritePersistentProperty`函数将属性持久化
6. 调用`property_changed()`通知属性发生了变化

一开始属性是不存在的，因此会调用`__system_property_add`函数：

1. 调用`contexts_`的`GetSerialPropArea`函数得到`prop_area* serial_pa`，`serial_pa`代表着文件`/dev/__properties__/properties_serial`的内存映射，为"global serial"。
2. 调用`contexts_->GetPropAreaForName(name)`函数得到`prop_area* pa`，`pa`代表着属性名为`name`的selinux上下文同名文件的内存映射，`init`进程通过`open(filename, O_RDWR | O_CREAT | O_NOFOLLOW | O_CLOEXEC | O_EXCL, 0444)`打开该文件，因此如果这个文件不存在就会创建出来，然后调用`mmap(nullptr, pa_size_, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0)`将文件以读写共享的方式进行内存映射。这样`init`进程可以写该区域从而对属性值进行设置。
3. 调用`prop_area`的`add`函数：`add(name, namelen, value, valuelen)`设置属性。
4. 由于设置了新的属性，因此"global serial"发生了变化，值变为原来的值加1，此时调用`__futex_wake`通知等待在此`serial`处的进程，后边讲述属性同步机制的时候还会提及。

`prop_area`结构描述了具有同一个selinux上下文的所有属性区域，它的成员变量为：

```
uint32_t bytes_used_; //prop_area总共分配的字节大小

atomic_uint_least32_t serial_; //序列号，初始为0，当添加了新的属性或者属性值更新以后会自增1

uint32_t magic_; //0x504f5250

uint32_t version_; //0xfc6ed0ab

uint32_t reserved_[28]; //全为0的保留区域

char data_[0]; //属性区域，由二叉搜索树构成，树的每个结点类型为prop_bt
```

`prop_area`区域的属性区域由变种的字典树/二叉搜索树组成，树的每个结点类型为`prop_bt`：

```
struct prop_bt {

  uint32_t namelen; //名称长度

  atomic_uint_least32_t prop; //如果这个结点对应于一个属性，那么prop就是指向prop_area内存区域的prop_info结构的偏移

  atomic_uint_least32_t left; //左子树，其中所有结点的名称 < 此结点的名称

  atomic_uint_least32_t right;//右子树，其中所有结点的名称 > 此结点的名称

  atomic_uint_least32_t children;//下一级子结点

  char name[0];//名称所在的可变数组

}
```

`prop_bt`代码上面的注释也说明了：
假设有`ro.secure=1`属性
![](upload/attach/202210/607812_G38DCBBEJCMEUSA.png)
1. `ro`属性先插入到树中，`net`和`sys`后插入，因为和`ro`同级，所以成为`ro`的左右子树。
2. `ro.com`属性的`com`节点和`secure`节点同级，它是后插入到树中的，因此`com`节点成为`secure`节点的左子树。
3. `ro.secure`属性值等于1，所以`secure`节点的`prop`指向`prop_info`结构代表着属性的名称和值。

再来看一下`prop_info`结构：

```
struct prop_info {

  atomic_uint_least32_t serial;//修改属性值时用到的同步变量，同时也用来描述属性值的长度。

  union {

    char value[PROP_VALUE_MAX];//属性值字符串

    struct {

      char error_message[kLongLegacyErrorBufferSize];

      uint32_t offset;

    } long_property; //和长属性相关，属性值长度>=PROP_VALUE_MAX的为长属性

  };

  char name[0]; //属性名称字符串

}
```

再回到`prop_area`类的`bool add(const char* name, unsigned int namelen, const char* value, unsigned int valuelen)`函数，这个函数用来往`prop_area`中添加属性，也是二叉搜索树中的插入操作，理解这个操作以后遍历和更新属性也就好理解了，因为它们就是二叉搜索树的遍历和更新操作而已：

增加属性的`prop_area::add`函数调用的是`find_property`函数，查找属性的`prop_area::find`函数调用的也是`find_property`函数，两者主要区别在于最后一个参数`bool alloc_if_needed`，`alloc_if_needed`为`true`时，查找不到对应结点的时候会创建结点。

### `prop_area::find_property`函数分析：

1. 以`.`分割属性名称字符串，从树的根结点开始进入`while(true)`循环依次处理每一段分割开的子字符串。
2. 每一段循环都先查找当前处理结点的子结点，如果`alloc_if_needed`参数为`true`且没找到子结点，则调用`new_prop_bt`函数创建出一个子结点`prop_bt`对象。
3. 在子结点中查找当前循环处理的子字符串所对应的结点，如果这个子字符串等于此子结点的名称，那么就返回子结点做为下一次循环处理的结点，如果子字符串小于此子结点的名称，则返回子结点的左子树做为下一次循环处理的结点，如果子字符串大于此子结点的名称，则返回子结点的右子树做为下一次循环处理的结点。判断字符串大小逻辑为：先比较长度，长度小的就算小，长度相同的则取`strncmp`返回值。如果左右子树不存在且`alloc_if_needed`为`true`，同样的则调用`new_prop_bt`函数创建出左右子树的结点`prop_bt`对象。
4. 处理到最后一段子字符串时循环结束，如果是`find`查询操作则直接返回当前结点的`prop`指向的`prop_info`结构即可（有可能为`null`），如果是`add`增加操作则调用`new_prop_info`函数添加一个新的`prop_info`结构并返回这个结构，属性值长度`<<24`放于`prop_info`结构的`serial`成员变量中。

回到`init`进程的`SecondStageMain()`函数，接下来调用`StartPropertyService`函数开启socket等待其他进程的设置属性请求：`PROP_MSG_SETPROP`和`PROP_MSG_SETPROP2`，至此App就可以使用属性系统了。

普通APP对于`/dev/__properties__/properties_serial`和`/dev/__properties__/property_info`都是有权限读取的，但是以selinx上下文命名的文件并不是所有的文件app都可以读取，这也就表示有的属性app是无法读取的。在app遍历属性的时候会调用`ContextNode::CheckAccessAndOpen`函数依次检查并打开所有的`/dev/__properties__`目录下的selinux上下文文件，对于selinux禁止读取的文件该函数返回`false`，app就遍历不到对应的属性。app有权限读取的属性在`system/sepolicy/public/domain.te`中指定，如：

```
# Public readable properties

get_prop(domain, debug_prop)

get_prop(domain, exported_config_prop)

get_prop(domain, exported_default_prop)

get_prop(domain, exported_dumpstate_prop)

get_prop(domain, exported_fingerprint_prop)

get_prop(domain, exported_radio_prop)

get_prop(domain, exported_secure_prop)

get_prop(domain, exported_system_prop)

get_prop(domain, exported_vold_prop)

get_prop(domain, exported2_default_prop)

get_prop(domain, logd_prop)
```
