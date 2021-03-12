## 目标

使用 LL 算法构建 AST
### 准备知识：
#### AST
AST——抽象语法树。代码在计算机的分析过程中，首先将编程语言分词，分词之后得到一个层层嵌套的树形结构，最后根据语法树进行解析执行。构建抽象语法树的过程被称为语法分析。最著名的语法分析算法：LL、LR（LL 表示从左到右扫描，从左到右规约）。

#### 四则运算
##### 词法定义
四则运算的字符主要有：数字、操作符以及一些格式化的字符。真正有意义的输入被称为 token，即数字、操作符。
- TokenNumber: . 1 2 3 4 5 6 7 8 9 0 的组合
- Operator: + - * / 之一
- 格式化的字符：空格、换行等

##### 语法定义

- Expression
    - AdditiveExpression EOF
- AdditiveExpression
    - MultiplicativeExpression
    - AdditiveExpression + MultiplicativeExpression
    - AdditiveExpression - MultiplicativeExpression
- MultiplicativeExpression(最贴近终结符)
    - Number
    - MultiplicativeExpression * Number
    - MultiplicativeExpression / Number

'EOF' - 结束标识符(End of File)
认为加法由左右两个乘法组成，可以连加、加法是重复自身的序列 （递归）。

##### LL语法分析
根据以上语法定义从左到右扫描，遇到的第一个字符可能是 MultiplicativeExpression，也可能是 AdditiveExpression，如果是 MultiplicativeExpression，这个乘法表达式很可能是一个还未解析的状态，因此还要依据乘法表达式的语法定义来做一系列解析。MultiplicativeExpression 有可能遇到的情况有Number、MultiplicativeExpression、AdditiveExpression，当遇到乘法表达式时，应该当做乘法表达式继续处理还是当做加法，需要看第二个字符是什么操作符。

#### 正则表达式
做词法分析，首先要利用正则表达式来获取有效 token（数组、操作符）。
```
    var regexp = /([0-9\.]+)|([\t]+)|([\r\n]+)|(\*)|(\/)|(\+)|(\-)/g;
```

对 regexp 进行分析：
'|' 分隔表示多个分支，每一次匹配只会匹配到某一个分支里
'()' 表示多个捕获组。匹配结果不仅会将整体匹配结果返回，还会返回每个捕获组的匹配状态。

##### RegExp.prototype.exec()
exec() 方法在一个指定字符串中执行一个搜索匹配。返回一个结果数组或 null。<br />
在设置了 global 或 sticky 标志位的情况下（如 /foo/g or /foo/y），JavaScript RegExp 对象是有状态的。他们会将上次成功匹配后的位置记录在 lastIndex 属性中。使用此特性，exec() 可用来对单个字符串中的多次匹配结果进行逐条的遍历（包括捕获到的匹配），而相比之下， String.prototype.match() 只会返回匹配到的结果。<br />
exec() 的返回值，如果匹配成功，返回一个数组，否则返回 null，返回数组含义如下：
- [0]——匹配的全部字符串	
- [1], ...[n ]——捕获组的匹配结果

利用 exec() 方法，我们可以添加标识数组，给匹配到的每一个字符添加标识，即字典：
```
 var dictionary = ['Number', 'Whitespace', 'LineTerminator', '*', '/', '+', '-'];
```

为了防止匹配到的字符中有我们不认识的字符，导致前进的长度与匹配的长度不一致，因此需要添加一个函数级别的 lastIndex 变量，记录每一次匹配结果的 lastIndex。用每一次匹配的结果中的 lastIndex 和上一次匹配结果的 lastIndex 的差值与当前匹配字符长度相比较，大于则表示有我们不认识的字符。

#### 迭代器与生成器

迭代器与生成器提供了一种机制来自定义 for...of 循环的行为（for-of 专用于可迭代对象）。<br />
迭代器与生成器是一个对象，它定义一个序列，并在终止时可能返回一个返回值。 更具体地说，迭代器是通过使用 next() 方法实现 Iterator protocol 的任何一个对象，该方法返回具有两个属性的对象： value，这是序列中的 next 值；和 done ，如果已经迭代到序列中的最后一个值，则它为 true 。如果 value 和 done 一起存在，则它是迭代器的返回值。<br />
在视频中使用的方法是 for...of ，循环里可以直接拿到 value 值。使用 next() 方式结果如下：

```
let generator = tokenize('1024 + 10 * 25');
console.log(generator.next()); //{value: {type: "Number", value: "1024"}, done: false}
console.log(generator.next()); //{value: {type: "Number", value: " "}, done: false}
console.log(generator.next()); //{value: {type: "Number", value: "+"}, done: false}
...
```

#### MultiplicativeExpression 方法实现

根据四则运算的语法定义：
- MultiplicativeExpression(最贴近终结符)
    - Number
    - MultiplicativeExpression * Number
    - MultiplicativeExpression / Number
    
我们分析出它的第一个输入可能有两种情况：'Number'、'MultiplicativeExpression';<br />
第二个输入也可能有两种情况：'*'、'/';<br />

- Number:
    Number 要被认为是一个0次的乘法，因此需要新建一个乘法表达式类型，并递归调用 MultiplicativeExpression
- MultiplicativeExpression、*
    因为 MultiplicativeExpression 是最贴近终结符的，所以这一种情况是属于符合乘法产生式的，
- MultiplicativeExpression、/
    同上

#### AdditiveExpression 方法实现

根据四则运算的语法定义：
- AdditiveExpression
    - MultiplicativeExpression
    - AdditiveExpression + MultiplicativeExpression
    - AdditiveExpression - MultiplicativeExpression

通过分析可以得出与 MultiplicativeExpression 相类似的三个分支。与 MultiplicativeExpression 不同的是，AdditiveExpression 包含了 MultiplicativeExpression 的逻辑，因此每一次需要先走一次 MultiplicativeExpression，然后依次去找。并且在操作符之后，还需要调用一次 MultiplicativeExpression。

- MultiplicativeExpression
    一个单独的乘法也是作为一个0次的加法

#### Expression 方法实现

根据四则运算的语法定义：
- Expression
    - AdditiveExpression EOF

如果第一个输入是 AdditiveExpression，第二个输入是 EOF，则说明可以结束了。否则首先需要调用 AdditiveExpression。<br />
最后的表达式应该只有 AdditiveExpression 和 EOF，通常所有的语法分析最后都会得到一个单根。
#### 问题
加了 lastIndex 后，只有一个 number 被打印出来，经调试后发现，是匹配出来的长度与前进的长度不一致，表示有我们不能识别的字符。原来是空格没有被识别为 'Whitespace'，原正则：
```
var regexp = /([0-9\.]+)|([\t]+)|([\r\n]+)|(\*)|(\/)|(\+)|(\-)/g;
```
修改后的正则如下：
```
var regexp = /([0-9\.]+)|([\t\s]+)|([\r\n]+)|(\*)|(\/)|(\+)|(\-)/g;
```