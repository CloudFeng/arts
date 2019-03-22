# Go语言移除切片或数组中的重复元素

## 问题描述

  在统计一个字符串出现在哪些文件里面，本想想构建一个类似 `map[string] set[string]`类似，发现Go居然没有set类型。没有想明白，为啥Go没有内置的Set的。后面只能构建一个`map[string] []string`,然后在对切片进行删除重复的元素。

## 解决方法

使用map指示每个元素是否在切片或数组中；若不在加入并设置为true；否则过滤掉。具体实现如下：

```Go
// 删除slice中重复的元素
func removeDuplicatesFromSlice(s []string) []string {
    // 创建一个map指示每个元素是否在切片或数组中
    m := make(map[string]bool)
    // 用于存放结果
	result := []string{}

    // 遍历
	for _, entry := range s {
		if _, value := m[entry]; !value {
			m[entry] = true
			result = append(result, entry)
		}
	}
	return result
}
```