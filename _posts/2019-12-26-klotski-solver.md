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

> 要死磕，也要选值得死磕的通用题材。否则，应放弃。

## Toc

1. [开篇](#开篇)
2. [解题思路](#解题思路)
    1. [精彩题解参考](#精彩题解参考)
    2. [广度优先搜索](#广度优先搜索)
    3. [位运算与移动](#位运算与移动)
3. [代码框架](#代码框架)
    1. [通用模板](#通用模板)
    2. [自顶向下编程](#自顶向下编程)
4. [补充细节](#补充细节)
    1. [记录路径](#记录路径)
    2. [可能的移动](#可能的移动)
    3. [棋局压缩与镜像](#棋局压缩与镜像)
5. [踩坑复盘](#踩坑复盘)
6. [后记](#后记)

## 开篇

华容道，源自三国时期，曹操赤壁之战后败走华容道，最后被关羽放走。其衍生的此款游戏属于一种滑块游戏。参阅 [维基百科](https://zh.wikipedia.org/zh-cn/%E8%8F%AF%E5%AE%B9%E9%81%93_(%E9%81%8A%E6%88%B2))，[百度百科](https://baike.baidu.com/item/%E5%8D%8E%E5%AE%B9%E9%81%93/23619)。

华容道可抽象为 **非对弈双方的** *(不同于五子棋)*、**无唯一解的** *(可走不同路径，可扩展为曹操必走某处)*、**局部目标固定的** *(只要求曹操出来，可扩展为要求关羽或小兵位置)*、**移动滑块** *(可扩展为支持移形换位——跳着移动)*、**求最短路径** *(最快解出未必是最短路径)* 的问题。

## 解题思路

先来几道惊险又刺激的 leetcode 热热身如何？
- [单词接龙](https://leetcode-cn.com/problems/word-ladder/)，何为广度优先搜索
- [滑动谜题](https://leetcode-cn.com/problems/sliding-puzzle/)，体会滑块移动
- [N皇后](https://leetcode-cn.com/problems/n-queens-ii/)，棋盘与位运算

### 精彩题解参考与对比
- [HeroKern 华容道算法基础版](https://blog.csdn.net/qq_21792169/article/details/88708968)、[HeroKern  华容道算法之性能优化](https://blog.csdn.net/qq_21792169/article/details/89216377)
    - C/C++?
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
    - 同样采用BFS *([画的最好看](#广度优先搜索))*
    - 压缩策略 [Zobrist hashing](https://en.wikipedia.org/wiki/Zobrist_hashing)
        - 提到了上述 Simon 利用 `javascript` 数据类型特点的方式仍然需要 `O(n²)` 的时间复杂度
        - Zobrist hashing 通过特殊的哈希表与哈希函数，有利于处理棋盘类游戏重复性判断的问题
        - 只需要 `O(1)` 的时间复杂度即可计算下一个棋盘状态
        - 计算镜像棋盘状态 *(左右对称的棋盘)* 也是 `O(1)`
- 本人题解思路的异同
    - 个人觉得，上述三篇精彩题解中，属 jeantimex 的最为出彩
    - 着实可惜，本人在解题前尚未体会上述精彩题解的精要，一心只想解题，也因此饶了不少弯路，末尾的 [踩坑复盘](#踩坑复盘) 将重现这些愚钝的过程
    - 相同的思路
        - 广度优先搜索求最短路径
        - 考虑镜像棋局状态，成倍减少可能的移动
        - 利用数据类型特点，压缩棋局存储，利用哈希表更快判重
        - 在BFS的同时记录路径，最终以栈形式倒序输出结果
    - 不同的思路
        - 以二进制表示棋子位置
            - 优点：同时表示了棋盘大小，解决了整个滑块移动的问题
            ```java
            private QuickSolverIV() {
                standard = new int[10];
                standard[0] = 0b0110_0110_0000_0000_0000; // 曹操
                standard[1] = 0b0000_0000_0110_0000_0000; // 关于
                standard[2] = 0b1000_1000_0000_0000_0000; // 五虎上将
                // ...
                standard[6] = 0b0000_0000_0000_0000_1000; // 小兵
                // ...
            }
            ```
        - 通过位运算移动棋子
            - 优点：由于二维棋盘被 **拉伸** 成了一维的形式，天然解决了连续移动、拐角移动的问题
            - 详见 [位运算与移动](#位运算与移动)
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
                    blocks[i] = original << 5; // 向上再左滑动
                    if (validate(blocks)) result.add(blocks.clone());
                }
                if (i > 5 && !this.isRightBorder(original)) {
                    blocks[i] = original << 3; // 向上再右滑动
                    if (validate(blocks)) result.add(blocks.clone());
                }
            }
            ```

### 广度优先搜索

### 位运算与移动

## 代码框架

### 通用模板

### 自顶向下编程

## 补充细节

### 记录路径

### 可能的移动

### 棋局压缩

### 镜像棋局

## 踩坑复盘

### 曹操是食人族？

### 积木Debug！

### 关二哥穿越了...

### 优化压缩算法

### 就差两行排序

## 后记

### 心得体会

- 解题背景
    - 本人自小对棋盘、RTS、全局策略类游戏之喜爱，即便根本没有时间玩，但每次看到游戏更新的内容，都能让我为之热血
    - 几个月前为了教女儿认识交通灯，用积木搭了一个 **滑动窗口** 的红绿灯玩具，顿悟「乐高反着搭可以完美滑动」*（积木块凸起高度为正常块1/4）*，由此引申「要不做个滑块游戏」的冲动，这就有了后来的 **乐高版华容道**
    - 刚搭好那几天，我与太太心血来潮，埋头苦试「横刀立马」之解法，最终太太喜提 `First Blood`
    - 对华容道的不甘心，加之近期对算法的刻意练习，让我更坚定了要啃下华容道的决

### 跟拍花絮
