---
title: 多数元素I
categories: 算法
tags:
  - LeetCode
  - 摩尔投票
  - 哈希表
cover: 'https://bu.dusays.com/2023/06/13/648837aae27ce.png'
abbrlink: a5831ec7
date: 2023-06-16 00:01:34
---

# 多数元素I
LeetCode 第169题 {% label 简单题 green %}

给定一个大小为 `n` 的数组 `nums` ,返回其中的多数元素。多数元素是指在数组中出现次数 大于 `⌊ n/2 ⌋` 的元素。
你可以假设数组是非空的,并且给定的数组总是存在多数元素。

{% label 示例1 green %}
> 输入:nums = [3,2,3]
> 输出:3

{% label 示例2 green %}
> 输入:nums = [2,2,1,1,1,2,2]
> 输出:2
 

{% folding yellow, 查看提示 %}
- `n == nums.length`
- `1 <= n <= 5 * 104`
- `-109 <= nums[i] <= 109`
{% endfolding %}

 
进阶:尝试设计时间复杂度为 $$O(n)$$、空间复杂度为 $$O(1)$$ 的算法解决此问题。

## HashMap法
{% folding yellow, HashMap法解题思路 %}
遍历整个数组,对记录每个数值出现的次数(利用 `HashMap`,其中 `key` 为数值,`value` 为出现次数)
再去遍历这个 `HashMap` ,如果这个数值出现的次数 > `⌊ n/2 ⌋` 的话,那这个数值就是要寻找的值

{% endfolding %}
```java
class Solution {
    public int majorityElement(int[] nums) {
        int n = nums.length  >> 1 ;
        HashMap<Integer, Integer> map = new HashMap<>(n);
        for (int i = 0; i < nums.length; i++) {
            if (map.containsKey(nums[i])){
                map.put(nums[i], map.get(nums[i]) + 1);
            }else {
                map.put(nums[i],1);
            }
        }
        for (Integer key : map.keySet()) {
            if (map.get(key) > n){
                return key;
            }
        }
        return -1;
    }
}
```

## 摩尔投票法
{% folding yellow, 摩尔投票法解题思路 %}
候选人(`cand`)初始化为 `nums[0]`,票数 `count` 初始化为 1。
当遇到与 `cand` 相同的数,则票数 `count = count + 1`,否则票数 `count = count - 1`。
当票数 `count` 为 `0` 时,更换候选人,并将票数 `count` 重置为 1。
遍历完数组后,`cand` 即为最终答案。

投票法是遇到相同的则 `票数 + 1`,遇到不同的则 `票数 - 1`。
且"多数元素"的个数 > `⌊ n/2 ⌋`,其余元素的个数总和 <= `⌊ n/2 ⌋`。
因此“多数元素”的个数 - 其余元素的个数总和 的结果 肯定 >= 1。
这就相当于每个 “多数元素” 和其他元素 两两相互抵消,抵消到最后肯定还剩余 至少1个 “多数元素”。

无论数组是 1 2 1 2 1,亦或是 1 2 2 1 1,总能得到正确的候选人。
{% endfolding %}

```java
class Solution {
    public int majorityElement(int[] nums) {
        int cand = nums[0], count = 1;
        for (int i = 0; i < nums.length; i++) {
            if (cand == nums[i]) {
                count++;
            } else if (--count == 0) {
                cand = nums[i];
                count = 1;
            }
        }
        return cand;
    }
}
```


{% label 解题思路: purple %}
{% note simple %}
    {% link Java-3种方法(计数法/排序法/摩尔投票法),,https://leetcode.cn/problems/majority-element/solution/3chong-fang-fa-by-gfu-2/ %}

{% endnote %}