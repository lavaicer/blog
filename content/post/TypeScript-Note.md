---
title: "TypeScript Note"
date: 2022-06-01T11:27:08+08:00
draft: true
weight: 9
categories: ["Language"]
tags: ["typescript", "language"]
author: ["lavaicer"]
# author: ["Me", "You"] # multiple authors
---

## 简介

TypeScript 是带有类型语法的 JavaScript

## 安装

- node
- npm install -g typescript

## 变量声明

- let
- const
- `${}` 直接使用变量

## 数据类型

- number
- boolean
- string
- any
- undefined
- literal
- 类型的并集

### 枚举类型

- enum

## 逻辑控制

- if/else
- switch
- for
- while
- try{throw}/catch

## 数组

- push | pop
- shift | unshift
- slice
- splice
- indexOf
- sort
- split | join
- 作为元组展开

## 对象类型

- 定义
- 嵌套
- JSON

## 函数

- 可选参数 s?:string
- 默认值
- 可变参数列表
- 重载（不建议使用）
- 对象类型参数
- 给对象增加方法

## 函数式编程

> - 变量类型可以是函数
> - 值可以是函数
> - 函数的参数可以是函数
> - 函数的返回值可以是函数
> - 对象的字段可以是函数

箭头函数
(a:number, b:number) => a-b

- 闭包

## 