## 目标
1. 学习寻路算法：
    - 熟悉广度优先搜索
    - 学习深度搜索算法
    - 学习 JavaScript 中特有的与搜索相关的特性
    - 算法步骤可视化

## 地图编辑器
1. 数据结构的选择：
    - 二维数组（可选），缺点：开销较大
    - 一维数组（可选）
    > 一维数组与二维数组之间的转换：下标 = 100 * 行+ 列
2. 逻辑处理：
    - 需要将地图数据保存到本地，利用 localStorage
    - 填满一个数组的方式
        > a.循环数组，依次填入 <br />
        > b.Array.prototype.fill()——用一个固定值填充一个数组中从起始索引到终止索引内的全部元素。不包括终止索引
    - 在鼠标按下经过小方格时，切换选中状态
```
<!-- 清除右键菜单 -->
    document.addEventListener('contextmenu', e => e.preventDefault());
```

## 寻路问题
给定一个起点，一个终点。寻找一条可以从起点走到终点的路径。
首先思考，起点可以走哪些地方？无非就是上、下、左、右四个点，再找到其他的点能走的点（能走且没有走过）。

#### 寻路算法
##### 思想
递归：
若是使用递归，找到一个点的可走路线，会继续找它周围所有点的可走路线，就变成了深度优先搜索，但是对于寻路问题来说，广度优先搜索会更适合。

广度优先搜索实现思想：
使用队列，先将起点的可走点入队列，再依次将队列中的点的可走点入队列，直到队列为空为止。

那么我们应该如何使用 JavaScript 来实现队列这种数据结构呢呢？<br />
我们可以利用数组的 shift()、unshift()、pop()、push() 等方法结合起来实现队列
> 栈：先进后出 push() & pop() —— 也可使用 shift 与 unshift，但由于操作对象是数组，因此会影响性能 <br />
> 队列：先进先出 push() & shift()

##### 实现
首先，思考寻路算法出口，即队列是否为空
```
function path(map, start, end) {
    let queue.length = [start];

    while(queue.length) {
        
    }
}
```
接下来，处理每一个点，将它所有可走点入队列，直到找到终点
```
function path(map, start, end) {
    let queue.length = [start];

    while(queue.length) {
        let [x, y] = queue.shift();
        if (x === end[0] && y === end[1]) {
            return true;
        }
        // 否则插入上下左右四个点
        insert(x - 1, y);
        insert(x + 1, y);
        insert(x, y - 1);
        insert(x, y + 1);
    }
}
```
下面就该实现 insert 方法了，这个方法主要是判断该点是否可走，是不是边界、墙或者已走过的点
```
function insert(x, y) {
    if (x < 0 || x >= 100 || y < 0 || y >= 100) {
        return false;
    }
    if (map[x * 100 + y]) {
        return false;
    }
    map[x * 100 + y] = 2; // 标志已走过
    queue.push([x, y]);
}
```
> 总结：深度优先搜索与广度优先搜索的区别，在于存储可走点的数据结构，使用栈，就是深度优先搜索，因为栈是后进先出，也就是始终会围绕某一个点来找它周围的所有的可走路线，即深度优先搜索的思想。
##### 利用 async 和 await 实现可视化
insert() 方法中，在插入前添加一个定时器，使其延迟插入，并将即将插入的点使用其他颜色显示。

> 总结：<br />
>   广度优先搜索是以扩散的形式来寻路<br />
>   如果地图中存在闭环，那么闭环中的点永远达不到<br />
>   C 形阻碍会加大寻路的难度

#### 处理最终路径
我们现在可以得到所有的可走点，但却没有一个最终路径，那么如何找到最终路径呢？<br />
针对某一个点的来说，我们是知道它的前一个点的，因此在插入时需要将它的前一个点存储下来。为了方便存储并不改变原地图数据，克隆一个地图数据的副本，用来保存走过的点
```
function insert(x, y, prev) {
    // 判断是否可插入,边界情况、是否可走、是否走过
    if (x < 0 || x >= 100 || y < 0 || y >= 100) {
        return false;
    }
    if (mapClone[x * 100 + y]) {
        return false;
    }
    mapContainer.children[x * 100 + y].style.background = 'skyblue';
    mapClone[x * 100 + y] = prev; // 保存前一个节点
    queue.push([x, y]);
}
```
找到终点后，需要将所有的前置节点点亮
```
if (x === end[0] && y === end[1]) {
    console.log('寻路成功')
    let path = [];

    while (x !== start[0] || y !== start[1]) {
        path.push([x, y]);
        [x, y] = mapClone[x * 100 + y]; // 循环的每一次将 x,y 复制为该点的上一个节点，直到到起点为止
        await sleep(30);
        mapContainer.children[x * 100 + y].style.background = 'lightgreen';
    }

    return path;
}
```
#### 启发式寻路
单纯使用广度优先搜素，过程会非常慢，所以需要一种方式可以加速搜索，即启发式寻路（来找到最优路径）。启发式寻路需要我们判断点的扩展优先级，优先寻找权值大的点，根据这种规则，一定能找到最佳路径，这条路径被称为 A*<br />

那么该怎么应用到我们寻路问题中呢？只要修改队列的数据结构，把队列修改为能够以一定的优先级来提供点的数据结构即可。
> 创建 Sorted 数据结构，保证每一次 take 拿到的都是最小的
```
class Sorted {
    constructor(data, compare) {
        this.data = data.slice();
        this.compare = compare || ((a, b) => a - b);
    }

    take() {
        if (!this.data.length) {
            return; // 不能return null，因为 null 会参与比较
        }
        let min = this.data[0];
        let minIndex = 0;

        for (let i = 0; i < this.data.length; i++) {
            if (this.compare(this.data[i], min) < 0) {
                min = this.data[i];
                minIndex = i;
            }
        }

        // 数组要遵循的原则：尽量就是少挪动数组里的元素，每一次找最小的
        // take 之后改如何删除该元素呢，最简单的方式是使用 splice() 方法，但是这个方法的时间复杂度是 O(n)，我们需要找到一个时间复杂度为O(1)的方式。既然数组本身是无序的，因此可以将最后一个元素直接赋值到需要删除元素的索引下，再删除最后一个元素即可。
        this.data[minIndex] = this.data[this.data.length - 1];
        this.data.pop();
        return min;
    }

    give(value) {
        this.data.push(value);
    }
}
```
##### 遇到的问题
在用 Sorted 数据结构实现队列时，出现插入了 undefined 的情况。通过打断点找原因，发现是在给数组赋值时出错，取最后一个元素时越界，完全是因为粗心大意。
> 错误代码：this.data[minIndex] = this.data[this.data.length];<br />
> 正确代码：this.data[minIndex] = this.data[this.data.length - 1];