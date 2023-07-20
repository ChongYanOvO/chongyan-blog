---
title: 多数元素II
categories: LeetCode
tags:
  - 摩尔投票
  - 哈希表
cover: "https://bu.dusays.com/2023/06/13/64880f6b82d0c.png"
abbrlink: d988e980
date: 2023-06-17 13:49:20
---

# 多数元素 II

LeetCode 第 229 题 {% label 中等题 orange %}

给定一个大小为  `n`  的整数数组,找出其中所有出现超过  `⌊ n/3 ⌋`  次的元素。

{% label 示例1 green %}

> 输入:nums = [3,2,3]
> 输出:[3]

{% label 示例2 green %}

> 输入:nums = [1]
> 输出:[1]

{% label 示例3 green %}

> 输入:nums = [1,2]
> 输出:[1,2]

{% folding yellow, 查看提示 %}
- `1 <= nums.length <= 5 * 104`
- `-109 <= nums[i] <= 109`
{% endfolding %}

进阶:尝试设计时间复杂度为 $$O(n)$$、空间复杂度为 $$O(1)$$的算法解决此问题。

## HashMap 法

{% folding yellow, HashMap法解题思路 %}
遍历整个数组,对记录每个数值出现的次数(利用 `HashMap`,其中 `key` 为数值,`value` 为出现次数)
再去遍历这个 `HashMap` ,如果这个数值出现的次数 > `⌊ n/3 ⌋` 的话,那这个数值就是要寻找的值
出现的数值最终要放到 list 中返回

{% endfolding %}

```java
class Solution {
    public List<Integer> majorityElement(int[] nums) {
        int n = nums.length  / 3 ;
        HashMap<Integer, Integer> map = new HashMap<>(n);
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < nums.length; i++) {
            if (map.containsKey(nums[i])){
                map.put(nums[i], map.get(nums[i]) + 1);
            }else {
                map.put(nums[i],1);
            }
        }
        for (Integer key : map.keySet()) {
            if (map.get(key) > n){
                list.add(key);
            }
        }
        return list;
    }
}
```

## 摩尔投票法

{% folding yellow, 摩尔投票法解题思路 %}
超过  `⌊ n/3 ⌋`  次的元素很显然的可以得知最多可能是 2 个
同理也可以推到出现 `k` 次的元素最多可能是 `k - 1` 次
定义两个候选者和两个投票计数器,先遍历 `nums` 数组,找到这两个候选者
> 判断候选者和当前数组元素是否相等,相等的话投票计数器 + 1
> 不相等的话两个投票计数器都 - 1
> 注意当计数器为 0 时,要将候选者替换为当前的数组元素
将投票计数器置0后再次遍历 `nums` 数组,确定这两个候选者的出现次数
满足票数 > `⌊ n/3 ⌋`的话就放到list中返回

{% endfolding %}

```java
class Solution {
    public List<Integer> majorityElement(int[] nums) {
        int cand1 = nums[0];
        int cand2 = nums[0];
        int count1 = 0;
        int count2 = 0;
        for (int i = 0; i < nums.length; i++) {
            if (cand1 == nums[i]) {
                count1++;
            } else if (cand2 == nums[i]) {
                count2++;
            } else if (count1 == 0) {
                cand1 = nums[i];
                count1 = 1;
            } else if (count2 == 0) {
                cand2 = nums[i];
                count2 = 1;
            } else {
                count1--;
                count2--;
            }
        }
        count1 = 0;
        count2 = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == cand1) {
                count1++;
            } else if (nums[i] == cand2) {
                count2++;
            }
        }
        ArrayList<Integer> list = new ArrayList<>();
        if (count1 > nums.length / 3) {
            list.add(cand1);
        }
        if (count2 > nums.length / 3) {
            list.add(cand2);
        }
        return list;
    }
}
```

{% label 解题思路: purple %}
{% note simple %}
{% link 【宫水三叶の相信科学系列】详解摩尔投票为何能推广到 n / k 的情况,,https://leetcode.cn/problems/majority-element-ii/solution/gong-shui-san-xie-noxiang-xin-ke-xue-xi-ws0rj/ %}

{% endnote %}
