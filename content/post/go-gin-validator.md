---
title: "Gin 模型验证 Validator"
date: 2018-09-03T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

Gin 默认用就是 [go-playground/validator](https://github.com/go-playground/validator) 这个库, 通过 tag 可以设置结构体字段的校验规则, `go-playground/validator` [自带了差不多一百种](https://github.com/go-playground/validator/blob/v9/baked_in.go)吧, 比如必须有值(required), 验证长度(len), 有效邮箱(email), 如果这些都还不能满足的话还能够根据需要自定义.

```go
type Category struct {
    Name string `form:"name" json:"name" binding:"required"`
    Slug string `form:"slug" json:"slug" binding:"required"`
}
```

这里 `binding` 是 v8 版本的写法, 也就是 gin 当前(2018.9)引用的 validator 版本, 但是, **go-playground/validator** 早就更新到了 v9 了, 并且 `binding` 换成了 `validate`, 还有其他一些用法因为变动比较大, 虽然老早有人提了 [PR](https://github.com/gin-gonic/gin/pull/1015) 但是目前貌似还合不了. **go-playground/validator** 索性自己给了一个[升级方案](https://github.com/go-playground/validator/tree/v9/_examples/gin-upgrading-overriding)出来.

v8 的自定义规则类似长这样的

```go
func bookableDate(
    v *validator.Validate, topStruct reflect.Value, currentStructOrField reflect.Value,
    field reflect.Value, fieldType reflect.Type, fieldKind reflect.Kind, param string,
) bool {
    if date, ok := field.Interface().(time.Time); ok {
        today := time.Now()
        if today.Year() > date.Year() || today.YearDay() > date.YearDay() {
            return false
        }
    }
    return true
}

```

一堆反射的参数看着头都晕了, 换成了 v9 以后简洁多了, 反射什么的按需自取就是了, 结合 **gorm** 写的一个判断数据库字段唯一的自定义规则

```go
func ValidateUniq(fl validator.FieldLevel) bool {
    var result struct{ Count int }
    currentField, _, _ := fl.GetStructFieldOK()
    table := modelTableNameMap[currentField.Type().Name()] // table name
    value := fl.Field().String()                           // value
    column := fl.FieldName()                               // column name
    sql := fmt.Sprintf("select count(*) from %s where %s='%s'", table, column, value)
    db.PG.Raw(sql).Scan(&result)
    dup := result.Count > 0
    return !dup
}
```


-----------------


###### 一个简单的示例

```go
package model

import (
    "coconut/db"
    "fmt"

    validator "gopkg.in/go-playground/validator.v9"

    "github.com/gin-gonic/gin/binding"

    "github.com/gin-gonic/gin"
    "github.com/jinzhu/gorm"
)

type Category struct {
    gorm.Model
    Name string `form:"name" json:"name" binding:"required,is-uniq"`
    Slug string `form:"slug" json:"slug" binding:"required"`
}

// CATEGORY VALIDATOR
type CategoryValidator struct {
    CategoryModel Category `json:"category"`
}

func (s *CategoryValidator) Bind(c *gin.Context) error {
    b := binding.Default(c.Request.Method, c.ContentType())
    err := c.ShouldBindWith(s, b)
    if err != nil {
        return err
    }
    return nil
}

func ValidateUniq(fl validator.FieldLevel) bool {
    var result struct{ Count int }
    currentField, _, _ := fl.GetStructFieldOK()
    table := modelTableNameMap[currentField.Type().Name()] // table name
    value := fl.Field().String()                           // value
    column := fl.FieldName()                               // column name
    sql := fmt.Sprintf("select count(*) from %s where %s='%s'", table, column, value)
    db.PG.Raw(sql).Scan(&result)
    dup := result.Count > 0
    return !dup
}
```

方法 `Bind` 返回的 error 是结构体下所有违规的字段错误, 所以可以这么处理(**v9**)

```go
type CommonError struct {
    Errors map[string]interface{} `json:"errors"`
}

func NewValidatorError(err error) CommonError {
    res := CommonError{}
    res.Errors = make(map[string]interface{})
    errs := err.(validator.ValidationErrors)

    for _, e := range errs {
        res.Errors[e.Field()] = e.ActualTag()
    }
    return res
}
```