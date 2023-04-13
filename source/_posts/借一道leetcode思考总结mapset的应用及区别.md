---
title: 借一道leetcode思考总结map/set的应用及区别
date: 2023-04-13
tags: [c++, 算法, 哈希算法]
categories: 自己事情靠自己
---



# 前言

原题是leetcode349，要求两个数组的交集

![ ](https://s2.loli.net/2023/04/13/WmNqiBn9R6Lh7f4.png)
这题本身不难，主要是要考虑到：

1. 原题只需求“频率”，无需考虑“顺序”，则应使用哈希表结构，而不是顺序结构+两个for暴力遍历
2. 用于作键值key的是数字而非字母，所以应该用正儿八经的set/map，而不是用vector搞伪hash（否则当数字键值很大且稀疏时，vector会浪费大量空间） 
3. 不需要设置明确的key，所以用set，而不是map
4. 不考虑顺序，所以用unordered_set

上述思路理清之后，代码自然就出来了
```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) 
    {
        unordered_set<int> result_set; // 存放结果，之所以用set是为了给结果集去重
        unordered_set<int> nums_set(nums1.begin(), nums1.end());
        
        for (auto iter : nums2) 
        {
            // 发现nums2的元素 在nums_set里又出现过
            if (nums_set.find(iter) != nums_set.end()) 
            {
                result_set.insert(iter); 
            }
        }
        return vector<int>(result_set.begin(), result_set.end());
    }
};
```

由此题我们可以一窥set/map的具体使用场景，下面对其差别和应用进行简单总结

---

# 一、定义和类型
STL中的部分容器（vector、list、deque等）底层为线性序列的数据结构，故将这些容器统称为**序列式容器**，里面存储的是元素本身；对应的，有另外一种用<key，value>键值对方式储存数据的数据结构，我们称其为**关联式容器**，典型的有set类和map类容器。

这种关联式容器的motivition应该是用某种特殊的底层数据结构来代替线性序列，以避免线性结构容易导致的空间浪费问题，同时提高curd效率 -- -- 线性序列为O(n)，那再提高就是O(log n)和O(1)，对应的是啥捏？

***树和hash table嘛！这也正是set\map的底层实现方式***

具体如下：

| 集合               | 底层实现 | 是否有序 | 数值可重复 | 数值可更改 | curd效率 |
| ------------------ | -------- | -------- | ---------- | ---------- | -------- |
| std::set           | 红黑树   | 有序     | 否         | 否         | O(log n) |
| std::multiset      | 红黑树   | 有序     | 是         | 否         | O(logn)  |
| std::unordered_set | 哈希表   | 无序     | 否         | 否         | O(1)     |

| 集合               | 底层实现 | 是否有序 | 数值可重复  | 数值可更改 | curd效率 |
| ------------------ | -------- | -------- | ----------- | ---------- | -------- |
| std::map           | 红黑树   | 有序     | key不可重复 | 否         | O(log n) |
| std::multimap      | 红黑树   | 有序     | key可重复   | 否         | O(logn)  |
| std::unordered_set | 哈希表   | 无序     | key不可重复 | 否         | O(1)     |

其需要注意的是，set\multiset\map\multimap的实现都为红黑树，红黑树是一种平衡二叉搜索树，所以key有序且不能修改，修改key会导致整棵树的错乱；
而我们要用集合来解决hash问题时，优先使用unordered，因为其底层使用hash table，curd效率最高（只需执行一次hash function，复杂度为O(1)）

---

# 二、set类说明
1. 与map/multimap不同，map中存储的是真正的键值对<key, value>，set中只放value，但在底层实际存放的是由<value, value>构成的键值对（即一个元素的value同时也会标识它，value就是key）。故set中插入元素时，**只需要插入value即可，不需要构造键值对**
2. set中的元素不可以重复，因此可以使用set进行**去重**
3. set中的元素有序（默认升序），故可用iteration**遍历set得有序序列**
4. set中的元素**不允许修改**（元素总是const）
5. set中的count()方法只能返回0或1，所以其实就是个find()。。而find()返回的是查找元素的位置指针，没有则返回set.end()
6. multiset与set的区别是**前者中的元素可重复**，其它都一样
7. unordered_set与set的区别是**前者中的元素不会排序**

代码示例：

```cpp
#include<iostream>
using namespace std;
#include <set>
#include <unordered_set>


void TestSet()
{
	// 用数组array中的元素构造set
	int array[] = { 1, 3, 5, 7, 9, 2, 4, 6, 8, 0, 1, 3, 5, 7, 9, 2, 4, 6, 8, 0 };
	set<int> s;
	for (auto e : array)
		s.insert(e);

	cout << "set中的元素个数为: " << s.size() << endl;
	
	// 正向打印set中的元素，从打印结果中可以看出：set可去重
    cout << "正向打印set中的元素: " ;
	for (auto& e : s)
		cout<< e << " ";
	cout << endl;

	// 使用迭代器逆向打印set中的元素
    cout << "逆向打印set中的元素: " ;
	for (auto it = s.rbegin(); it != s.rend(); ++it)
		cout << *it << " ";
	cout << endl;

	// set中值为3的元素出现了几次
	cout << "set中值为x的元素出现了几次：" << s.count(0) << endl;

    multiset<int> muls(array, array + sizeof(array) / sizeof(array[0]));
    cout <<  "正向打印multiset中的元素: " ;
	for (auto& e : muls)
		cout <<e << " ";
	cout << endl;

    unordered_set<int> unorders(array, array + sizeof(array) / sizeof(array[0]));
    cout <<  "打印unordered_set中的元素: " ;
	for (auto& e : unorders)
		cout <<e << " ";
	cout << endl;
}

int main()
{
	TestSet();
	return 0;
}
```

运行结果：
 ![](https://s2.loli.net/2023/04/13/w8kQyJztoIBaq1E.png)


---
# 三、map类说明
1. 需要**构造键值对**
2. map支持下标访问符，即在[]中放入key，就可以找到与key对应的value；也支持.at()方法，但二者有所不同（见下面代码）
3. multimap和map的唯一不同就是：map中的**key是唯一**的，而multimap中key是**可以重复的**
4. unordered_map和map : : unordered_map存储元素时是**没有顺序的**，只是根据key的哈希值，将元素存在指定位置，所以根据key查找单个value时非常高效

代码示例（来源: [C++ STL中 set和map介绍以及使用方法](https://blog.csdn.net/qq_61635026/article/details/126070134?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166593410616782412589556%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=166593410616782412589556&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-6-126070134-null-null.142^v56^control_1,201^v3^control_2&utm_term=stl%20set%E5%92%8Cmap%E7%9A%84%E5%8C%BA%E5%88%AB&spm=1018.2226.3001.4187)）

```cpp
#include <iostream>
#include <set>
#include <map>

using namespace std;
 

void TestMap()
{
	map<string, string> m;
	// 向map中插入元素的方式：
	// 将键值对<"peach","桃子">插入map中，用pair直接来构造键值对
	m.insert(pair<string, string>("peach", "桃子"));
	// 将键值对<"peach","桃子">插入map中，用make_pair函数来构造键值对
	m.insert(make_pair("banan", "香蕉"));

	/*
	operator[]的原理是：
	 用<key, T()>构造一个键值对，然后调用insert()函数将该键值对插入到map中
	 如果key已经存在，插入失败，insert函数返回该key所在位置的迭代器
	 如果key不存在，插入成功，insert函数返回新插入元素所在位置的迭代器
	 operator[]函数最后将insert返回值键值对中的value返回
	*/

	// 将<"apple", "">插入map中，插入成功，返回value的引用，将“苹果”赋值给该引用结果
	m["apple"] = "苹果";

	// key不存在时抛异常
	//m.at("waterme") = "水蜜桃";

	cout << m.size() << endl;
	// 用迭代器去遍历map中的元素，可以得到一个按照key排序的序列
	for (auto& e : m)
		cout << e.first << "--->" << e.second << endl;
	cout << endl;
	// map中的键值对key一定是唯一的，如果key存在将插入失败
	auto ret = m.insert(make_pair("peach", "another桃子"));
	if (ret.second)
		cout << "<peach, another桃子>不在map中, 已经插入" << endl;
	else
		cout << "键值为peach的元素已经存在：" << ret.first->first << "--->"
		<< ret.first->second << " 插入失败" << endl;
	// 删除key为"apple"的元素
	m.erase("apple");
	if (1 == m.count("apple"))
		cout << "apple还在" << endl;
	else
		cout << "apple被吃了" << endl;
}

int main()
{
	TestMap();

	return 0;
}
```

运行结果：



 ![](https://s2.loli.net/2023/04/13/A3rM12B8DJKzVS7.png)
如果用at()查值，则key不在时抛出异常

```cpp
m.at("waterme") = "水蜜桃";
```

![](https://s2.loli.net/2023/04/13/bMPIHgarpTBKlu1.png)

---
# 总结
**1. 用set类还是map类？**
如果需要建立明确的**键值对应关系**（如示例中的水果），那只能用map；如果只需知道“**存在与否**”，那用set就够了（如leetcode例题，其实没有体现一个明确的key，coding时关心的也是value而不是key）

**2. 用set类还是array伪hash？**
如果key分布在一个**不大的连续区间**内（ 如26个字母），则可以直接用array，这样更快，因为set不仅占用空间比数组大，而且速度要比数组慢，set把数值映射到key上都要做hash计算的；		
但如果key随机则用set，如key为**分布稀疏**的大数字时，用数组就非常浪费空间，只能用set。

**3. 用set还是unordered_set？（map同理）**
有序set（红黑树），无序unordered_set（hash table）
	
**PS.** py中的in关键字在不同结构中（tuple, list, dict, set）查找元素时效率是相差很大的，因为dict, set底层是一个hash table；而tuple, list只是一个单纯类于数组的线性结构。。