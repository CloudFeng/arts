# 前言

`Go`是一门新的语言，尽管它从现有的语言中借鉴了很多想法，但其特有的属性让高效的`Go`程序不同于它亲族语言写的代码。直接把`C++`、`Java`程序改写为`Go`,并不会有一个满意的结果，是因为程序使用`Java`写，而非`Go`。另外一方面，从`Go`语言的角度面对问题，会写出成功但十分不同的程序。总之，理解`Go`的特性和习惯写法，比如命名、格式化、程序的构建等等，对写出好的`Go`代码是十分重要的。熟悉这些，以便写出让其他`Go`程序员易读的代码。

此文档为写简洁、地道的`Go`代码给出相关的提示，并且是语言规范](http://docs.studygolang.com/ref/spec)、[go旅途](http://tour.golang.org/)和 [如何写go程序](http://docs.studygolang.com/doc/code.html)的补充，上述提到的资料都是你需要提前读。

## 示例

[`Go`语言包源码](http://docs.studygolang.com/src/)不仅是核心库，同时也是如何使用`Go`语言的示例程序。甚至，很多包都包含可工作、可执行的示例，你可以在[golang.org](golang.org) 执行，比如[example_Map](http://golang.org/pkg/strings/#example_Map)（如果有需要，点击“例子”就可以打开）。若你有解决问题存在疑问或者实现方面存在难点，库中的文档、代码和示例都可能为你提供答案、思路和背景知识。