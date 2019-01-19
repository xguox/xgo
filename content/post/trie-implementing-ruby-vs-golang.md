---
title: "Trie 的实现, Ruby vs Go"
date: 2019-01-03T16:01:23+08:00
draft: false
tags: ["Go", "Ruby"]
categories: ["Go", "Ruby"]
---

[上一篇](https://xguox.me/gin-router-conflicts.html)看到 Trie 的数据结构, 想着用 Ruby 和 Go 大概实现一下对比看看, 顺便看看一下 Benchmark.

(挺没意义的一个事 = 。 =)

普通的 trie 是一个字符一个结点, 压缩 trie 的结点可能是一个字符串, 空间更省一些吧.

[20k.txt](https://github.com/first20hours/google-10000-english/blob/master/20k.txt) 是两万个英文单词, github 地址 [https://github.com/first20hours/google-10000-english/blob/master/20k.txt](https://github.com/first20hours/google-10000-english/blob/master/20k.txt).

#### Trie in Ruby

```ruby
# ruby 2.4
require 'benchmark'

class Node
  attr_reader :char, :children

  def initialize(c)
    @char = c
    @children = []
  end

  def build(c)
    child = find(c)
    if child.nil?
      child = Node.new(c)
      @children << child
    end
    return child
  end

  def find(c)
    @children.each do |child|
      return child if child.char == c
    end
    nil
  end
end

class Trie
  attr_reader :root

  def initialize
    @root = Node.new(nil)
  end

  def build(word)
    node = @root
    word.chars.each do |char|
      child = node.build(char)
      node = child
    end
  end

  def has?(word)
    node = @root
    word.chars.each do |char|
      found = node.find(char)
      return false if found.nil?
      node = found
    end

    return true
  end
end

class Trie2 < Hash
  def initialize
    super
  end

  def build(string)
    string.chars.inject(self) do |h, char|
      h[char] ||= { }
    end
  end

  def has?(string)
    tr = self
    string.chars.each do |char|
      return false if tr[char].nil?
      tr = tr[char]
    end
    return true
  end
end


t = Trie.new
File.readlines('./20k.txt').each do |line|
  t.build(line.gsub('\n', ''))
end

puts Benchmark.measure {
  200_000.times do
    t.has?('42082')
    t.has?('oops')
    t.has?('Supercalifragilisticexpialidocious')
  end
}

t2 = Trie2.new
File.readlines('./20k.txt').each do |line|
t2.build(line.gsub('\n', ''))
end

puts Benchmark.measure {
  200_000.times do
    t2.has?('42082')
    t2.has?('oops')
    t2.has?('Supercalifragilisticexpialidocious')
  end
}

# Benchmark
# 2.950000   0.010000   2.960000 (  2.978725)
# 1.320000   0.010000   1.330000 (  1.325868)
```

#### Trie in Golang

```go
// go 1.11
package main

import (
    "bufio"
    "log"
    "os"
    "testing"
)

type Node struct {
    Char     rune
    Children []*Node
}

func NewNode(r rune) *Node {
    return &Node{Char: r}
}

func (n *Node) insert(r rune) *Node {
    child := n.get(r)
    if child == nil {
        child = NewNode(r)
        n.Children = append(n.Children, child)
    }

    return child
}

func (n *Node) get(r rune) *Node {
    for _, child := range n.Children {
        if child.Char == r {
            return child
        }
    }
    return nil
}

type Trie struct {
    Root *Node
}

func NewTrie() *Trie {
    var r rune
    trie := Trie{Root: NewNode(r)}
    return &trie
}

func (tr *Trie) Build(word string) {
    node := tr.Root
    runeArr := []rune(word)
    for _, char := range runeArr {
        child := node.insert(char)
        node = child
    }
}

func (tr *Trie) Has(word string) bool {
    node := tr.Root
    runeArr := []rune(word)
    for _, char := range runeArr {
        found := node.get(char)
        if found == nil {
            return false
        }
        node = found
    }
    return true
}

func BenchmarkTrieFind(b *testing.B) {
    var trie1 = NewTrie()
    file, err := os.Open("./20k.txt")
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()

    scanner := bufio.NewScanner(file)

    for scanner.Scan() {
        trie1.Build(scanner.Text())
    }

    b.ResetTimer()

    for i := 0; i < b.N; i++ {
        trie1.Has("42082")
        trie1.Has("oops")
        trie1.Has("Supercalifragilisticexpialidocious")
    }
}

// goos: darwin
// goarch: amd64
// BenchmarkTrieFind-8   	 5000000	       255 ns/op	     144 B/op	       1 allocs/op
// PASS
// ok  	_/Users/xguox/Desktop	1.591s
// Success: Benchmarks passed.

```

Ruby 的 Benchmark 是 200_000 次的总和, 单位是 **s**, 换成 go 实现下面的 **ns** 大概 **6629 ns/op**. 