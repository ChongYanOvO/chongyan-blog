---
title: KMP算法
categories: 算法
tags:
  - KMP
  - 字符串
  - LeetCode
cover: 'https://bu.dusays.com/2023/07/31/64c7ac61b5f73.png'
ai: true
mathjax: true
abbrlink: 2da0528d
date: 2023-07-31 11:41:31
---
# KMP算法

KMP 算法是一个快速查找匹配串的算法,它的作用其实就是本题问题:如何快速在「原字符串」中找到「匹配字符串」
在朴素解法中,不考虑剪枝的话复杂度是 O(m∗n) 的,而 KMP 算法的复杂度为 O(m+n)
KMP 之所以能够在O(m+n) 复杂度内完成查找,是因为其能在「非完全匹配」的过程中提取到有效信息进行复用,以减少「重复匹配」的消耗

匹配过程
前缀: 对于字符串 abc...efg,我们称 abc 属于 ab...efg 的某个前缀
后缀: 对于字符串 abc...efg,我们称 efg 属于 ab...efg 的某个后缀

假设原串为 abeababeabf,匹配串为 abeabf

## 朴素匹配

不使用 KMP,采用最朴素的暴力匹配方法
首先在「原串」和「匹配串」分别各自有一个指针指向当前匹配的位置
首次匹配的「发起点」是第一个字符 a 显然,后面的 abeab 都是匹配的,两个指针会同时往右移动(黑标)
在都能匹配上 abeab 的部分,「朴素匹配」和「KMP」并无不同

![](https://bu.dusays.com/2023/08/01/64c89c23beb5e.png)
![](https://bu.dusays.com/2023/08/01/64c8a328a2d66.png)

直到出现第一个不同的位置(红标):
接下来,正是「朴素匹配」和「KMP」出现不同的地方:
先看下「朴素匹配」逻辑:
将原串的指针移动至本次「发起点」的下一个位置(b 字符处)；匹配串的指针移动至起始位置,
尝试匹配,发现对不上,原串的指针会一直往后移动,直到能够与匹配串对上位置,

![](https://bu.dusays.com/2023/08/01/64c8a3ce279fc.png)


也就是说,对于「朴素匹配」而言,一旦匹配失败,将会将原串指针调整至下一个「发起点」,匹配串的指针调整至起始位置,然后重新尝试匹配,

这也就不难理解为什么「朴素匹配」的复杂度是O(m∗n) 了,

## KMP 匹配
### 匹配过程
![](https://bu.dusays.com/2023/08/01/64c8a95fdf3aa.png)


首先匹配串会检查之前已经匹配成功的部分中里是否存在相同的「前缀」和「后缀」,如果存在,则跳转到「前缀」的下一个位置继续往下匹配:

![](https://bu.dusays.com/2023/08/01/64c8a9e0e9e73.png)


跳转到下一匹配位置后,尝试匹配,发现两个指针的字符对不上,并且此时匹配串指针前面不存在相同的「前缀」和「后缀」,这时候只能回到匹配串的起始位置重新开始,
到这里,你应该清楚 KMP 为什么相比于朴素解法更快:
- 因为 KMP 利用已匹配部分中相同的「前缀」和「后缀」来加速下一次的匹配
- 因为 KMP 的原串指针不会进行回溯(没有朴素匹配中回到下一个「发起点」的过程)


### 分析实现
我们分析一下复杂度,如果严格按照上述解法的话,最坏情况下我们需要扫描整个原串,复杂度为 O(n),在每一次匹配失败时,去检查已匹配部分的相同「前缀」和「后缀」,跳转到相应的位置,如果不匹配则再检查前面部分是否有相同「前缀」和「后缀」,再跳转到相应的位置,这部分的时间复杂度是 $O(m^2)$,因此整体的复杂度是 $O(n \cdot m^2)$,而我们的朴素解法是 $O(m \cdot n)$ 的
说明还有一些性质我们没有利用到
显然,扫描完整原串操作这一操作是不可避免的,我们可以优化的只能是「检查已匹配部分的相同前缀和后缀」这一过程
再进一步,我们检查「前缀」和「后缀」的目的其实是「为了确定匹配串中的下一段开始匹配的位置」
同时我们发现,对于匹配串的任意一个位置而言,由该位置发起的下一个匹配点位置其实与原串无关
举个例子,对于匹配串 abcabd 的字符 d 而言,由它发起的下一个匹配点跳转必然是字符 c 的位置,因为字符 d 位置的相同「前缀」和「后缀」字符 ab 的下一位置就是字符 c
可见从匹配串某个位置跳转下一个匹配位置这一过程是与原串无关的,我们将这一过程称为找 next 点
显然我们可以预处理出 next 数组,数组中每个位置的值就是该下标应该跳转的目标位置(next点)
当我们进行了这一步优化之后,复杂度是多少呢?
预处理 next 数组的复杂度未知,匹配过程最多扫描完整个原串,复杂度为 $O(n)$
因此如果我们希望整个 KMP 过程是 $O(m + n)$的话,那么我们需要在 $O(m)$的复杂度内预处理出 next数组
所以我们的重点在于如何在 $O(m)$ 复杂度内 next 数组

### next 数组的构建
![](https://bu.dusays.com/2023/08/01/64c8ad9de451f.png)

![](https://bu.dusays.com/2023/08/01/64c8ae926118a.png)

![](https://bu.dusays.com/2023/08/01/64c8aec9ac042.png)

![](https://bu.dusays.com/2023/08/01/64c8af03f18f1.png)

这就是整个 next 数组的构建过程,时空复杂度均为 $O(m)$,至此整个 KMP 匹配过程复杂度是 $O(m + n)$的

### 算法实现

[LeetCode 第 28 题 找出字符串中第一个匹配项的下标](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/description/)

```java
class Solution {
    public int strStr(String haystack, String needle) {
        if (needle.length() == 0) {
            return 0;
        }

        int[] next = new int[needle.length()];
        getNext(next, needle);

        int j = 0;
        for (int i = 0; i < haystack.length(); i++) {
            while (j > 0 && haystack.charAt(i) != needle.charAt(j)) {
                j = next[j - 1];
            }
            if (haystack.charAt(i) == needle.charAt(j)) {
                j++;
            }
            if (j == needle.length()) {
                return i - needle.length() + 1;
            }
        }
        return -1;
    }

    public void getNext(int[] next, String s) {
        int j = 0;
        next[0] = 0;

        for (int i = 1; i < next.length; i++) {
            while (j > 0 && s.charAt(i) != s.charAt(j)) {
                j = next[j - 1];
            }
            if (s.charAt(i) == s.charAt(j)) {
                j++;
            }
            next[i] = j;
        }
    }
}
```

**参考文章:** [【宫水三叶】简单题学 KMP 算法
](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/solutions/575568/shua-chuan-lc-shuang-bai-po-su-jie-fa-km-tb86/)
