---
title: '[LeetCode]313. Super Ugly Number'
permalink: leetcode-313/
date: 2017-09-10 14:31:23
tags:
- LeetCode
- 算法
- Legacy
categories:
- 技术笔记
---

> 原题链接：[https://leetcode.com/problems/super-ugly-number/description/](https://leetcode.com/problems/super-ugly-number/description/ "313. Super Ugly Number")

### Description

Write a program to find the nth super ugly number.

Super ugly numbers are positive numbers whose all prime factors are in the given prime list primes of size k. For example, [1, 2, 4, 7, 8, 13, 14, 16, 19, 26, 28, 32] is the sequence of the first 12 super ugly numbers given primes = [2, 7, 13, 19] of size 4.

Note:
(1) 1 is a super ugly number for any given primes.
(2) The given numbers in primes are in ascending order.
(3) 0 < k ≤ 100, 0 < n ≤ 106, 0 < primes[i] < 1000.
(4) The nth super ugly number is guaranteed to fit in a 32-bit signed integer.

<!--more-->

---

### 翻译

写一个找出第n个超级丑陋数的程序。

超级丑陋数是指所有素因子均属于一个长度为k的给定素数集的正数。比如，[1, 2, 4, 7, 8, 13, 14, 16, 19, 26, 28, 32]是关于长度为4的给定素数集primes = [2, 7, 13, 19]的前12个超级丑陋数的序列。

注意：
（1） 1是关于任意一个素数集的超级丑陋数。
（2） 素数集中的数为升序。
（3） 0 < k ≤ 100, 0 < n ≤ 106, 0 < primes[i] < 1000。
（4） 所求的第n个超级丑陋数可以保证在可用32位有符号整数表示范围之内。

---

### 思路

主要的思路是通过给定的素数集元素凑出前n个超级丑陋数，存在结果集`dp[n]`中，最后取出最后一项即为结果。同时，我们还需要一个辅助数组`idx[primes.size()]`来存储下一次计算每个素数分别该乘结果集中的哪一个元素。注意结果集中的数一定是由既存结果中的数与素数相乘得出来的，所以每一轮要计算最小的`dp[idx[j]] * primes[j]`来更新结果集，之后更新对应的`idx[j]`，只要和结果集更新的元素对应乘积相等的都需要更新对应的`idx[j]`，否则结果集会有重复元素。

---

### 代码

```C++
class Solution {
public:
	int nthSuperUglyNumber(int n, vector<int>& primes) {
		vector<int> dp(n, 1), idx(primes.size(), 0);
		for (int i = 1; i < n; ++i) {
			dp[i] = INT_MAX;
			for (int j = 0; j < primes.size(); ++j) {
				dp[i] = min(dp[i], dp[idx[j]] * primes[j]);
			}
			for (int j = 0; j < primes.size(); ++j) {
				if (dp[i] == dp[idx[j]] * primes[j]) {
					++idx[j];
				}
			}
		}
		return dp.back();
	}
}
```

---

**Reference Source:**

1. [[LeetCode] Super Ugly Number 超级丑陋数](http://www.cnblogs.com/grandyang/p/5144918.html "http://www.cnblogs.com/grandyang/p/5144918.html")
