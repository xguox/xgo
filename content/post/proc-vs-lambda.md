---
title: "Lambda 和 Proc 的区别在哪儿"
date: 2011-11-18T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

proc是代码块的对象形式,它的行为就像一个代码块.Lambda的的行为略有不同,它的行为更像方法而非代码块.调用一个proc则像对代码块进行yield,而调用一个lambda则像调用一个方法.在Ruby1.9中的,可以通过Proc对象的实例方法 lambda? 来判定该实例是一个proc还是lambda,如果返回值为真,那么它是一个lambda,否则为一个proc.

##### 代码块、proc和lambda中的return语句

在一个代码块中的return语句不仅仅会从调用代码块的迭代器返回,它还会从调用迭代器的方法返回.例如:

```ruby
  def  test
    puts "entering method"
    1.times{puts "entering block";return}
    puts "exiting method"
  end
  test
```

proc与代码块类似,因此如果调用的proc执行一个return语句,它会试图从代码块(这个代码块被转换为一个proc)所在的方法中返回.比如:

```ruby
  def  test
    puts "entering method"
    p =Proc.new {puts  "entering proc";return}
    p.call
    puts "exiting method"
  end
  test
```

不过,因为proc经常在不同方法间传递,在proc中使用return语句会十分诡异.在proc被调用时,在句法上包含该proc的方法已经返回:


```ruby
  def procBuilder(message)
    Proc.new {puts message ;return}
  end

  def test
    puts "entering method"
    p = procBuilder("entering proc")
    p.call
    puts "exiting method"
  end
  test
```

在把代码块转换成对象后,可以四处传递该对象,并且在"上下文"之外使用.如果这样做,则要承担从一个已经返回的方法中返回的风险.就像本例所示那样.当这种情况发生时,Ruby会抛出一个LocalJumpError异常.

当然,在这个臆造的例子中,可以通过去掉多余的return语句来修复这个问题.不过return语句并非总是多余的,这时可以通过使用lambda而非proc来修复这个问题.如前所述,lambda更像方法而非代码块.这样,在lambda中的return语句仅仅从lambda自身返回.而不会从产生lambda的方法中返回:

```ruby
def  test
  puts "entering method"
  p = lambda{puts  "entering lambda";return}
  p.call
  puts "exiting method"
end
test
```

Lambda中的return仅仅从lambda自身返回,这个事实意味着我们根本无须考虑LocalJumpError;

```ruby
def lambdaBuilder(message)
  lambda {puts message;return}
end

def test
  puts   "exiting method"
  l =lambdaBuilder("entering lambda")
  l.call
  puts  "exiting method"
end
test
```

##  代码块、proc和lambda中的break语句

当我们用Proc.new创建一个proc对象时,这个Proc.new就是break语句应该返回的地方,当我们调用proc对象的时候,这个迭代器已经返回了,因此,用Proc.new创建一个顶级break语句是说不通的:

```ruby
  def test
    puts  "entering test method "
    proc =Proc.new{puts "entering proc";break}
    proc.call
    puts  "exiting test method"
  end
  test
```

如果通过迭代器方法的&参数方式创建一个proc,我们可以调用它让该迭代器方法返回；

```ruby
  def  iterator (&proc)
    puts  "entering iterator"
    proc.call
    puts  "exiting test method"
  end

  def test
    iterator {puts "entering proc";break)
  end
  test
```

Lambda类似于方法,因此,如果把一个break语句单独地放在那里,而不是出现在循环或者迭代方法中是说不通的.下面的语句,没有任何东西可以被break,你可能认为它会失败.
不过,在这种情况下,break的行为与return一样;

```ruby
def test
  puts "entering test method"
  lambda = lambda{puts "entering lambda";break;puts "exiting    lambda"}
  lambda.call
  puts "exiting  test  method"
end
test
```