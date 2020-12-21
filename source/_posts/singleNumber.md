---
title: singleNumber
date: 2019-09-22 13:45:38
categories: 
- web前端
tags: leetCode
---
## 只出现一次的数字
给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。
说明：

你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？

示例 1:

输入: [2,2,1]
输出: 1
示例 2:

输入: [4,1,2,1,2]
输出: 4

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/single-number
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

### 解法1 应该是最慢的方法。。。违背了题意

刚看到这个题的时候我以为是用对象就出结果了，尴尬。然后仔细想了下这不是消消乐那种嘛，虽然不怎么玩，但是想起来简单做起来难， 我第一次的解法确实最繁琐的。
解法是利用了很多外部空间是数组加对象
- 利用数组判断出每个数字的循环次数
- 再用对象找出最小的那个。。。
```
/**
 * @param {number[]} nums
 * @return {number}
 */
var singleNumber = function(nums) {
	if (nums.length < 2) {
		return nums[0];
	}
	let map = {};
	let n = 1;
	let res = '';
	for (let i = 0; i < nums.length; i++) {
		n = 1;
		for (let j = 0; j < nums.length; j++) {
			if (nums[i] === nums[j]) {
				map[nums[i]] = n++;
			}
		}
	}
	console.log(map, 'map');
	let min = 1;
	for (let key in map) {
		if (map[key] < min) {
			min = map[key];
		}
		console.log(min, 'min');
		if (map[key] === min) {
			res = key;
		}
	}
	console.log(res);
	return res;
};
let t = [ 2, 1, 2 ];
let t1 = [ 4, 1, 2, 1, 2 ];
singleNumber(t);
```
然后运行时长看图：
![1.png](http://ww1.sinaimg.cn/large/006xVFBigy1g78gkzf509j30as0a8aa7.jpg)

### 解法2 

- 新建对象
- 相同的就删除
  

```
var singleNumber1 = function(nums) {
	var obj = {};
	for (let i = 0; i < nums.length; i++) {
		obj[nums[i]] = !obj[nums[i]];
		if (!obj[nums[i]]) {
			delete obj[nums[i]];
		}
	}
	console.log(obj);
	return Object.keys(obj)[0];
};

let t = [ 2, 1, 2 ];
let t1 = [ 4, 1, 2, 1, 2 ];
let t2 = [ 1, 3, 1, -1, 3 ];
singleNumber1(t1);
```

运行图解-第一次用这个软件
![l1.png](http://ww1.sinaimg.cn/large/006xVFBigy1g79oh84oi6j30n2158q6d.jpg)

运行时长
![222.png](http://ww1.sinaimg.cn/large/006xVFBigy1g79ojg56dwj309r06e3yj.jpg)

## 一些大神的解法 ：
### 解法3 
- 因为 除了某个元素只出现一次以外，其余每个元素均出现两次， 所以我们可以先排序，在以2为基数循环  
- 第一步先排序  冒号排序972ms..., 自带的排序88ms
- 以2为基数每次循环
- 因为是排好序的又是相同的是两两出现当前的不等于后一个就是唯一的，如果没有找到不相等的就返回最后一个

```
var singleNumber2 = function(nums) {
	if (nums.length < 2) {
		return nums[0];
	}
	let tmp = 0;
	// 排序
	for (let i = 0; i < nums.length; i++) {
		for (let j = 0; j < nums.length - i - 1; j++) {
			if (nums[j] > nums[j + 1]) {
				tmp = nums[j];
				nums[j] = nums[j + 1];
				nums[j + 1] = tmp;
			}
		}
	}
	for (let i = 0; i < nums.length; i += 2) {
		if (nums[i] != nums[i + 1]) {
			return nums[i];
		}
	}
	return nums[nums.length - 1];
};
```
运行时长：
![微信截图_20190925171519.png](http://ww1.sinaimg.cn/large/006xVFBigy1g7bvvn7ft2j30bm09r3yo.jpg)

### 解法三 异或运算
先来看下异或运算的描述
- 任何数与0异或都不会改变他的数值 x ^ 0 = x
- 任何数与自身异都为0 x^x = 0

正好利用以上两点完美的解决。
```
var singleNumber = function(nums) {
	let num = 0;
	for (let i = 0; i < nums.length; i++) {
		num = num ^ nums[i];
	}
	console.log(num);
	return num;
};
```
执行时长

![2.png](http://ww1.sinaimg.cn/large/006xVFBigy1g7dxgzjpi2j30ap08oaa6.jpg)