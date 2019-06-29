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