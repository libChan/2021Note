## 牛客剑指 Offer 9.变态跳台阶

#### 题目链接

https://www.nowcoder.com/practice/22243d016f6b47f2a6928b4313c85387?tpId=13&tqId=11162&rp=1&ru=%2Fta%2Fcoding-interviews&qru=%2Fta%2Fcoding-interviews%2Fquestion-ranking&tab=answerKey

#### 题目描述

一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

```
输入：3
输出：4
```

#### 思路

动规，最后一步可以跳任意(n)阶，则$f(n)=f(n-1)+f(n-2)+f(n-3)+...+f(0), (f(0)=1)$

可以将数组空间优化到线性

```java
//数组记忆
public class Solution {
    public int jumpFloorII(int target) {
        int[] dp = new int[target+1];
        dp[0] = 1;
        int sum = 0;
        for(int i = 1;i<target + 1;i++){
            sum += dp[i-1];
            dp[i] = sum;
        }
        return dp[target];
    }
}
```

```java
//空间优化
public class Solution {
    public int jumpFloorII(int target) {
        int sum = 0;
        int tmp = 1;
        for(int i = 1;i<target + 1;i++){
            sum += tmp;
            tmp = sum;
        }
        return sum;
    }
}
```

