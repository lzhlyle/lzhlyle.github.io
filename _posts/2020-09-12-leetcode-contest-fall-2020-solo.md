---
layout:     post
title:      战队赛求组团呀
subtitle:   LCCUP 2020 力扣杯 秋季编程大赛 个人赛
date:       2020-09-12
author:     lyle
header-img: "img/in-post/leetcode-contest-fall-2020-solo/banner.jpg"
tags:
    - LeetCode 竞赛
    - 二分
    - 动态规划
    - 深度优先
    - 记忆化
---

在快餐店打比赛还是很有趣，两个半小时，为了省电都使出「间隙性睡眠」的招数了，就当做练练白板编程也不错。

赛中只拿了 T1、T2，赛后补的 T3、T4。    
T5 放弃，也不想花时间看了，本文只留下题目。

> 4 分，750+ 名

<p><img src="/img/in-post/leetcode-contest-fall-2020-solo/rank.jpg" alt="image.png" width="0px"></p>

## 题目

1. [速算机器人](#t1-速算机器人)
2. [早餐组合](#t2-早餐组合)
3. [秋叶收藏集](#t3-秋叶收藏集)
4. [快速公交](#t4-快速公交)
5. [追逐游戏](#t5-追逐游戏)
6. [赛后复盘](#赛后复盘)

## T1. 速算机器人

#### 题目

<div class="css-1fsq2v0" style="padding: 0px; margin: 13px 0px;"><p>小扣在秋日市集发现了一款速算机器人。店家对机器人说出两个数字（记作 <code>x</code> 和 <code>y</code>），请小扣说出计算指令：</p>
<ul>
<li><code>"A"</code> 运算：使 <code>x = 2 * x + y</code>；</li>
<li><code>"B"</code> 运算：使 <code>y = 2 * y + x</code>。</li>
</ul>
<p>在本次游戏中，店家说出的数字为 <code>x = 1</code> 和 <code>y = 0</code>，小扣说出的计算指令记作仅由大写字母 <code>A</code>、<code>B</code> 组成的字符串 <code>s</code>，字符串中字符的顺序表示计算顺序，请返回最终 <code>x</code> 与 <code>y</code> 的和为多少。</p>
<p><strong>示例 1：</strong></p>
<blockquote>
<p>输入：<code>s = "AB"</code></p>
<p>输出：<code>4</code></p>
<p>解释：<br>
经过一次 A 运算后，x = 2, y = 0。<br>
再经过一次 B 运算，x = 2, y = 2。<br>
最终 x 与 y 之和为 4。</p>
</blockquote>
<p><strong>提示：</strong></p>
<ul>
<li><code>0 &lt;= s.length &lt;= 10</code></li>
<li><code>s</code> 由 <code>'A'</code> 和 <code>'B'</code> 组成</li>
</ul>
</div>

#### 题解思路

- 暴力模拟

#### 参考代码

```java
public int calculate(String s) {
    int x = 1, y = 0;
    for (char c : s.toCharArray()) {
        if (c == 'A') x = 2 * x + y;
        else y = 2 * y + x;
    }
    return x + y;
}
```

#### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(1)`

## T2. 早餐组合

#### 题目

<div class="css-1fsq2v0" style="padding: 0px; margin: 13px 0px;"><p>小扣在秋日市集选择了一家早餐摊位，一维整型数组 <code>staple</code> 中记录了每种主食的价格，一维整型数组 <code>drinks</code> 中记录了每种饮料的价格。小扣的计划选择一份主食和一款饮料，且花费不超过 <code>x</code> 元。请返回小扣共有多少种购买方案。</p>
<p>注意：答案需要以 <code>1e9 + 7 (1000000007)</code> 为底取模，如：计算初始结果为：<code>1000000008</code>，请返回 <code>1</code></p>
<p><strong>示例 1：</strong></p>
<blockquote>
<p>输入：<code>staple = [10,20,5], drinks = [5,5,2], x = 15</code></p>
<p>输出：<code>6</code></p>
<p>解释：小扣有 6 种购买方案，所选主食与所选饮料在数组中对应的下标分别是：<br>
第 1 种方案：staple[0] + drinks[0] = 10 + 5 = 15；<br>
第 2 种方案：staple[0] + drinks[1] = 10 + 5 = 15；<br>
第 3 种方案：staple[0] + drinks[2] = 10 + 2 = 12；<br>
第 4 种方案：staple[2] + drinks[0] = 5 + 5 = 10；<br>
第 5 种方案：staple[2] + drinks[1] = 5 + 5 = 10；<br>
第 6 种方案：staple[2] + drinks[2] = 5 + 2 = 7。</p>
</blockquote>
<p><strong>示例 2：</strong></p>
<blockquote>
<p>输入：<code>staple = [2,1,1], drinks = [8,9,5,1], x = 9</code></p>
<p>输出：<code>8</code></p>
<p>解释：小扣有 8 种购买方案，所选主食与所选饮料在数组中对应的下标分别是：<br>
第 1 种方案：staple[0] + drinks[2] = 2 + 5 = 7；<br>
第 2 种方案：staple[0] + drinks[3] = 2 + 1 = 3；<br>
第 3 种方案：staple[1] + drinks[0] = 1 + 8 = 9；<br>
第 4 种方案：staple[1] + drinks[2] = 1 + 5 = 6；<br>
第 5 种方案：staple[1] + drinks[3] = 1 + 1 = 2；<br>
第 6 种方案：staple[2] + drinks[0] = 1 + 8 = 9；<br>
第 7 种方案：staple[2] + drinks[2] = 1 + 5 = 6；<br>
第 8 种方案：staple[2] + drinks[3] = 1 + 1 = 2；</p>
</blockquote>
<p><strong>提示：</strong></p>
<ul>
<li><code>1 &lt;= staple.length &lt;= 10^5</code></li>
<li><code>1 &lt;= drinks.length &lt;= 10^5</code></li>
<li><code>1 &lt;= staple[i],drinks[i] &lt;= 10^5</code></li>
<li><code>1 &lt;= x &lt;= 2*10^5</code></li>
</ul>
</div>

#### 题解思路

- 俩都排序，一个枚举，一个二分
- 优化：对数量少的枚举

#### 参考代码

```java
private static final int MOD = 1000000007;

public int breakfastNumber(int[] aarr, int[] barr, int x) {
    Arrays.sort(aarr); Arrays.sort(barr);
    int cnt = 0;
    for (int i = 0; i < aarr.length && aarr[i] < x; i++) {
        int target = x - aarr[i];
        // bs: except larger than
        int l = 0, r = barr.length - 1;
        while (l < r) {
            int mid = l + (r - l + 1) / 2;
            if (barr[mid] > target) r = mid - 1;
            else l = mid;
        }
        if (barr[l] <= target) cnt += l + 1;
        if (cnt >= MOD) cnt -= MOD;
    }
    return cnt;
}
```

#### 复杂度分析

- 时间复杂度：`O(nlog(n))`，`n = max(aarr.length, barr.length)`
- 空间复杂度：`O(1)`

## T3. 秋叶收藏集

#### 题目

<div class="css-1fsq2v0" style="padding: 0px; margin: 13px 0px;"><p>小扣出去秋游，途中收集了一些红叶和黄叶，他利用这些叶子初步整理了一份秋叶收藏集 <code>leaves</code>， 字符串 <code>leaves</code> 仅包含小写字符 <code>r</code> 和 <code>y</code>， 其中字符 <code>r</code> 表示一片红叶，字符 <code>y</code> 表示一片黄叶。<br>
出于美观整齐的考虑，小扣想要将收藏集中树叶的排列调整成「红、黄、红」三部分。每部分树叶数量可以不相等，但均需大于等于 1。每次调整操作，小扣可以将一片红叶替换成黄叶或者将一片黄叶替换成红叶。请问小扣最少需要多少次调整操作才能将秋叶收藏集调整完毕。</p>
<p><strong>示例 1：</strong></p>
<blockquote>
<p>输入：<code>leaves = "rrryyyrryyyrr"</code></p>
<p>输出：<code>2</code></p>
<p>解释：调整两次，将中间的两片红叶替换成黄叶，得到 "rrryyyyyyyyrr"</p>
</blockquote>
<p><strong>示例 2：</strong></p>
<blockquote>
<p>输入：<code>leaves = "ryr"</code></p>
<p>输出：<code>0</code></p>
<p>解释：已符合要求，不需要额外操作</p>
</blockquote>
<p><strong>提示：</strong></p>
<ul>
<li><code>3 &lt;= leaves.length &lt;= 10^5</code></li>
<li><code>leaves</code> 中只包含字符 <code>'r'</code> 和字符 <code>'y'</code></li>
</ul>
</div>

#### 题解思路

- 动态规划
- `dp[n][3]`，`dp[i][j]` 表示第 `i` 位的叶子，计划归入 `j` 区
- 则每个叶子都依赖于
    - 前一片叶子所在区域
    - 当前叶子颜色

#### 参考代码

```java
public int minimumOperations(String leaves) {
    char[] arr = leaves.toCharArray();
    int n = arr.length;

    int[][] dp = new int[n][3];

    dp[0][0] = arr[0] == 'r' ? 0 : 1;
    for (int i = 1; i < n - 2; i++) {
        dp[i][0] = (arr[i] == 'r' ? 0 : 1) 
            + dp[i - 1][0];
    }
    
    dp[1][1] = (arr[1] == 'y' ? 0 : 1) + dp[0][0];
    for (int i = 2; i < n - 1; i++) {
        dp[i][1] = (arr[i] == 'y' ? 0 : 1) 
            + Math.min(dp[i - 1][0], dp[i - 1][1]);
    }

    dp[2][2] = (arr[2] == 'r' ? 0 : 1) + dp[1][1];
    for (int i = 3; i < n; i++) {
        dp[i][2] = (arr[i] == 'r' ? 0 : 1) 
            + Math.min(dp[i - 1][1], dp[i - 1][2]);
    }

    return dp[n - 1][2];
}
```

#### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(n)`

## T4. 快速公交

#### 题目

<div class="css-1fsq2v0" style="padding: 0px; margin: 13px 0px;"><p>小扣打算去秋日市集，由于游客较多，小扣的移动速度受到了人流影响：</p>
<ul>
<li>小扣从 <code>x</code> 号站点移动至 <code>x + 1</code> 号站点需要花费的时间为 <code>inc</code>；</li>
<li>小扣从 <code>x</code> 号站点移动至 <code>x - 1</code> 号站点需要花费的时间为 <code>dec</code>。</li>
</ul>
<p>现有 <code>m</code> 辆公交车，编号为 <code>0</code> 到 <code>m-1</code>。小扣也可以通过搭乘编号为 <code>i</code> 的公交车，从 <code>x</code> 号站点移动至 <code>jump[i]*x</code> 号站点，耗时仅为 <code>cost[i]</code>。小扣可以搭乘任意编号的公交车且搭乘公交次数不限。</p>
<p>假定小扣起始站点记作 <code>0</code>，秋日市集站点记作 <code>target</code>，请返回小扣抵达秋日市集最少需要花费多少时间。由于数字较大，最终答案需要对 1000000007 (1e9 + 7) 取模。</p>
<p>注意：小扣可在移动过程中到达编号大于 <code>target</code> 的站点。</p>
<p><strong>示例 1：</strong></p>
<blockquote>
<p>输入：<code>target = 31, inc =  5, dec = 3, jump = [6], cost = [10]</code></p>
<p>输出：<code>33</code></p>
<p>解释：<br>
小扣步行至 1 号站点，花费时间为 5；<br>
小扣从 1 号站台搭乘 0 号公交至 6 * 1 = 6 站台，花费时间为 10；<br>
小扣从 6 号站台步行至 5 号站台，花费时间为 3；<br>
小扣从 5 号站台搭乘 0 号公交至 6 * 5 = 30 站台，花费时间为 10；<br>
小扣从 30 号站台步行至 31 号站台，花费时间为 5；<br>
最终小扣花费总时间为 33。</p>
</blockquote>
<p><strong>示例 2：</strong></p>
<blockquote>
<p>输入：<code>target = 612, inc =  4, dec = 5, jump = [3,6,8,11,5,10,4], cost = [4,7,6,3,7,6,4]</code></p>
<p>输出：<code>26</code></p>
<p>解释：<br>
小扣步行至 1 号站点，花费时间为 4；<br>
小扣从 1 号站台搭乘 0 号公交至 3 * 1 = 3 站台，花费时间为 4；<br>
小扣从 3 号站台搭乘 3 号公交至 11 * 3 = 33 站台，花费时间为 3；<br>
小扣从 33 号站台步行至 34 站台，花费时间为 4；<br>
小扣从 34 号站台搭乘 0 号公交至 3 * 34 = 102 站台，花费时间为 4；<br>
小扣从 102 号站台搭乘 1 号公交至 6 * 102 = 612 站台，花费时间为 7；<br>
最终小扣花费总时间为 26。</p>
</blockquote>
<p><strong>提示：</strong></p>
<ul>
<li><code>1 &lt;= target &lt;= 10^9</code></li>
<li><code>1 &lt;= jump.length, cost.length &lt;= 10</code></li>
<li><code>2 &lt;= jump[i] &lt;= 10^6</code></li>
<li><code>1 &lt;= inc, dec, cost[i] &lt;= 10^6</code></li>
</ul>
</div>

#### 题解思路

- [吃掉 N 个橘子的最少天数](/leetcode-contest-weekly-202/#t4-%E5%90%83%E6%8E%89-n-%E4%B8%AA%E6%A9%98%E5%AD%90%E7%9A%84%E6%9C%80%E5%B0%91%E5%A4%A9%E6%95%B0) 的升级版
- 从 `target` 倒着走
    - 往后走几步，可以搭公交
    - 往前走几步，可以搭公交
- 深度优先 + 记忆化
- 易错点：`map.put(curr, (long) MOD);` 提前放入记忆，避免无限走远

#### 参考代码

```java
private long inc, dec;
private int[] jump, cost;
private int MOD = 1000000007;

public int busRapidTransit(int target, int inc, int dec, int[] jump, int[] cost) {
    this.inc = inc;
    this.dec = dec;
    this.jump = jump;
    this.cost = cost;
    return (int) (dfs(target, new HashMap<>()) % MOD);
}

private long dfs(int curr, Map<Integer, Long> map) {
    if (curr == 0) return 0;
    if (map.containsKey(curr)) return map.get(curr);

    map.put(curr, (long) MOD); // avoid infinite
    long res = (long) curr * inc;
    for (int i = 0; i < jump.length; i++) {
        // next <----- bus <- x
        res = Math.min(res, dfs(curr / jump[i], map) + cost[i] + (curr % jump[i]) * inc);

        // next ............. x -> bus
        // ^------------------------|
        res = Math.min(res, dfs((curr / jump[i]) + 1, map) + cost[i] + (jump[i] - (curr % jump[i])) * dec);
    }
    map.put(curr, res);
    return res;
}
```

#### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(n)`

## T5. 追逐游戏

#### 题目

<div class="css-1fsq2v0" style="padding: 0px; margin: 13px 0px;"><p>秋游中的小力和小扣设计了一个追逐游戏。他们选了秋日市集景区中的 N 个景点，景点编号为 1~N。此外，他们还选择了 N 条小路，满足任意两个景点之间都可以通过小路互相到达，且不存在两条连接景点相同的小路。整个游戏场景可视作一个无向连通图，记作二维数组 <code>edges</code>，数组中以 <code>[a,b]</code> 形式表示景点 a 与景点 b 之间有一条小路连通。</p>
<p>小力和小扣只能沿景点间的小路移动。小力的目标是在最快时间内追到小扣，小扣的目标是尽可能延后被小力追到的时间。游戏开始前，两人分别站在两个不同的景点 <code>startA</code> 和 <code>startB</code>。每一回合，小力先行动，小扣观察到小力的行动后再行动。小力和小扣在每回合可选择以下行动之一：</p>
<ul>
<li>移动至相邻景点</li>
<li>留在原地</li>
</ul>
<p>如果小力追到小扣（即两人于某一时刻出现在同一位置），则游戏结束。若小力可以追到小扣，请返回最少需要多少回合；若小力无法追到小扣，请返回 -1。</p>
<p>注意：小力和小扣一定会采取最优移动策略。</p>
<p><strong>示例 1：</strong></p>
<blockquote>
<p>输入：<code>edges = [[1,2],[2,3],[3,4],[4,1],[2,5],[5,6]], startA = 3, startB = 5</code></p>
<p>输出：<code>3</code></p>
<p>解释：<br>
<img src="/img/in-post/leetcode-contest-fall-2020-solo/t5-eg1.jpg" alt="image.png" height="250px"></p>
<p>第一回合，小力移动至 2 号点，小扣观察到小力的行动后移动至 6 号点；<br>
第二回合，小力移动至 5 号点，小扣无法移动，留在原地；<br>
第三回合，小力移动至 6 号点，小力追到小扣。返回 3。</p>
</blockquote>
<p><strong>示例 2：</strong></p>
<blockquote>
<p>输入：<code>edges = [[1,2],[2,3],[3,4],[4,1]], startA = 1, startB = 3</code></p>
<p>输出：<code>-1</code></p>
<p>解释：<br>
<img src="/img/in-post/leetcode-contest-fall-2020-solo/t5-eg2.jpg" alt="image.png" height="250px"></p>
<p>小力如果不动，则小扣也不动；否则小扣移动到小力的对角线位置。这样小力无法追到小扣。</p>
</blockquote>
<p><strong>提示：</strong></p>
<ul>
<li><code>edges</code> 的长度等于图中节点个数</li>
<li><code>3 &lt;= edges.length &lt;= 10^5</code></li>
<li><code>1 &lt;= edges[i][0], edges[i][1] &lt;= edges.length 且 edges[i][0] != edges[i][1]</code></li>
<li><code>1 &lt;= startA, startB &lt;= edges.length 且 startA != startB</code></li>
</ul>
</div>

## 赛后复盘

- 题目都出的挺好，有二分，有动态规划，也送有暴力模拟
- T3 在赛中的动态规划思路不够清晰，转入了暴力计算的胡同
- T4 赛中没品出老题的味道，最后补交的作业也是挺顺利，以后这类型是不应该再放过了！
- 下周战队赛加油！（单枪匹马求组队…
