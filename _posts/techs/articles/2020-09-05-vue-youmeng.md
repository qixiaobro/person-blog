---
title: Vue和小程序项目接入友盟
date: 2020-09-05 13:48:26
permalink: /docs/techs/youmeng
key: youmeng
sharing: true
show_author_profile: true
show_subscribe: true
articles:
  excerpt_type: html
sidebar:
  nav: tech-nav
categories:
  - 前端
  - 技术文章
tags:
  - 友盟
  - Vue.js
---
# 友盟是啥
<!--more-->
**友盟+**的功能特别多，如下图所示，有统计分析、开发者工具，营销增长、企业数据银行等。对我们来说最主要的功能就是统计分析：分为移动端app分析、web网站分析、小程序统计。
![截屏2020-08-10 下午5.13.18.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597050808649-b86f49c3-3db3-49df-8a47-02292d6fed74.png#align=left&display=inline&height=730&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-10%20%E4%B8%8B%E5%8D%885.13.18.png&originHeight=730&originWidth=2508&size=250765&status=done&style=none&width=2508)
# 友盟统计分析有哪些功能
统计分析功能能够：

1. 统计网站访问次数
1. 当前在线人数
1. 访问ip列表
1. 受访页面次数
1. 热点图（某个页面鼠标点击次数的分布图）
1. ....

通过这些统计数据运营和开发者就能够准确的知道产品的各种情况，从而能够制定精准的运营方案以及优化用户体验，提升产品的流量，增加用户，增加盈利。
![截屏2020-08-10 下午5.19.14.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597051169022-6dc00cd3-307b-420d-bad5-758f7cb62749.png#align=left&display=inline&height=728&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-10%20%E4%B8%8B%E5%8D%885.19.14.png&originHeight=728&originWidth=330&size=44302&status=done&style=none&width=330)![截屏2020-08-10 下午5.19.40.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597051184216-14057f12-4f96-48e7-9476-67e3916be455.png#align=left&display=inline&height=920&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-10%20%E4%B8%8B%E5%8D%885.19.40.png&originHeight=920&originWidth=328&size=53986&status=done&style=none&width=328)![截屏2020-08-10 下午5.20.01.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597051206463-fc2f448f-3f51-4b68-a961-ddedb41be671.png#align=left&display=inline&height=672&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-10%20%E4%B8%8B%E5%8D%885.20.01.png&originHeight=672&originWidth=330&size=40364&status=done&style=none&width=330)![截屏2020-08-10 下午5.20.17.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597051220883-eae7c9db-2f75-4393-9e89-47ff48fe33c1.png#align=left&display=inline&height=670&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-10%20%E4%B8%8B%E5%8D%885.20.17.png&originHeight=670&originWidth=344&size=37082&status=done&style=none&width=344)


# Vue项目中接入友盟
## 一：添加应用（站点）
进入友盟的工作台，点击添加web应用
![截屏2020-08-10 下午5.25.41.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597051645484-4ca5e905-9a0b-4d46-a1f3-bde88bc99fd1.png#align=left&display=inline&height=265&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-10%20%E4%B8%8B%E5%8D%885.25.41.png&originHeight=1034&originWidth=1660&size=227782&status=done&style=none&width=426) 
![截屏2020-08-10 下午5.26.40.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597051692101-dd717992-4855-4b73-a025-d25ef651e928.png#align=left&display=inline&height=217&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-10%20%E4%B8%8B%E5%8D%885.26.40.png&originHeight=568&originWidth=1128&size=60654&status=done&style=none&width=430)
![截屏2020-08-10 下午5.29.00.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597051963021-de22120b-b192-493b-aba5-3d1eb7e99140.png#align=left&display=inline&height=322&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-10%20%E4%B8%8B%E5%8D%885.29.00.png&originHeight=806&originWidth=1096&size=85842&status=done&style=none&width=438)
## 二：配置虚拟PV跟踪
虚拟PV(页面浏览量)跟踪用于统计AJAX、异步加载页面，友情链接，下载链接的流量。
在第一步点击确定后会跳转到代码拷贝页面：


![截屏2020-08-10 下午5.34.06.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597052218447-8aca899a-4fc1-4190-a06f-6d9c08e36c5c.png#align=left&display=inline&height=1542&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-10%20%E4%B8%8B%E5%8D%885.34.06.png&originHeight=1542&originWidth=2136&size=448725&status=done&style=none&width=2136)
对于普通的项目可以直接将上述任意一种代码复制粘贴到项目的</body>标签前，但是对于Vue、React这种单页面应用则不行。
在Vue项目的App.vue文件内。添加以下代码：
```javascript
  mounted(){
    const script = document.createElement('script')
    script.src = 'https://s95.cnzz.com/z_stat.php?id=1111111111&web_id=11111111'
    script.language = 'JavaScript'
    document.body.appendChild(script)
  },
  watch:{
    '$route' () {
      if (window._czc) {
        let location = window.location
        let contentUrl = location.pathname + location.hash
        let refererUrl = '/'
        window._czc.push(['_trackPageview', contentUrl, refererUrl])
      }82
    }
  },   
```
将上述代码中的**id**和**web_id**替换为你自己的应用的**id**即可。保存部署后访问线上页面。稍等片刻在站点列表对应应用中点击 **查看报表 **即可查看应用的访问统计数据
![截屏2020-08-10 下午5.42.12.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597052637185-02f5cfc8-5d3c-4e71-a3fe-7c47af250b09.png#align=left&display=inline&height=424&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-10%20%E4%B8%8B%E5%8D%885.42.12.png&originHeight=424&originWidth=2466&size=111919&status=done&style=none&width=2466)


## 三：api列表
| API | 功能描述 |
| --- | --- |
| _trackPageview | 用于发送某个URL的PV统计请求，适用于统计AJAX、异步加载页面，友情链接，下载链接的流量。 |
| _trackEvent | 用于发送页面上按钮等交互元素被触发时的事件统计请求。如视频的“播放、暂停、调整音量"，页面上的"返回顶部"、"赞"、"收藏等。也可用于发送Flash事件统计请求。 |
| _setCustomVar | 用于发送自定义访客标记。如访客是否登录、会员访客级别等。 |
| _deleteCustomVar | 用于删除自定义访客标记 |
| _setAutoPageView | 关闭页面上统计代码的自动PV统计，可配合trackPageview共同使用。 |
| _setAccount | 当页面上添加了多个CNZZ统计代码时，通过本方法可绑定需要哪个siteid对应的统计代码接受API发送的请求。未绑定的siteid将忽略相关请求。 |



### `_trackPageview`
用于发送某个URL的PV统计请求，适用于统计AJAX、异步加载页面，友情链接，下载链接的流量。

| 参数 | 必填/选填 | 类型 | 功能 | 备注 |
| --- | --- | --- | --- | --- |
| content_url | 必填 | string | 自定义虚拟PV页面的URL地址 | 填写以斜杠开头的相对路径，系统会自动补全域名,支持中文 |
| referer_url | 选填 | string | 自定义该受访页面的来源页URL地址 | 建议填写该异步加载页面的母页面。不填，则来路按母页面的来路计算。填为空",即"",则来路按"直接输入网址或书签计算。 |

语法：`_czc.push(["_trackPageview",cotent_url,referer_url])`


例子：
对于Vue项目，只需在上述配置虚拟PV跟踪中使用即可。其余地方如下载文件时，在方法里添加下方例子类似代码。
```javascript
methods:{
  download(){
      if(window._czc){
        window._czc.push(['_trackPageview', '/下载文件', '/'])
      }
    //other code
  }
}
```
通过api改写url,就能够在统计报表中清晰看到此下载操作的次数、访客等数据![截屏2020-08-11 上午11.58.33.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597118501670-6e3c4349-6312-4cb1-a808-16e7d7d64b3a.png#align=left&display=inline&height=67&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-11%20%E4%B8%8A%E5%8D%8811.58.33.png&originHeight=244&originWidth=2728&size=73897&status=done&style=none&width=746)


### `_trackEvent`
用于发送页面上按钮等交互元素被触发时的事件统计请求。如视频的“播放、暂停、调整音量”，页面上的“返回顶部”、“赞”、“收藏”等。也可用于发送Flash事件统计请求。

| 参数 | 必填/选填 | 类型 | 功能 | 备注 |
| --- | --- | --- | --- | --- |
| category | 必填 | string | 表示事件发生在谁身上，如"视频、“小说"、"轮显层"等等。 |  |
| action | 必填 | string | 表示访客跟元素交互的行为动作，如"播放"、"收藏"、"翻层"等 |  |
| label | 选填 | string | 用于更详细的描述事件，如具体是哪个视频，哪部小说。 |  |
| value | 选填 | int | 用于填写打分型事件的分值，加载时间型事件的时长，订单型事件的价格。 | 请填写整数数值，如果填写为其他形式，系统将按0处理。为浮点小数，系统会自动取整去掉小数点。 |
| nodeid | 选填 | string | 填写事件元素的div元素id | 请填写class id，暂不支持name. |

语法：
`_czc.push(["_trackEvent",category,action,label,value,nodeid]);`


例子：
```javascript
methods:{
  fetchRowDetail(){
      if(window._czc){
        window._czc.push(['_trackEvent',"详情",'点击','1',''])
      }
    	//other code
  }
}
```
在统计列表的事件菜单下可以看到统计的事件数据。![截屏2020-08-11 上午11.59.36.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597119616648-be798814-7206-44f2-9ed4-585c33ce4a31.png#align=left&display=inline&height=1546&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-11%20%E4%B8%8A%E5%8D%8811.59.36.png&originHeight=1546&originWidth=2732&size=270362&status=done&style=none&width=2732)


### `_setCustomVar`
用于发送为访客打自定义标记的请求，用来统计会员访客、登录访客、不同来源访客的浏览数据。



| 参数 | 必填/选填 | 类型 | 功能 | 备注 |
| :--- | :--- | :--- | :--- | :--- |
| name | 必填 | string | 自定义访客种类，用来描述观察访客的角度，如“会员级别”、“访客来源”等等。 |   |
| value | 必填 | string | 自定义访客值，表示对访客类型的具体描述，如"VIP1"、"VIP2"等等。 |   |
| time | 选填 | int | 有效时长，表示本自定义访客标记的生效时长。 | 不填或填“1”表示长期有效。
填“0”表示仅在发包页面有效。
填“2”表示仅在本访次有效。
填具体数值，表示生效时长，单位“秒”。 |



语法：`_czc.push(["_setCustomVar",name,value,time])`


例子：在用户访问是打上访客的标志，在用户登录后根据等级打上用户级别，这样在统计报表中就能够看到访客的、不同等级用户的访问数据了。
```javascript
//首页,用户访问被打上访客标志
  mounted() {
      if(window._czc){
        window._czc.push(['_setCustomVar',"访客",'等级1','1'])
      }
  },
 
//登录页，登录成功，打上会员标志
 methods:{
   asyncLogin(){
     //....
     this.$message({
       type: "success",
       message: "登录成功！"
     });
     if(window._czc){
       window._czc.push(['_setCustomVar',"会员",'等级2','1'])
      }
     //....
   }
 }
```
### `_deleteCustomer`
发送删除自定义访客标签的请求。将访客身上已被标记的自定义访客类型去掉，去掉后不再继续统计。



| 参数 | 必填/选填 | 类型 | 功能 | 备注 |
| :--- | :--- | :--- | :--- | :--- |
| name | 必填 | string | 需要被删除的自定义访客类型。 | 填写自定义访客类型种类名name。 |



语法：`_czc.push(['_deleteCustomVar',name])`


例子：用户退出，删除登录标志。
```javascript
methods:{
  asyncLogot(){
    this.$message({
      type: "success",
      message: "退出成功！"
    });
    if(window._czc){
      window._czc.push(['_deleteCustomVar',"会员退出"])
    }
  }
}
```


### `_setAutoPageview`
主动关闭或开启PV跟踪统计



| 参数 | 必填/选填 | 类型 | 功能 | 备注 |
| :--- | :--- | :--- | :--- | :--- |
| autopageview | 必填 | boolean | 是否自动发送页面PV的统计请求。 | 关闭自动发送，填false
开启自动发送，为true，不调用时默认为true |



语法：`_czc.push(﻿["_setAutoPageview",autopageview])`


例子：统计视频某个时间段的播放量，则当用户点击播放时需要发送统计数据，当用户点击关闭，则需要停止发送统计数据。
```javascript
methods:{
  	videoPlay(){
      if(window._czc){
      	window._czc.push(['_setAutoPageview',true])
    	}
    },
    videoClose(){
      if(window._czc){
      	window._czc.push(['_setAutoPageview',false])
    	}
    }
}
```


### `_setAccount`
当您的页面上添加了多个CNZZ统计代码时，需要用到本方法绑定需要哪个siteid对应的统计代码来接受API发送的请求。未绑定的siteid将忽略相关请求。比如需要某个项目被多个应用统计，且每个应用统计的是不同的操作，有些是播放量，有些是点击量。则在相对应的代码中可以设置不同的统计代码来接收API请求。



| 参数 | 必填/选填 | 类型 | 功能 | 备注 |
| :--- | :--- | :--- | :--- | :--- |
| siteid | 必填 | int | 绑定要接受API请求的统计代码siteid。 |   |



语法：`_czc.push(['_setAccount',siteid])`


比较适用于多页面应用，单页面应用暂未找到如何正确使用。因为本API一定要放在其他API（如`_trackPageview`）和网站统计代码之前，但是单页面应用在app.vue文件中监听路由变化有使用`_trackPageview`


## 四：热点图
热点图功能记录页面访客的鼠标点击行为，通过颜色区分不同区域点击热度。 支持"关注范围"的设定，按来路细分点击热度等功能。
在报表菜单-》热点图中，添加热点图，填写对应信息即可。
![截屏2020-08-11 下午2.46.44.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597128513446-ff9b0e0c-b2f1-4eb7-8f98-c53055f4dde0.png#align=left&display=inline&height=391&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-11%20%E4%B8%8B%E5%8D%882.46.44.png&originHeight=1448&originWidth=2722&size=733403&status=done&style=none&width=735)
![截屏2020-08-11 下午2.47.37.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597128521656-0acd8e37-b37c-4d4f-b518-b1c5c928dab9.png#align=left&display=inline&height=411&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-11%20%E4%B8%8B%E5%8D%882.47.37.png&originHeight=868&originWidth=1574&size=128402&status=done&style=none&width=746)


查看热点图：需要网站已支持https和跨越，才能看到网页背景。
![截屏2020-08-11 下午2.49.21.png](https://cdn.nlark.com/yuque/0/2020/png/377922/1597128575484-dae4d86b-6a2d-4b77-bac6-7b9454b3a5cb.png#align=left&display=inline&height=1500&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2020-08-11%20%E4%B8%8B%E5%8D%882.49.21.png&originHeight=1500&originWidth=3128&size=272271&status=done&style=none&width=3128)
# 如何在小程序项目中接入友盟
暂未实践，待续...
