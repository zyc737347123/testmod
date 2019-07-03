# GO学习笔记


## 项目代码结构方案
在学习阶段先以GOPATH组织代码结构，在正式开始项目后采用GO module组织代码结构


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