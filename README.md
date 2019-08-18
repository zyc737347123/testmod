# Go学习笔记

## 项目代码结构方案
在学习阶段先以GOPATH组织代码结构，在正式开始项目后采用go module组织代码结构

## Go module踩坑记录
- [go module使用介绍](https://roberto.selbach.ca/intro-to-go-modules/)，好学易懂

- go module是采用[语义版本控制](https://gist.github.com/5d/5252217)，所以使用go module的项目的版本号有明确的意义和命名规则

- go module是根据分布式仓库例如GitHub上，依赖包的tag和go.mod进行依赖包的版本管理的

- go module方案下，依赖包的主版本更新可以不向后兼容

- 项目主版本更新的同时必须更新go.mod文件，不然go module无法正确处理

## Go的两种内存分配方式
### make和new
看起来二者没有什么区别，都在堆上分配内存，但是它们的行为不同，适用于不同的类型

- new(T) 为每个新的类型T分配一片内存，初始化为 0 并且返回类型为*T的内存地址：这种方法 返回一个指向类型为 T，值为 0 的地址的指针，**它适用于值类型如数组和结构体**；它相当于 &T{}
- make(T) **返回一个类型为 T 的初始值**，**它只适用于3种内建的引用类型：切片、map 和 channel**
![](https://raw.githubusercontent.com/Unknwon/the-way-to-go_ZH_CN/master/images/7.3_fig7.3.png)
- 具体来说最大区别就是new返回一个数组切片类型的指针，make返回一个数组切片变量

### new的语法糖--字面量初始化 
- 使用 new 初始化：
![](https://raw.githubusercontent.com/Unknwon/the-way-to-go_ZH_CN/master/eBook/images/10.1_fig10.1-1.jpg)
- 作为结构体字面量初始化：
![](https://raw.githubusercontent.com/Unknwon/the-way-to-go_ZH_CN/master/eBook/images/10.1_fig10.1-2.jpg)
- PS：T{}是不是比&T{}节省一个指针大小的内存？

## Go的数组和切片
### 数组
- Go 语言中的数组是一种**值类型**（不像 C/C++ 中是指向首元素的指针），这样的结果就是当把一个数组赋值给另一个时，会发生一次数组内存的拷贝操作，两个数组间是独立的两个值
- 在Go中将如果想按指针传递数组，有两种方式传递**数组的指针，传递数组的切片**。一般选择传递数组的切片，因为传递数组的指针，必须指明数组的大小，有很大的局限性
![](https://raw.githubusercontent.com/Unknwon/the-way-to-go_ZH_CN/master/images/7.1_fig7.1.png)
### 切片
- 切片（slice）是对数组一个连续片段的引用（该数组我们称之为**相关数组，通常是匿名的**），所以切片是一个引用类型，切片在内存中的组织方式实际上是一个有 3 个域的结构体：指向相关数组的指针，切片长度以及切片容量。下图给出了一个长度为 2，容量为 4 的切片y
![](https://raw.githubusercontent.com/Unknwon/the-way-to-go_ZH_CN/master/images/7.2_fig7.2.png)
- 切片是左闭右开的
- 如果 s2 是一个 slice，你可以将 s2 向后移动一位 `s2 = s2[1:]`，但是末尾没有移动。切片只能向后移动，`s2 = s2[-1:]`会导致编译错误。切片不能被重新分片以获取数组的前一个元素
- 仅对访问下标时，*寻址运算符允许不写！
- **修改变什么就传什么，想改数组就传切片(数组的引用<=>指针)；想改切片，就传切片的指针**

### 切片的追加和复制

- 如果想增加切片的容量，我们必须创建一个新的更大的切片并把原分片的内容都拷贝过来

- `func append(s[]T, x ...T) []T` 其中 append 方法将 0 个或多个具有相同类型 s 的元素追加到切片后面并且返回新的切片；追加的元素必须和原切片的元素同类型。如果 s 的容量不足以存储新增元素，append 会分配新的切片来保证已有切片元素和新增元素的存储。因此，返回的切片可能已经指向一个不同的相关数组了。append 方法总是返回成功，除非系统内存耗尽了

- 如果你想将切片 y 追加到切片 x 后面，只要将第二个参数扩展成一个列表即可：`x = append(x, y...)`

- `func copy(dst, src []T) int` copy 方法将类型为 T 的切片从源地址 src 拷贝到目标地址 dst，覆盖 dst 的相关元素，并且返回拷贝的元素个数。源地址和目标地址可能会有重叠。拷贝个数是 src 和 dst 的长度最小值。如果 src 是字符串那么元素类型就是 byte。如果你还想继续使用 src，在拷贝结束后执行 `src = dst`

## Go的map

- map 是一种特殊的数据结构：一种元素对（pair）的**无序集合**，pair 的一个元素是 key，对应的另一个元素是 value，所以这个结构也称为关联数组或字典。这是一种快速寻找值的理想结构：给定 key，对应的 value 可以迅速定位

- key 可以是任意可以用 == 或者 != 操作符比较的类型，比如 string、int、float。所以数组、切片和结构体不能作为 key (含有数组切片的结构体不能作为 key，只包含内建类型的 struct 是可以作为 key 的），但是指针和接口类型可以

- value 可以是任意类型的；通过使用空接口类型，我们可以存储任意值，但是使用这种类型作为值时需要先做一次类型断言；map 也可以用函数作为自己的值，这样就可以用来做分支结构，key 用来选择要执行的函数

- **不要使用 new，永远用 make 来构造 map**

  **注意** 如果你错误的使用 new() 分配了一个引用对象，你会获得一个空引用的指针，相当于声明了一个未初始化的变量并且取了它的地址
- map 可以根据新增的 key-value 对动态的伸缩，因此它不存在固定长度或者最大限制。但是你也可以选择标明 map 的初始容量 `capacity`，就像这样：`make(map[keytype]valuetype, cap)`；**当 map 增长到容量上限的时候，如果再增加新的 key-value 对，map 的大小会自动加 1。所以出于性能的考虑，对于大的 map 或者会快速扩张的 map，即使只是大概知道容量，也最好先标明**

### 用切片作为 map 的值

- 通过将 value 定义为 `[]int` 类型或者其他类型的切片

  ```go
  mp1 := make(map[int][]int) // 用于修改切片指向的数组
  mp2 := make(map[int]*[]int) // 用于修改切片本身
  ```

- 再解释一下，第一种方式是增加一个对切片相关数组的引用；第二种方式是保存切片(指针)本身；所以第一种情况下，原来的那个切片指向新数组后(例如append)，还会指向原来的数组；第二种会指向最新的那个数组

### 测试键值对是否存在

- `val1, isPresent = map1[key1]` isPresent 返回一个 bool 值：如果 key1 存在于 map1，val1 就是 key1 对应的 value 值，并且 isPresent为true；如果 key1 不存在，val1 就是一个空值，并且 isPresent 会返回 false
- 从 map1 中删除 key1：直接 `delete(map1, key1)` 

### map类型的切片

- 假设我们想获取一个 map 类型的切片，我们必须使用两次 `make()` 函数，第一次分配切片，第二次分配 切片中每个 map 元素

  ```go
  items := make([]map[int]int, 5)
  for i:= range items {
  		items[i] = make(map[int]int, 1) // 真正分配了map的内存，而且记录在切片中
  		items[i][1] = 2
  }
  ```

### map的排序
- map 默认是无序的，不管是按照 key 还是按照 value 默认都不排序。如果你想为 map 排序，需要将 key（或者 value）拷贝到一个切片，再对切片排序，然后可以使用切片的 for-range 方法打印出所有的 key 和 value
- [示例](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/08.5.md)

## Go的struct

### struct的定义

```go
var t T // t是结构体的一个实例
t := new(T) // t是指向一个结构体实例的指针
t := T{a, b} or t:= T{key:value, key:value} // t是结构体的一个实例
t := &T{a, b} // t是指向一个结构体实例的指针
```

- 结构体类型和字段的命名遵循可见性规则，一个导出的结构体类型中有些字段是导出的，另一些不是，这是可能的
- **Go的可见性规则无处不在，函数，变量，结构体，结构体的字段和方法都是有可见性的！！！**
- **Go语言中符号的可访问性是包一级的而不是类型一级的**

### 结构体的内存布局

Go 语言中，结构体和它所包含的数据在内存中是以连续块的形式存在的，即使结构体中嵌套有其他的结构体，这在性能上带来了很大的优势。不像 Java 中的引用类型，一个对象和它里面包含的对象可能会在不同的内存空间中，这点和 Go 语言中的指针很像。下面的例子清晰地说明了这些情况：

```go
type Rect1 struct {Min, Max Point }
type Rect2 struct {Min, Max *Point }
```
![](https://raw.githubusercontent.com/Unknwon/the-way-to-go_ZH_CN/master/eBook/images/10.1_fig10.2.jpg)

### 递归结构体
- go的结构体字段类型可以是自身的指针（方便实现链表或二叉树）

### 结构体的“构造函数”

```go
type File struct {
    fd      int     // 文件描述符，注意这个变量是私有的
    name    string  // 文件名，注意这个变量是私有的
}
```

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }

    return &File{fd, name}
}
```

```go
f := NewFile(10, "./test.txt")
```

- 构造函数还有一个注意点，**构造函数的返回值应该是结构体指针而不是结构体**，如果返回结构体就会增加一次内存分配（拷贝了一个新的结构体实例出来）
- 还可以应用可见性规则禁止使用 new 函数，强制用户使用构造方法，从而使类型变成私有的，就像在面向对象语言中那样

```go
type matrix struct {
    ...
}

func NewMatrix(params) *matrix {
    m := new(matrix) // 初始化 m
    return m
}
```

```go 
package main
import "matrix"
...
wrong := new(matrix.matrix)     // 编译失败（matrix 是私有的）
right := matrix.NewMatrix(...)  // 实例化 matrix 的唯一方式
```

### 带标签的结构体

结构体中的字段除了有名字和类型外，还可以有一个可选的标签（tag）：它是一个附属于字段的字符串，可以是文档或其他的重要标记。标签的内容不可以在一般的编程中使用，只有包 `reflect` 能获取它，[具体例子](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/10.4.md)

### 匿名字段
- 结构体可以包含一个或多个 匿名（或内嵌）字段，即这些字段没有显式的名字，只有字段的类型是必须的，此时类型就是字段的名字

- 在一个结构体中对于每一种数据类型只能有一个匿名字段

- Go的结构体支持内嵌结构体，同样地结构体也是一种数据类型，所以它也可以作为一个匿名字段来使用

### 命令冲突
1. 外层名字会覆盖内层名字（但是两者的内存空间都保留），这提供了一种重载字段或方法的方式
2. 如果相同的名字在同一级别出现了两次，如果这个名字被程序使用了，将会引发一个错误（不使用没关系）。没有办法来解决这种问题引起的二义性，必须由程序员自己修正

## 方法
在 Go 语言中，结构体就像是类的一种简化形式，那么面向对象程序员可能会问：类的方法在哪里呢？在 Go 中有一个概念，它和方法有着同样的名字，并且大体上意思相同：Go 方法是作用在接收者（receiver）上的一个函数，接收者是某种类型的变量。因此方法是一种特殊类型的函数

接收者类型可以是（几乎）任何类型，不仅仅是结构体类型：任何类型都可以有方法，甚至可以是函数类型，可以是 int、bool、string 或数组的别名类型。但是接收者不能是一个接口类型，因为接口是一个抽象定义，但是方法却是具体实现；如果这样做会引发一个编译错误：**invalid receiver type…**

- **在 Go 中，类型的代码和绑定在它上面的方法的代码可以不放置在一起，它们可以存在在不同的源文件，唯一的要求是：它们必须是同一个包的**

  **类型和作用在它上面定义的方法必须在同一个包里定义，这就是为什么不能在 int、float 或类似这些的类型上定义方法。试图在 int 类型上定义方法会得到一个编译错误**

  **但是有一个间接的方式：可以先定义该类型（比如：int 或 float）的别名类型，然后再为别名类型定义方法。或者像下面这样将它作为匿名类型嵌入在一个新的结构体中。当然方法只在这个别名类型上有效**

- 类型 T（或 *T）上的所有方法的集合叫做类型 T（或 *T）的**方法集**

- 定义方法的一般格式如下：

  ```go
  func (recv receiver_type) methodName(parameter_list) (return_value_list) { … }
  ```

  如果 `recv` 是 receiver 的实例，Method1 是它的方法名，那么方法调用遵循传统的 `object.name` 选择器符号：**recv.methodName()**

  如果 `recv` 是一个指针，Go 会自动解引用

- 如果方法不需要使用 `recv` 的值(类的静态方法)，可以用 **_** 替换它，比如：

  ```go
  func (_ receiver_type) methodName(parameter_list) (return_value_list) { … }
  ```

  `recv` 就像是面向对象语言中的 `this` 或 `self`，但是 Go 中并没有这两个关键字。随个人喜好，你可以使用 `this` 或 `self` 作为 receiver 的名字
  
### 函数和方法的区别

- 函数将变量作为参数：**Function1(recv)**

- 方法在变量上被调用：**recv.Method1()**

- 在接收者是指针时，方法可以改变接收者的值（或状态），这点函数也可以做到（当参数作为指针传递，即通过引用调用时，函数也可以改变参数的状态）。

- **不要忘记 Method1 后边的括号 ()，否则会引发编译器错误：method recv.Method1 is not an expression, must be called**

- 接收者必须有一个显式的名字，这个名字必须在方法中被使用。

- **receiver_type** 叫做 **（接收者）基本类型**，这个类型必须在和方法**同样的包**中被声明。

- 在 Go 中，（接收者）类型关联的方法不写在类型结构里面，就像类那样；耦合更加宽松；类型和方法之间的关联由接收者来建立。

- **方法没有和数据定义（结构体）混在一起：它们是正交的类型；表示（数据）和行为（方法）是独立的**

### 方法和未导出字段
- 这可以通过面向对象语言一个众所周知的技术来完成：提供 getter 和 setter 方法。对于 setter 方法使用 Set 前缀，对于 getter 方法只使用成员名

  ```go
  package person
  
  type Person struct {
  	firstName string
  	lastName  string
  }
  
  func (p *Person) FirstName() string {
  	return p.firstName
  }
  
  func (p *Person) SetFirstName(newName string) {
  	p.firstName = newName
  }
  ```

### 内嵌类型的方法和继承
- 当一个匿名类型被内嵌在结构体中时，匿名类型的可见方法也同样被内嵌，这在效果上等同于外层类型 **继承** 了这些方法

- 内嵌将一个已存在类型的字段和方法注入到了另一个类型里：匿名字段上的方法“晋升”成为了外层类型的方法。当然类型可以有只作用于本身实例而不作用于内嵌“父”类型上的方法

- 可以覆写方法（像字段一样）：和内嵌类型方法具有同样名字的外层类型的方法会覆写内嵌类型对应的方法

- 结构体内嵌字段类型和结构体本身在同一个包中时，可以彼此访问对方所有的字段和方法

### 如何在类型中嵌入功能
主要有两种方法来实现在类型中嵌入功能：
- 聚合（或组合）：包含一个所需功能类型的具名字段。

- 内嵌：内嵌（匿名地）所需功能类型

### SetFinalizer
如果需要在一个对象 obj 被从内存移除前执行一些特殊操作，比如写到日志文件中，可以通过如下方式调用函数来实现：
```go
runtime.SetFinalizer(obj, func(obj *typeObj))
```
func(obj *typeObj) 需要一个 typeObj 类型的指针参数 obj，特殊操作会在它上面执行。func 也可以是一个匿名函数。在对象被 GC 进程选中并从内存中移除以前，SetFinalizer 都不会执行，即使程序正常结束或者发生错误

## Go的接口
- 接口方法的接受者是指针类型，那么必须用指针变量转换为接口；如果接受者是值类型，则没有限制，[例子](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/11.6.md)
- 指针方法可以通过指针调用
- 值方法可以通过值调用
- 接收者是值的方法可以通过指针调用，**因为指针会首先被解引用**
- 接收者是指针的方法不可以通过值调用，**因为存储在接口中的值没有地址**
- 将一个值赋值给一个接口时，编译器会确保所有可能的接口方法都可以在此值上被调用，因此不 正确的赋值在编译期就会失败
```go
  cannot use q (type Square) as type Shaper in array or slice literal:
          Square does not implement Shaper (Area method has pointer receiver)
```
- 一个接口可以包含一个或多个其他的接口，这相当于直接将这些内嵌接口的方法列举在外层接口中一样
- Go 语言规范定义了接口方法集的调用规则：
  - 类型 *T 的可调用方法集包含接受者为 *T 或 T 的所有方法集
  - 类型 T 的可调用方法集包含接受者为 T 的所有方法
  - 类型 T 的可调用方法集不包含接受者为 *T 的方法