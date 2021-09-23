---
title: "golang 内置函数 copy"
date: 2021-09-22T23:29:28+08:00
draft: false
toc: true
images:
tags: 
  - golang
  - copy
---

golang 提供内置函数 copy() 将源切片的元素复制到目标切片元素中，如果两个切片不一致，那么会按照切片最小元素个数进行复制。

需要注意的是 copy() 函数只支持切片。

```
// dst：目标切片，src：源切片
// 返回复制的元素数，len(dst)和len(src)的最小值
func copy(dst, src []Type) int
```

## 代码实例
下面介绍 copy() 的使用
```
slice1 := []int{1, 2, 3, 4, 5}
slice2 := []int{9, 8, 7}

// 将 slice2 复制到 slice1 中
n := copy(slice1, slice2)

log.Println(slice1) // [9,8,7,4,5]
log.Println(slice2) // [9,8,7]
log.Println(n) // 3
```
上述示例中 slice2 存在3个元素[9,8,7]，将 slice2 复制到 slice1 中，可以理解为把 slice2 中的[0,1,2]三个元素依次赋值给slice1中的[0,1,2]三个元素。

copy()函数的 dst 和 src 类型必须一致，否则会出现类型不一致，无法进行 copy
```
slice1 := []int{1, 2, 3, 4, 5}
slice2 := []int64{9, 8, 7}
// Cannot use 'slice1' (type []int) as type []int64
copy(slice1, slice2)
```

从指定位置进行替换
```
slice1 := []int{1,2,3,4,5,6,7,8}
slice2 := []int{11,12,13,14,15,16,17,18}
n :=copy(slice1[3:7], slice2[2:4])

log.Println(slice1) // 1,2,3,13,14,6,7,8
log.Println(slice2) // 11,12,13,14,15,16,17,18
log.Println(n) // 2
```