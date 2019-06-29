# GO学习笔记


## 项目代码结构方案
在学习阶段先以GOPATH组织代码结构，在正式开始项目后采用GO module组织代码结构


## Go module踩坑记录
- [go module使用介绍](https://roberto.selbach.ca/intro-to-go-modules/)，好学易懂

- go module是采用[语义版本控制](https://gist.github.com/5d/5252217)，所以使用go module的项目的版本号有明确的意义和命名规则

- go module是根据分布式仓库例如GitHub上，依赖包的tag和go.mod进行依赖包的版本管理的

- go module方案下，依赖包的主版本更新可以不向后兼容

- 项目主版本更新的同时必须更新go.mod文件，不然go module无法正确处理

## Go的两种变量内存分配
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

- **注意** 绝对不要用指针指向 slice。切片本身已经是一个引用类型，所以它本身就是一个指针!!

### 切片的追加和复制

- 如果想增加切片的容量，我们必须创建一个新的更大的切片并把原分片的内容都拷贝过来
- `func append(s[]T, x ...T) []T` 其中 append 方法将 0 个或多个具有相同类型 s 的元素追加到切片后面并且返回新的切片；追加的元素必须和原切片的元素同类型。如果 s 的容量不足以存储新增元素，append 会分配新的切片来保证已有切片元素和新增元素的存储。因此，返回的切片可能已经指向一个不同的相关数组了。append 方法总是返回成功，除非系统内存耗尽了
- 如果你想将切片 y 追加到切片 x 后面，只要将第二个参数扩展成一个列表即可：`x = append(x, y...)`