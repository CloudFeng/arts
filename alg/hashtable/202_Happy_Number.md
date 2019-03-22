# 202. Happy Number

## 问题描述

Write an algorithm to determine if a number is "happy".

A happy number is a number defined by the following process: Starting with any positive integer, replace the number by the sum of the squares of its digits, and repeat the process until the number equals 1 (where it will stay), or **it loops endlessly in a cycle which does not include 1**. Those numbers for which this process ends in 1 are happy numbers.

### 例子

<blockquote>

Input: 19       <br>
Output: true    <br>
Explanation:    <br>
12 + 92 = 82    <br>
82 + 22 = 68    <br>
62 + 82 = 100   <br>
12 + 02 + 02 = 1<br>

</blockquote>


## 思路与代码

想到的思路也是比较简单地，主要难点是在处理循环上面：可以借鉴`set`的唯一性做判断。

```Java
    class Solution {
        private Set<Integer> splitSquareSumSet = new HashSet<Integer>();
        private int splitNumAndSquareSum(int n) {
            if (n < 10) {
                return n * n;
            }
            int squareSum = 0;
            while (n != 0) {
                //splitSet.add(n % 10);
                squareSum += Math.pow(n % 10, 2);
                n /= 10;
            }
            return squareSum;
        }
        
        public boolean isHappy(int n) {
            if (n <= 0) {
                return false;
            }
            
            int squareSum = splitNumAndSquareSum(n);
            
            if (1 == squareSum) {
                return true;
            }
            if (!splitSquareSumSet.add(squareSum)) {
                return false;
            }
            return isHappy(squareSum);
            
        }
    }
```

上面这个方法利用了递归，使用的空间复杂度比较大，若n比较大时，可能会超过最大的栈深度。可以将上面的代码改写为循环处理的方法。

```Java
    public boolean isHappy(int n) {
        Set<Integer> set = new HashSet<>();
        while(set.add(n)){
            int sum = 0;
            while(n != 0){
                sum += (n % 10) * ( n % 10);
                n /= 10;
            }
            n = sum;
        }
        return n == 1;
    }
```
---------