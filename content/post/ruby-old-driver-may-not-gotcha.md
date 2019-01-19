---
title: "Ruby 老司机也未必 Gotcha "
date: 2016-10-22T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

[原 Google Docs](https://docs.google.com/presentation/d/1cqdp89_kolr4q1YAQaB-6i5GXip8MHyve8MvQ_1r6_s/present?slide=id.p)

```ruby
2.3.1 056 > str = "foo"
 => "foo"
str.upcase
 => "FOO"
str
 => "foo"
str.upcase!
 => "FOO"
str
 => "FOO"
str.upcase! # 这返回值惊呆了
 => nil
str
 => "FOO"
```

##### Constant

```ruby
FOO = 5
 => 5
2.3.1 002 > FOO = 7
(irb):2: warning: already initialized constant FOO
(irb):1: warning: previous definition of FOO was here
 => 7
FOO
 => 7
```

即使 **freeze** 了

```ruby
BAR = 7
 => 7
BAR.freeze
 => 7
BAR = 9
(irb):6: warning: already initialized constant BAR
(irb):4: warning: previous definition of BAR was here
 => 9
BAR
 => 9
```

##### Comparator

```ruby
1 == 1.0
 => true
1.eql? 1.0
 => false
a = "foo"
 => "foo"
b = "foo"
 => "foo"
a == b
 => true
a.eql? b
 => true
a.equal? b
 => false
a.equal? a
 => true
```

**== 比较的是值, eql? 比较的是值和类, equal? 比较的是对象**

```ruby
1 === 1
 => true
Fixnum === 1
 => true
1 === Fixnum
 => false
Class === Class
 => true
Object === Object
 => true
Class === Object
 => true
Object === Class
 => true
Fixnum === Fixnum
 => false
(1..3) === 2
 => true
2 === (1..3)
 => false
```

```ruby
 x = true && false
 => false
x
 => false
x = true and false
 => false
x
 => true
```

##### Whitespace

```ruby
def method; 42; end
 => :method
num = 21
 => 21
method/num
 => 2
method / num
 => 2
method/ num
 => 2
method /num
# SyntaxError:
# unterminated regexp
```

```ruby
def one; 1; end
 => :one
one - 1
 => 0
one-1
 => 0
one- 1
 => 0
one -1
ArgumentError: wrong number of arguments (given 1, expected 0)
  from (irb):13:in `one'
  from (irb):17
  from /Users/xguox/.rvm/rubies/ruby-2.3.1/bin/irb:11:in `<main>'

```

---------

##### Class Variable

```ruby
class Parent
  @@value = 6
  def self.value
    @@value
  end

  def self.inc_value
    @@value += 1
  end
end
 => :inc_value
class Child < Parent
  @@value = 42
end
 => 42
Parent.value  # 呵呵哒
 => 42
Parent.inc_value
 => 43
Child.value
 => 43
```

```ruby
3.times do |loop_num|
    sum ||= 0
  sum += 1
  puts sum
end
1
1
1
 => 3
for loop_num in 1..3
  sum ||= 0
  sum += 1
  puts sum
end
1
2
3
 => 1..3
```

##### Freeze Array

```ruby
arr = ["one", "two", "three"]
 => ["one", "two", "three"]
arr.freeze
 => ["one", "two", "three"]
arr << "four"
RuntimeError: can't modify frozen Array
  from (irb):43
  from /Users/xguox/.rvm/rubies/ruby-2.3.1/bin/irb:11:in `<main>'
arr[0] = "eno"
RuntimeError: can't modify frozen Array
  from (irb):44
  from /Users/xguox/.rvm/rubies/ruby-2.3.1/bin/irb:11:in `<main>'
arr[0].object_id
 => 70307108991340
arr[0].reverse!
 => "eno"
arr
 => ["eno", "two", "three"]
arr
 => ["eno", "two", "three"]
arr[0].object_id
 => 70307108991340
```

```ruby
arr = [1, 2, 3, 4]
 => [1, 2, 3, 4]
arr.freeze
 => [1, 2, 3, 4]
arr << 5
RuntimeError: can't modify frozen Array
  from (irb):52
  from /Users/xguox/.rvm/rubies/ruby-2.3.1/bin/irb:11:in `<main>'
arr[0] += 2
RuntimeError: can't modify frozen Array
  from (irb):53
  from /Users/xguox/.rvm/rubies/ruby-2.3.1/bin/irb:11:in `<main>'
1.object_id
 => 3
3.object_id
 => 7
```

