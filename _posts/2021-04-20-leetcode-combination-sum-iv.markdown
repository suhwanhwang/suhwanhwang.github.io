---
layout: post
title:  "LeetCode Combination Sum IV"
date:   2021-04-20 07:30:00 +0900
categories: algorithm
---

<https://leetcode.com/problems/combination-sum-iv/>

모든 조합으로 더한 값이 target과 같은 경우를 구하는 전형적인 Dynamic programming 문제이다. 

우리가 구하고 싶은 sum이 되는 가지수를 F(sum)이라고 하면 아래와 같이 정의할 수 있다.

```
F(sum) = F(sum - nums[0]) 
       + F(sum - nums[1]) 
       + ... 
       + F(sum - nums[nums.length - 1])
```

결과값을 `dp[sum]`에 저장하면 된다. 코드로 구현하면 아래와 같다.

{% highlight java %}
class Solution {
    private int[] dp;
    public int combinationSum4(int[] nums, int target) {
        dp = new int[1000 + 1];
        Arrays.fill(dp, -1);
        return helper(nums, target, 0);
    }
    
    private int helper(int[] nums, int target, int sum) {
        if (sum > target) {
            return 0;
        }
        
        if (sum == target) {
            return 1;
        }
        
        if (dp[sum] >= 0) {
            return dp[sum];
        }
        
        int count = 0;
        for (int num : nums) {
            count += helper(nums, target, sum + num);
        }
        dp[sum] = count;
        return count;
    }
}
{% endhighlight %}
