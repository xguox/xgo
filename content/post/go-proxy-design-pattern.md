---
title: "Go 的代理模式(Proxy Pattern)"
date: 2018-09-21T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

相比[之前写的组合模式](https://xguox.me/go-composite-design-pattern.html), 代理模式实现起来并不需要太费劲.

主要特性:

- 隐藏或限制被代理对象
- 易于为被代理的对象提供新的抽象层(拦截, 重定义)

[https://github.com/tmrts/go-patterns/blob/master/structural/proxy.md](https://github.com/tmrts/go-patterns/blob/master/structural/proxy.md)

```go
// To use proxy and to object they must implement same methods
type IObject interface {
    ObjDo(action string)
}

// Object represents real objects which proxy will delegate data
type Object struct {
    action string
}

// ObjDo implements IObject interface and handel's all logic
func (obj *Object) ObjDo(action string) {
    // Action behavior
    fmt.Printf("I can, %s", action)
}

// ProxyObject represents proxy object with intercepts actions
type ProxyObject struct {
    object *Object
}

// ObjDo are implemented IObject and intercept action before send in real Object
func (p *ProxyObject) ObjDo(action string) {
    if p.object == nil {
        p.object = new(Object)
    }
    if action == "Run" {
        p.object.ObjDo(action) // Prints: I can, Run
    }
}
```

[https://github.com/bvwells/go-patterns/blob/master/structural/proxy.go](https://github.com/bvwells/go-patterns/blob/master/structural/proxy.go)

```go
package structural

import (
	"fmt"
	"io"
	"os"
)

var outputWriter io.Writer = os.Stdout // modified during testing

// ITask is an interface for performing tasks.
type ITask interface {
	Execute(taskType string)
}

// Task implements the ITask interface for performing tasks.
type Task struct {
	taskName string
}

// Execute implements the task.
func (t *Task) Execute(taskType string) {
	fmt.Fprint(outputWriter, "Performing task type: "+taskType)
}

// ProxyTask represents a proxy task with re-routes tasks.
type ProxyTask struct {
	task *Task
}

// NewProxyTask creates a new instance of a ProxyTask.
func NewProxyTask() *ProxyTask {
	return &ProxyTask{task: &Task{}}
}

// Execute intercepts the Execute command and re-routes it to the Task Execute command.
func (t *ProxyTask) Execute(taskType string) {
	if taskType == "Run" {
		t.task.Execute(taskType)
	}
}
```

[https://github.com/monochromegane/go_design_pattern/blob/master/proxy/proxy.go](https://github.com/monochromegane/go_design_pattern/blob/master/proxy/proxy.go)

```go
package proxy

type printable interface {
	SetPrinterName(name string)
	GetPrinterName() string
	Print(str string) string
}

type printer struct {
	name string
}

func (self *printer) SetPrinterName(name string) {
	self.name = name
}

func (self *printer) GetPrinterName() string {
	return self.name
}

func (self *printer) Print(str string) string {
	return self.name + ":" + str
}

type PrinterProxy struct {
	Name string
	real *printer
}

func (self *PrinterProxy) SetPrinterName(name string) {
	if self.real != nil {
		self.real.SetPrinterName(name)
	}
	self.Name = name
}

func (self *PrinterProxy) GetPrinterName() string {
	return self.Name
}

func (self *PrinterProxy) Print(str string) string {
	self.realize()
	return self.real.Print(str)
}

func (self *PrinterProxy) realize() {
	if self.real == nil {
		self.real = &printer{self.Name}
	}
}

```