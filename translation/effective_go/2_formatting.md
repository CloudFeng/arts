# 格式化

格式引起最多争议的，而且没有一定之规。程序员可以采用不同的格式化风格，但最好不要这么做。若是每个人遵守同样的格式化风格，那么有没必要为此争论不休。问题在如何达到每个遵守的风格但又没有长篇大论的风格指南。

在Go中我们采用不同的解决方案（注：不是那种风格指南）：让机器去关注这些格式化的问题。`gofmt`程序（`go fmt`命令作用于包级别的而不是源文件级别）读取`Go`程序，以一种标准的缩进、对齐、保留或者再需要的时候重新格式化注释格式输入文件。若想知道如何处理新的风格情况，只需要执行`gofmt`即可。若是结果不是很明确，你可以重新组织你的程序（或者gofmt工具bug列表），没有必要为此纠结。

作为一个例子，没有必需花时间对齐结构（类）中成员进行格式化。`Gofmt`会为你做的，下面是一个结构体声明：

```GO
type T struct {
    name string // name of the object
    value int // its value
}
```

`gofmt`将格式化列：

```GO
type T struct {
    name    string // name of the object
    value   int    // its value
}
```

All Go code in the standard packages has been formatted with gofmt.

Some formatting details remain. Very briefly:

- 缩进

我们在`gofmt`中使用制表符作用于缩进，除非你有必要才使用空格。

- 行长度

`Go`没有行长度的限制，无需担心超过打孔机的长度。如一行太长了，可以写在多行上，可以使用额外的制表符插入其中。

- 括号

相对于Java和C所需要的括号要少得多，控制结构（`if`、`for`、`switch`)再其语法上面无括号。同时，操作符优先级更为简短和名字，因此正如空格提示，此不同于其他的语言。

`x<<8 + y<<16`