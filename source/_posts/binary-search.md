---
title: 【算法基础】二分查找（Binary Search）
date: 2023-04-09 13:08:17
tags: [算法,二分]
math: true
---

在力扣周赛中，面对 **“最大值最小”** 或者 **“最小值最大”** 此类问题时总是没有头绪，知道要用二分但往往不知道应该对谁做二分，这篇文章整理一下二分的常见问题。

<!-- more -->

## 二分的两种模板

- 查找不小于x的第一个位置

  ``` cpp
  int l = 0, r = n - 1;
  while(l < r) {
    int mid = l + r >> 1;
    if(nums[mid] < x) l = mid + 1;
    else r = mid;
  }
  ```

- 查找不大于x的最后一个位置

  ``` cpp
  int l = 0, r = n - 1;
  while(l < r) {
    int mid = l + r + 1 >> 1;
    if(nums[mid] <= x) l = mid;
    else r = mid - 1;
  }
  ```

在头文件`<algorithm>`中

- `lower_bound` 用于在指定区域内查找不小于目标值的第一个元素

- `upper_bound` 用于在指定范围内查找大于目标值的第一个元素


## LeetCode真题

### 🟡[**6359. 最小化数对的最大差值**](https://leetcode.cn/problems/minimize-the-maximum-difference-of-pairs/)

给你一个下标从 **0** 开始的整数数组 `nums` 和一个整数 `p` 。请你从 `nums` 中找到 `p` 个下标对，每个下标对对应数值取差值，你需要使得这 `p` 个差值的 **最大值** **最小**。同时，你需要确保每个下标在这 `p` 个下标对中最多出现一次。

对于一个下标对 `i` 和 `j` ，这一对的差值为 `|nums[i] - nums[j]|` ，其中 `|x|` 表示 `x` 的 **绝对值** 。

请你返回 `p` 个下标对对应数值 **最大差值** 的 **最小值** 。

**示例 1：**

```
输入：nums = [10,1,2,7,1,3], p = 2
输出：1
解释：第一个下标对选择 1 和 4 ，第二个下标对选择 2 和 5 。
最大差值为 max(|nums[1] - nums[4]|, |nums[2] - nums[5]|) = max(0, 1) = 1 。所以我们返回 1 。
```

**示例 2：**

```
输入：nums = [4,2,1,2], p = 1
输出：0
解释：选择下标 1 和 3 构成下标对。差值为 |2 - 2| = 0 ，这是最大差值的最小值。
```

**提示：**

- `1 <= nums.length <= 105`
- `0 <= nums[i] <= 109`
- `0 <= p <= (nums.length)/2`

### 解答

``` cpp
class Solution {
public:
    int minimizeMax(vector<int>& nums, int p) {
        int n = nums.size();
        sort(nums.begin(), nums.end());
        int l = 0, r = nums.back() - nums.front();
        while(l < r) {
            int mid = l + r >> 1, cnt = 0;
            for(int i = 1; i < n; i ++) {
                if(nums[i] - nums[i - 1] <= mid) {
                    cnt ++;
                    i ++;
                }
            }
            if(cnt < p) l = mid + 1;
            else r = mid;
        }
        return l;
    }
};
```