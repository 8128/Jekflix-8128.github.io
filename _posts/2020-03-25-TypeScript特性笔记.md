---
layout: post
title:  TypeScript特性笔记
date:   2020-03-25 21:35:00
description: TypeScript特性笔记总结
subtitle:
comments: true
image: https://raw.githubusercontent.com/8128/PicGo/master/20200326151932.png
optimized_image:
category: code
tags:
  - code
  - javascript
author: tty
paginate: false

---

谷歌对特殊符号的不支持真是让我无语！很多查询都变得无比艰难！总结一下，我可不想再查一次了

提前说明这不是tutorial，我也没加tutorial标签。这里只有我想记一下的内容

关于[TypeScript](https://ts.xcatliu.com/)

## Ts属性后的感叹号

来源[七月时光](https://www.leevii.com/2018/10/what-does-the-exclamation-point-behind-the-ts-attribute-mean.html)

首先看一段代码

```js
import { Vue, Component, Prop } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @Prop(Number) propA!: number
  @Prop({ default: 'default value' }) propB!: string
  @Prop([String, Boolean]) propC: string | boolean
}
```

这是vue组件的ts写法，仔细观察，里面有一段这样的代码`@Prop(Number) propA!: number`，这是什么鬼？

查了一下ts的文档说明，原来感叹号是非`null`和非`undefined`的类型断言，所以上面的写法就是对`propA`这个属性进行非空断言。文档的相关说明在[这里](https://github.com/Microsoft/TypeScript/wiki/What's-new-in-TypeScript#non-null-assertion-operator)。

官方文档上的一个例子很好的说明了这个问题

```js
interface Entity {
  name: string
}

// Compiled with --strictNullChecks
function validateEntity(e?: Entity) {
  // Throw exception if e is null or invalid entity
}

function processEntity(e?: Entity) {
  validateEntity(e);
  let s = e!.name;  // Assert that e is non-null and access name
}
```

如果直接使用`let s = e.name;`，编译器会抛出`e`可能不存在的错误，但是使用非空断言，则表示`e`肯定是存在的，从而不会产生编译问题。