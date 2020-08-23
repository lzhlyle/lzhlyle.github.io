---
layout:     post
title:      以为的差一点，其实差得远
subtitle:   LeetCode 第 202 场周赛
date:       2020-08-16
author:     lyle
header-img: "img/in-post/leetcode-contest-weekly-202/banner.jpg"
tags:
    - LeetCode 竞赛
---

> 痛苦的不是失败，而是「我本可以」。
>
> 更痛苦的不是「我本可以」，而是「自以为我本可以」。

本周的周赛比较适合上分，大神们都是手速场。自信以为能挺进 150，结果 T4 都没 AC，600 开外慢走不送……

特此复盘，有感而记。

## 题目

1. [存在连续三个奇数的数组](#t1-存在连续三个奇数的数组)
2. [使数组中所有元素相等的最小操作数](#t2-使数组中所有元素相等的最小操作数)
3. [两球之间的磁力](#t3-两球之间的磁力)
4. [吃掉 N 个橘子的最少天数](#t4-吃掉-n-个橘子的最少天数)
5. [赛后复盘](#赛后复盘)

## T1. 存在连续三个奇数的数组

#### 题目

> 给你一个整数数组 `arr`，请你判断数组中是否存在连续三个元素都是奇数的情况：如果存在，请返回 `true` ；否则，返回 `false` 。

示例 1

```
输入：arr = [2,6,4,1]
输出：false
解释：不存在连续三个元素都是奇数的情况。
```

示例 2

```
输入：arr = [1,2,34,3,4,5,7,23,12]
输出：true
解释：存在连续三个元素都是奇数的情况，即 [5,7,23] 。
```

提示

- `1 <= arr.length <= 1000`
- `1 <= arr[i] <= 1000`

#### 题解思路

- 遍历数组，统计连续奇数个数 `cnt`
- 易错点
    - 要求 `cnt >= 3` 而非 `cnt == 3`
    - 遍历结束需要再判断一次 `cnt`，经典范式了都

#### 参考代码

```java
public boolean threeConsecutiveOdds(int[] arr) {
    int cnt = 0;
    for (int v : arr) {
        if ((v & 1) == 1) cnt++;
        else if (cnt >= 3) return true;
        else cnt = 0;
    }
    return cnt >= 3;
}
```

#### 复杂度分析

- 时间复杂度：`O(n)`
- 空间复杂度：`O(1)`

## T2. 使数组中所有元素相等的最小操作数

#### 题目

> 存在一个长度为 `n` 的数组 `arr` ，其中 `arr[i] = (2 * i) + 1` （ `0 <= i < n` ）。
>
> 一次操作中，你可以选出两个下标，记作 `x` 和 `y` （ `0 <= x, y < n` ）并使 `arr[x]` 减去 `1` 、`arr[y]` 加上 `1` （即 `arr[x] -=1` 且 `arr[y] += 1` ）。最终的目标是使数组中的所有元素都 **相等** 。题目测试用例将会 **保证** ：在执行若干步操作后，数组中的所有元素最终可以全部相等。
>
> 给你一个整数 `n`，即数组的长度。请你返回使数组 `arr` 中所有元素相等所需的 **最小操作数** 。

示例 1

```
输入：n = 3
输出：2
解释：arr = [1, 3, 5]
第一次操作选出 x = 2 和 y = 0，使数组变为 [2, 3, 4]
第二次操作继续选出 x = 2 和 y = 0，数组将会变成 [3, 3, 3]
```

示例 2

```
输入：n = 6
输出：9
```

提示

- `1 <= n <= 10^4`

#### 题解思路

- 步长为 `2` 的递增等差数列，头尾配对操作，最终都为中位数 `to`
- 只需求前半段，共 `n / 2` 个元素，的等差数列和
- 数据范围不大，遍历 `O(n)` 也能过
- 易错点：注意 `n` 的奇偶处理

#### 参考代码

```java
public int minOperations(int n) {
    int to = (2 * (n - 1) + 1) / 2 + 1;
    int a0 = 1 + (to & 1), an = to - 1;
    return (a0 + an) * (n / 2) / 2;
}
```

#### 复杂度分析

- 时间复杂度：`O(1)`
- 空间复杂度：`O(1)`

## T3. 两球之间的磁力

#### 题目

> 在代号为 C-137 的地球上，Rick 发现如果他将两个球放在他新发明的篮子里，它们之间会形成特殊形式的磁力。Rick 有 `n` 个空的篮子，第 `i` 个篮子的位置在 `position[i]` ，Morty 想把 `m` 个球放到这些篮子里，使得任意两球间 **最小磁力** 最大。
>
> 已知两个球如果分别位于 `x` 和 `y`，那么它们之间的磁力为 `|x - y|`。
>
> 给你一个整数数组 `position` 和一个整数 `m`，请你返回最大化的最小磁力。

示例 1

![t3 eg1](/img/in-post/leetcode-contest-weekly-202/t3-eg1.jpg)

```
输入：position = [1,2,3,4,7], m = 3
输出：3
解释：将 3 个球分别放入位于 1，4 和 7 的三个篮子，两球间的磁力分别为 [3, 3, 6]。
最小磁力为 3 。我们没办法让最小磁力大于 3 。
```

示例 2

```
输入：position = [5,4,3,2,1,1000000000], m = 2
输出：999999999
解释：我们使用位于 1 和 1000000000 的篮子时最小磁力最大。
```

提示

- `n == position.length`
- `2 <= n <= 10^5`
- `1 <= position[i] <= 10^9`
- 所有 position 中的整数 互不相同
- `2 <= m <= position.length`

#### 题解思路

- 尽量都间隔开，要求放完所有球
- 对结果二分，统计能放下的个数 `cnt`

#### 参考代码

```java
public int maxDistance(int[] arr, int m) {
    Arrays.sort(arr);
    
    int l = 1, r = arr[arr.length - 1] - arr[0];
    while (l < r) {
        int mid = l + (r - l + 1) / 2;
        
        // check
        int cnt = 1;
        for (int i = 1, last = arr[0]; i < arr.length; i++) {
            if (arr[i] - last >= mid) {
                cnt++;
                last = arr[i];
            }
        }
        
        if (cnt < m) r = mid - 1;
        else l = mid;
    }
    return l;
}
```

#### 复杂度分析

- 时间复杂度：`O(nlog(n))`
- 空间复杂度：`O(1)`

## T4. 吃掉 N 个橘子的最少天数

#### 题目

> 厨房里总共有 `n` 个橘子，你决定每一天选择如下方式之一吃这些橘子：
> 
> - 吃掉一个橘子。
> - 如果剩余橘子数 `n` 能被 `2` 整除，那么你可以吃掉 `n/2` 个橘子。
> - 如果剩余橘子数 `n` 能被 `3` 整除，那么你可以吃掉 `2*(n/3)` 个橘子。
> 
> 每天你只能从以上 3 种方案中选择一种方案。
>
> 请你返回吃掉所有 `n` 个橘子的最少天数。

示例 1

```
输入：n = 10
输出：4
解释：你总共有 10 个橘子。
第 1 天：吃 1 个橘子，剩余橘子数 10 - 1 = 9。
第 2 天：吃 6 个橘子，剩余橘子数 9 - 2*(9/3) = 9 - 6 = 3。（9 可以被 3 整除）
第 3 天：吃 2 个橘子，剩余橘子数 3 - 2*(3/3) = 3 - 2 = 1。
第 4 天：吃掉最后 1 个橘子，剩余橘子数 1 - 1 = 0。
你需要至少 4 天吃掉 10 个橘子。
```

示例 2

```
输入：n = 6
输出：3
解释：你总共有 6 个橘子。
第 1 天：吃 3 个橘子，剩余橘子数 6 - 6/2 = 6 - 3 = 3。（6 可以被 2 整除）
第 2 天：吃 2 个橘子，剩余橘子数 3 - 2*(3/3) = 3 - 2 = 1。（3 可以被 3 整除）
第 3 天：吃掉剩余 1 个橘子，剩余橘子数 1 - 1 = 0。
你至少需要 3 天吃掉 6 个橘子。
```

示例 3

```
输入：n = 1
输出：1
```

示例 4

```
输入：n = 56
输出：6
```

提示

- `1 <= n <= 2*10^9`

#### 题解思路

- 分阶段决策，首先想到 DP
- 数据范围较大，[Bottom-Up](#dp-bottom-up) 会超出空间限制
- 考虑 Top-Down：DFS + Memorize（深度优先 + 记忆化）
  - 当前还差多少能够起跳（跳 `2` 或跳 `3`）

#### 参考代码

```java
public int minDays(int n) {
    return dfs(n, new HashMap<>());
}

private int dfs(int n, Map<Integer, Integer> map) {
    if (n == 0) return 0;
    if (map.containsKey(n)) return map.get(n);
    
    int res = n;
    res = Math.min(res, dfs(n / 2, map) + 1 + n % 2); // 还差 n % 2 可跳
    res = Math.min(res, dfs(n / 3, map) + 1 + n % 3); // 还差 n % 3 可跳
    
    map.put(n, res);
    return res;
}
```

#### 复杂度分析

- 时间复杂度：`O((log(n))^2)`
    - 根据 `T(n) = T(n/2) + T(n/3) + O(1)`
    - 得，递归树为二叉树，第 `i` 层共 `i` 个 `O(1)` 的计算量，树高 `log(n)`
    - 即，`1 + 2 + ... + log(n)` 共 `log(n)` 个元素的等差数列求和，得 `(log(n))^2`
- 空间复杂度：`O((log(n))^2)`
    - 每个 `O(1)` 结果都计入

#### 曾经尝试

##### DP Bottom-Up

- `dp` 超出空间限制
- 比较中规中矩的动态规划

```java
public int minDays(int n) {
    if (n == 1) return 1;
    if (n <= 3) return 2; 
    
    int[] dp = new int[n + 1]; // 设剩下 i 个橘子，空间 O(n)
    dp[1] = 1;
    dp[2] = dp[3] = 2;
    
    for (int i = 4; i <= n; i++) {
        dp[i] = dp[i - 1] + 1;
        if ((i % 2) == 0) dp[i] = Math.min(dp[i], dp[i / 2] + 1);
        if ((i % 3) == 0) dp[i] = Math.min(dp[i], dp[i / 3] + 1);
    }
    
    return dp[n];
}
```

##### DP Top-Down

- StackOverflowError
- 由 DP Bottom-Up 转换而来
- 直接 `dfs(n - 1, map)` 就给干崩了

```java
public int minDays(int n) {
    return dfs(n, new HashMap<>());
}

private int dfs(int n, Map<Integer, Integer> map) {
    if (n == 0) return 0;
    if (map.containsKey(n)) return map.get(n);
    
    int res = dfs(n - 1, map) + 1; // O(n)
    if ((n % 2) == 0) res = Math.min(res, dfs(n / 2, map) + 1);
    if ((n % 3) == 0) res = Math.min(res, dfs(n / 3, map) + 1);
    
    map.put(n, res);
    return res;
}
```

##### DP Top-Down Adv

- AC，赛后才出来，心痛
- 优化 DP Top-Down 而来
- 比赛时放弃了 dfs，转了贪心

```java
public int minDays(int n) {
    return dfs(n, new HashMap<>());
}

private int dfs(int n, Map<Integer, Integer> map) {
    if (n == 0) return 0;
    if (map.containsKey(n)) return map.get(n);
    
    int res = n;
    if ((n % 2) == 0) res = Math.min(res, dfs(n / 2, map) + 1); // 能起跳
    else res = Math.min(res, dfs(n / 2, map) + 2); // 差一步起跳

    if ((n % 3) == 0) res = Math.min(res, dfs(n / 3, map) + 1); // 能起跳
    else if (((n - 1) % 3) == 0) res = Math.min(res, dfs(n / 3, map) + 2); // 差一步起跳
    else res = Math.min(res, dfs(n / 3, map) + 3); // 差两步起跳
    
    map.put(n, res);
    return res;
}
```

##### Greedy

- WA
- 心态崩溃，小青蛙都跳上来了

```java
public int minDays(int n) {
    if (n == 1) return 1;
    if (n <= 3) return 2;
    
    int end = 1, step = 1;
    while (end < n) {
        step++;
        int next = end + 1;
        if (2 * end <= n) next = 2 * end;
        if (3 * end <= n) next = 3 * end;
        end = next;
    }
    return step;
}
```

## 赛后复盘

- T1 与以往一样，都是数组遍历的简单题
- T2 求最小操作数一般都有捷径可走
- T3 求最小距离的最大值，有点像供暖、路灯。求有范围的单值结果，考虑二分。这类型的题总算是出来一次了，团队赛那次差在了区间调整，还是思路要清晰呀
- T4 时间很充裕，却一直在自顶向下的动态规划和贪心里绕，思路不够清晰

总体是历来难度较小的一次了，没能把握住有些遗憾吧

有时候感觉还能再抢救一下，到最后才发现，其实差的不只是几行代码，而是整个题解思路的稳步推进。动态规划也好，链表、树的遍历也罢，都还需要练习。思路的差距，往往大于代码的差距。

> 认知的距离，往往大于行动的距离。
