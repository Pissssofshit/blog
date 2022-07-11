[TOC]
## Go可比较类型
基础类型: Boolean(布尔值)、Integer(整型)、Floating-point(浮点数)、Complex(复数)、String(字符)
指针 ：如果两个指针指向同一个变量，或者两个指针类型相同且值都为 nil，则它们相等。注意，指向不同的零大小变量的指针可能相等，也可能不相等。
Channel (通道)具有可比性：如果两个通道值是由同一个 make 调用创建的，则它们相等。

interface (接口值)具有可比性：如果两个接口值具有相同的动态类型和相等的动态值，则它们相等。

## 不可比较类型
slice、map、function 这些是不可以比较的，但是也有特殊情况，那就是当他们值是 nil 时，可以与 nil 进行比较。

## struct 是否可比较
如果所有字段都具有可比性，则 struct (结构体值)具有可比性：如果它们对应的非空字段相等，则两个结构体值相等。
如果 array(数组)元素类型的值是可比较的，则数组值是可比较的：如果它们对应的元素相等，则两个数组值相等。

如果一定要比较struct,可以使用reflect.DeepEqual
```
func main() {
    v1 := Value{Name: "煎鱼", GoodAt: []string{"炸", "煎", "蒸"}}
    v2 := Value{Name: "煎鱼", GoodAt: []string{"炸", "煎", "蒸"}}
    if reflect.DeepEqual(v1, v2) {
        fmt.Println("脑子进煎鱼了")
        return
    }

    fmt.Println("脑子没进煎鱼")
```

## reflect.DeepEqual 原理
应该是类型判断，是slice就逐个比较这样吧