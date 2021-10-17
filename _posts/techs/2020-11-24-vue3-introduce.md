---
title: Vue3简单上手
date: 2020-11-24 16:08:22
permalink: /docs/techs/vue3
key: docs-techs-vue3
articles:
  excerpt_type: html
categories:
  - 前端
  - 技术文章
tags:
  - Vue.js
---

2020 年 09 月 18 日，终于。。`Vue.js 3.0`正式版发布了。
扶我起来，我还能学！
<!--more-->
这里展示学习`Vue.js 3.0`新增特性和变更的特性的所有 demo

## 响应性

`Vue.js 3.0`的数据监听使用`ES6`新特性`Proxy`重写了。使得对对象、数组等数据结构的操作监听更加强大。
其中`Vue.js 3.0`新增了许多响应性 API，这里展示几个常用的。

### `reactive`

为 `JavaScript` 对象创建响应式状态

```js
import { reactive } from 'vue'

const state = reactive({
  count: 0,
})
```


响应式转换是“深层”的——它影响所有嵌套 `property`。在基于 `ES2015 Proxy` 的实现中，返回的 `proxy` 是不等于原始对象的。建议只使用响应式 `proxy`，避免依赖原始对象。
{:.warning}
即

```js
const state = { count: 0 }
const state2 = reactive(state)
console.log(state === state2) //false
```

### `ref`

让一个原始值变成响应式

```js
import { ref } from 'vue'

const count = ref(0)
```

使用`ref`API,会返回一个可变的响应式对象，该对象作为它的内部值——一个响应式的引用。在`setup()`API 里需使用`.value`来访问,在实例创建完后，则可以直接使用
`this.count`访问

```js
//setup()
console.log(count.value) //0

//other
console.log(this.count) //0
```

#### `ref`展开

当 `ref` 作为渲染上下文 (从 `setup()` 中返回的对象) 上的 `property` 返回并可以在模板中被访问时，它将自动展开为内部值。不需要在模板中追加 `.value`：

```html
<span>count:{{count}}</span>
<!--模版中会自动展开，不需要.value-->
```

#### 访问响应式对象

```js
const count = ref(0)
const state = reactive({
  count,
})

console.log(state.count) // 0

state.count = 1
console.log(count.value) // 1
```

如果将新的 ref 赋值给现有 ref 的 property，将会替换旧的 ref：

```js
const otherCount = ref(2)

state.count = otherCount
console.log(state.count) // 2
console.log(count.value) // 1
```

[在 codeOpen 里尝试上述 demo](https://codepen.io/qixiaobro/pen/WNGeXXE){:.button.button--secondary.button--pill}

`ref` 展开仅发生在被响应式 `Object` 嵌套的时候。当从 `Array` 或原生集合类型如 `Map`访问 `ref` 时，不会进行展开,需要使用`.value`访问
{:.warning}

```js
const books = reactive([ref('Vue 3 Guide')])
// 这里需要 .value
console.log(books[0].value)

const map = reactive(new Map([['count', ref(0)]]))
// 这里需要 .value
console.log(map.get('count').value)
```

### 响应式状态解构

响应式对象如果解构，会导致响应性丢失。
{:.warning}

```js
const book = reactive({
  author: 'Vue Team',
  year: '2020',
  title: 'Vue 3 Guide',
  description: 'You are reading this book right now ;)',
  price: 'free',
})
let { author } = book
book.author = 'zxd'
console.log(book.author) //zxd
console.log(author) //"Vue Team"
```

#### 解决办法

##### `toRef`

为单个属性保持响应式连接。

```js
const year = toRef(book, 'year')
book.year = '2021'
console.log(year.value) //2021  需使用.value方式访问
console.log(book.year) //2021

//当您要将 prop 的 ref 传递给复合函数时，toRef 很有用：
export default {
  setup(props) {
    useSomeFeature(toRef(props, 'title'))
  },
}
```

##### `toRefs`

为一组属性保持响应式连接

```js
let { title, price } = toRefs(book)

book.title = 'toRefs'
book.price = 'noFree'

console.log(title.value) //"toRefs"
console.log(price.value) //"noFree"
console.log(book.title) //"toRefs"
console.log(book.price) //"noFree"
```

### readonly

防止更改响应式对象

```js
const original = reactive({ count: 0 })

const copy = readonly(original)

original.count++

copy.count++ // 警告 "Set operation on key 'count' failed: target is readonly."
```

[在 codeOpen 里尝试上述 demo](https://codepen.io/qixiaobro/pen/mdrbzdK){:.button.button--secondary.button--pill}

### watchEffect

立即执行传入的一个函数，同时响应式追踪其依赖，并在其依赖变更时重新运行该函数。

```js
const count = ref(0)

const stop = watchEffect(() => console.log(count.value)) //0

setTimeout(() => {
  count.value++
}, 100) //1
```

#### 停止侦听

```js
setTimeout(() => {
  stop()
  count.value++
}, 200) //无输出
```

#### 清除副作用

有时候我们会通过监听变量，来发起请求，获取数据。但是有可能在请求返回数据之前，变量又发生了改变或者停止了侦听。那么上一次返回的数据就是脏数据，会导致界面展示错误等问题。通过`watchEffect`就可以减少调用

我们可以通过传入`onInvalidate` 函数作入参，用来注册清理失效时的回调。
这个回调触发时机：

- 副作用即将重新执行时
- 侦听器被停止 (如果在 `setup()` 或生命周期钩子函数中使用了 `watchEffect`，则在组件卸载时)

```js
watchEffect(onInvalidate => {
  const token = performAsyncOperation(id.value)
  onInvalidate(() => {
    // id has changed or watcher is stopped.
    // invalidate previously pending async operation
    token.cancel() //取消api的调用
  })
})

//通常是异步的
const data = ref(null)
watchEffect(async onInvalidate => {
  onInvalidate(() => {...}) // 我们在Promise解析之前注册清除函数
  data.value = await fetchData(props.id)
})
```

#### 副作用刷新时机

当一个用户定义的副作用函数进入队列时，默认情况下，会在所有的组件 `update` 前执行

如果需要在组件更新后重新运行侦听器副作用，我们可以传递带有 flush 选项的附加 `options` 对象 (默认为 `'pre'`)：

```js
// 在组件更新后触发，这样你就可以访问更新的 DOM。
// 注意：这也将推迟副作用的初始运行，直到组件的首次渲染完成。
watchEffect(
  () => {
    /* ... */
  },
  {
    flush: 'post',
  }
)
```

#### 侦听器调试

`onTrack` 和 `onTrigger` 选项可用于调试侦听器的行为。

`onTrack` 将在响应式 `property` 或 `ref` 作为依赖项被追踪时被调用。
`onTrigger` 将在依赖项变更导致副作用被触发时被调用。
这两个回调都将接收到一个包含有关所依赖项信息的调试器事件。建议在以下回调中编写 `debugger` 语句来检查依赖关系

```js
watchEffect(
  () => {
    /* 副作用 */
  },
  {
    onTrigger(e) {
      debugger
    },
  }
)
```

### watch

`watch`跟 vue2 的用法差不多，需要指定特定的侦听数据源。
与 `watchEffect` 比较，`watch` 允许我们：

- 执行副作用；
- 更具体地说明什么状态应该触发侦听器重新运行；
- 访问侦听状态变化前后的值。

#### 侦听单个数据源

```js
// 侦听一个 getter,直接侦听一个对象的属性值
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)

// 直接侦听ref
const count = ref(0)
watch(count, (count, prevCount) => {
  /* ... */
})
```

#### 侦听多个数据源

```js
watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
  /* ... */
})
```


[在 codeOpen 里尝试上述 demo](https://codepen.io/qixiaobro/pen/MWjWWyv){:.button.button--secondary.button--pill}

## 组合式`API`

在之前我们用`Vue.js`写项目有一个噩梦，就是当一个页面功能逻辑繁多的时候，代码长度会非常长。导致维护起来经常需要上下滚动，查找变量、方法。让人头疼。`Vue3`的组合式`API`就是为了更好的编写页面代码，解决这个问题。那么怎么使用呢？`Vue3`提供了一个`setup()`API 做为入口。

### `setup()`

`setup()`在创建组件实例前被执行，因此在`setup()`中没有`this`，相应的也就不能够访问组件中声明的任何属性——本地状态、计算属性或方法。在`setup()`中返回的属性能够在组件的本地状态、计算属性或方法中都能够访问到。
{:.warning}

### 参数

`setup()`接收两个参数,`setup(props, context)`

### `props`

`setup()`函数中的 `props` 是响应式的，当传入新的 `prop` 时，它将被更新。

props 是响应式的，你不能使用 ES6 解构，因为它会消除 prop 的响应性。可以使用`toRefs`进行解构
{:.warning}

### `context`

`context`是非响应式的，可以使用 ES6 解构
`attrs`、`slot`、`emit`
`attrs` 和 `slots` 是有状态的对象，它们总是会随组件本身的更新而更新。这意味着你应该避免对它们进行解构，并始终以 `attrs.x` 或 `slots.x` 的方式引用 `property`。请注意，与 `props` 不同，`attrs` 和 `slots` 是非响应式的。如果你打算根据 `attrs` 或 `slots` 更改应用副作用，那么应该在 `onUpdated` 生命周期钩子中执行此操作。

```js
  props: {
    title: String,
    author: String
  },
  setup(props, ctx) {
    const { title, author } = toRefs(props);

    const { attrs, slots, emit } = ctx;

    // Attribute (非响应式对象)
    console.log(context.attrs)

    // 插槽 (非响应式对象)
    console.log(context.slots)

    // 触发事件 (方法)
    console.log(context.emit)
  }
```

### 返回渲染函数

`setup` 还可以返回一个渲染函数，该函数可以直接使用在同一作用域中声明的响应式状态

```js
import { h, ref, reactive } from 'vue'

export default {
  setup() {
    const readersNumber = ref(0)
    const book = reactive({ title: 'Vue 3 Guide' })
    // Please note that we need to explicitly expose ref value here
    return () => h('div', [readersNumber.value, book.title])
  },
}
```

### 生命周期钩子

`setup` 是围绕 `beforeCreate` 和 `created` 生命周期钩子运行的,所以不需要这两个钩子了。
在`setup`使用生命周期钩子

```js
import {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
  onActivated,
  onDeactivated,
  onErrorCaptured,
} from 'vue'

export default {
  setup() {
    onBeforeMount(() => {
      console.log('Before Mount!')
    })
    onMounted(() => {
      console.log('Mounted!')
    })
    onBeforeUpdate(() => {
      console.log('Before Update!')
    })
    onUpdated(() => {
      console.log('Updated!')
    })
    onBeforeUnmount(() => {
      console.log('Before Unmount!')
    })
    onUnmounted(() => {
      console.log('Unmounted!')
    })
    onActivated(() => {
      console.log('Activated!')
    })
    onDeactivated(() => {
      console.log('Deactivated!')
    })
    onErrorCaptured(() => {
      console.log('Error Captured!')
    })
  },
}
```

### 提供/注入

在`setup()`中使用`provide/inject`。

```html
<!-- src/components/MyMap.vue -->
<template>
  <MyMarker />
</template>

<script>
  import { provide, reactive, readonly, ref } from 'vue'
  import MyMarker from './MyMarker.vue

  export default {
    components: {
      MyMarker
    },
    setup() {
      const location = ref('North Pole')
      const geolocation = reactive({
        longitude: 90,
        latitude: 135
      })

      const updateLocation = () => {
        location.value = 'South Pole'
      }

      provide('location', readonly(location)) //防止被子组件更改
      provide('geolocation', readonly(geolocation))
      provide('updateLocation', updateLocation) //提供修改父组件数据的方法
    }
  }
</script>
```

```html
<!-- src/components/MyMarker.vue -->
<script>
  import { inject } from 'vue'

  export default {
    setup() {
      const userLocation = inject('location', 'The Universe')
      const userGeolocation = inject('geolocation')
      const updateUserLocation = inject('updateLocation')

      return {
        userLocation,
        userGeolocation,
        updateUserLocation,
      }
    },
  }
</script>
```

### 模板引用

```js
<template>
  <div ref="root">This is a root element</div>
</template>

<script>
  import { ref, onMounted } from 'vue'

  export default {
    setup() {
      const root = ref(null)

      onMounted(() => {
        // DOM元素将在初始渲染后分配给ref
        console.log(root.value) // <div>这是根元素</div>
      })

      return {
        root
      }
    }
  }
</script>
```

#### `v-for`中的用法

组合式 `API` 模板引用在 `v-for` 内部使用时没有特殊处理。相反，请使用函数引用执行自定义处理：

```html
<template>
  <div v-for="(item, i) in list" :ref="el => { if (el) divs[i] = el }">
    {{ item }}
  </div>
</template>

<script>
  import { ref, reactive, onBeforeUpdate } from 'vue'

  export default {
    setup() {
      const list = reactive([1, 2, 3])
      const divs = ref([])

      // 确保在每次更新之前重置ref
      onBeforeUpdate(() => {
        divs.value = []
      })

      return {
        list,
        divs,
      }
    },
  }
</script>
```

::: tip
看起来好像跟选项式 API 没有什么区别，还是一直往`setup`里添加代码。没错是这样的，但是`setup`可以从外部引入变量和方法。
我们可以将负责不同功能的代码统一编写，在引入到`setup`里。
:::

比如下面是两个负责不能功能的文件

```js
// src/composables/useUserRepositories.js

import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted, watch } from 'vue'

export default function useUserRepositories(user) {
  const repositories = ref([])
  const getUserRepositories = async () => {
    repositories.value = await fetchUserRepositories(user.value)
  }

  onMounted(getUserRepositories)
  watch(user, getUserRepositories)

  return {
    repositories,
    getUserRepositories,
  }
}
```

```js
// src/composables/useRepositoryNameSearch.js

import { ref, computed } from 'vue'

export default function useRepositoryNameSearch(repositories) {
  const searchQuery = ref('')
  const repositoriesMatchingSearchQuery = computed(() => {
    return repositories.value.filter((repository) => {
      return repository.name.includes(searchQuery.value)
    })
  })

  return {
    searchQuery,
    repositoriesMatchingSearchQuery,
  }
}
```

引入到`setup`里使用

```js
// src/components/UserRepositories.vue
import useUserRepositories from '@/composables/useUserRepositories'
import useRepositoryNameSearch from '@/composables/useRepositoryNameSearch'
import { toRefs } from 'vue'

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: { type: String }
  },
  setup (props) {
    const { user } = toRefs(props)

    const { repositories, getUserRepositories } = useUserRepositories(user)

    const {
      searchQuery,
      repositoriesMatchingSearchQuery
    } = useRepositoryNameSearch(repositories)

    return {
      repositories: repositoriesMatchingSearchQuery,
      getUserRepositories,
      searchQuery,
    }
  },
  data () {
    return {
      filters: { ... }, // 3
    }
  },
  computed: {
    filteredRepositories () { ... }, // 3
  },
  methods: {
    updateFilters () { ... }, // 3
  }
}
```

或者也还是能够在一个文件里编写代码，但是通过定义不同的方法，使负责同一块功能的代码写在一起

```js
import { toRefs,ref, onMounted, watch,computed } from 'vue'
import { fetchUserRepositories } from '@/api/repositories'


function useUserRepositories(user) {
  const repositories = ref([])
  const getUserRepositories = async () => {
    repositories.value = await fetchUserRepositories(user.value)
  }

  onMounted(getUserRepositories)
  watch(user, getUserRepositories)

  return {
    repositories,
    getUserRepositories
  }
}


function useRepositoryNameSearch(repositories) {
  const searchQuery = ref('')
  const repositoriesMatchingSearchQuery = computed(() => {
    return repositories.value.filter((repository) => {
      return repository.name.includes(searchQuery.value)
    })
  })

  return {
    searchQuery,
    repositoriesMatchingSearchQuery,
  }
}

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: { type: String }
  },
  setup (props) {
    const { user } = toRefs(props)

    const { repositories, getUserRepositories } = useUserRepositories(user)

    const {
      searchQuery,
      repositoriesMatchingSearchQuery
    } = useRepositoryNameSearch(repositories)

    return {
      repositories: repositoriesMatchingSearchQuery,
      getUserRepositories,
      searchQuery,
    }
  },
  data () {
    return {
      filters: { ... }, // 3
    }
  },
  computed: {
    filteredRepositories () { ... }, // 3
  },
  methods: {
    updateFilters () { ... }, // 3
  }
}
```

## `Teleport`
`Teleport` 提供了一种干净的方法，允许我们控制在 DOM 中哪个父节点下呈现 HTML，而不必求助于全局状态或将其拆分为两个组件。  
将组件挂载在`id="app"`外的指定 Dom 上。  
用法：`<teleport to="target"></teleport>`
```js
//挂载到 id 为 endofbody 的标签上
const app = Vue.createApp({
  template: `
    <h1>Root instance</h1>
    <parent-component />
  `
})

app.component('parent-component', {
  template: `
    <h2>This is a parent component</h2>
    <teleport to="#endofbody">
      <child-component name="John" />
    </teleport>
  `
})

app.component('child-component', {
  props: ['name'],
  template: `
    <div>Hello, {{ name }}</div>
  `
})
```
```js
//挂载到body标签上
app.component('modal-button', {
  template: `
    <button @click="modalOpen = true">
        Open full screen modal! (With teleport!)
    </button>

    <teleport to="body">
      <div v-if="modalOpen" class="modal">
        <div>
          I'm a teleported modal! 
          (My parent is "body")
          <button @click="modalOpen = false">
            Close
          </button>
        </div>
      </div>
    </teleport>
  `,
  data() {
    return { 
      modalOpen: false
    }
  }
})
```

[在 codeOpen 里尝试上述 demo](https://codepen.io/team/Vue/pen/gOPNvjR){:.button.button--secondary.button--pill}
## 触发组件选项

### 事件名

事件名不存在任何自动化的大小写转换。而是触发的事件名需要完全匹配监听这个事件所用的名称。
推荐使用`kebab-case`的事件名

```js
this.$emit('my-event')
```

```html
<my-component @my-event="doSomething"></my-component>
```

### 定义自定义事件

可以在`emits`选项定义已发出的事件

```js
//数组语法
app.component('custom-form', {
  emits: ['in-focus', 'submit'],
})

//对象语法
app.component('custom-form', {
  emits: {
    'in-focus': null,
    'submit': null,
  },
})
```

#### 验证抛出的错误
使用对象语法时，可以给事件添加验证，如果发射的事件不符合限制，则抛出警告.

要添加验证，将为事件分配一个函数，该函数接收传递给 `$emit` 调用的参数，并返回一个<mark>布尔值</mark>以指示事件是否有效。
```js
app.component('custom-form', {
  emits: {
    // 没有验证
    click: null,
    // 验证submit 事件
    submit: ({ email, password }) => {
      if (email && password) {
        return true
      } else {
        console.warn('Invalid submit event payload!')
        return false
      }
    }
  },
  methods: {
    submitForm() {
      this.$emit('submit', { email, password })
    }
  }
})
```

### `v-model`参数
`v-model`有了一些变化
prop：`value` -> `modelValue`;
event：`input` -> `update:modelValue`;
去除了选项中的`model`选项。
现在定义`v-model`的方式：
```js
const app = Vue.createApp({})

app.component('my-component', {
  props: {
    foo: String
  },
  template: `
    <input 
      type="text"
      :value="foo"
      @input="$emit('update:foo', $event.target.value)">
  `
})
```
```html
<my-component v-model:foo="bar"></my-component>
```

### 多个`v-model`绑定
因为去除了`model`选项和更改了`prop`和`event`。使得可以定义多个`v-model`。
```html
<user-name
  v-model:first-name="firstName"
  v-model:last-name="lastName"
></user-name>
```
```js
const app = Vue.createApp({})

app.component('user-name', {
  props: {
    firstName: String,
    lastName: String
  },
  template: `
    <input 
      type="text"
      :value="firstName"
      @input="$emit('update:firstName', $event.target.value)">

    <input
      type="text"
      :value="lastName"
      @input="$emit('update:lastName', $event.target.value)">
  `
})
```

### 处理`v-model`修饰符
在 2.x 中，我们对组件 v-model 上的 `.trim` 等修饰符提供了硬编码支持。但是，如果组件可以支持自定义修饰符，则会更有用。在 3.x 中，添加到组件 `v-model` 的修饰符将通过 `modelModifiers` prop 提供给组件
添加到组件 v-model 的修饰符将通过 `modelModifiers`  prop 提供给组件。在下面的示例中，我们创建了一个组件，其中包含默认为空对象的 ` modelModifiers`  prop。

请注意，当组件的 `created`  生命周期钩子触发时，` modelModifiers`  prop 包含 ` capitalize` ，其值为 true——因为它被设置在 ` v-model`  绑定 ` v-model.capitalize="bar"` 。


示例：
#### 不带参数的`v-model`  
`modelModifiers`

```html
<div id="app">
  <my-component v-model.capitalize="myText"></my-component>
  {{ myText }}
</div>
```
```js
const app = Vue.createApp({
  data() {
    return {
      myText: ''
    }
  }
})

app.component('my-component', {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  methods: {
    emitValue(e) {
      let value = e.target.value
      if (this.modelModifiers.capitalize) {
        value = value.charAt(0).toUpperCase() + value.slice(1)
      }
      this.$emit('update:modelValue', value)
    }
  },
  template: `<input
    type="text"
    :value="modelValue"
    @input="emitValue">`
})

app.mount('#app')
```
#### 带参数的 `v-model`  
`arg + "Modifiers"`

```html
<my-component v-model:foo.capitalize="bar"></my-component>
```
```js
app.component('my-component', {
  props: ['foo', 'fooModifiers'],
  template: `
    <input type="text" 
      :value="foo"
      @input="$emit('update:foo', $event.target.value)">
  `,
  created() {
    console.log(this.fooModifiers) // { capitalize: true }
  }
})
```

## `setup`语法糖
如果`<script></script>`标签里只有`setup()`API。可以使用下面的语法糖缩写
```html
<template>
  <button @click="inc">{{ count }}</button>
</template>

<script setup>
  import { ref } from 'vue'

  export const count = ref(0)
  export const inc = () => count.value++
</script>
```
```html
<script setup="props">
import { computed } from 'vue'

export default {
  props: {
    msg: String,
  },
  inheritAttrs: false,
}

export const computedMsg = computed(() => props.msg + '!!!')
</script>
```
## `style vars` 变量
在单文件组件里，通过变量控制`style`
```html
<template>
  <div class="text">hello</div>
</template>

<script>
export default {
  data() {
    return {
      color: 'red'
    }
  }
}
</script>

<style vars="{ color }">
.text {
  color: var(--color);
}
</style>
```

## `::v-deep ::deep` 深度选择器
`vue3`抛弃了`deep`，改为下面的方式定义
```html
<style scoped>
/* deep selectors 深度选择器*/ 
::v-deep(.foo) {}
/* shorthand  缩写*/
:deep(.foo) {}

/* targeting slot content 模板插槽选择*/
::v-slotted(.foo) {} 
/* shorthand 缩写*/  
:slotted(.foo) {} 

/* one-off global rule 全局一次性选择规则*/
::v-global(.foo) {}
/* shorthand 缩写*/
:global(.foo) {}
</style>
```
