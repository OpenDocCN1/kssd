
所以我们可以通过 `constantViolations`、`referenced`、`references`、`referencePaths` 多个参数来判断变量是否可以被删除，AST 处理代码如下：

```javascript
const parser = require("@babel/parser");
const generate = require("@babel/generator").default
const traverse = require("@babel/traverse").default
const code = `
const a = 1;
const b = a \* 2;
const c = 2;
const d = b + 1;
const e = 3;
console.log(d)
`
const ast = parser.parse(code)
const visitor = {
VariableDeclarator(path){
const binding = path.scope.getBinding(path.node.id.name);
// 如标识符被修改过，则不能进行删除动作。
if (!binding || binding.constantViolations.length > 0) {
return;
}
// 未被引用
if (!binding.referenced) {
path.remove();
}
// 被引用次数为0
// if (binding.references === 0) {
// path.remove();
// }
// 长度为0，变量没有被引用过
// if (binding.referencePaths.length === 0) {
// path.remove();
// }
}
}
traverse(ast, visitor)
const result = generate(ast)
console.log(result.code)
```

处理后的代码（未使用的 b、c、e 变量已被删除）：

```javascript
const a = 1;
const b = a \* 2;
const d = b + 1;
console.log(d);
```

## 删除冗余逻辑代码

有时候为了增加逆向难度，会有很多嵌套的 `if-else` 语句，大量判断为假的冗余逻辑代码，同样可以利用 AST 将其删除掉，只留下判断为真的，示例代码如下：

```javascript
const example = function () {
let a;
if (false) {
a = 1;
} else {
if (1) {
a = 2;
}
else {
a = 3;
}
}
return a;
};
```

观察 AST，判断条件对应的是 `test` 节点，`if` 对应的是 `consequent` 节点，`else` 对应的是 `alternate` 节点，如下图所示：

![16](upload/attach/202204/929075\_8YPW8A9M4KT6UUZ.jpg)

AST 处理思路以及代码：

1. 筛选出 `BooleanLiteral` 和 `NumericLiteral` 节点，取其对应的值，即 `path.node.test.value`；
2. 判断 `value` 值为真，则将节点替换成 `consequent` 节点下的内容，即 `path.node.consequent.body`；
3. 判断 `value` 值为假，则替换成 `alternate` 节点下的内容，即 `path.node.alternate.body`；
4. 有的 `if` 语句可能没有写 `else`，也没有 `alternate`，所以这种情况下判断 `value` 值为假，则直接移除该节点，即 `path.remove()`

```javascript
const parser = require("@babel/parser");
const generate = require("@babel/generator").default
const traverse = require("@babel/traverse").default
const types = require('@babel/types');
const code = `
const example = function () {
let a;
if (false) {
a = 1;
} else {
if (1) {
a = 2;
}
else {
a = 3;
}
}
return a;
};
`
const ast = parser.parse(code)
const visitor = {
enter(path) {
if (types.isBooleanLiteral(path.node.test) || types.isNumericLiteral(path.node.test)) {
if (path.node.test.value) {
path.replaceInline(path.node.consequent.body);
} else {
if (path.node.alternate) {
path.replaceInline(path.node.alternate.body);
} else {
path.remove()
}
}
}
}
}
traverse(ast, visitor)
const result = generate(ast)
console.log(result.code)
```

处理结果：

```javascript
const example = function () {
let a;
a = 2;
return a;
};
```

## switch-case 反控制流平坦化

控制流平坦化是混淆当中最常见的，通过 `if-else` 或者 `while-switch-case` 语句分解步骤，示例代码：

```javascript
const \_0x34e16a = '3,4,0,5,1,2'['split'](',');
let \_0x2eff02 = 0x0;
while (!![]) {
switch (\_0x34e16a[\_0x2eff02++]) {
case'0':
let \_0x38cb15 = \_0x4588f1 + \_0x470e97;
continue;
case'1':
let \_0x1e0e5e = \_0x37b9f3[\_0x50cee0(0x2e0, 0x2e8, 0x2e1, 0x2e4)];
continue;
case'2':
let \_0x35d732 = [\_0x388d4b(-0x134, -0x134, -0x139, -0x138)](\_0x38cb15 >> \_0x4588f1);
continue;
case'3':
let \_0x4588f1 = 0x1;
continue;
case'4':
let \_0x470e97 = 0x2;
continue;
case'5':
let \_0x37b9f3 = 0x5 || \_0x38cb15;
continue;
}
break;
}
```

AST 还原思路：

1. 获取控制流原始数组，将 `'3,4,0,5,1,2'['split'](',')` 之类的语句转化成 `['3','4','0','5','1','2']` 之类的数组，得到该数组之后，也可以选择把 `split` 语句对应的节点删除掉，因为最终代码里这条语句就没用了；
2. 遍历第一步得到的控制流数组，依次取出每个值所对应的 `case` 节点；
3. 定义一个数组，储存每个 `case` 节点 `consequent` 数组里面的内容，并删除 `continue` 语句对应的节点；
4. 遍历完成后，将第三步的数组替换掉整个 `while` 节点，也就是 `WhileStatement`。

不同思路，写法多样，对于如何获取控制流数组，可以有以下思路：

1. 获取到 `While` 语句节点，然后使用 `path.getAllPrevSiblings()` 方法获取其前面的所有兄弟节点，遍历每个兄弟节点，找到与 `switch()` 里面数组的变量名相同的节点，然后再取节点的值进行后续处理；
2. 直接取 `switch()` 里面数组的变量名，然后使用 `scope.getBinding()` 方法获取到它绑定的节点，然后再取这个节点的值进行后续处理。

所以 AST 处理代码就有两种写法，方法一：（`code.js` 即为前面的示例代码，为了方便操作，这里使用 `fs` 从文件中读取代码）

```javascript
const parser = require("@babel/parser");
const generate = require("@babel/generator").default
const traverse = require("@babel/traverse").default
const types = require("@babel/types")
const fs = require("fs");
const code = fs.readFileSync("code.js", {encoding: "utf-8"});
const ast = parser.parse(code)
const visitor = {
WhileStatement(path) {
// switch 节点
let switchNode = path.node.body.body[0];
// switch 语句内的控制流数组名，本例中是 \_0x34e16a
let arrayName = switchNode.discriminant.object.name;
// 获得所有 while 前面的兄弟节点，本例中获取到的是声明两个变量的节点，即 const \_0x34e16a 和 let \_0x2eff02
let prevSiblings = path.getAllPrevSiblings();
// 定义缓存控制流数组
let array = []
// forEach 方法遍历所有节点
prevSiblings.forEach(pervNode => {
let {id, init} = pervNode.node.declarations[0];
// 如果节点 id.name 与 switch 语句内的控制流数组名相同
if (arrayName === id.name) {
// 获取节点整个表达式的参数、分割方法、分隔符
let object = init.callee.object.value;
let property = init.callee.property.value;
let argument = init.arguments[0].value;
// 模拟执行 '3,4,0,5,1,2'['split'](',') 语句
array = object[property](argument)
// 也可以直接取参数进行分割，方法不通用，比如分隔符换成 | 就不行了
// array = init.callee.object.value.split(',');
}
// 前面的兄弟节点就可以删除了
pervNode.remove();
});
// 储存正确顺序的控制流语句
let replace = [];
// 遍历控制流数组，按正确顺序取 case 内容
array.forEach(index => {
let consequent = switchNode.cases[index].consequent;
// 如果最后一个节点是 continue 语句，则删除 ContinueStatement 节点
if (types.isContinueStatement(consequent[consequent.length - 1])) {
consequent.pop();
}
// concat 方法拼接多个数组，即正确顺序的 case 内容
replace = replace.concat(consequent);
}
);
// 替换整个 while 节点，两种方法都可以
path.replaceWithMultiple(replace);
// path.replaceInline(replace);
}
}
traverse(ast, visitor)
const result = generate(ast)
console.log(result.code)
```

方法二：

```javascript
const parser = require("@babel/parser");
const generate = require("@babel/generator").default
const traverse = require("@babel/traverse").default
const types = require("@babel/types")
const fs = require("fs");
const code = fs.readFileSync("code.js", {encoding: "utf-8"});
const ast = parser.parse(code)
const visitor = {
WhileStatement(path) {
// switch 节点
let switchNode = path.node.body.body[0];
// switch 语句内的控制流数组名，本例中是 \_0x34e16a
let arrayName = switchNode.discriminant.object.name;
// 获取控制流数组绑定的节点
let bindingArray = path.scope.getBinding(arrayName);
// 获取节点整个表达式的参数、分割方法、分隔符
let init = bindingArray.path.node.init;
let object = init.callee.object.value;
let property = init.callee.property.value;
let argument = init.arguments[0].value;
// 模拟执行 '3,4,0,5,1,2'['split'](',') 语句
let array = object[property](argument)
// 也可以直接取参数进行分割，方法不通用，比如分隔符换成 | 就不行了
// let array = init.callee.object.value.split(',');
// switch 语句内的控制流自增变量名，本例中是 \_0x2eff02
let autoIncrementName = switchNode.discriminant.property.argument.name;
// 获取控制流自增变量名绑定的节点
let bindingAutoIncrement = path.scope.getBinding(autoIncrementName);
// 可选择的操作：删除控制流数组绑定的节点、自增变量名绑定的节点
bindingArray.path.remove();
bindingAutoIncrement.path.remove();
// 储存正确顺序的控制流语句
let replace = [];
// 遍历控制流数组，按正确顺序取 case 内容
array.forEach(index => {
let consequent = switchNode.cases[index].consequent;
// 如果最后一个节点是 continue 语句，则删除 ContinueStatement 节点
if (types.isContinueStatement(consequent[consequent.length - 1])) {
consequent.pop();
}
// concat 方法拼接多个数组，即正确顺序的 case 内容
replace = replace.concat(consequent);
}
);
// 替换整个 while 节点，两种方法都可以
path.replaceWithMultiple(replace);
// path.replaceInline(replace);
}
}
traverse(ast, visitor)
const result = generate(ast)
console.log(result.code)
```

以上代码运行后，原来的 `switch-case` 控制流就被还原了，变成了按顺序一行一行的代码，更加简洁明了：

```javascript
let \_0x4588f1 = 0x1;
let \_0x470e97 = 0x2;
let \_0x38cb15 = \_0x4588f1 + \_0x470e97;
let \_0x37b9f3 = 0x5 || \_0x38cb15;
let \_0x1e0e5e = \_0x37b9f3[\_0x50cee0(0x2e0, 0x2e8, 0x2e1, 0x2e4)];
let \_0x35d732 = [\_0x388d4b(-0x134, -0x134, -0x139, -0x138)](\_0x38cb15 >> \_0x4588f1);
```

# 参考资料

本文有参考以下资料，也是比较推荐的在线学习资料：
