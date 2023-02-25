---
title: 力扣算法2-打家劫舍
date: 2023-02-23
tags: 算法
---

## 题目

https://leetcode.cn/problems/house-robber-iv/

## 解法

初步思路是先获取窃取房屋的组合情况、再取得每一个组合的最高金额、再拿到所有最高金额中最小值。

```javascript
var minCapability = function (nums, k) {
  if (k === 1) return Math.min(...nums);
  // 1、窃取房屋的组合情况
  function getNextIndex(index, count, number) {
    // 如果后边的房间不满足窃取规则，不做任何操作
    if (nums.length - index - 1 < 2 * (k - count)) return;
    if (k === count + 1) {
      // 如果是最后一个可窃取的房间，那么拿到后边房间都可以成为组合，返回最小值即可
      if (index + 2 === nums.length - 1)
        return Math.min(number, nums[index + 2]);
      return Math.min(number, Math.min(...nums.slice(index + 2)));
    }
    // 3、返回最高金额列表中的最小值
    return Math.min(
      ...nums
        .slice(index + 2)
        .map((sec, secIdx) => {
          // 2、拿到组合中最高金额
          return getNextIndex(
            index + 2 + secIdx,
            count + 1,
            Math.max(number, sec)
          );
        })
        .filter((v) => v)
    );
  }
  // 3、返回最高金额列表中的最小值
  return Math.min(
    ...nums.map((item, index) => getNextIndex(index, 1, item)).filter((v) => v)
  );
};
```

测试一下，好使。

```javascript
// minCapability([5038,3053,2825,3638,4648,3259,4948,4248,4940,2834,109,5224,5097,4708,2162,3438,4152,4134,551,3961,2294,3961,1327,2395,1002,763,4296,3147,5069,2156,572,1261,4272,4158,5186,2543,5055,4735,2325,1206,1019,1257,5048,1563,3507,4269,5328,173,5007,2392,967,2768,86,3401,3667,4406,4487,876,1530,819,1320,883,1101,5317,2305,89,788,1603,3456,5221,1910,3343,4597], 28)
```

但是太慢，leetCode 还是过不去！
题为求能偷的 k 间房屋中最小金额。于是想到用二分法来做：
1. 对金额先做二分，设定两个指针，一个指最左，一个指最右，设定一个 mid 值为此平均值
2. 判断 mid 是否符合 k 间房屋的设定；遍历 nums 中所有小于 mid 的值，存在不相邻的 k 个即符合设定
3. 如果符合说明它还可能不是最小的金额，要往左查找；如果不符合往右查找

```javascript
/*
 * @param {number[]} nums 房屋及其价值
 * @param {number} k 最少能偷多少家
 * @return {number}
 */
var minCapability = function (nums, k) {
  let L = Math.min(...nums);
  let R = Math.max(...nums);
  function check(n) {
    let cnt = 0;
    for (let i = 0; i < nums.length; i++) {
      if (nums[i] <= n) {
        cnt++;
        i++;
      }
    }
    return cnt >= k;
  }
  while (L < R) {
    let mid = Math.floor((L + R) / 2);
    if (check(mid)) {
      R = mid;
    } else {
      L = mid + 1;
    }
  }
  return R;
};
```
