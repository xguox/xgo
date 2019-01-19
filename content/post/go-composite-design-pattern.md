---
title: "Go 的组合模式(Composite Pattern)"
date: 2018-09-13T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

**Go** 没有传统面向对象语言(如 Ruby, Java) 的继承特性, 取而代之, 更多的是用 **组合模式** 来达到类似效果.

##### 组合设计模式 Composite Design Pattern

**组合构建的是一个树形的层级对象, 一个对象包含有其他一些拥有各自独立的字段和方法的对象.** 换一个说法讲, **组合代表「拥有 has」 关系, 而继承则代表「是 is」关系.** 这种模式可以解决(多)继承的问题, 典型的比如, 两个实体分别继承自两个不同的类, 而这两个实体之间实际上并没有任何关联关系.

举个例子, Athlete(运动员)类有一个 Train(训练)方法, 然后 SwimmerAthlete(游泳运动员) 继承自 Athlete, 并有自己的 Swim(游泳)方法. 可能还会有一个 Rider(骑手)类也继承自 Athlete, 然后有一个自己的 Ride(骑行)方法.

另外的, 还有一个 Animal(动物)类, 有 Eat, Sleep 等方法. 然后类似有 Dog(🐶) 类继承于它, 并有着特有的 Bark(叫)方法 一切看起来都没什么特别之处, 很常规的面向对象的继承就可以实现.  但是, **问题来了**, 如果有一个 Fish(鱼) 类, 同样也有一个 Swim 方法, 那么就不好办了, Fish 不是 Athlete 啊. Fish 会 Swim 但不会 Train (不抬杠哈). 这时候的最佳实践可能会是定义一个需要实现 Swim 方法的 Swimmer 接口, 然后 SwimmerAthlete 和 Fish 都分别实现这个接口.  但结果即使是一模一样的方法, Swim 还是要被实现两次. 如果有一个 Triathlete(三项全能运动员)类呢? 又得再实现一次一模式样的?

**运用组合模式则可以很好的解决这个问题**, 用 Go 有两种实现这种组合的方式, 先来看第一种.

```go
type Athlete struct{}

func (a *Athlete) Train() {
    fmt.Println("Training")
}
```

接下来建立一个 Swimmer 结构体, 这里加上 A 用以标识是第一种区别于后边的第二种.

```go
type CompositeSwimmerA struct{
    MyAthlete Athlete
    MySwim func()
}
```

CompositeSwimmerA 这个类型有一个 `Athlete` 类型的 `MyAthlete` 字段, 和一个 `func()` 类型的 `MySwim` 字段.

定义一个 Swim 方法之后可以赋值到上述的 `MySwim` 字段.

```go
func Swim(){
    fmt.Println("Swimming!")
}


swimmer := CompositeSwimmerA{
    MySwim: Swim,
}
swimmer.MyAthlete.Train()
swimmer.MySwim()
```

这里的 swimmer 对象因为没有赋值给 `MyAthlete` 字段, 所以默认为`Athlete` 类型的零值.

```go
$ go run main.go
Training
Swimming!
```

那么鱼呢?

```go
type Animal struct{}

func (r *Animal)Eat() {
    println("Eating")
}

type Fish struct{
    Animal
    Swim func()
}

nimo := Fish{
    Swim: Swim,
}
nimo.Eat()
nimo.Swim()
```

「继承」了对应的类型并拥有公用的 Swim 方法

```go
$ go run main.go
Eating
Swimming!
```

这里 Fish 的实现与 Athlete 有一些不一样的地方是用了嵌套类型. 顺便就有了我们前面提的第二种方式.

定义一个需要实现 Swim 方法的 Swimmer 接口, 以及一个实现了该接口的结构体 `SwimmerImpl`, 最后嵌套在 `CompositeSwimmerB` 这个结构体里边(相应鱼的实现也类似).

```go
type Swimmer interface {
    Swim()
}

type Trainer interface {
    Train()
}
type SwimmerImpl struct{}

func (s *SwimmerImpl) Swim(){
    println("Swimming!")
}

type CompositeSwimmerB struct{
    Trainer
    Swimmer
}

```

**这种做法的好处是, 里层字段对象更可控, 不会一不小心变成了零值**, 使用方式:

```go
swimmer := CompositeSwimmerB{
    &Athlete{},
    &SwimmerImpl{},
}

swimmer.Train()
swimmer.Swim()

```