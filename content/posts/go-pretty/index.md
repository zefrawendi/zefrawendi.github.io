---
title: "Go Pretty"
date: 2022-10-19T15:44:48+08:00
lastmod: 2022-10-20T09:44:48+08:00
draft: false
description: "go pretty库"
categories: ["开发"]
tags: ["go","第三方库"]
slug: "go_pretty"
image: "ops.png"
---

用于美化表格，列表，进度条，文本等的控制台输出

[jedib0t/go-pretty: Table-writer and more in golang! (github.com)](https://github.com/jedib0t/go-pretty)

## table

可以在输出美化的表格

```go
package main  
  
import (  
   "os"  
  
   "github.com/jedib0t/go-pretty/v6/table"
)  
  
type Student struct {  
   ID     int  
   Name   string  
   Age    int  
   School string  
}  
  
var students = []Student{  
   {ID: 1, Name: "张三", Age: 18, School: "清华大学"},  
   {ID: 2, Name: "李四", Age: 19, School: "北京大学"},  
   {ID: 3, Name: "王五", Age: 20, School: "复旦大学"},  
   {ID: 4, Name: "赵六", Age: 21, School: "上海交通大学"},  
}  
  
func (s Student) toRow() table.Row {  
   return table.Row{s.ID, s.Name, s.Age, s.School}  
}  
  
func main() {  
   t := table.NewWriter()  
   t.SetOutputMirror(os.Stdout)  
   t.AppendHeader(table.Row{"ID", "姓名", "年龄", "学校"})
   for _, s := range students {  
      t.AppendRow(s.toRow())  
   }  
   t.SetStyle(table.StyleColoredDark)  
   t.Render()  
}
```
运行可以在终端中得到

![](ops.png)



Render同时也支持渲染html、csv等...
