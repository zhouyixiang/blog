---
layout: post
title:  "Laravel MessageBag与Validation"
date:   2016-09-28 09:30:56 +0800
categories: 开发
tags: [php, laravel, dingo]
---

# Message Bag
## Message Bag数据结构

``` php
[
  'key1' => [
    'message 1',
    'message 2',
  ],
  'key2' => [
    'message 1',
    'message 2',
  ],
]

```
## Message Bag读取接口

format: ":key的消息是:message"

返回一个array，和message bag内部存储结构相同。消息按照format参数格式化。format默认格式是“:message”，也就是不变。
``` php
[
  'key1' => [
    'key1的消息是message 1',
    ...
  ],
  ...
]
```

# Validation
Laravel的验证框架有下面几个构件：

* MessageBag 用于存储验证失败信息
* Rule 验证规则
* Message Rule验证失败的错误消息
* Data 验证的数据


# Validator MessageBag生成过程

``` php
$validator = app(Factory::class)->make([
  'attr1' => 'some value',
  'attr2' => 'some value',
], [
  'rule1' => ':attribute不满足:rule1Replacer',
  'rule2' => ':attribute不满足:rule2Replacer',
], [
  'attr1' => 'rule1|rule2|rule3',
  'attr2' => 'rule1|rule2|rule3',
]);
```

MessageBag的format只需要/支持:key和:message两个占位符，Validator自己根据规则扩展了占位符，在Validator内部自己负责替换。
Validator内部的MessageBag以校验的数据attribute为key。当某个属性的某条验证规则（例如: "min:6"）失败时，会提取出该条验证规则对应的message，然后替换掉message中的占位符（例如:attribute, :min），将替换后的消息存放到MessageBag中。

``` php
[
  'attribute1' => [
    'key1的消息是message 1',
    'key1的消息是message 2',
  ],
  'attribute1' => [
    'key2的消息是message 1',
    'key2的消息是message 2',
  ],
]
```
