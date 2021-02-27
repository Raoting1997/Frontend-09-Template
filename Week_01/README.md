学习笔记
## TicTacToe
#### 编程思路
1. 首先循环棋盘数组，显示到页面上(数组值 0 1 2)；
2. 添加事件，点击 双方交替落子；
3. 判断胜负（行、列、斜）
4. 添加局部 AI
5. 添加 bestChoice 函数
    - 递归
    - 胜负剪枝，去掉不必要的循环
    - 对方输，正方赢

> 编程小技巧：
    - 对称数公式：对称数1 = 和 - 对称数2；

#### className 与 classList 的区别
className 返回所有类名的字符串，即使只想要修改某一个类型，也必须每次都设置整个字符串的值。<br />
classList 却极为方便，是一个对象，包含以下属性和方法：
- length 长度
- item(): 取得每个元素
- add(): 添加指定类名，如果存在则不添加
- contains(): 是否包含某个类名
- remove(): 移除某个类名
- toggle(): 判断类名，如果存在，删除它；不存在，添加它。

#### Object.create
Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。<br />

###### 参数
- proto —— 新创建对象的原型对象。
- propertiesObject —— 可选。需要传入一个对象，对象的 key 为想给新对象添加的属性，值为该属性的属性标识符。例如：
```
foo: {
    writable:true,
    configurable:true,
    value: "hello"
},
```

#### 克隆一个对象的方法
- Object.creat 适用于对象只有一层时
- JSON.parse(JSON.stringify(obj))

## JavaScript 的异步机制
JavaScript 的异步机制共三种
- callback 
    - 函数回调实现异步
    - 缺点：回调地狱。不美观
- promise 
- async/await 
    - 是 promise 的语法糖，运行时依靠 promise 来管理异步