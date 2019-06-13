# GO学习笔记


## 项目代码结构方案
在学习阶段先以GOPATH组织代码结构，在正式开始项目后采用GO module组织代码结构


## Go module踩坑记录
- [go module使用介绍](https://roberto.selbach.ca/intro-to-go-modules/)，好学易懂
- go module是采用[语义版本控制](https://gist.github.com/5d/5252217)，所以使用go module的项目的版本号有明确的意义和命名规则

- go module是根据分布式仓库例如GitHub上，依赖包的tag和go.mod进行依赖包的版本管理的

- go module方案下，依赖包的主版本更新可以不向后兼容

- 项目主版本更新的同时必须更新go.mod文件，不然go module无法正确处理

  