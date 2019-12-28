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

华容道，源自三国时期，曹操赤壁之战后败走华容道，最后被关羽放走。其衍生的此款游戏属于一种滑块游戏。参阅 [维基百科](https://zh.wikipedia.org/zh-cn/%E8%8F%AF%E5%AE%B9%E9%81%93_(%E9%81%8A%E6%88%B2))，[百度百科](https://baike.baidu.com/item/%E5%8D%8E%E5%AE%B9%E9%81%93/23619)。

华容道可抽象为 **非对弈双方的** *(不同于五子棋)*、**无唯一解的** *(可走不同路径，可扩展为曹操必走某处)*、**局部目标固定的** *(只要求曹操出来，可扩展为要求关羽或小兵位置)*、**移动滑块** *(可扩展为支持移形换位——跳着移动)*、**求最短路径** *(用时最少出未必是最短路径)* 的问题。

本文以解华容道经典开局「横刀立马」为例，基于广度优先搜索，通过位运算移动棋子，以 Java 语言实现程序解华容道。

![opening](/img/in-post/klotski-solver/opening.png)
<small class="img-hint">横刀立马开局</small>

## 解题思路

先来几道惊险又刺激的 leetcode 热热身如何？
- [单词接龙](https://leetcode-cn.com/problems/word-ladder/)，何为广度优先搜索
- [滑动谜题](https://leetcode-cn.com/problems/sliding-puzzle/)，体会滑块移动
- [N皇后](https://leetcode-cn.com/problems/n-queens-ii/)，棋盘与位运算

### 精彩题解参考

- [HeroKern 华容道算法基础版](https://blog.csdn.net/qq_21792169/article/details/88708968)、[HeroKern  华容道算法之性能优化](https://blog.csdn.net/qq_21792169/article/details/89216377)
    - C/C++? 解
    - 提到了「通过栈先进后出策略完成最终路径寻找」
    - 比较了通过链表/红黑树实现查找算法 *(判断棋局重复性)*
- [Simonsays](http://simonsays-tw.com/web/Klotski/Klotski.html)
    - javascript 解，状态动作采用 [kineticjs](http://kineticjs.com/)
    - 提到华容道游戏技巧，配以动图解释
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
    - 压缩策略 [Zobrist hashing](https://en.wikipedia.org/wiki/Zobrist_hashing)
        - 提到了上述 Simon 利用 `javascript` 数据类型特点的方式仍然需要 `O(n²)` 的时间复杂度
        - Zobrist hashing 通过特殊的哈希表与哈希函数，有利于处理棋盘类游戏重复性判断的问题
        - 只需要 `O(1)` 的时间复杂度即可计算下一个棋盘状态
        - 计算镜像棋盘状态 *(左右对称的棋盘)* 也是 `O(1)`

### 本文思路比较

笔者认为，上述三篇精彩题解中，属 jeantimex 的最为出彩。不过着实可惜，笔者在解题前尚未体会上述精彩题解的精要，一心只想解题，也因此饶了不少弯路，末尾的 *[踩坑复盘](#踩坑复盘)* 将重现这些愚钝的过程。


相同的思路

- 广度优先搜索求最短路径
- 考虑镜像棋局状态，成倍减少可能的移动
- 利用数据类型特点，压缩棋局存储，利用哈希表更快判重
- 在BFS的同时记录路径，最终以栈形式倒序输出结果


不同的思路

- 以二进制表示棋子位置
    - 优点：同时表示了棋盘大小，解决了多格滑块整体移动的问题
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
        - 已知道需经过N个路口，只求找到一种路径
- **广度优先搜索**——*蝙蝠回声探路*
    - 鸣人：你站着别动，我每个路口都影分身找你！
    - 小樱：要是走到死胡同呢？
    - 鸣人：我每个影分身都 **同步向前** 走，分身之间可互通信息，这条不行总有条一定行！
        - *同步向前：类似飞行棋移动，即便有四架飞机都在外可移动，但每次只能移动一个棋子。*
        - *每个影分身都同步向前，意味着下一次移动机会要让给另一个分身，而不是总是一个分身继续向前移动（除非只剩下一个分身）*
        - *总是这个分身向前移动的话，就是深度优先搜索*
    - 适用：路口分岔少，不只要找到，还要经过最少路口——**求最少步数解华容道**
- **双向广度优先搜索**——*那就…两只蝙蝠*
    - 鸣人：诶？你不是也会影分身吗，我们一起分身找对方啊！
    - 小樱：你走我也走不会彼此错过吗？
    - 鸣人：我们每个路口都分身，地毯式互找，错不了！
    - 适用：已知起点、终点——华容道终点棋盘样式太多，枚举起终点反而数量巨大

华容道 BFS
- 引自 [jeantimex 解释华容道 bfs](https://github.com/jeantimex/klotski#breadth-first-search-bfs)

![jeantimex-klotski-bfs](/img/in-post/klotski-solver/bfs.png)
<small class="img-hint">华容道 广度优先搜索</small>

- 类比路口寻找
    - 开局棋盘找终局棋盘 *(只要红色曹操出来即可)*
    - 每次可能的移动就是每个路口可能的分岔
    - 镜像棋盘 *(Mirror)* 算作走过的路口，不需要再走 *(剪枝 pruning)*
    - 重复棋盘 *(Duplicate)* 属于其他影分身走过的路口，也要剪掉
    - 回看上图，广度优先搜索是 **一层一层找**
        - 每层的每个棋局，都发散出下一层可能的棋局
        - 若是一条路走到底，就属深度优先搜索

### 位运算与移动

[位操作](https://zh.wikipedia.org/wiki/%E4%BD%8D%E6%93%8D%E4%BD%9C) 是程序设计中对位模式或二进制数的一元和二元操作。如二进制 `001100` 向左移 `1` 位后，得到 `0011000` 。

![opening](/img/in-post/klotski-solver/opening.png)
<small class="img-hint">横刀立马开局</small>

1. 使用 **二进制** 表示每个棋子在棋盘中的位置
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
    - `0b` 自 java 7 之后表示二进制前缀
    - 每一行使用下划线分割（java 编译后会处理掉）
    - 曹操 `0110_0110_0000_0000_0000` 表示曹操 **正占用着** 第一行中间、第二行中间
    - 同时也表示了棋盘大小：`5` 行，`4` 列
        ```java
        0110
        0110
        0000  --(二维棋盘拉伸成一维)->  0110_0110_0000_0000_0000
        0000
        0000
        ```
    - 只需要曹操出来的终局 `target` ，可表示为 `0b0000_0000_0000_0110_0110`
2. 使用 **左移**、**右移** 表示棋子移动
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
        - 相当于拉伸成一维后右移 `4` 位
    - 小兵（在棋盘左下角） `0000_0000_0000_0000_1000` 向东北方（右上方）移动（拐角移动计 `1` 步），得 `0000_0000_0000_0100_0000`
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
3. 通过位移后 `& 1` 判断是否到边
    ```java
    private boolean isLeftBorder(int block) {
        // 左边界数组，二进制对低位为0算起，3表示最下一行的左边界
        // 以此类推，7, 11, 15, 19 表示其他各行的左边界
        int[] leftBorders = new int[]{3, 7, 11, 15, 19}; 
        for (int leftBorder : leftBorders) if (((block >> leftBorder) & 1) == 1) return true;
        return false;
    }
    ```
    - `(x & 1) == 1` 指的是 `x` 最低位（最右位最低）是否为 `1`
    - 竖着的五虎将 `1000_1000_0000_0000_0000` 在左上角，右移 `15` 位后得 `0000_0000_0000_0001_0001` （别管当前对应什么位置，仅利用纯位移来判断原来是否是某行的左边），最低位是 `1` ，故 `return true;`
    - 利用位运算特点：位移不改变原值，而是拷贝新值后再位移
4. 利用 `int` ，可高效排序，在压缩棋盘时，表示对同类型棋子，**总是** 无所谓位置在哪（详见下文 *[棋局压缩](#棋局压缩)*）
    ```java
    // 每次压缩棋局前，先对同类型的棋子排序
    // v(vertical) 表示竖着的五虎将们
    // ...少了横着的(horizon)关羽(h)就是四虎了 
    int[] vArr = new int[]{blocks[2], blocks[3], blocks[4], blocks[5]};
    Arrays.sort(vArr);
    ```
5. 利用只记录各棋子最低位的 `1` ，来实现 `long` ( `64b` ，实际只需 `50b`) 存储一个棋局（详见下文 *[棋局压缩](#棋局压缩)*）
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
6. 利用 `&` 作棋子重叠判断， `|=` 作占位累计（详见下文 *[可能的移动](#可能的移动)*）
    ```java
    int occupying = 0b0; // 已占位的
    for (int block : blocks) {
        // ... 越上界
        if ((occupying & block) != 0) return false; // 重叠
        occupying |= block; // 占位累计
    }
    ```

在理解了 *[解题思路](#解题思路)* 之后，接下来两节 *[代码框架](#代码框架)*、*[补充细节](#补充细节)* 将从零开始完成每一行代码。其中不乏可升级优化、适应更多棋局的改动，都将记录在 *[抽象优化](#抽象优化)* 一节中。

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
    - 看看哪些分岔其他分身曾走过——剪枝，去重
- 每个分身都有平等机会 **同步向前**——队列
    - 所有可能的下一步都先别急着走，先放入队列
    - 把这一轮（bfs的一层）所有分身都移动一遍之后，再看下一层
    - 下一层都在队列里

> Talk is cheap, show you the code.

入口方法

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

递归搜索

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
        List<int[]> possibilities = this.getPossibilities(current);
        for (int[] possibility : possibilities) {
            // pruning: visited 看看哪些分岔其他分身走过了
            if (visited.contains(this.compress(possibility))) continue; // 若走过，则跳过不再考虑

            // can be possible

            // 下一条路就是终点，则返回经过了多少批分身（即多少个路口）
            if (possibility[0] == target) return step; // win

            // 若不是终点，则入选下一轮分岔路队列
            nextQueue.add(possibility);

            // 记录这个路口已经有影分身占位准备要往前走了，其他分身不必再来
            visited.add(this.compress(possibility));
            // 同时，镜像路口也算走过，不必再走
            visited.add(this.compress(this.getMirror(possibility)));
        }
    }

    // drill down 所有分身接着向前走
    return this.bfs(step, nextQueue, target, visited);
}
```

IDE生成的空方法

```java
// 返回棋局的镜像棋局
private int[] getMirror(int[] blocks) {
    // TODO
    return new int[0];
}

// 查找当前棋局的、所有可能的、移动后的棋局
private List<int[]> getPossibilities(int[] blocks) {
    // TODO
    return Collections.emptyList();
}

// 压缩棋局
private Long compress(int[] blocks) {
    // TODO
    return -1L;
}
```

## 补充细节

自顶向下编程

写代码也犹如广度优先：不应一股脑深入具体方法的实现（深度优先），先把大框架搭好，保证思路清晰、变量及方法命名合适，再借用IDE能力快速生成空的方法实现。即「自顶向下编程」。

接下来我们就实现上述代码中的三个空方法
- *[可能的移动](#可能的移动)* `getPossibilities(int[] blocks): List<int[]>`
- *[棋局压缩](#棋局压缩)* `compress(int[] blocks): Long`
- *[镜像棋局](#镜像棋局)* `getMirror(int[] blocks): int[]`

以及为了棋盘输出所需的 *[记录路径](#记录路径)*

### 可能的移动

### 棋局压缩

### 镜像棋局

### 记录路径

## 踩坑复盘

### 曹操是食人族？

### 积木Debug！

### 关二哥穿越了...

### 优化压缩算法

### 就差两行排序

## 后记

### 抽象优化

### 心得体会

- 解题背景
    - 笔者自小对棋盘、RTS、全局策略类游戏之喜爱，即便根本没有时间玩，但每次看到游戏更新的内容，都能让我为之热血
    - 几个月前为了教女儿认识交通灯，用积木搭了一个 **滑动窗口** 的红绿灯玩具，顿悟「乐高反着搭可以完美滑动」*（积木块凸起高度为正常块1/4）*，由此引申「要不做个滑块游戏」的冲动，这就有了后来的 **乐高版华容道**
    - 刚搭好那几天，我与太太心血来潮，埋头苦试「横刀立马」之解法，最终由太太首先解出
    - 对华容道的不甘心，加之近期对算法的浓厚兴趣，让我更坚定了要啃下华容道的决心

### 跟拍花絮
