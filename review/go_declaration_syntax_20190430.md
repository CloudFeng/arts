# Go的声明语法

- 标题：Go's Declaration Syntax
- 作者：Rob Pike

## 阅读笔记

此篇文章是主要是解答为何Go语言将类型放在变量后面，先说明了C是如何声明变量和函数的，一旦出现嵌套事情就变得复杂起来，可读性和明确性变差了。比如:
`int (*(*fp)(int (*)(int, int), int))(int, int)`。 为了可读性和明确性，Go将类型声明在变量后面，遵循了**从左往右读的习惯**。比如上面的C语言例子在Go中：`f func(func(int,int) int, int) func(int, int) int`