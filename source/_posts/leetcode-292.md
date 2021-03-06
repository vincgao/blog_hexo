---
title: LeetCode 292 - Nim Game 解析
tags:
- 算法
categories:
- Android
comments: true
date: 2016-02-29 18:11:43
updated:
desc: leetcode 292 - Nim Game 解析
---

问题链接： https://leetcode.com/problems/nim-game/

> You are playing the following Nim Game with your friend: There is a heap of stones on the table, each time one of you take turns to remove 1 to 3 stones. The one who removes the last stone will be the winner. You will take the first turn to remove the stones.
>
> Both of you are very clever and have optimal strategies for the game. Write a function to determine whether you can win the game given the number of stones in the heap.
>
> For example, if there are 4 stones in the heap, then you will never win the game: no matter 1, 2, or 3 stones you remove, the last stone will always be removed by your friend.
>
> Hint:
> If there are 5 stones in the heap, could you figure out a way to remove the stones such that you will always be the winner?

AC code：

```java
public class Solution {
    public boolean canWinNim(int n) {
        return !(n % 4 == 0);
    }
}
```

解释：
在初始状态一定的情况下，博弈的结果不是必胜就是必败，所以必败的状态就是剩余4个石子的情况，同样4的倍数都是必败的。
关键要推导出必胜或必败的公式。(http://blog.csdn.net/tbestcc/article/details/49090531)

是我把问题想复杂了，竟然还想到了树形结构的解法。。其实这个问题可以这样简化：当我拿的时候，剩4个石头的必输。题目要求中说到：
> Both of you are very clever and have optimal strategies for the game.

所以我就要想办法给对方剩4个石头，由于是我先来，所以当石头数目是5~7个时，都会赢，到8个石头时就完了。再到9个石头，其实可以简化为5个石头的情形，前5个石头处理好，给对方剩4个，就赢了。

换句话说，剩余4的倍数的石头数必败。

