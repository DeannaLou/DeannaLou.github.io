---
title: 力扣算法1-发 Leet coin-草稿
date: 2023-01-31
tags: 算法
---

## 题目
https://leetcode.cn/problems/coin-bonus
## 解法

最开始的思路是:
1. 遍历 leadership，建一个 map 表示领导到直属下级的关系，类似 <code>{ 1: { coin, memberList: [2,4,6] }</code>
2. 遍历 operations，如果遇到方式 1 就增加 map[1].coin，遇到方式 2 就拿到所有子下属（递归遍历 map[leader].memberList 拿到所有子下属），并增加他们的 coin，遇到方式 3 再拿到所有子下属的 coin 加和。

代码很快就写完了，但是执行超时！于是重新审视这道题，如果第 2 步中的那道所有子下属列表的操作能在第 1 步中拿到，那么就不需要多次去遍历了，修改如下：

```javascript
var bonus = function (n, leadership, operations) {
  // 此时把每个人的上级、下级的关系静态化，而不是只记录直系领导和下属，是处于性能的考虑。
  // operations 传参中会多次要求输出，如果每一次输出时都临时遍历某员工的上级下级，成本是巨大的
  let memberMap = {
    /**
     * 1: {
     *  memberList: [], // 存储部门下所有的人
     *  leaderList: [], // 存储此人的领导们
     *  coin: 0, // 存储部门所有人获得的 coin 之和
     * }
     * */
  };
  leadership.forEach(([leader, member]) => {
    if (!memberMap[leader])
      memberMap[leader] = {
        memberList: [], // 可用 Set 也可Array，但执行次数多的情况下 Array 性能比 Set 好
        leaderList: [],
        coin: 0,
      };
    if (!memberMap[member])
      memberMap[member] = {
        memberList: [], 
        leaderList: [],
        coin: 0,
      };
    memberMap[member].leaderList.push(leader);
    memberMap[leader].memberList.push(member);
    memberMap[leader].leaderList.forEach((v) => {
      // 成员的领导人数变更。领导的领导也是领导
      if (memberMap[member].leaderList.indexOf(v) === -1) { // indexOf 的性能比 some 的性能好
        memberMap[member].leaderList.push(v);
      }
      // 领导们的部门人数变更
      if (memberMap[v].memberList.indexOf(member) === -1) {
        memberMap[v].memberList.push(member);
      }
    });
  });

  let result = [];
  operations.forEach(([type, member, coin]) => {
    // 给成员发，该成员及领导们的部门 coin += coin
    if (type === 1) {
      memberMap[member].coin += coin;
      memberMap[member].leaderList.forEach((v) => (memberMap[v].coin += +coin));
      return;
    }
    const addCoin = coin * (memberMap[member].memberList.length + 1);
    // 给成员及其下属发
    if (type === 2) {
      // 该成员及领导们的部门 coin += 人数 * coin
      memberMap[member].coin += addCoin;
      memberMap[member].leaderList.forEach(
        (v) =>
          (memberMap[v].coin += addCoin)
      );
      // 下属们的部门 coin += 下属的部门人数 * coin
      memberMap[member].memberList.forEach(v => {
        memberMap[v].coin += coin * (memberMap[v].memberList.length + 1)
      });
      return;
    }
    // 返回部门 coin
    result.push(memberMap[member].coin % 1000000007);
  });
  return result;
};
```
## 测试用例
https://leetcode.cn/submissions/detail/398908470/testcase/

没错的测试用例执行还是超！时！了！
// TODO 未完待续
