---
title: 记位运算符(＞＞)使用不当引起的一次bug
date: 2023-04-13
tags: [bug, 算法, leetcode]
categories: 自己事情靠自己
---



# 问题描述

今天刷leetcode时遇到个死活也想不通的bug

原题很简单，线性数组插值问题，暴力遍历和二分法都可以做

 ![](https://s2.loli.net/2023/04/13/ifSOReA2dapUFsh.png)
不假思索用区间左闭右开的二分法，三下五除二就整了出来，胸有成竹😋

```c
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int left = 0, right = nums.size();
        while(left < right)
        {
            int middle = left + (right - left) >> 1;
            if (nums[middle] > target )
                right = middle;
            else if (nums[middle] < target )
                left = middle + 1;
            else return middle;
        }
        return right;
    }
};
```
谁知提交时一直在示例3（即输入数组为[1 3 5 6]，查询值target = 7）卡bug，提示超时。。


---



# 原因分析：

原以为是区间边界条件设置不当，反复检查，手动演算，脑袋想破了也觉得没问题。。

无奈debug，发现执行这一句后，middle一值变化很奇怪。。

```c
int middle = left + (right - left) >> 1;
```

突然虎躯一震，意识到可能是运算符优先级一问题，速google之，果然！

![](https://s2.loli.net/2023/04/13/ildhp5643oy9vXw.png)

***原来加减符的优先级是要高于位运算符的！***
一验证发现也的确如此
![](https://s2.loli.net/2023/04/13/r8KHytNjmulQXIv.png)



![](https://s2.loli.net/2023/04/13/pCmWrFxzQE8AY1T.png)


---

# 解决方案：
加个括号即可
```c
int middle = left +( (right - left) >> 1);
```
问题解决，顺利通关！



![](https://s2.loli.net/2023/04/13/vtzFBo3YmZqg4XV.png)

---

# 总结反思：
1. 善于使用括号，尤其是主观上希望某个式子部分先运算时。
2. 老老实实用乘除得了，别整些什么花里胡哨trick。。代码省下几毫秒，debug多花几十分钟。。