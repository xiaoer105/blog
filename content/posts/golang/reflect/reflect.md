---
title: "golang 源码解析系列之 reflect"
date: 2021-08-30T16:30:59+08:00
hidden: false
draft: false
toc: true
tags:
  - golang
  - golang 源码解析
  - reflect
keywords: 
  - reflect
description: ""
slug: "Theme Preview"
---

## 反射是什么
通过反射可以获取更加丰富的类型信息

使用官方的话来说就是在程序执行时，允许程序操作任意类型的对象。

在golang中，反射通过`reflect`包实现。

## 反射定律
golang 存在三条反射定律
1. 反射从接口值到对象
2. 反射从反射对象到接口
3. 要修改反射对象，该对象必须是可设置的

## 类型和值
反射主要分为两种类型，分别是类型和值，通过使用 reflect 包提供的 reflect.ValueOf 和 reflect.TypeOf 可以将对象转换成 reflect 包中的 Type 和 Value 类型, 对对象进行其他的操作。

### Type
golang 源码 reflect 目录下的 type.go 记录了

```
// 类型是 Go 类型的表示形式
// 调用时需要注意，并非所有的方法都是用于所有类型，有些方法方法注明了调用的条件。
// 所以在调用方法过程中，需要保证调用的方法支持对象类型，否则会出现 panic 导致程序运行时死机
type Type interface {
	// 适用于所有类型方法
	// Align 在内存中分配时返回此类型值的对齐方式（以字节为单位）
	Align() int

	// FieldAlign 在用作 struct 中的字段时，返回此类型的对齐方式（以字节为单位）
	FieldAlign() int

	// Method 返回类型第 i 个方法实例，i 必须在 0~NumMethod 范围内，否则出现 panic
	// 只有提供给外部调用的方法才可以访问，并且他们按照字典顺序排序 a~z
	Method(int) Method

	// MethodByName 根据方法名返回类型的方法实例，以及提示是否找到该方法的 bool 类型
	// 对于非接口类型 T 或者 *T，返回的方法的 type 和 Func 字段描述其第一个参数为接收方函数
	// 对于接口类型，返回的方法的 type 字段给出方法签名，没有接收者，Func 字段为 nil
	MethodByName(string) (Method, bool)

	// NumMethod 返回类型的方法集中可导出（外部访问）方法的数量
	NumMethod() int

	// Name 返回定义类型的包的类型名称（不好含包名），对于其他自定义类型，返回空字符串。
	Name() string

	// PkgPath 返回定义类型的包路径，即唯一识别包的导入路径（import），如 "encoding/base64"。
	// 如果类型已经预先声明（string, error）或未定义（*T、struct{}、[]int或A，其中A是未定义类型的别名），则包路径将返回空字符串
	PkgPath() string

	// Size 返回存储给定类型的值所需的字节数（rtype.size），类似 unsafe.Sizeof
	Size() uintptr

	// String 返回类型名，PkgPath()+Name()
	String() string

	// Kind 返回 rtype.kind 类型的特定种类
	Kind() Kind

	// Implements 检查该类型是否实现接口类型 u
	Implements(u Type) bool

	// AssignableTo 检查该类型是否可以分配给类型 u
	AssignableTo(u Type) bool

	// ConvertibleTo 检查该类型的值是否可以转换为 u 类型
	ConvertibleTo(u Type) bool

	// Comparable 检查该类型是否可以用于比较运算（类型底层是否绑定 typeAlg 的 equal 方法）
	Comparable() bool

	// Methods 返回类型的位大小
	// 该方法仅适用于某些类型，比如:
	//	Int*, Uint*, Float*, Complex*: Bits
	//	Array: Elem, Len
	//	Chan: ChanDir, Elem
	//	Func: In, NumIn, Out, NumOut, IsVariadic.
	//	Map: Key, Elem
	//	Ptr: Elem
	//	Slice: Elem
	//	Struct: Field, FieldByIndex, FieldByName, FieldByNameFunc, NumField
	// 如果类型的种类不是大小相同或不相同的 int、uint、float 或 complex 种类之一，会出现 panic
	Bits() int

	// ChanDir 返回通道类型的方向，如果不是 chan 会 panic
	ChanDir() ChanDir

	// IsVariadic 检查返回函数类型的最后一个参数（t.In(t.NumIn() - 1)）是不是可变数量 ... 类型（[]t），
	// 例如 t 表示 func(x int, y ... float64), 那么
	//	t.NumIn() == 2
	//	t.In(0) is the reflect.Type for "int"
	//	t.In(1) is the reflect.Type for "[]float64"
	//	t.IsVariadic() == true
	// 如果类型的 kind 不是 func 会 panic
	IsVariadic() bool

	// Elem 返回类型的元素类型（指向底层的元素）
	// 如果类型不是 Array、Chan、Map、Ptr 或者 Slice 会出现 panic 
	Elem() Type

	// Field 返回 struct 类型第 i 个字段
	// 如果不是 struct 会 panic
	// 如果范围不在 0~NumField 范围内，会 panic
	Field(i int) StructField

	// FieldByIndex 返回索引序列对应的 struct 字段
	// 可以同时查找多个字段，依次按照每个索引 i 调用字段
	// 如果不是 struct 会 panic
	FieldByIndex(index []int) StructField

	// FieldByName 根据字段名查找该字段，返回查找 struct 字段和字段是否存在的 bool 值
	// 如果不是 struct 会 panic
	FieldByName(name string) (StructField, bool)

	// FieldByNameFunc 返回 struct 字段，其中名称满足匹配函数和 bool 值，提示是否找到该字段
	// 如果多个字段满足匹配函数，则它们互相取消，并且 FieldByNameFunc 返回不匹配。
	FieldByNameFunc(match func(string) bool) (StructField, bool)

	// In 返回函数类型的 i 输入参数的类型
	// 如果类型的 kind 没有 func，会出现 panic
	// 如果 i 的范围不在 0~NumIN 之前，会出现 panic
	In(i int) Type

	// Key 返回 map 的 类型，如果不是 map 会 panic
	Key() Type

	// Len 返回 array 类型的长度，如果不是 array 会 panic
	Len() int

	// NumField 返回 struct 类型字段数量
	// 如果不是 struct 会 panic
	// 如果类型的 kind 没有 struct 会 panic
	NumField() int

	// NumIn 返回 func 类型的输入参数数量，如果不是 func 会 panic
	NumIn() int

	// NumOut 返回 func 类型的输出参数数量
	// 如果不是 func 会 panic
	// 如果类型的 kind 没有 func 会 panic
	NumOut() int

	// Out 返回函数类型的 i'th 输出参数的类型
	// 如果类型的 kind 不是 func 会发生 panic
	// 如果 i 的返回不在 0~NumOut 之间会 panic
	Out(i int) Type

	common() *rtype
	uncommon() *uncommonType
}
```
使用 `reflect.TypeOf()` 函数可以得到值的类型 `reflect.Type`，`reflect.Type` 提供了一系列的获取值类型信息的方法。
```
var num int
t := reflect.TypeOf(num)
log.Println(t.Name(), t.Kind(), t.String()) // 输出 int int int
```
上述代码，定义名为 num 的 int 类型，通过 reflect.TypeOf() 得到 num 的类型对象，依次输出 num 的类型名、类型种类、字符串。

### Value
与 Type 不同的是，Value 声明为结构体，没有对外暴露字段，但是提供了写入或读取数据等一系列方法。
```
// Value is the reflection interface to a Go value.
//
// Not all methods apply to all kinds of values. Restrictions,
// if any, are noted in the documentation for each method.
// Use the Kind method to find out the kind of value before
// calling kind-specific methods. Calling a method
// inappropriate to the kind of type causes a run time panic.
//
// The zero Value represents no value.
// Its IsValid method returns false, its Kind method returns Invalid,
// its String method returns "<invalid Value>", and all other methods panic.
// Most functions and methods never return an invalid value.
// If one does, its documentation states the conditions explicitly.
//
// A Value can be used concurrently by multiple goroutines provided that
// the underlying Go value can be used concurrently for the equivalent
// direct operations.
//
// To compare two Values, compare the results of the Interface method.
// Using == on two Values does not compare the underlying values
// they represent.
type Value struct {
	// typ 反射生成出来值的类型
	typ *rtype

	// 指针值数据，或者，如果设置了 flagIndir，指向数据的指针
	// 在设置 flagIndir 时有效，或者 typ.pointers() 为 true
	ptr unsafe.Pointer

	// 保存有关该值的元数据
	// 最低位时标识位:
	//	- flagStickyRO: 通过未被实施的未嵌入字段获得，所以只读
	//	- flagEmbedRO: 通过未被实施的嵌入式现场获得，所以只读
	//	- flagIndir: val包含指向数据的指针
	//	- flagAddr: v.canaddr 为 true（implies flagIndir）
	//	- flagMethod: v是方法值。
	// The next five bits give the Kind of the value.
	// This repeats typ.Kind() except for method values.
	// The remaining 23+ bits give a method number for method values.
	// If flag.kind() != Func, code can assume that flagMethod is unset.
	// If ifaceIndir(typ), code can assume that flagIndir is set.
	flag

	//方法值表示一些接收器R的r.read等核心方法调用。 Typ + Val + Flag位描述了接收器R，但标志的种类表示Func（方法是函数），并且标志的顶部位给出了R类型的方法表中的方法编号。
	// A method value represents a curried method invocation
	// like r.Read for some receiver r. The typ+val+flag bits describe
	// the receiver r, but the flag's Kind bits say Func (methods are
	// functions), and the top bits of the flag give the method number
	// in r's type's method table.
}

// Cap 返回 v 的容量.
// 如果 v 不是 Array、Chan、Slice 类型，会引发 panic
func (v Value) Cap() int

// Close 关闭通道 v
// 如果 v 不是 Chan 类型，会引发 panic
func (v Value) Close()

// Elem 返回接口 v 包含或指针指向的值
// 如果 v 不是 Interface、Ptr 类型，会引发 panic
// 如果 v 为 nil，则返回零值
func (v Value) Elem() Value          

// Field 返回 v 第 i 个字段，主要用于遍历 struct
// 如果 v 不是 struct 类型，会引发 panic
// 如果 i 不在 0~len(findids) 范围内，会引发 panic
func (v Value) Field(i int) Value  

// FieldByIndex 返回对应索引的嵌套字段，底层调用 Field() 方法
// 如果 v 不是 struct 类型，会引发 panic
// 如果 v 是 Ptr 并且底层类型为 Struct 时，v 为 nil 会引发 panic
func (v Value) FieldByIndex(index []int) Value  

// FieldByName 根据给定的名称返回 struct 字段
// 如果找不到任何字段，返回零值
// 如果 v 不是 struct 类型，会引发 panic
func (v Value) FieldByName(name string) Value   

// FieldByNameFunc 返回具有满足匹配函数的名称的 struct 字段
// 如果 v 不是 struct 类型，会引发 panic
// 如果找不到任何字段，返回零值
func (v Value) FieldByNameFunc(match func(string) bool) Value          

// Float 返回 v 的底层值，类型为 float64
// 如果 v 不是 Float32 或 Float64 类型，会引发 panic
func (v Value) Float() float64     

// Index 返回 v 的第 i 个元素
// 如果 v 不是 Array、Slice、String 类型或者 i 超出了范围长度，会引发 panic
func (v Value) Index(i int) Value   

// Int 返回 v 的底层值，类型为 int64
// 如果 v 不是 Int、Int8、Int16、Int32 或者 Int64 类型时，会引发 panic
func (v Value) Int() int64     

// CanInterface 检查是否可以使用接口，避免 panic.
func (v Value) CanInterface() bool   

// Interface 将 v 的当前值作为 interface 返回
// 它相当于:
// var i interface{} = (v's underlying value)
// 如果 flag 为 0，会引发panic
func (v Value) Interface() (i interface{})       

// InterfaceData returns the interface v's value as a uintptr pair.
// It panics if v's Kind is not Interface.
func (v Value) InterfaceData() [2]uintptr

// IsNil 检查 v 是否为 nil
// 如果 v 不是 Chan、Func、Map、Ptr、UnsafePointer 或 Interface、Slice 类型会引发 panic
// 需要注意的是 IsNil 并不总是与 nil 正常比较。
// 例如，如果通过调用 ValueOf 与未初始化的接口变量 i 进行比较（i == nil），结果为 true，但 v.IsNil 将 panic，v 为零值 
func (v Value) IsNil() bool

// IsValid reports whether v represents a value.
// It returns false if v is the zero Value.
// If IsValid returns false, all other methods except String panic.
// Most functions and methods never return an invalid value.
// If one does, its documentation states the conditions explicitly.
func (v Value) IsValid() bool        

// IsZero 检查 v 是否是其类型的零值
// 如果类型无效，会引发 panic
func (v Value) IsZero() bool

// Kind 返回 v 的类型
// 如果 v 是零值（IsValid 返回 false）,kind 返回无效
func (v Value) Kind() Kind

// Len 返回 v 的长度
// 如果 v 不是 Array、Chan、Map、Slice 或 String 类型，会引发 panic
func (v Value) Len() int

// MapIndex 返回 map 中 key 相关联的 value 值
// 如果 v 不是 map 类型，会引发 panic
// 如果在 map 中找不到 key 或 v 为 nil，则返回零值 Value{}
// 如上，关键的值必须将其分配给 map key 类型
func (v Value) MapIndex(key Value) Value

// MapKeys 返回 map 中所有的 key，以未指定的顺序
// 如果 v 不是 map 类型，会引发 panic
// 如果 v 为 nil map，则返回空切片
func (v Value) MapKeys() []Value
    
// MapRange returns a range iterator for a map.
// It panics if v's Kind is not Map.
//
// Call Next to advance the iterator, and Key/Value to access each entry.
// Next returns false when the iterator is exhausted.
// MapRange follows the same iteration semantics as a range statement.
//
// Example:
//
//	iter := reflect.ValueOf(m).MapRange()
// 	for iter.Next() {
//		k := iter.Key()
//		v := iter.Value()
//		...
//	}
//
func (v Value) MapRange() *MapIter
                                 
// Method returns a function value corresponding to v's i'th method.
// The arguments to a Call on the returned function should not include
// a receiver; the returned function will always use v as the receiver.
// Method panics if i is out of range or if v is a nil interface value.
func (v Value) Method(i int) Value 

// NumMethod returns the number of exported methods in the value's method set.
func (v Value) NumMethod() int  

// MethodByName returns a function value corresponding to the method
// of v with the given name.
// The arguments to a Call on the returned function should not include
// a receiver; the returned function will always use v as the receiver.
// It returns the zero Value if no method was found.
func (v Value) MethodByName(name string) Value   

// NumField returns the number of fields in the struct v.
// It panics if v's Kind is not Struct.
func (v Value) NumField() int        

// Recv receives and returns a value from the channel v.
// It panics if v's Kind is not Chan.
// The receive blocks until a value is ready.
// The boolean value ok is true if the value x corresponds to a send
// on the channel, false if it is a zero value received because the channel is closed.
func (v Value) Recv() (x Value, ok bool)   

// TryRecv attempts to receive a value from the channel v but will not block.
// It panics if v's Kind is not Chan.
// If the receive delivers a value, x is the transferred value and ok is true.
// If the receive cannot finish without blocking, x is the zero Value and ok is false.
// If the channel is closed, x is the zero value for the channel's element type and ok is false.
func (v Value) TryRecv() (x Value, ok bool)

// Send sends x on the channel v.
// It panics if v's kind is not Chan or if x's type is not the same type as v's element type.
// As in Go, x's value must be assignable to the channel's element type.
func (v Value) Send(x Value)       

// TrySend attempts to send x on the channel v but will not block.
// It panics if v's Kind is not Chan.
// It reports whether the value was sent.
// As in Go, x's value must be assignable to the channel's element type.
func (v Value) TrySend(x Value) bool

// Set assigns x to the value v.
// It panics if CanSet returns false.
// As in Go, x's value must be assignable to v's type.
func (v Value) Set(x Value)  

// SetBool sets v's underlying value.
// It panics if v's Kind is not Bool or if CanSet() is false.
func (v Value) SetBool(x bool)    

// SetBytes sets v's underlying value.
// It panics if v's underlying value is not a slice of bytes.
func (v Value) SetBytes(x []byte)   

// setRunes sets v's underlying value.
// It panics if v's underlying value is not a slice of runes (int32s).
func (v Value) setRunes(x []rune)     

// SetFloat sets v's underlying value to x.
// It panics if v's Kind is not Float32 or Float64, or if CanSet() is false.
func (v Value) SetFloat(x float64)      

// SetInt sets v's underlying value to x.
// It panics if v's Kind is not Int, Int8, Int16, Int32, or Int64, or if CanSet() is false.
func (v Value) SetInt(x int64)      

// SetLen sets v's length to n.
// It panics if v's Kind is not Slice or if n is negative or
// greater than the capacity of the slice.
func (v Value) SetLen(n int)      

// SetCap sets v's capacity to n.
// It panics if v's Kind is not Slice or if n is smaller than the length or
// greater than the capacity of the slice.
func (v Value) SetCap(n int)   

// SetMapIndex sets the element associated with key in the map v to elem.
// It panics if v's Kind is not Map.
// If elem is the zero Value, SetMapIndex deletes the key from the map.
// Otherwise if v holds a nil map, SetMapIndex will panic.
// As in Go, key's elem must be assignable to the map's key type,
// and elem's value must be assignable to the map's elem type.
func (v Value) SetMapIndex(key, elem Value)         

// SetUint sets v's underlying value to x.
// It panics if v's Kind is not Uint, Uintptr, Uint8, Uint16, Uint32, or Uint64, or if CanSet() is false.
func (v Value) SetUint(x uint64)    

// SetPointer sets the unsafe.Pointer value v to x.
// It panics if v's Kind is not UnsafePointer.
func (v Value) SetPointer(x unsafe.Pointer)  

// SetString sets v's underlying value to x.
// It panics if v's Kind is not String or if CanSet() is false.
func (v Value) SetString(x string)       

// Slice returns v[i:j].
// It panics if v's Kind is not Array, Slice or String, or if v is an unaddressable array,
// or if the indexes are out of bounds.
func (v Value) Slice(i, j int) Value        

// Slice3 is the 3-index form of the slice operation: it returns v[i:j:k].
// It panics if v's Kind is not Array or Slice, or if v is an unaddressable array,
// or if the indexes are out of bounds.
func (v Value) Slice3(i, j, k int) Value  

// String returns the string v's underlying value, as a string.
// String is a special case because of Go's String method convention.
// Unlike the other getters, it does not panic if v's Kind is not String.
// Instead, it returns a string of the form "<T value>" where T is v's type.
// The fmt package treats Values specially. It does not call their String
// method implicitly but instead prints the concrete values they hold.
func (v Value) String() string  

// Type returns v's type.
func (v Value) Type() Type   

// Uint returns v's underlying value, as a uint64.
// It panics if v's Kind is not Uint, Uintptr, Uint8, Uint16, Uint32, or Uint64.
func (v Value) Uint() uint64                                                                                                                                        
```

在使用过程中，需要注意的一点是，类型值不能使用 Value 所有的方法，调用之前最好先使用 Kind() 获取值的类型，根据类型调用相应的方法，避免出现 panic。

例如：
```
package main

import (
    "log"
    "reflect"
)

func main() {
    var f float64 = 1
    fv := reflect.ValueOf(f)
    fvLen := fv.Len()
    log.Println("fvLen:", fvLen)
}
```
以上代码运行后出现 panic: reflect: call of reflect.Value.Len on float64 Value 的错误，导致程序崩溃。

我们对上面的代码稍加修改
```
package main

import (
	"log"
	"reflect"
)

func main() {
	var f float64 = 1
	fLen:= GetLength(f)
	log.Println("fLen:", fLen) // 输出 0

	var array = []int64{1,2,3,4}
	arrayLen:= GetLength(array)
	log.Println("arrayLen:", arrayLen) // 输出 4
}

// GetLength 得到长度
func GetLength(i interface{}) int {
	v := reflect.ValueOf(i)
	switch v.Kind(){// 类型判断
	case reflect.Array, reflect.String, reflect.Slice:
		return v.Len()
	default:
		return 0
	}
}
```
通过对类型的判断，可以有效地避免程序 panic。

## 应用场景

### Struct Tag

###
```

```

value 中 flag 什么情况下会为 0



