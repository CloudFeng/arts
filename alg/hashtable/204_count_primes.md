# 204. Count Primes

## 问题描述

Count the number of prime numbers less than a non-negative number, n.

### 例子

<blockquote>

Input: 10
Output: 4
Explanation: There are 4 prime numbers less than 10, they are 2, 3, 5, 7.

</blockquote>


## 思路与代码

开始想到的思路，比较直观，对每个`2~n`做判断，若是素数就累计加1，循环结束就返回结果。但此方法在n特别大的时候耗费的时间很长。时间复杂是`O(n^3/2)`，近似约为`O(n^2)`。最为耗费的时间是在判断素数的方法，没有很好地利用素数的特性。

```Java
     public int countPrimes(int n) {
		 int count = 0;
		 for (int i = 2; i < n; ++i) {
			 if (isPrime(i)) {
				 ++count;
			 }
		 } 
		 return count;
	 }
	 
	 private boolean isPrime(int n) {
		 if (n <= 2) {
			 return n > 1;
		 }
		 if (n % 2 == 0) {
			 return false;
		 }
		 for (int i = 3; i * i <= n; i += 2) {
			 if (n % i == 0) {
				 return false;
			 }
		 } 
		 return true;
	 }
```

在判断素数的时候就进行统计，利用素数的特性，若一个数`i`是素数，那么对于`i*j<n`的`i*j`都不是素数。
这个方法，我没有想出来，leetcode上标签为`hashtable`，就一直往`map/set`方向考虑。其实还有变种，
类似于位图一般。

```Java
    public int countPrimes(int n) {
        int count = 0;
        boolean []notPrime = new boolean[n];
        for (int i = 2; i < n; ++i) {
            // 是素数
            if (!notPrime[i]) {
                ++count;
                // i * j 不是素数
                for (int j = 2; i * j < n; j++) {
                    notPrime[i*j] = true;
                }
            }
        }
        return count;
    }
```
---------