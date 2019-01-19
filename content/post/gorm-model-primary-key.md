---
title: "Gorm 笔记"
date: 2018-10-31T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

有一些 model 结构没有嵌套 `gorm.Model`, 只是自己自定义了 `CreatedAt` `UpdatedAt`, 然后插入数据的时候都会插两次, 原因是漏了 `ID`, 准确的说是漏了 **primary_key** = . =


