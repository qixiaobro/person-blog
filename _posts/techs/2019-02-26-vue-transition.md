---
title: Vue transition 进入离开&列表过渡
date: 2019-02-26 15:27:01
permalink: /docs/techs/vue-transition
key: vue-transition
categories:
  - 前端
  - 技术文章
tags:
  - Vue.js
---
Vue 中的进入离开&列表过渡学习。<!--more-->
![](https://cdn.nlark.com/yuque/0/2020/svg/377922/1594123660671-25998841-e30e-4b8f-bdf3-defb16e594dd.svg)
# 单元素/组件的过渡
## css过渡
```html
  <div>
    <button @click="show = !show">
      toggle
    </button>

    <transition name="slide-fade">
      <p v-if="show">
        hello
      </p>
    </transition>
  </div>
```
```css
.slide-fade-enter-active{
  transition:all .3s ease;
}
.slide-fade-leave-active{
  transition:all .3s ease
}
.slide-fade-enter{
  transform: translateY(30px);
  opacity: 0;
}
.slide-fade-leave-to{
  transform: translateX(30px);
  opacity:0;
}
```
## css动画
CSS 动画用法同 CSS 过渡，区别是在动画中 `v-enter` 类名在节点插入 DOM 后不会立即删除，而是在 `animationend` 事件触发时删除。
```html
  <div>
    <button @click="show = !show">
      toggle
    </button>

    <transition name="bounce">
      <p v-if="show">
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris
        facilisis enim libero, at lacinia diam fermentum id. Pellentesque
        habitant morbi tristique senectus et netus.
      </p>
    </transition>
  </div>
```
```css
.bounce-enter-active {
  animation: bounce-in .5s
}
.bounce-leave-active{
  animation: bounce-in .5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
```


## 自定义过渡类名
在`<transition></transition>`标签上添加下列属性，自定义不同的过渡类名。当我们使用动画插件时，很方便，可以直接使用动画插件的类名。

- `enter-class`
- `enter-active-class`
- `enter-to-class` (2.1.8+)
- `leave-class`
- `leave-active-class`
- `leave-to-class` (2.1.8+)



## 同时使用过渡和动画
在一些场景中，你需要给同一个元素同时设置两种过渡动效，比如 `animation` 很快的被触发并完成了，而 `transition` 效果还没结束。如果你只使用一种过渡动效，Vue能够自动识别以哪个效果的时长为结束时长，但是在这种情况中，你就需要使用 `type` attribute 并设置 `animation` 或 `transition` 来明确**声明**你需要 Vue 监听的类型。

- 设为`animation`,不管`transition`时长是多长，只会以`animation`的时长渲染动效，反之亦然。
```html
  <div>
    <button @click="show = !show">
      toggle
    </button>

    <transition name="bounce" type="animation">
      <p v-if="show">
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris
        facilisis enim libero, at lacinia diam fermentum id. Pellentesque
        habitant morbi tristique senectus et netus.
      </p>
    </transition>
  </div>
```
```css
<style lang="scss" scoped>
.bounce-enter,.bounce-leave-to{
  transform: translateX(50px)
}
.bounce-enter-active {
  animation: bounce-in .1s;
  transition: all 5s ease;
}
.bounce-leave-active{
  animation: bounce-in .1s reverse;
  transition: all 5s ease;
}
@keyframes bounce-in {
  0% {
    color:red
  }
  50% {
    color:green
  }
  100% {
    color:blue;
  }
}
</style>
```
![Jul-07-2020 11-01-17.gif](https://cdn.nlark.com/yuque/0/2020/gif/377922/1594090973108-c983efca-aab9-411f-9ac1-6710262cdd3a.gif#align=left&display=inline&height=239&margin=%5Bobject%20Object%5D&name=Jul-07-2020%2011-01-17.gif&originHeight=320&originWidth=640&size=223468&status=done&style=none&width=477)

- 设为`transition`
```html
 <transition name="bounce" type="transition">
      <p v-if="show">
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris
        facilisis enim libero, at lacinia diam fermentum id. Pellentesque
        habitant morbi tristique senectus et netus.
      </p>
 </transition>
```
![Jul-07-2020 11-02-30.gif](https://cdn.nlark.com/yuque/0/2020/gif/377922/1594091049902-2cce377f-809a-4a52-b190-562576fc876b.gif#align=left&display=inline&height=259&margin=%5Bobject%20Object%5D&name=Jul-07-2020%2011-02-30.gif&originHeight=320&originWidth=636&size=704643&status=done&style=none&width=514)


## 显性的过渡时长
在很多情况下，Vue 可以自动得出过渡效果的完成时机。默认情况下，Vue 会等待其在过渡效果的根元素的第一个 `transitionend` 或 `animationend` 事件。但是如果我们的过渡效果有一系列呢，`transition`、`animation`动画都需要完整渲染呢？那么就可以通过自定义设置时长。

- `<transition :duration="1000">...</transition>`
- `<transition :duration="{ enter: 500, leave: 800 }">...</transition>`

     
```html
<transition name="bounce" :duration="5000">
  <p v-if="show">
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris
    facilisis enim libero, at lacinia diam fermentum id. Pellentesque
    habitant morbi tristique senectus et netus.
  </p>
</transition>
```
```css
<style lang="scss" scoped>
.bounce-enter,.bounce-leave-to{
  transform: translateX(50px)
}
.bounce-enter-active {
  animation: bounce-in .1s;
  transition: all 5s ease;
}
.bounce-leave-active{
  animation: bounce-in .3s reverse;
  transition: all 5s ease;
}
@keyframes bounce-in {
  0% {
    color:red
  }
  50% {
    color:green
  }
  100% {
    color:blue;
  }
}
</style>
```
![Jul-07-2020 11-08-45.gif](https://cdn.nlark.com/yuque/0/2020/gif/377922/1594091396520-1e5df3b8-24ae-4fdc-b497-eab8912471bd.gif#align=left&display=inline&height=214&margin=%5Bobject%20Object%5D&name=Jul-07-2020%2011-08-45.gif&originHeight=320&originWidth=636&size=827472&status=done&style=none&width=426)
可以看到css过渡和css动画两种动效都有展示。


## javaScript钩子
除了使用css来编写动画，也能够通过js来编写动画。而对应的js钩子如下：

- `before-enter` 对应 `v-enter`
- `enter`对应`v-enter-active`
- `after-enter`对应`v-enter-to`
- `enter-cancelled` 取消进入
- `before-leave` 对应 `v-leave`
- `leave`对应`v-enter-leave`
- `after-leave`对应`v-leave-to`
- `leave-cancelled` 取消离开
```html
<transition
  @before-enter="beforeEnter" 
  @enter="enter"
  @after-enter="afterEnter"
  @enter-cancelled="enterCancelled"

  @before-leave="beforeLeave"
  @leave="leave"
  @after-leave="afterLeave"
  @leave-cancelled="leaveCancelled"
>
</transition>
```
```javascript
methods: {
  // --------
  // 进入中
  // --------

  beforeEnter: function (el) {
    // ...
  },
  // 当与 CSS 结合使用时
  // 回调函数 done 是可选的
  enter: function (el, done) {
    // ...
    done()
  },
  afterEnter: function (el) {
    // ...
  },
  enterCancelled: function (el) {
    // ...
  },

  // --------
  // 离开时
  // --------

  beforeLeave: function (el) {
    // ...
  },
  // 当与 CSS 结合使用时
  // 回调函数 done 是可选的
  leave: function (el, done) {
    // ...
    done()
  },
  afterLeave: function (el) {
    // ...
  },
  // leaveCancelled 只用于 v-show 中
  leaveCancelled: function (el) {
    // ...
  }
}
```

- 当只用 JavaScript 过渡的时候，**在 `enter` 和 `leave` 中必须使用 `done` 进行回调**。否则，它们将被同步调用，过渡会立即完成。
- 推荐对于仅使用 JavaScript 过渡的元素添加 `v-bind:css="false"`，Vue 会跳过 CSS 的检测。这也可以避免过渡过程中 CSS 的影响
{:.warning}


示例：
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>  
<div>
    <button @click="show = !show">
      Toggle
    </button>
    <transition
      v-on:before-enter="beforeEnter"
      v-on:enter="enter"
      v-on:leave="leave"
      v-bind:css="false"
    >
      <p v-if="show">
        Demo
      </p>
    </transition>
  </div>
```
```javascript
    beforeEnter: function(el) {
      el.style.opacity = 0;
      el.style.transformOrigin = "left";
    },
    enter: function(el, done) {
      Velocity(el, { opacity: 1, fontSize: "1.4em" }, { duration: 300 });
      Velocity(el, { fontSize: "1em" }, { complete: done });
    },
    leave: function(el, done) {
      Velocity(el, { translateX: "15px", rotateZ: "50deg" }, { duration: 600 });
      Velocity(el, { rotateZ: "100deg" }, { loop: 2 });
      Velocity(
        el,
        {
          rotateZ: "45deg",
          translateY: "30px",
          translateX: "30px",
          opacity: 0,
        },
        { complete: done }
      );
    },
```


# 初始渲染的过渡
我们可以发现上面设置的动画只有在我们点击按钮的时候进行展示。当刷新浏览器初始渲染时并没有过渡效果。
可以通过 `appear` attribute 设置节点在初始渲染的过渡。
`apperar`也可以自定义类名，自定义JavaScript钩子。
```html
<transition
  appear
  appear-class="custom-appear-class"
  appear-to-class="custom-appear-to-class"
  appear-active-class="custom-appear-active-class"
>
  <!-- ... -->
</transition>

```
```html
<transition
  appear
  v-on:before-appear="customBeforeAppearHook"
  v-on:appear="customAppearHook"
  v-on:after-appear="customAfterAppearHook"
  v-on:appear-cancelled="customAppearCancelledHook"
>
  <!-- ... -->
</transition>
```
# 多个元素的过渡
在实际业务场景中经常会有多元素的过渡。当有相同标签名的元素切换时，需要通过 `key` attribute 设置唯一的值来标记以让 Vue 区分它们，否则 Vue 为了效率只会替换相同标签内部的内容。即使在技术上没有必要，给在 `<transition>` 组件中的多个元素设置 key 是一个更好的实践。
```html
  <div>
    <transition>
      <button :key="isEditing" @click="isEditing=!isEditing">
        {{ isEditing ? "Save" : "Edit" }}
      </button>
    </transition>
  </div>
```
## 过渡模式
“Save”按钮和“Edit”按钮的过渡中，两个按钮都被重绘了，一个离开过渡的时候另一个开始进入过渡。这是 `<transition>` 的默认行为 - 进入和离开同时发生。怎么版？
Vue提供了一下过渡模式：

- `in-out`:新元素先进行过渡，完成之后当前元素过渡离开；
- `out-in`:旧元素先进行过渡离开，完成之后新元素过渡进入。
```html
  <div class="page">
    <transition name="fade" mode="in-out">
      <button style="position:absolute;width:40px" :key="isEditing" @click="isEditing=!isEditing">
        {{ isEditing ? "Save" : "Edit" }}
      </button>
    </transition>
  </div>
```
```css
.page{
  margin-left:100px;
  position: static;
}
.fade{
  position: absolute;
}
.fade-enter{
  transform: translateX(30px);
  opacity: 0.3;
}
.fade-enter-to{
  opacity: 1;
}
.fade-leave-to{
  transform: translateX(-30px);
  opacity: 0.3;
}
.fade-enter-active,.fade-leave-active{
  transition: all .5s ease
}
```
![Jul-07-2020 13-44-49.gif](https://cdn.nlark.com/yuque/0/2020/gif/377922/1594100732923-2f2debfd-893c-456a-aa49-037d2688348b.gif#align=left&display=inline&height=320&margin=%5Bobject%20Object%5D&name=Jul-07-2020%2013-44-49.gif&originHeight=320&originWidth=352&size=113664&status=done&style=none&width=352)


# 多个组件的过渡
多个组件的过渡简单很多 - 我们不需要使用 key attribute。相反，我们只需要使用动态组件：
```html
<transition name="component-fade" mode="out-in">
  <component v-bind:is="view"></component>
</transition>
```
# 列表过渡
## 列表的进入离开过渡
使用`<transition-group></transition-group>`

- 不同于 `<transition>`，它会以一个真实元素呈现：默认为一个`<span>`。你也可以通过 `tag` attribute 更换为其他元素。
- 过渡模式不可用，因为我们不再相互切换特有的元素。
- **内部元素总是需要提供唯一的 `key` attribute 值。**
- CSS 过渡的类将会应用在**内部的元素**中，而不是这个组/容器本身。
## 列表的排序过渡
`<transition-group>` 组件还有一个特殊之处。不仅可以进入和离开动画，还可以改变定位。要使用这个新功能只需了解新增的 `v-move` class，它会在元素的改变定位的过程中应用。像之前的类名一样，可以通过`name` attribute来自定义前缀，也可以通过`move-class` attribute 手动设置。内部的实现，Vue 使用了一个叫 FLIP 简单的动画队列，使用 `transforms` 将元素从之前的位置平滑过渡新的位置。

需要注意的是使用 FLIP 过渡的元素不能设置为` display: inline` 。作为替代方案，可以设置为 `display: inline-block` 或者放置于 `flex` 中
{:.warning}


## 列表的交错过渡
通过js钩子函数进行设置过渡
# 可复用的过渡
过渡可以通过 Vue 的组件系统实现复用。要创建一个可复用过渡组件，你需要做的就是将 `<transition>` 或者 `<transition-group>` 作为根组件，然后将任何子组件放置在其中就可以了。推荐使用函数式组件
```javascript
Vue.component('my-special-transition', {
  functional: true,
  render: function (createElement, context) {
    var data = {
      props: {
        name: 'very-special-transition',
        mode: 'out-in'
      },
      on: {
        beforeEnter: function (el) {
          // ...
        },
        afterEnter: function (el) {
          // ...
        }
      }
    }
    return createElement('transition', data, context.children)
  }
})
```


# 动态过渡
通过动态`name`attribute以及js钩子来创建动态过渡


**唯一的限制是你的想象力。**
