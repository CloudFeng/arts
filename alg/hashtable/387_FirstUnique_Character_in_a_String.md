# ARTS之HashTable系列（一）

## 387. First Unique Character in a String

### 问题描述

Given a string, find the first non-repeating character in it and return it's index. If it doesn't exist, return -1.

### 例子

<blockquote>

s = "leetcode" <br/>
return 0.

<br>

s = "loveleetcode",<br/>
return 2.

</blockquote>

### Note

You may assume the string contain only lowercase letters.

## 思路与代码

遍历`s`使用`Map`记录每一个字符的个数，再次遍历`s`取出计数器，若为1，则直接返回，结束就返回-1。
代码如下：

    ```Java
    public int firstUniqChar(String s) {
        if (null == s || s.isEmpty()) {
            return -1;
        }
        
        Map<Character, Integer> charCount = new HashMap<Character, Integer>();
        
        for (int i = 0; i < s.length(); ++i) {
            char tmp = s.charAt(i);
            int count = charCount.getOrDefault(tmp, 0);
            charCount.put(tmp, count + 1);
        }
        for (int i = 0; i < s.length(); ++i) {
            if (1 == charCount.get(s.charAt(i))) {
                return i;
            }
        }
        return -1;
    }
    ```

换一种计数方式，都是小写，只有26个字符，那么可以申请一个固定数组，存放每个字符出现的个数。然后遍历取出
计数数组，若为1则直接返回，否则循环结束，返回-1。

    ```Java
    public int firstUniqChar (String s) {
		if (null == s || s.isEmpty()) {
			return -1;
		}
		char [] charArr = s.toCharArray();
		int [] hashTmp = new int[26];
		
		for (char c : charArr) {
			hashTmp[c - 'a']++;
		}
		for (int i = 0; i < s.length(); ++i) {
			if (1 == hashTmp[s.charAt(i) - 'a']) {
				return i;
			}
		}
		return -1;
	}
    ```

---------