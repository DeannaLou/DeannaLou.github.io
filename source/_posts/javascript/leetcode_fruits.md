---
title: 力扣算法题3-重排水果-草稿
date: 2023-02-23
tags: Javascript
---

## 题目
https://leetcode.cn/problems/rearranging-fruits/

## 解法
// TODO 未完待续

```javascript
/**
 * @param {number[]} basket1
 * @param {number[]} basket2
 * @return {number}
 */
var minCost = function(basket1, basket2) {
  const left = basket1.reduce((a,b) => a+b);
  const right = basket2.reduce((a,b) => a+b);
  if (left === right) return 0;
  const delta = Math.abs(left-right)/2;
  if (delta % 1 !== 0) return -1;
  let result = 0;
  let flag = false;
  flag = basket1.some((item, index) => {
      const rightValue = left > right ? item + delta : item - delta;
      const rightIdx = basket2.indexOf(rightValue);
      // 看看1个元素组成的组合中，差值相抵的情况
      if (rightIdx !== -1) {
        result = Math.min(item, basket2[rightIdx]);
        return true;
      }
      let count = 2;
      let find = false;
      while (!find && count <= basket1.length - index - 1) {
        
        count += 1;
      }
      // 看看多个元素组成的组合中，差值相抵的情况
      return find;
  });
  return result;
};

// 获取数组内，count 个元素相加的和的组合情况
function getSum(array, count, startIdx = 0) {
  if (count === 1) return array;
  let cur = 2;
  while (cur < array.length) {
    array.forEach((v, index) => {})
  }
}

// 获取数组内，count 个元素相加等于 sum 的元素下标集合
function getIndex(array, sum, count, list, startIdx = 0) {
  if (count === 1) return array.indexOf(sum) === -1 ? [-1] : list.concat(startIdx + array.indexOf(sum));
  let result = [];
  array.forEach((v, index) => {
      const res = getIndex(array.slice(index+1), sum-v, count-1, list.concat(startIdx+index), startIdx+index+1);
      if (res.includes(-1)) return;
      result.push(res);
  });
  return result;
}
console.log(getIndex([4, 2, 2, 3, 2, 1, 1, 5], 6, 3, []));
```