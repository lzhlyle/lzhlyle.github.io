---
layout:     post
title:      当「华容道」遇上位运算与广度优先搜索
subtitle:   借位运算解构棋子移动，BFS求解最短路径问题
date:       2019-12-22
author:     lyle
header-img: "img/in-post/klotski-solver/banner.jpg"
tags:
    - 棋类题解
    - 位运算
    - 广度优先搜索
---

> 譬如朝露，去日苦多。

本文以解经典开局「横刀立马」为例，基于广度优先搜索，通过位运算移动棋子，以 Java 语言实现程序解华容道。源码仓库 [klotski](https://github.com/lzhlyle/klotski)。

![opening](/img/in-post/klotski-solver/opening.png)
<small class="img-hint">横刀立马开局</small>

## 索引

1. [开篇](#开篇)
2. [解题思路](#解题思路)
    1. [精彩题解参考](#精彩题解参考)
    2. [本文思路比较](#本文思路比较)
    3. [广度优先搜索](#广度优先搜索)
    4. [位运算与移动](#位运算与移动)
3. [代码框架](#代码框架)
    1. [以终为始](#以终为始)
    2. [通用模板](#通用模板)
4. [补充细节](#补充细节)
    1. [可能的移动](#可能的移动)
    2. [棋局压缩](#棋局压缩)
    3. [镜像棋局](#镜像棋局)
    4. [记录路径](#记录路径)
5. [踩坑复盘](#踩坑复盘)
6. [后记](#后记)

## 开篇

华容道，源自三国时期，曹操赤壁之战后败走华容道，最后被关羽放走的典故。其衍生的此款游戏属于一种滑块游戏。参阅 [维基百科](https://zh.wikipedia.org/zh-cn/%E8%8F%AF%E5%AE%B9%E9%81%93_(%E9%81%8A%E6%88%B2))，[百度百科](https://baike.baidu.com/item/%E5%8D%8E%E5%AE%B9%E9%81%93/23619)。

华容道题解，可抽象为 **非双方对弈的** *(不同于五子棋)*、**无唯一解的** *(可走不同路径，可扩展为曹操必走某处)*、**局部目标固定的** *(只要求曹操出来，可扩展为要求关羽或小兵位置)*、**移动滑块** *(可扩展为支持移形换位——跳着移动)*、**求最短路径** *(用时最少未必是最短路径)* 的问题。

## 解题思路

先来几道惊险又刺激的 leetcode 算法题热热身如何？
- [单词接龙](https://leetcode-cn.com/problems/word-ladder/)，何为广度优先搜索
- [滑动谜题](https://leetcode-cn.com/problems/sliding-puzzle/)，体会滑块移动
- [N皇后](https://leetcode-cn.com/problems/n-queens-ii/)，棋盘与位运算

### 精彩题解参考

- [HeroKern 华容道算法基础版](https://blog.csdn.net/qq_21792169/article/details/88708968)、[HeroKern  华容道算法之性能优化](https://blog.csdn.net/qq_21792169/article/details/89216377)
    - C/C++? 解
    - 提到了「通过栈先进后出策略完成最终的路径输出」
    - 比较了通过链表、红黑树实现查找算法 *(判断棋局重复性)*
- [Simonsays](http://simonsays-tw.com/web/Klotski/Klotski.html)
    - javascript 解，状态动作采用 [kineticjs](http://kineticjs.com/)
    - 提到华容道游戏技巧，并配以动图
    - 图解何为广度优先搜索(BFS)
    - 棋盘压缩为 `36 bits` ，存入散列表以判断棋盘状态树的重复性
    - 解释了如何计算步数
- [jeantimex](https://github.com/jeantimex/klotski)
    - javascript 解
    - 定义了滑块的形状、位置，棋盘，逃离目标点
        ```javascript
        var game = {
            blocks: [ // 滑块
                { "shape": [2, 2], "position": [0, 1] }, // 曹操
                // ...
                { "shape": [1, 2], "position": [2, 1] }, // 五虎上将
                { "shape": [1, 1], "position": [3, 1] }  // 小兵
                // ...
            ],
            boardSize: [6, 6], // 棋盘
            escapePoint: [2, 4], // 目标
        };
        ```
    - 移动方向针对每个滑块做了限制
        ```javascript
        { down: 0, right: 1, up: 2, left: 3 } // 移动方向
        { "shape": [2, 1], "position": [0, 0], directions: [0, 2] }
        ```
    - 同样采用BFS *(画的最好，详见下文 [广度优先搜索](#广度优先搜索))*
    - 压缩策略采用 [Zobrist hashing](https://en.wikipedia.org/wiki/Zobrist_hashing)
        - 提到了上述 Simon 利用 javascript 数据类型特点的方式仍然需要 O(n²) 的时间复杂度
        - Zobrist hashing 通过特殊的哈希表与哈希函数，有利于处理棋盘类游戏重复性判断的问题
        - 只需要 O(1) 的时间复杂度即可计算下一个棋盘状态
        - 计算镜像棋盘状态 *(左右对称的棋盘)* 也是 O(1)

### 本文思路比较

笔者认为，上述三篇精彩题解中，属 jeantimex 的最为出彩。不过着实可惜，笔者在解题前尚未体会上述精彩题解的精要，一心只想解题，也因此饶了不少弯路，末尾的 *[踩坑复盘](#踩坑复盘)* 将重现这些愚钝的过程。


相同的思路

- 广度优先搜索求最短路径
- 考虑镜像棋局状态，成倍减少可能的移动
- 利用数据类型特点，压缩棋局存储，利用哈希表更快判重
- 在BFS的同时记录路径，利用栈特点倒序确定路径，以输出结果


不同的思路

- 以二进制表示棋子位置
    - 优点：同时表示了棋盘大小，解决了多格滑块整体移动的问题
    - 详见 *[以终为始](#以终为始)*
    ```java
    private QuickSolverIV() {
        standard = new int[10];
        standard[0] = 0b0110_0110_0000_0000_0000; // 曹操
        standard[1] = 0b0000_0000_0110_0000_0000; // 关羽
        standard[2] = 0b1000_1000_0000_0000_0000; // 五虎上将
        // ...
        standard[6] = 0b0000_0000_0000_0000_1000; // 小兵
        // ...
    }
    ```
- 通过位运算移动棋子
    - 优点：由于二维棋盘被 **拉伸** 成了一维的形式，天然解决了连续移动、拐角移动的问题
    - 详见 *[位运算与移动](#位运算与移动)*
    ```java
    // up
    blocks[i] = original << 4; // 上移一格
    if (validate(blocks)) {
        result.add(blocks.clone());
        if (i != 0) {
            blocks[i] = original << 8; // 上移两格
            if (validate(blocks)) result.add(blocks.clone());
        }
        if (i > 5 && !this.isLeftBorder(original)) {
            blocks[i] = original << 5; // 上左滑动
            if (validate(blocks)) result.add(blocks.clone());
        }
        if (i > 5 && !this.isRightBorder(original)) {
            blocks[i] = original << 3; // 上右滑动
            if (validate(blocks)) result.add(blocks.clone());
        }
    }
    ```

### 广度优先搜索

[广度优先搜索(维基百科)](https://zh.wikipedia.org/zh/%E5%B9%BF%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2)（英语：Breadth-First-Search，缩写 BFS），又名 [宽度优先搜索(百度百科)](https://baike.baidu.com/item/%E5%AE%BD%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2/5224802?fromtitle=%E5%B9%BF%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2&fromid=2148012)，是一种图形搜索算法。与之相关的还有 [深度优先搜索(维基百科)](https://zh.wikipedia.org/wiki/%E6%B7%B1%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2)（英语：Depth-First-Search，缩写 DFS）。


场景类比：男主、女主站在同一城市的不同路口，男主要找女主
- **深度优先搜索**——*不撞南墙不回头*
    - 男主：你站着别动，我去找你！
    - 女主：要是走到死胡同呢？
    - 男主：那我再退回上个路口，换条路，继续找！
    - 适用：
        - 路口分岔多，只要找到就行，不求经过最少路口
        - 要求经过N个路口，只求找到一种可行的路径
- **广度优先搜索**——*蝙蝠回声探路*
    - 鸣人：你站着别动，我每个路口都影分身找你！
    - 小樱：要是走到死胡同呢？
    - 鸣人：我每个影分身都 **同步向前** 走，分身之间可互通信息，这条不行总有条一定行！
        - *同步向前：类似飞行棋移动，即便有四架飞机都在外可移动，但每次只能移动一个棋子。*
        - *每个影分身都同步向前，意味着下一次移动机会要让给另一个分身，而不是总是一个分身继续向前移动（除非只剩下一个分身）*
        - *若总是这个分身向前移动，就是深度优先搜索*
    - 适用：路口分岔少，不只要找到，还要求经过最少路口——**最优步数解华容道**
- **双向广度优先搜索**——*好比…两只蝙蝠*
    - 鸣人：诶？你不是也会影分身吗，我们一起分身找对方啊！
    - 小樱：你走我也走不会彼此错过吗？
    - 鸣人：我们每个路口都分身，地毯式互找，错不了！
    - 适用：已知起点、终点——华容道终点棋盘样式太多，枚举起终点反而数量巨大。可参考笔者在 [单词接龙的题解](https://leetcode-cn.com/problems/word-ladder/solution/shuang-duan-yan-du-you-xian-di-gui-yu-chu-li-44ms-/)

华容道 BFS
- 引自 [jeantimex 解释华容道 bfs](https://github.com/jeantimex/klotski#breadth-first-search-bfs)

![jeantimex-klotski-bfs](/img/in-post/klotski-solver/bfs.png)
<small class="img-hint">华容道 广度优先搜索</small>

- 类比路口寻找
    - 开局棋盘找终局棋盘 *(只要红色曹操出来即可)*
    - 每次可能的移动就是每个路口可能的分岔情况
    - 镜像棋盘 *(Mirror)* 算作走过的路口，不需要再走 *(剪枝 pruning)*
    - 重复棋盘 *(Duplicate)* 属于其他影分身走过的路口，也要剪掉
    - 回看上图，广度优先搜索是 **一层一层找**
        - 每层的每个棋局，都发散出下一层可能的棋局
        - 若是一条路走到底，就是深度优先搜索

### 位运算与移动

[位操作](https://zh.wikipedia.org/wiki/%E4%BD%8D%E6%93%8D%E4%BD%9C) 是程序设计中对位模式或二进制数的一元和二元操作。如二进制 `0001100` 向左移 `1` 位后，得到 `0011000` 。

![opening](/img/in-post/klotski-solver/opening.png)
<small class="img-hint">横刀立马开局</small>

1. 使用 **二进制** 表示每个棋子在棋盘中的位置，完整代码详见 *[以终为始](#以终为始)*
    ```java
    private QuickSolverIV() {
        standard = new int[10];
        standard[0] = 0b0110_0110_0000_0000_0000; // 曹操
        standard[1] = 0b0000_0000_0110_0000_0000; // 关羽
        standard[2] = 0b1000_1000_0000_0000_0000; // 其他四虎上将
        // ... 3, 4, 5
        standard[6] = 0b0000_0000_0000_0000_1000; // 小兵
        // ... 7, 8, 9
    }
    ```
    - 定义长度为 `10` 的数组分别表示10个棋子
    - 固定每个棋子在数组中的索引，如 `standard[0]` 为曹操
    - `0b` 自 java 7 之后可表示二进制前缀
    - 每一行使用下划线分割（java 编译后会处理掉）
    - 曹操 `0110_0110_0000_0000_0000` 表示 **正占用着** 第一行中间两格、第二行中间两格
    - 同时也表示了棋盘大小：`5` 行，`4` 列
        ```java
        0110
        0110
        0000  --(二维棋盘拉伸成一维)->  0110_0110_0000_0000_0000
        0000
        0000
        ```
    - 只需要曹操出来的终局 `target` ，可表示为 `0000_0000_0000_0110_0110`
2. 使用 **左移**、**右移** 表示棋子移动，代码实现详见 *[可能的移动](#可能的移动)*
    - 曹操 `0110_0110_0000_0000_0000` 右移 `1` 列，得到 `0011_0011_0000_0000_0000`
        ```java
        0110             0011
        0110             0011
        0000 --(>> 1)->  0000 --(拉伸成一维)-> 0011_0011_0000_0000_0000
        0000             0000
        0000             0000
        ```
    - 曹操 `0110_0110_0000_0000_0000` 下移 `1` 行，得到 `0000_0110_0110_0000_0000`
        ```java
        0110             0000
        0110             0110
        0000 --(>> 4)->  0110 --(拉伸成一维)-> 0000_0110_0110_0000_0000
        0000             0000
        0000             0000
        ```
        - 相当于拉伸成一维后，再右移 `4` 位
    - 小兵（当前在棋盘左下角） `0000_0000_0000_0000_1000` 向东北方（右上方）移动（拐角移动计 `1` 步），得 `0000_0000_0000_0100_0000` ，代码实现详见 *[完善拐点移动](#完善拐点移动)* 
        ```java
        0000             0000             0000
        0000             0000             0000
        0000 --(<< 4)->  0000 --(>> 1)->  0000 --> 0000_0000_0000_0100_0000
        0000             1000             0100
        1000             0000             0000
        ```
        - 相当于上移 `1` 行，再右移 `1` 列
        - 相当于左移 `4` 位，再右移 `1` 位
        - 相当于左移 `3` 位
3. 通过位移后 `& 1` 判断是否到边，详见 *[实现是否靠左边](#实现是否靠左边)*
    ```java
    private boolean isLeftBorder(int block) {
        // 左边界数组，二进制对低位为0算起，3表示最下一行的左边界
        // 依此类推，7, 11, 15, 19 表示其他各行的左边界
        int[] leftBorders = new int[]{3, 7, 11, 15, 19}; 
        for (int leftBorder : leftBorders) if (((block >> leftBorder) & 1) == 1) return true;
        return false;
    }
    ```
4. 利用 `int` ，可高效排序，在压缩棋盘时，表示对同类型棋子，**总是** 无所谓位置在哪，详见 *[棋局压缩](#棋局压缩)*
    ```java
    // 每次压缩棋局前，先对同类型的棋子排序
    // v(vertical) 表示竖着的五虎将们
    // ...少了横着的(horizon)关羽(h)就是四虎了 
    int[] vArr = new int[]{blocks[2], blocks[3], blocks[4], blocks[5]};
    Arrays.sort(vArr);
    ```
5. 利用只记录各棋子最低位的 `1` ，来实现 `long` ( `64b` ，实际只需 `50b`) 存储一个棋局，详见 *[棋局压缩](#棋局压缩)*
    ```java
    int[] temp = new int[10];
    // ... 将棋局经同类型排序处理后，存入 temp 数组
    
    // 曹操顶左上角时: 1100_1100_0000_0000_0000 = (1<<19) + (1<<18) + (1<<15) + (1<<14)
    // 最大的最低位为19:10011,10个block共 5*10=50 bit即可
    long res = 0b0; // 64 bit
    // each block
    for (int i = 0; i < temp.length; i++) {
        // 找最低位的1的位置
        long lowest = 0; // 64 bit
        for (int n = 0; n < 20; n++) {
            if (((temp[i] >> n) & 1) == 1) {
                lowest = n;
                break;
            }
        }
        res |= (lowest << (i * 5));
    }
    return res;
    ```
6. 利用 `&` 作棋子重叠判断， `|=` 作占位累计，详见 *[可能的移动](#可能的移动)*
    ```java
    int occupying = 0b0; // 已占位的
    for (int block : blocks) {
        // ... 越上界
        if ((occupying & block) != 0) return false; // 重叠
        occupying |= block; // 占位累计
    }
    ```

在理解了 *[解题思路](#解题思路)* 之后，接下来两节 *[代码框架](#代码框架)*、*[补充细节](#补充细节)* 我们将从零开始一起完成每一行代码。其中不乏可升级、适应更多棋局的优化思路，都将简要记录在 *[抽象优化](#抽象优化)* 一节。

## 代码框架

既然基于广度优先搜索实现，那么我们先来搭建广度优先搜索的框架代码（与解华容道无关），再在框架内完善题解华容道相关的顶层变量及方法名。

### 以终为始

首先我们基于搭建题解类，确定入口参数 `int[]` 开局棋盘及返回类型 `int` 步数。

![opening](/img/in-post/klotski-solver/opening.png)
<small class="img-hint">横刀立马开局</small>

```java
public class QuickSolverIV {
    // 开局棋盘
    private int[] standard;

    // 确定开局，以横刀立马为例
    private QuickSolverIV() {
        standard = new int[10];
        standard[0] = 0b0110_0110_0000_0000_0000; // Square 大方块曹操
        standard[1] = 0b0000_0000_0110_0000_0000; // Horizon 横放关羽
        standard[2] = 0b1000_1000_0000_0000_0000; // Vertical 竖放武将
        standard[3] = 0b0001_0001_0000_0000_0000; // Vertical 竖放武将
        standard[4] = 0b0000_0000_1000_1000_0000; // Vertical 竖放武将
        standard[5] = 0b0000_0000_0001_0001_0000; // Vertical 竖放武将
        standard[6] = 0b0000_0000_0000_0000_1000; // Cube 小兵
        standard[7] = 0b0000_0000_0000_0100_0000; // Cube 小兵
        standard[8] = 0b0000_0000_0000_0010_0000; // Cube 小兵
        standard[9] = 0b0000_0000_0000_0000_0001; // Cube 小兵
    }
    
    // 传入开局棋盘
    // 返回最小步数
    public int minSteps(int[] opening) {
        // 要找的目标
        int target = 0b0000_0000_0000_0110_0110;
    
        // TODO
        return -1;
    }
}
```

### 通用模板

回顾 *[广度优先搜索](#广度优先搜索)* 的思路，代码需实现：
- 每个路口都做着一样的事情——递归
    - 看看这个路口有几条分岔——找所有可能的下一步
    - 看看哪些分岔其他分身已经走过——剪枝，去重
- 每个分身都有平等的机会 **同步向前**——队列
    - 所有可能的下一步都别急着走，先放入队列
    - 把这一轮（bfs的一层）所有分身都移动一遍之后，再看下一层
    - 下一层都在队列里

> Talk is cheap, show you the code.

#### 入口方法

```java
// 传入开局棋盘
// 返回最小步数
public int minSteps(int[] opening) {
    // 要找的目标
    int target = 0b0000_0000_0000_0110_0110;

    // 初始层的所有可能，存入队列
    Queue<int[]> queue = new LinkedList<>(Collections.singleton(opening));

    // 分身走过路口的记录
    Set<Long> visited = new HashSet<>(Collections.singleton(this.compress(opening)));

    return this.bfs(step: 0, queue, target, visited);
}
```

#### 递归主体

```java
// 广度优先搜索
private int bfs(int step, Queue<int[]> queue, int target, Set<Long> visited) {
    // terminator 递归终止条件
    if (queue.isEmpty()) return -1; // 所有路口都已经无路可走，找不到

    // process 当前这批分身要做的事
    step++; // 走一批，计数一次
    Queue<int[]> nextQueue = new LinkedList<>(); // 下一批可走的所有分岔
    // 这批每个分身都有平等机会 同步向前
    while (!queue.isEmpty()) {
        int[] current = queue.remove(); // 逐个分身走
        // 看看这个路口有几条分岔
        List<int[]> possibilities = getPossibilities(current);
        for (int[] possibility : possibilities) {
            // pruning: visited 看看哪些分岔其他分身走过了
            if (visited.contains(compress(possibility))) continue; // 若走过，则跳过不再考虑

            // can be possible

            // 下一条路就是终点，则返回经过了多少批分身（即多少个路口）
            if (possibility[0] == target) return step; // win

            // 若不是终点，则入选下一轮分岔路队列
            nextQueue.add(possibility);

            // 记录这个路口已经有影分身占位准备要往前走了，其他分身不必再来
            visited.add(compress(possibility));
            // 同时，镜像路口也算走过，不必再走
            visited.add(compress(getMirror(possibility)));
        }
    }

    // drill down 所有分身接着向前走
    return this.bfs(step, nextQueue, target, visited);
}
```

IDE生成的空方法

```java
// 返回棋局的镜像棋局
private int[] getMirror(int[] blocks) { return new int[0]; }

// 查找当前棋局的、所有可能的、移动后的棋局
private List<int[]> getPossibilities(int[] blocks) { return Collections.emptyList(); }

// 压缩棋局
private Long compress(int[] blocks) { return -1L; }
```

## 补充细节

自顶向下编程

写代码也犹如广度优先：不应一股脑深入具体某个方法的实现（深度优先），先把大框架搭好，保证思路清晰，变量与方法命名合适，再借用IDE能力快速生成空的方法实现。即「自顶向下编程」。

接下来我们就实现上述代码中的三个空方法
- *[可能的移动](#可能的移动)* `getPossibilities()`
- *[棋局压缩](#棋局压缩)* `compress()`
- *[镜像棋局](#镜像棋局)* `getMirror()`

以及为了棋盘输出所需的 *[记录路径](#记录路径)*

### 可能的移动

```java
// 查找当前棋局的、所有可能的、移动后的棋局
private List<int[]> getPossibilities(int[] blocks) {
    List<int[]> possibilities = new ArrayList<>();
    // move 逐个棋子试着移动，移动有效则计入可能的移动
    for (int i = 0; i < blocks.length; i++) {
        // 记录棋子原来位置，最后需还原
        int original = blocks[i];

        // up 试着向上移动
        blocks[i] = original << 4; // up
        if (validate(blocks)) possibilities.add(blocks.clone());

        // down 试着向下移动
        blocks[i] = original >> 4; // down
        if (validate(blocks)) possibilities.add(blocks.clone());

        // 若到最左边，则不可再左移动
        // left 试着向左移动
        if (!isLeftBorder(original)) {
            blocks[i] = original << 1; // left
            if (validate(blocks)) possibilities.add(blocks.clone());
        }

        // 若到最右边，则不可再右移动
        // right 试着向右移动
        if (!isRightBorder(original)) {
            blocks[i] = original >> 1; // right
            if (validate(blocks)) possibilities.add(blocks.clone());
        }

        // reverse 还原这颗棋子的位置
        blocks[i] = original;
    } // end for
    
    return possibilities;
}
```

关键点

- 完全可通过判断空位来决定哪些棋子可移动，而不是每个棋子、每个方向都尝试，详见 *[由空位定移动](#由空位定移动)*
- 对形状的抽象优化，详见 *[抽象形状判断](#抽象形状判断)*
- 至于 **连续移动两步**、**拐点移动** 没有体现？只是在现有基础上增加 `if` 逻辑分支 深入处理即可，并非难点，暂且跳过，后续完善
- `int[]` 需 `clone()` 后才能加入 `possibilities`
- 别忘了 `blocks[i] = original;` 还原棋子位置
- 同样自顶向下编程，出现后续待完善的三个空方法
    ```java
    // 是否靠右边
    private boolean isRightBorder(int block) { return false; }
    
    // 是否靠左边
    private boolean isLeftBorder(int block) { return false; }
    
    // 校验棋盘有效性
    private boolean validate(int[] blocks) { return false; }
    ```

#### 实现校验棋盘有效性

```java
// 校验棋盘有效性
private boolean validate(int[] blocks) {
    if (blocks.length != 10) return false; // 缺少棋子
    int occupying = 0b0; // 已占位的
    for (int block : blocks) {
        if (block > ((1 << 20) - 1)) return false; // 越上界
        if ((occupying & block) != 0) return false; // 重叠
        occupying |= block; // 占位累计
    }
    // 消除最低位的17个1后，应该还有1——越下界
    for (int i = 0; i < 17; i++) occupying &= occupying - 1;
    return occupying != 0;
}
```

关键点

- `int` 共 `32` 位，最高位（最左边）为符号位，其后都是数值
- 越上界
    - `1 << 20` 指 `1` 向左移动了 `20` 位，得到 `000...000_1_000...000` ，`1` 后面 `20` 个 `0`
    - `(1 << 20) - 1` 得到 `000...000_0_111...111` ，最右边 `20` 个 `1`
    - 若棋子超过此数，说明有 `1` 在五行四列的 `20` 位二进制棋盘之外
- 占位累计
    - `occupying` 为全 `0` ，看作空棋盘
    - 随着遍历棋子，通过 `|=` 不断将占位之处 **并** 入 `occupying`
        - `|` 理解为有 `1` 则 `1`
    - 以开局棋盘为例
        - 先并入曹操 `0110_0110_0000_0000_0000` ， `occupying` 为 `0110_0110_0000_0000_0000`
        - 再并入关羽 `0000_0000_0110_0000_0000` ， `occupying` 为 `0110_0110_0110_0000_0000`
    - 最终将是一个有两个 `0` 位的、其他全 `1` 的 `20` 位二进制表示占位情况
- 重叠
    - 通过 `occupying` 与每个棋子进行 `&` 操作，最终应该全是 `0`，否则出现棋子重叠
        - `&` 理解为有 `0` 则 `0`
- 越下界
    - 上述越上界，指有 `1` 出现在 `20` 位之外
    - 越下界，检查是否少了 `1`
    - 与判断 缺少棋子 的区别
        - 缺少棋子，指的是缺少棋子个数，如整块棋子不见了
        - 越下界，更准确的称呼应是「**棋子完整性**」，如曹操是否缺胳膊少腿，少了某些 `1` ，但棋子总数未少
        - 为了对应越上界，且位运算是通过 **逐步去 `1`** 实现，姑且称作「越下界」
    

#### 实现是否靠左边

```java
// 是否靠左边
private boolean isLeftBorder(int block) {
    // 左边界数组，二进制对低位为0算起，3表示最下一行的左边界
    // 依此类推，7, 11, 15, 19 表示其他各行的左边界
    int[] leftBorders = new int[]{3, 7, 11, 15, 19};
    for (int leftBorder : leftBorders) if (((block >> leftBorder) & 1) == 1) return true;
    return false;
}
```

关键点

- `(x & 1) == 1` 指的是 `x` 最低位（最右位为最低位）是否为 `1`
- 竖着的五虎将 `1000_1000_0000_0000_0000` 在左上角，右移 `15` 位后得 `0000_0000_0000_0001_0001` （别管当前对应什么位置，仅利用纯位移来判断原来是否是某行的左边），最低位是 `1` ，故 `return true;`
- 利用位运算特点：位移不改变原值，而是拷贝新值后再位移

> 至于 `isRightBorder()` ，先别看，试着参照 `isLeftBorder()` 写一写？

#### 同理，实现是否靠右边

```java
// 是否靠右边
private boolean isRightBorder(int block) {
    // 右边界数组，二进制对低位为0算起，0表示最下一行的右边界
    // 依此类推，4, 8, 12, 16 表示其他各行的右边界
    int[] rightBorders = new int[]{0, 4, 8, 12, 16};
    for (int rightBorder : rightBorders) if (((block >> rightBorder) & 1) == 1) return true;
    return false;
}
```

#### 完善连续移动两步

```java
// up 试着向上移动
blocks[i] = original << 4; // up
if (validate(blocks)) {
    possibilities.add(blocks.clone());

    // 非曹操才可能连续移动
    if (i != 0) {
        blocks[i] = original << 8; // up 2 cells
        if (validate(blocks)) possibilities.add(blocks.clone());
    }
}
```

关键点

- 能移动一步，才可能移动两步，否则就是跳棋咯
- 在原始基础 `original` 上连移 `2` 格
- 此处只展示了连续上移，还有连续下移、左移、右移

#### 完善拐点移动

```java
// up 试着向上移动
blocks[i] = original << 4; // up
if (validate(blocks)) {
    possibilities.add(blocks.clone());

    // 非曹操才可能连续移动
    if (i != 0) {
        blocks[i] = original << 8; // up 2 cells
        if (validate(blocks)) possibilities.add(blocks.clone());
    }

    // 小兵才可能拐点移动
    if (i > 5 && !isLeftBorder(original)) {
        blocks[i] = original << 5; // up left
        if (validate(blocks)) possibilities.add(blocks.clone());
    }
    if (i > 5 && !isRightBorder(original)) {
        blocks[i] = original << 3; // up right
        if (validate(blocks)) possibilities.add(blocks.clone());
    }
}
```

关键点

- 也是基于能移动一步，才可能拐点移动
- 在原始基础 `original` 上拐点移动 `(4+1)` 或 `(4-1)` 格
- 此处只展示了先上后左/右，还有先下、先左、先右

最终，整个 `getPossibilities()` 将不在本文赘述，详见源码仓库 [klotski](https://github.com/lzhlyle/klotski)。

### 棋局压缩

压缩思路

- 压缩目的
    - 最小存储代价存储当前棋局各棋子所在位置
    - 能快速比较两个棋局是否相同
    - 同形状的棋子即便位置互换也算相同棋局
- 利用每个棋子都对应一个索引的关系，详见 *[以终为始](#以终为始)* 
    - 则，取每个棋子最低位（或取最高位亦可）即能代表这个棋子的位置
    - 那么，可能的最低位有 `[0, 19]` 共20种
    - 其中，最高位的 `19` 转换二进制表示为 `10011` —— 共 `5 bit`
    - 若所有棋子都采用 `5` 位二进制表示，则最多 `10 * 5 = 50 (bit)`
    - 一个 `long` 支持 `64 bit` ，从简单实现的角度，选用 `long` 表示这 `10` 个棋子的的位置

![opening](/img/in-post/klotski-solver/opening.png)
<small class="img-hint">横刀立马开局</small>

- 压缩示例
    - 回顾开局的 `int[]` 表示
    ```java
    int[] standard = new int[10];
    standard[0] = 0b0110_0110_0000_0000_0000; // Square 大方块曹操
    standard[1] = 0b0000_0000_0110_0000_0000; // Horizon 横放关羽
    standard[2] = 0b1000_1000_0000_0000_0000; // Vertical 竖放武将
    standard[3] = 0b0001_0001_0000_0000_0000; // Vertical 竖放武将
    standard[4] = 0b0000_0000_1000_1000_0000; // Vertical 竖放武将
    standard[5] = 0b0000_0000_0001_0001_0000; // Vertical 竖放武将
    standard[6] = 0b0000_0000_0000_0000_1000; // Cube 小兵
    standard[7] = 0b0000_0000_0000_0100_0000; // Cube 小兵
    standard[8] = 0b0000_0000_0000_0010_0000; // Cube 小兵
    standard[9] = 0b0000_0000_0000_0000_0001; // Cube 小兵
    ```
    - 各棋子最低位的 `1` 分别处在的位置（最右为 `0` 起计）
    - 最有 `5` 个位一组，**对应棋子索引**，**并入** 压缩结果，对应压缩值如下表
    
棋子 | 棋子索引 | 最低位 `1` 的位置 | 最低位的二进制 | 并入压缩
:---:|:---:|:---:|:---:|:---:
初始化 | —— | —— | —— | ...00000_00000_00000
曹操 | 0 | 13 | 01101 | ...00000\_00000\_**01101**
关羽 | 1 |  9 | 01001 | ...00000_**01001**_01101
武将 | 2 | 15 | 01111 | ...**01111**_01001_01101
... | ... | ... | ... | ...

总是先对同形状排序

```java
// 压缩棋局
private Long compress(int[] blocks) {
    int[] temp = new int[10];
    temp[0] = blocks[0]; // 曹操这形状棋子就一个，无需排序
    temp[1] = blocks[1]; // 关羽这形状棋子就一个，无需排序

    // 对同vertical形状的四虎将排序
    int[] vArr = new int[]{blocks[2], blocks[3], blocks[4], blocks[5]};
    Arrays.sort(vArr);
    temp[2] = vArr[0]; temp[3] = vArr[1]; temp[4] = vArr[2]; temp[5] = vArr[3];

    // 对同cube形状的小兵排序
    int[] cArr = new int[]{blocks[6], blocks[7], blocks[8], blocks[9]};
    Arrays.sort(cArr);
    temp[6] = cArr[0]; temp[7] = cArr[1]; temp[8] = cArr[2]; temp[9] = cArr[3];
}
```

关键点

- 完全可使用 `System.arraycopy()` 实现数组拷贝，此处为一目了然而设计
- 对形状的抽象优化，详见 *[抽象形状判断](#抽象形状判断)*

并入压缩

```java
// 压缩棋局
private Long compress(int[] blocks) {
    int[] temp = new int[10];
    // ... 总是先对同形状排序

    // eg: 1100_1100_0000_0000_0000 = (1<<19) | (1<<18) | (1<<15) | (1<<14)
    // 最大的最低位为19:10011,10个block共 5*10=50 bit即可
    long res = 0b0; // 64 bit
    // 遍历棋子
    for (int i = 0; i < temp.length; i++) {
        // 找最低位的1的位置
        long lowest = 0; // 64 bit
        for (int n = 0; n < 20; n++) {
            if (((temp[i] >> n) & 1) == 1) {
                lowest = n; break;
            }
        }
        // 左移后并入，左移5是跨过上一个棋子已经并入的5个位
        res |= (lowest << (i * 5));
    }
    return res;
}
```

思考

- 找最低位 `1` 的位置，是试着逐个位右移后，每次都判断移后最低位是否为 `1` ，更好的写法是？
    - 通过 `temp[i] & -temp[i]` 取最低位 `1` 后，再对数所得
    - `long lowest = 31 - Integer.numberOfLeadingZeros(temp[i] & -temp[i]);`
    - 减少了每个棋子都 `for` ，加快了计算速度
- 纵观全局，虽返回值为 `long` ，但判重的哈希集合 `visited` 类型为 `Set<Long>` ，意味着存在装箱、拆箱，可有更好方案？
- *[看看其他读者还有哪些方案](https://github.com/lzhlyle/klotski/issues/1)*

### 镜像棋局

```java
// 返回棋局的镜像棋局
private int[] getMirror(int[] blocks) {
    int len = blocks.length;
    int[] mirror = new int[len];
    // 逐个棋子找镜像位置，放入镜像棋局
    for (int i = 0; i < len; i++) {
        // 最左边的，右移2（曹操/关羽——横向已占2格的）或3格（非曹操关羽——横向只占1格的）
        // 1100 -> 0011, 1000 -> 0001
        if (this.isLeftBorder(blocks[i])) mirror[i] = blocks[i] >> (i < 2 ? 2 : 3);
        // 最右边，左移2或3格
        // 0011 -> 1100, 0001 -> 1000
        else if (this.isRightBorder(blocks[i])) mirror[i] = blocks[i] << (i < 2 ? 2 : 3);
        // 非曹操/关羽时（曹/关在中间时就是镜像位），可能需左/右移1格
        else if (i > 1) {
            // 左移1格到边的话，那么镜像位为右移1格
            // 0100 -> 0010
            if (this.isLeftBorder(blocks[i] << 1)) mirror[i] = blocks[i] >> 1;
            // 右移1格到边的话，那么镜像位为左移1格
            // 0010 -> 0100
            if (this.isRightBorder(blocks[i] >> 1)) mirror[i] = blocks[i] << 1;
        }
        // 当前与镜像同位置，直接放入
        // 0110 -> 0110
        else mirror[i] = blocks[i];
    }
    return mirror;
}
```

关键点

- 寻找镜像位，实际上只需左/右移动，无关上/下移动
- 再次利用到二进制表示法的优势：移动是整块棋子的移动，即找镜像位是棋子的所有行的 `1` ，都同时找镜像位
- 寻找 *[可能的移动](#可能的移动)* 时，是整个棋局 `int[]` 的 `clone()` ，因为 `int[]` 对象传递的是数组地址，需要克隆才能不影响原 `int[]` 棋局
- 而此处是 `int` 赋值，传递的是值本身，无需克隆

至此，程序已经可运行，并返回 `81` 的最优步数，运行时间平均在 `47ms` 。

```java
public static void main(String[] args) {
    QuickSolver4Blog solver = new QuickSolver4Blog();
    long sum = 0;
    int times = 100; // 100次
    for (int i = 0; i < times; i++) {
        Date begin = new Date();
        int res = solver.minSteps(solver.standard);
        Date end = new Date();
        sum += end.getTime() - begin.getTime();
    }
    System.out.println("average millis: " + sum / times); // 取平均值
}
```

### 记录路径

若你想输出最优步数的每一步，则需要增加跟踪路径的「侦探」。

记录路径的作法有多种，如

- 先BFS找出最优步数 `81` ，再DFS找满足 `81` 的路径，可参考笔者在 [单词接龙II的题解](https://leetcode-cn.com/problems/word-ladder-ii/solution/18ms-xian-shuang-xiang-bfszai-dfs-by-lzhlyle/)。
- BFS的同时，记录当前棋局与上一步的棋局，形成一棵大树 *（心中可有B树？）*，本文采用了此种方法
    - 使用 `Map<int[], int[]>` 作为路劲跟踪者，`key` 为某时刻的棋局，`value` 为上一棋局
    - 因最终需要打印棋局，而非快速计算最优步数，故此处无需压缩
    - 利用哈希表插入、查询都是 `O(1)` 的优势，可快速插入跟踪点、查询上游棋局

- 在入口方法 `minSteps()` 内初始跟踪者

```java
// 传入开局棋盘
// 返回最小步数
public int minSteps(int[] opening) {
    // ...

    // 路径记录
    Map<int[], int[]> paths = new HashMap<>();
    paths.put(opening, null);

    return this.bfs(0, queue, target, visited, paths); // 传入跟踪者
}
```

- 递归主体 `bfs()` 支持跟踪者
- 跟踪者执行记录 `paths.put(...);`

```java
// 广度优先搜索
// 支持传入跟踪者
private int bfs(int step, Queue<int[]> queue, int target, Set<Long> visited, Map<int[], int[]> paths) {
    // ...
    
    while (!queue.isEmpty()) {
        int[] current = queue.remove(); // 逐个分身走
        // 看看这个路口有几条分岔
        List<int[]> possibilities = this.getPossibilities(current);
        for (int[] possibility : possibilities) {
            // pruning: visited 看看哪些分岔其他分身走过了
            if (visited.contains(this.compress(possibility))) continue; // 若走过，则跳过不再考虑

            // can be possible
            paths.put(possibility, current); // 跟踪者执行记录

            // 下一条路就是终点，则返回经过了多少批分身（即多少个路口）
            if (possibility[0] == target) return step; // win

            // ...
        }
    }

    // drill down 所有分身接着向前走
    return this.bfs(step, nextQueue, target, visited, paths); // 传入跟踪者
}
```

#### 输出棋盘

根据棋盘的 `int[]` 表示，输出一点儿不成问题，本文给出样例。

在 `return step;` 之前调用 `output(paths, possibility);`

```java
// 输出路径
private void output(Map<int[], int[]> paths, int[] curr) {
    // 先利用栈的先进后出，倒序存行进步骤
    Stack<int[]> stack = new Stack<>();
    while (paths.get(curr) != null) {
        stack.push(curr);
        curr = paths.get(curr);
    }

    System.out.println("========================");
    System.out.println("开局");
    printLayout(curr);

    int count = 0;
    while (!stack.isEmpty()) {
        System.out.println("第 " + (++count) + " 步");
        int[] b = stack.pop();
        printLayout(b);

        // 输出镜像棋局
        // System.out.println("mirror:");
        // printLayout(this.getMirror(b));
    }
}
```

```java
// 输出棋局
private void printLayout(int[] curr) {
    // 每个棋子，每个1都以字符表示
    char[] symbol = new char[]{'S', 'H', 'V', 'V', 'V', 'V', 'C', 'C', 'C', 'C'};
    String[] bStrArr = new String[10];
    // 逐棋子补0后转String
    for (int i = 0; i < 10; i++) {
        StringBuilder builder = new StringBuilder(Integer.toBinaryString(curr[i]));
        int length = builder.length();
        for (int j = 0; j < 20 - length; j++) builder.insert(0, "0");
        bStrArr[i] = builder.toString().replace('1', symbol[i]);
    }

    // 将棋子存入五行四列的20个格子中
    char[] inline = new char[20];
    for (int i = 0; i < 20; i++) {
        for (String bStr : bStrArr) {
            if (bStr.charAt(i) != '0') { inline[i] = bStr.charAt(i); break; }
        }
    }

    // 逐行输出
    for (int i = 0; i < 5; i++) {
        int j = i * 4;
        System.out.print(inline[j++]);
        System.out.print(inline[j++]);
        System.out.print(inline[j++]);
        System.out.print(inline[j]);
        System.out.println();
    }
    System.out.println();
}
```

运行效果

```bash
========================
开局
VSSV
VSSV
VHHV
VCCV
C  C

第 1 步
VSSV
VSSV
VHHV
VCCV
 C C

第 2 步
VSSV
VSSV
 HHV
VCCV
VC C
 
...

第 80 步
VVVV
VVVV
HHCC
C SS
C SS

第 81 步
VVVV
VVVV
HHCC
CSS 
CSS 
```

## 踩坑复盘

如 *[本文思路比较](#本文思路比较)* 中提到的，因在题解之前，虽有搜索多篇华容道解法，但未细读各文，踩了不少坑。直到题解如上代码之后，准备本文撰写之前，才恍然原来这些坑别人早已踩过。

> 也好，坑之宝贵，踩后自知。

### 积木Debug！

若细读上文，也许你会发现，除了思路与模板的熟悉外，最重要、也最容易写错的，就属 *[可能的移动](#可能的移动)* 一节了。没错，这都是不断 Debug 才沉淀出来的血泪。

而 Debug 的方式，不仅限于断点调试，还真是用华容道玩具来了一拨硬核 Debug。下文 *[心得体会](#心得体会)* 中有提到，用乐高搭了一版华容道，直接硬件刚！

被迫 Debug 的原因，是在一次运行之后，结果只需要 `52` 步？！难道要改变世界啦？稳住，咱别闹笑话了，输出棋盘逐步走走看。可是只看 *[输出棋盘](#输出棋盘)* 的结果，太考验想象力了，连我这个「有画画功底」的「特长生」都比划不来。

![debug-1](/img/in-post/klotski-solver/debug-1.jpg)
<small class="img-hint">积木Debug！</small>

![debug-2](/img/in-post/klotski-solver/debug-2.jpg)
<small class="img-hint">积木Debug！</small>

### 曹操是食人族？

由于棋局有效性判断不严谨，少了 「棋子个数」 的检查（详见 *[实现校验棋盘有效性](#实现校验棋盘有效性)* ），直接导致走着走着少了棋子，让出了更多的空间，曹操只需要 `25` 步就走了出来……

![lost-1](/img/in-post/klotski-solver/lost-1.jpg)
<small class="img-hint">曹操是食人族？</small>

![lost-2](/img/in-post/klotski-solver/lost-2.jpg)
<small class="img-hint">曹操是食人族？</small>

### 关二哥穿越了...

由于可能的移动判断不严谨，少了 「左右界」 的检查（详见 *[可能的移动](#可能的移动)* ），导致棋子移动后一半在这行最右，另一半在下一行最左，腾出了不该腾出的空间，曹操又趁此溜之大吉……

![split](/img/in-post/klotski-solver/split.jpg)
<small class="img-hint">关二哥穿越了...</small>

### 优化压缩算法

最初的压缩方式并非 *[棋局压缩](#棋局压缩)* 的 `long` 存储，而是直接将 `10` 个棋子各 `20b` 的二进制塞入 `new Bit(200)` 内。而后经分析棋子位置的表示，才有了上文的压缩算法。

```java
private BitSet compress(int[] blocks) {
    BitSet bitSet = new BitSet(200);
    // each block
    for (int i = 0; i < blocks.length; i++) {
        // each bit in a block
        for (int j = 0; j < 20; j++) {
            bitSet.set((i * 20) + j, ((blocks[i] >> j) & 1) == 1);
        }
    }
    return bitSet;
}
```

缺点

- 存储空间较 `long` 更多，而大部分却都是 `0`
- `BitSet` 的 `equals()` 重写为如下形式，需逐个比对 `200` 个位，略伤
    ```java
    public boolean equals(Object obj) {
        if (!(obj instanceof BitSet))
            return false;
        if (this == obj)
            return true;

        BitSet set = (BitSet) obj;

        checkInvariants();
        set.checkInvariants();

        if (wordsInUse != set.wordsInUse)
            return false;

        // Check words in use by both BitSets
        for (int i = 0; i < wordsInUse; i++)
            if (words[i] != set.words[i])
                return false;

        return true;
    }
    ```

### 就差两行排序

首次编写完的程序，运行超级慢，还被迫使用 [启发式搜索 Heuristic Search](https://zh.wikipedia.org/wiki/%E5%90%AF%E5%8F%91%E5%BC%8F%E6%90%9C%E7%B4%A2)。原因是在压缩棋局时，忽略了「同形状不同位置也算相同棋局」的问题——少了两行 `Arrays.sort()` （详见 *[棋局压缩](#棋局压缩)* ）。导致 `visited` 存储了大量实际属于相同的棋局——走20步就超过了160w种可能，真是令人崩溃。

相似的，镜像棋局也未剪枝。啊~这都要硬解还真是可怕。

在被迫使用的 a-star-search 中，估价函数定为当前曹操位置与目标的距离

```java
Queue<int[]> allNextQueue = new PriorityQueue<>(
    Comparator.comparingInt((int[] b) -> Math.abs(target - blocks[0])));
```

每一次出队，都需要出最优的 `30000` 种可能，才能得到 `81` 步。

而这一切，就差了忽略同形异位的两行排序处理。

## 后记

### 抽象优化

#### 由空位定移动

寻找可能的移动时，先确定空位，减少逐棋子逐方向尝试，可大量剪枝，加快速度。

#### 抽象形状判断

对于移动前的形状筛选，以及压缩前的同形排序，都可通过抽象形状判断，而非固定索引值的方式来硬执行，由此可适应更多开局。

*[看看其他读者还提出了哪些优化方向](https://github.com/lzhlyle/klotski/issues/1)*

### 心得体会

解题背景

- 笔者自小对棋盘、RTS、全局策略类游戏之喜爱，即便根本没有时间玩，也会因看到游戏更新的内容，就能让我为之热血
- 几个月前为了教女儿认识交通灯，用积木搭了一个 **滑动窗口** 的红绿灯玩具，顿悟「乐高反着搭可以完美滑动」*（积木块凸起高度为正常块1/4）*，由此引申「要不做个滑块游戏」的冲动，这就有了后来的 **乐高版华容道**
- 刚搭好那几天，我与太太心血来潮，埋头苦试「横刀立马」之解法，最终由太太首先解出
- 对华容道的不甘心，加之近期对算法的浓厚兴趣，让我更坚定了要啃下华容道的决心

> 算法练习，别死磕。
>
> 典型算法题，即一解通杀型的，磕后要抽象提炼、总结要点、踩坑复盘。

### 鸣谢与花絮

感谢妻子的理解与支持，无论是题解、搭博客或记下本文，都给予了我一百二十分的心理与行动支持，才有了坚持向前的动力。

> 没有爱的代码，很容易急功近利、不思进取。

感谢好友 @Eugene 的支持，一同交流我进度的分享与跌宕起伏的爬坑之旅，对本文也提了诸多建议，才是现在的样子。一碗老友不够的话……

文末，贴上部分与 @Eugene 的有趣聊天片段，以结束此文。

![last-1](/img/in-post/klotski-solver/last-1.jpg)

![last-2](/img/in-post/klotski-solver/last-2.jpg)

![last-3](/img/in-post/klotski-solver/last-3.jpg)

![last-4](/img/in-post/klotski-solver/last-4.jpg)

![last-5](/img/in-post/klotski-solver/last-5.jpg)

## 2020，新年快乐！

