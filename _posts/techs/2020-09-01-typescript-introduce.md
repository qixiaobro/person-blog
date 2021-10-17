---
title: TypeScript 简单上手
date: 2020-09-01 15:45:26
permalink: /docs/techs/typescript
key: docs-techs-typescript
categories:
  - 前端
  - 技术文章
tags:
  - TypeScript
---

`TypeScript`是一门强类型语言，是`JavaScript`的超集。同时是一门渐进式的语言，可以不用深入学习即可上手。
<!--more-->

  <!-- <iframe :src="$withBase('/markmap/TypeScript-getting-started.html')" width="100%" height="1000" frameborder="1" scrolling="yes" leftmargin="0" topmargin="0"></iframe> -->

## 原始数据类型

### `string`

```ts
const a: string = '1'
```

### `number`

```ts
const b: number = 1 //同时也可以是NaN Infinity
```

### `boolean`

```ts
const c: boolean = true //false
```

### `void`

```ts
const d: void = undefined //严格模式只能存undefined,非严格模式可以是null
```

### `null`

```ts
const e: null = null
```

### `undefined`

```ts
const f: undefined = undefined
```

### `symbol`

```ts
const g: symbol = Symbol() //仅在target为ES2015以上才行
```

## `module scoped`

在两个不同文件定义了两个重名变量，怎么办。

1. 使用立即执行函数

```ts
;(function() {
  const a = 123
})()
```

2. 使用`export`

```ts
export {}
```

## `Object`类型

`Object`类型包含所有非原始类型！！并不单指对象
{:.info}

- 可以是所有非原始类型

```ts
const foo: Object = function() {}
```

- 单指对象需使用字面量的方式定义

```ts
const bar: {} = {}
```

- 对象里必须有一个 foo 属性且是字符串

```ts
const bar: { foo: string } = { foo: '123' }
```

## 数组类型

```ts
const a: Array<number> = [1, 2, 3]

const b: number[] = [1, 2, 3] //常用方式

const c: Array<object> = []
```

函数指定参数数组类型

```ts
function sum(...args: number[]) {
  return args.reduce((prev, current) => prev + current, 0)
}

sum(1, 2, 3, 'foo') //会报错 “类型"foo"的参数不能赋给类型“number”的参数。”
```

## 元组

元组类型允许表示一个<mark>已知元素数量和类型</mark>的数组，各元素的类型不必相同。

```ts
const tuple: [string, number] = ['str', 1]

//取值
const a = tuple[0]
//解构赋值
const { str, num } = tuple
```

## 枚举

使用枚举可以定义一些带名字的常量。 使用枚举可以清晰地表达意图或创建一组有区别的用例。

会入侵编译后的代码
{:.warning}

```ts
enum PostStatus {
  Draft = 0,
  Unpublished = 1,
  Published = 2,
}
```

- 可以不用等号赋值，默认从 0 开始累加

```ts
enum PostStatus {
  Draft,
  Unpublished,
  Published,
}
```

- 也可以赋第一个值，然后后续会累加，`Unpublished = 7，Published = 8`

```ts
enum PostStatus {
  Draft = 6,
  Unpublished,
  Published,
}
```

- 也可以是字符串，但是就不能累加

```ts
enum PostStatus {
  Draft = 'a',
  Unpublished = 'b',
  Published = 'c',
}
```

### 常量枚举

不会入侵编译后的代码

```ts
const enum PostStatus2 {
  Draft = 0,
  Unpublished = 1,
  Published = 2,
}

const post2 = {
  title: 'hello ts',
  content: 'awesome',
  status: PostStatus2.Draft,
}
```

## 函数类型

- 定义函数

```ts
function func1(a: number, b: number) {
  return 'func1'
}

func1(100, 100)
```

- 可选参数

```ts
function func2(a: number, b?: number) {
  return 'func2'
}

func2(100, 100)
func2(100)
```

- 参数默认值

```ts
function func3(a: number, b: number = 100) {
  return 'func3'
}

func3(100, 100)
func3(100)
```

- 多余参数

```ts
function func4(a: number, b: number = 100, ...rest: number[]) {
  return 'func4'
}

func4(100, 100, 100, 100)
```

### 声明式写法

为常量定义类型

```ts
//定义一个函数，此函数接收两个number类型的参数，并返回一个字符串
const func5: (a: number, b: number) => string = function(a: number, b: number) {
  return 'func5'
}
```

## `any`类型

在编程阶段还不清楚类型的变量指定一个类型

```ts
function stringify(value: any) {
  return JSON.stringify(value)
}

stringify(100)

stringify('str')

stringify(true)

let foo: any = 'a'
foo = 1
```

## 隐式类型推断

在初始化变量和成员，设置默认参数值和决定函数返回值时，不指定类型，会自动推断类型。

```ts
let age = 18

age = 'str' //报错 不能将类型“"str"”分配给类型“number”。

let foo //这种情况下，则自动推断为any类型

foo = 1

foo = 'str'
```

## 断言

你清楚地知道一个实体具有比它现有类型更确切的类型。通过类型断言这种方式可以告诉编译器，“相信我，我知道自己在干什么”。

```ts
const nums: number[] = [1, 2, 3]

const res = nums.find((i) => i > 1) //编译器会认为 res 的类型为 number | undefined
```

1. 通过关键字`as` 断言

```ts
const num1 = res as number
```

2. 通过`<>`断言

   在 JSX 语法里不可使用，跟标签冲突
   {:.warning}

```ts
const num2 = <number>res
```

## 接口

用来制定一个类型的规则

```ts
interface Post {
  title: string
  content: string
  subtitle?: string //可选成员
  readonly summary: string //只读成员
}

function printPost(post: Post) {
  console.log(post.title)
  console.log(post.content)
}

printPost({
  title: 'ass',
  content: 'aaaa',
  summary: 'sss',
})
```

- 动态成员

```ts
interface Cache {
  [prop: string]: string
}

const cache: Cache = {}
cache.foo = '11'
cache.bar = '22'
```

## 类

属性类型：

- `public` 公共（默认）
- `private` 私有的，只能在当前类中访问
- `protected` 受保护的，只能在当前类和其子类中访问
- `readonly` 只读的，不可修改

```ts
class Person {
  public name: string //默认是public，可以不用写
  private age: number //私有属性
  protected readonly gender: boolean //受保护、且只读属性

  constructor(name: string, age: number) {
    this.gender = true
  }

  sayHi(msg: string) {
    console.log(`hi,I am ${this.name},${msg}`)
    console.log(this.age)
    console.log(this.age)
    // this.gender = false //报错 无法分配  因为是只读的
  }
}

class Student extends Person {
  constructor(name: string, age: number) {
    super(name, age)
    // console.log(this.age) //报错 属性“age”为私有属性，只能在类“Person”中访问。
    console.log(this.gender) //受保护属性可被子类访问
  }
}

//类的访问修饰符
const tom = new Person('tom', 16)
console.log(tom.name)
// console.log(tom.age)//报错 只能在Person类里访问
// console.log(tom.gender)//报错 受保护，只能在Person类或者其子类里访问
const jack = new Student()
console.log(jack.age) //报错 只能在Person类里访问
// console.log(jack.gender) //报错 属性“gender”受保护，只能在类“Person”及其子类中访问。
```

构造函数也可以是私有的，需要通过一个静态方法来创建实例

```ts
class Student extends Person {
  private constructor(name: string, age: number) {
    super(name, age)
    console.log(this.gender)
    // console.log(this.age)//私有属性，只能在Person类里访问
  }
  static create(name: string, age: number) {
    return new Student(name, age)
  }
}
// const jack = new Student() //无法新建实例，构造函数是受保护的
const jack = Student.create('jack', 16)
```

## 类和接口的结合使用

```ts
interface Eat {
  eat(food: string): void
}

interface Run {
  run(distance: number): void
}

class Animal implements Eat, Run {
  eat(food: string): void {
    console.log(`呼噜呼噜吃${food}`)
  }

  run(distance: number): void {
    console.log(`四条腿走了${distance}公里`)
  }
}

class Person implements Eat, Run {
  eat(food: string): void {
    console.log(`优雅的吃${food}`)
  }

  run(distance: number): void {
    console.log(`直立行走，走了${distance}公里`)
  }
}
```

## 抽象类
比如说动物类，太多种类，吃的东西 和 走路的方法都不同，这时就可以定一个抽象类来约束`eat`和`run`方法

无法创建实例，只能被继承
{:.warning}

```ts
abstract class Animal {
  eat(food: string): void {
    console.log(`呼噜呼噜吃${food}`)
  }

  abstract run(distance: number): void //抽象方法，需要有abstract关键字且不能有方法体
}

class Dog extends Animal {
  run(distance: number): void {
    console.log(`狗是四条腿走的，走了${distance}`)
  }
}

// const pig = new Animal() 无法创建实例

const dog = new Dog()
dog.eat('骨头')
dog.run(100)
```

## 泛型
在函数定义时因为不知道实际类型，先不指定类型，运用时再定义。
使用`<T>`代表泛型

```ts
function createArray<T> (length: number, value: T): T[]{
  const arr = Array<T>(length).fill(value)
  return arr
}

const stringArr = createArray<string>(3, 'a')
```

## 类型声明
使用`declare`关键字，为一些暂时没有`TypeScript`实现的第三方插件，定义方法类型。

```ts
import {camelCase} from 'lodash'

declare function camelCase (input: string): string

const res = camelCase('hello world')
```

