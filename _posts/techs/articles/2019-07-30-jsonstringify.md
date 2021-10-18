---
title: 你真的懂stringify()吗？
date: 2019-10-26 15:45:26
permalink: /docs/techs/jsonstringfy
key: jsonstringfy
sharing: true
show_author_profile: true
show_subscribe: true
sidebar:
  nav: tech-nav
categories:
  - 前端
  - 技术文章
tags:
  - JS
---
# 你真的懂JSON.stringify()吗？
<!--more-->

`JSON.stringify()`方法能够将一个`javascript`值转换字符串。
![](https://cdn.nlark.com/yuque/0/2020/svg/377922/1593092276992-fb0fe988-670a-498f-a43d-229536d1de95.svg)## 参数

1. `value`

要转换为字符串的值。

2. `replacer`：（可选）

可以为

      - 函数：则在转换过程中，被转换的值的每个属性都会经过该函数的转换和处理。
      - 数组：只有包含在这个数组中的属性名才会被转换到最终的 JSON 字符串。
      - `null`或未提供：对象所有的属性都会被序列化。
3. `space`：（可选）

指定缩进用的空白字符串，用于美化输出。可以为

      - 数字：代表有多少的空格；上限为10。该值若小于1，则意味着没有空格。
      - 字符串：（当字符串长度超过10个字母，取其前10个字母），该字符串将被作为空格。
      - `null`或未提供：没有空格。



## 规则描述

- 转换值如果有 `toJSON()` 方法，该方法定义什么值将被序列化。
```javascript
let obj = {
  foo: 'foo',
  toJSON: function () {
    return 'bar';
  }
};
JSON.stringify(obj);      
// '"bar"'
JSON.stringify({x: obj}); 
// '{"x":"bar"}'
```

- **非数组对象的属性不能保证以特定的顺序出现在序列化后的字符串中**。
- 布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值。
```javascript
JSON.stringify([new Number(1), new String("false"), new Boolean(false)]); 
// '[1,"false",false]'
```

- `undefined`、**任意的函数**以及`symbol`值，在序列化过程中会被忽略（出现在非数组对象的属性值中时）或者被转换成 `null`（出现在数组中时）。
- 函数、`undefined` 被单独转换时，会返回 `undefined`，如`JSON.stringify(function(){})` or `JSON.stringify(undefined)`
```javascript
JSON.stringify({x: undefined, y: Object, z: Symbol("")}); 
// '{}'

//undefined单独转换
JSON.stringify(undefined)
//undefined

//函数被单独转换
JSON.stringify(function(){})
//undefined
```

- 对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。
- 所有以 `symbol` 为属性键的属性都会被完全忽略掉，即便 `replacer` 参数中强制指定包含了它们。
```javascript
JSON.stringify({[Symbol("foo")]: "foo"});                 
// '{}'
```

- `Date`日期调用了`toJSON()`将其转换为了`string` 字符串（同`Date.toISOString(`)），因此会被当做字符串处理。
- `NaN`和`Infinity`格式的数值及`null`都会被当做`null`。
```javascript
JSON.stringify({ a: NaN, b : Infinity,c:null })
//"{"a":null,"b":null,"c":null}"
```

- 其他类型的对象，包括 `Map/Set/WeakMap/WeakSet`，仅会序列化可枚举的属性。
```javascript
JSON.stringify( 
    Object.create(
        null, 
        { 
            x: { value: 'x', enumerable: false }, 
            y: { value: 'y', enumerable: true } 
        }
    )
);
// "{"y":"y"}"
```


## `replacer`参数
`replacer`参数可以是一个函数或者一个数组。
### 函数
接受两个参数，`key`和`value`。
在开始时, `replacer` 函数会被传入一个**空字符串**作为 `key` 值，代表着要被 `stringify` 的这个对象。随后每个对象或数组上的属性会被依次传入。
函数应当返回`JSON`字符串中的`value`, 如下所示:

- 如果返回一个 [`Number`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number), 转换成相应的字符串作为属性值被添加入 `JSON` 字符串。
- 如果返回一个 [`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/String), 该字符串作为属性值被添加入 `JSON`字符串。
- 如果返回一个 [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean), "`true`" 或者 "`false`" 作为属性值被添加入 `JSON` 字符串。
- 如果返回任何其他对象，该对象**递归地序列化**成`JSON`字符串，对每个属性调用 `replacer` 方法。除非该对象是一个函数，这种情况将不会被序列化成 `JSON` 字符串。
- 如果返回 `undefined`，该属性值不会在 `JSON` 字符串中输出。



`replacer`函数可以通过条件返回`undefined` 来使不需要转换的值不转换！
{:.info}
```javascript
//不序列化字符串类型的值
function replacer(key, value) {
  if (typeof value === "string") {
    return undefined;
  }
  return value;
}
let foo = {foundation: "Mozilla", model: "box", week: 45, transport: "car", month: 7};
let jsonString = JSON.stringify(foo, replacer);//{"week":45,"month":7} 
```


### 数组
如果 `replacer` 是一个数组，数组的值代表将被序列化成 `JSON` 字符串的属性名。
```javascript
let foo = {foundation: "Mozilla", model: "box", week: 45, transport: "car", month: 7};
JSON.stringify(foo, ['week', 'month']);  
// '{"week":45,"month":7}', 只保留 “week” 和 “month” 属性值。
```




## `space`参数
用来控制结果字符串里面的间距。可以是数字也可以字符串。

- 数字：在字符串化时每一级别会比上一级别缩进多这个数字值的空格（最多10个空格）
```javascript
JSON.stringify({a:2},null,2)
//"{
//  "a": 2
//}"

JSON.stringify({a:2},null,11)
//"{
//          "a": 2
//}"
```

- 字符串：每一级别会比上一级别多缩进该字符串（或该字符串的前10个字符）
```javascript
JSON.stringify({a:2},null,'as')
//"{
//as"a": 2
//}"

JSON.stringify({a:2},null,'12345678901')
//"{
//1234567890"a": 2
//}"
```




`JSON`不是`JavaScript`严格意义上的子集
{:.success}
