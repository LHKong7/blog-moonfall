## 滑动窗口 

### 滑动窗口简介
滑动窗口是一个问题解决方法来将双重循环的时间复杂度从 O(n^2) 降低到 O(n)， 在日常算法刷题中，滑动窗口也比较常见，比如 [无重复最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/?envType=study-plan-v2&envId=top-interview-150)。接下来就让我们一起学习一下滑动窗口的优化策略。


### 滑动窗口尝试解决什么类型的问题
1. 通过维护一个固定窗口，解决连续数中平均值问题：
2. 解决相邻对问题：处理有序数据结构中的相邻对
3. 查找子数组中目标值问题：当需要在原数组中，针对连续的元素找到目标值时，可以通过维护窗口的大小找到符合目标值的子数组
4. 最长/最短/最优的序列问题：当需要找到序列中相邻元素的目标值时，对比扫描整个序列，滑动窗口会更加有效。

总结为一句话：在一个大的序列中，求解连续子序列中的数据，可以使用滑动窗口来对暴力解法进行优化。

### 滑动窗口是如何降低时间复杂度的：
滑动窗口通过维护原数据的子数组或区间范围值来减少时间复杂度，对比暴力解法的针对于一个值遍历每一个对应的子数组，滑动窗口只需要维护在原数组中的连续元素的值/窗口。连续元素的值/窗口的大小可以是固定的或不固定的，通常使用哈希，双指针或者混合法来维护窗口内的数据。
- 混合法是双指针 + 哈希，通常哈希有着较快的查询速度可以提高滑动窗口算法的效率。而双指针通常来指向窗口的开始与结束。

### 滑动窗口解决法与模版
窗口的维护一般为：
- 固定窗口
- 不固定窗口
根据问题的定义，选择对应的窗口模式；以下提供了两种JS滑动窗口的模版，

- 滑动窗口的模版：

```js
const slidingWindow = (nums, k) => {
	let leftPtr = 0, rightPtr = 0; // 双指针维护窗口的起始与结束位置
	let myStore = new Map(); // 使用一个容器记录窗口中的数据

	while (rightPtr < nums.length) { // 当窗口结束位置并没有超过list长度时
		myStore.set(nums[rightPtr], rightPtr);
		rightPtr++;

		if (condition) { // 窗口中的数据满足计算的情况时
			/** 
             * 进行目标值计算 
             **/

			// 缩小窗口
			myStore.remove(nums[leftPtr];
			leftPtr++;
		}
	}
}
```
上面的模版适用于固定与不固定的窗口，如果算法题中的情况为不固定窗口时，我们则需要考虑到三个点来修改模版中 `if (condition)` 的部分：
- 何时扩大窗口
- 何时计算窗口中维护的值
- 如何修改窗口的大小

### leetcode实战

#### 固定窗口大小的问题
- [lc643.子数组最大平均数I](https://leetcode.cn/problems/maximum-average-subarray-i/description/)
> 题目描述:给你一个由 n 个元素组成的整数数组 nums 和一个整数 k 。请你找出平均数最大且 长度为 k 的连续子数组，并输出该最大平均数。

在本题中，在集合nums中，求解连续K个的数字之和的最大平均数，如果使用暴力解法，我们则需要针对每个长度为K的子数据求解最大的平均值并返回其中最大的。那么则需要O(n^2)，当中会有重复的计算。
```js
for (let i = 0; i < n-k+1; i++) {
    let currSum = nums[i];
    for (let j = 1; j < k; j++) {
        currSum += nums[i + j];
    }
    maxSum = Math.max(maxSum, currSum)
}
```
我们可以通过维护一个K长度的固定窗口来将时间复杂度优化为 `O(n)`, 减少重复计算。
```js
var findMaxAverage = function(nums, k) {
    let leftPtr = 0, rightPtr = 0;
    const n = nums.length;

    let maxSum = Number.MIN_SAFE_INTEGER;
    let currSum = 0;
    while (rightPtr < n) {
        currSum += nums[rightPtr];

        if ((rightPtr - leftPtr + 1) === k) { // 当目前窗口满足固定窗口的大小时
            maxSum = Math.max(currSum, maxSum);

            currSum -= nums[leftPtr];
            leftPtr++;
        }

        rightPtr++;
    }

    return maxSum/k;
};
```
对比针对于每个数据都计算当前的值，并与最大值进行比较选择最大值，我们可以直将窗口的最右和最左值进行移除与移入的操作，并每次当窗口中的数据长度为K时进行更新最大值的操作。

- [lc438.找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/description/)
> 题目描述：给定两个字符串 s 和 p，找到 s 中所有 p 的 异位词 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。异位词 指由相同字母重排列形成的字符串（包括相同的字符串）。
在题目中，我们可以确定需要找到s中所有p的子串，如果使用暴力解法则和上面643的解决大同小异，双重遍历（这里的双重遍历不包含判断两个子串是否满足异味词要求的计算时间复杂度）查找每个长度为`p.length`的子串并计算是否满足异位词的要求，如果满足则将结果加一。而复杂度高的原因则是存在着重复的子串计算。我们可以将 `p.length`作为固定窗口的大小来简化时间复杂度; 以下代码还有优化空间只是作为滑动窗口的例子
```js
var findAnagrams = function(s, p) {
    const cnts = new Array(26).fill(0); // 这里基于ASCII码的特性作为容器进行数据存储，可以使用Map
    for (let i = 0; i < p.length; i++){
        cnts[p[i].charCodeAt() - 'a'.charCodeAt()] += 1;
    }

    const isSame = (l1, l2) => { // toString这里可以选择其他更优雅的方式进行判断两个数组是否相等
        return l1.toString() === l2.toString() ? true : false;
    }

    let leftPtr = 0, rightPtr = 0;
    const n = s.length, fixedWindowSize = p.length;
    const res = [];
    let windowData = new Array(26).fill(0);

    while (rightPtr < n) {
        let currChar = s[rightPtr];
        windowData[currChar.charCodeAt() - 'a'.charCodeAt()] += 1
        rightPtr++;

        if (rightPtr - leftPtr === fixedWindowSize) {
            if (isSame(cnts, windowData)) {
                res.push(leftPtr);
            }

            windowData[s[leftPtr].charCodeAt() - 'a'.charCodeAt()] -= 1;
            leftPtr++;
        }
    }

    return res;
};
```

希望基于这两个问题，已经对固定窗口大小的滑动窗口有了了解，感兴趣的读者也可以通过[lc30.串联所有单词的子串](https://leetcode.cn/problems/substring-with-concatenation-of-all-words/description/)继续锻炼固定窗口的滑动窗口算法。

#### 不固定窗口大小的问题
- [lc3.无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)
> 题目描述：给定一个字符串 s ，请你找出其中不含有重复字符的 最长子串 的长度。
当遇到这个问题，如果使用暴力解法则需要针对于每一个字符，判断每一个连续的无重复子串的长度并进行计算。问题也和之前的两个问题一样，存在着重复计算。我们可以通过不固定窗口大小的滑动窗口算法来优化。我们可以确定问题中找到的是连续的子串，

```js
var lengthOfLongestSubstring = function(s) {
    if (!s) return 0;

    let leftPtr = 0, rightPtr = 0;
    let hashStore = new Set();
    const n = s.length;
    let res = 1;
    while (rightPtr < n) {
        let currChar = s[rightPtr];

        while (hashStore.has(currChar)) {
            hashStore.delete(s[leftPtr]);
            leftPtr++;
        }

        hashStore.add(currChar);
        res = Math.max(res, rightPtr - leftPtr + 1);
        rightPtr++;
    }

    return res;
};
```
- [lc76.最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)
> 给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" 。
> 注意：
> 对于 t 中重复字符，我们寻找的子字符串中该字符数量必须不少于 t 中该字符数量。如果 s 中存在这样的子串，我们保证它是唯一的答案 

```js
var minWindow = function(s, t) {
    if (s.length === 0 || t.length === 0) {
        return '';
    }
    const sLen = s.length
    const needs = new Map();
    for (let char of t) {
        if (needs.has(char)) {
			needs.set(char, needs.get(char) + 1)
		} else {
			needs.set(char, 1);
		}
    }

    let leftPtr = 0, rightPtr = 0;
    let minLen = Number.MAX_SAFE_INTEGER;
    let minSubStr = "";

    while (rightPtr < sLen) {
        let currRightChar = s[rightPtr];
		if (needs.has(currRightChar)) {
			needs.set(currRightChar, needs.get(currRightChar) - 1);
		}
        rightPtr++;
		while (checkExist(needs) && leftPtr <= rightPtr) {
			let currLeftChar = s[leftPtr];
			if (rightPtr - leftPtr < minLen) {
				minLen = rightPtr - leftPtr;
				minSubStr = s.slice(leftPtr, rightPtr)
			}
			if (needs.has(currLeftChar)) {
				needs.set(currLeftChar, needs.get(currLeftChar) + 1);
			}
			leftPtr++;
		}
    }

	return minSubStr;
};

const checkExist = (needs) => {
    for (let [_, val] of needs) {
        if (val > 0) return false;
    }

    return true;
}
```



## References：
- https://leetcode.com/discuss/interview-question/3722472/mastering-sliding-window-technique-a-comprehensive-guide
- [https://github.com/shichangzhi/fucking-algorithm-book/blob/main/第1章-核心套路篇/1.7-我写了一个模板，把滑动窗口算法变成了默写题.md](https://github.com/shichangzhi/fucking-algorithm-book/blob/main/%E7%AC%AC1%E7%AB%A0-%E6%A0%B8%E5%BF%83%E5%A5%97%E8%B7%AF%E7%AF%87/1.7-%E6%88%91%E5%86%99%E4%BA%86%E4%B8%80%E4%B8%AA%E6%A8%A1%E6%9D%BF%EF%BC%8C%E6%8A%8A%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E7%AE%97%E6%B3%95%E5%8F%98%E6%88%90%E4%BA%86%E9%BB%98%E5%86%99%E9%A2%98.md)