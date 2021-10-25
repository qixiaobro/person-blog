---
title: Node.js 入门
date: 2021-03-27 13:26:49
permalink: /docs/techs/nodejs-intor
key: nodejs-intor
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
  - Node.js
---

## 简单介绍
![xmind](/assets/images/node-intorduce.png)

## 入门教程

<!--more-->
### 如何从 Node.js 读取环境变量
Node.js 的`process`核心模块提供了`env`属性，该属性承载了在启动进程时设置的所有环境变量。<br />这是访问 NODE_ENV 环境变量的示例，该环境变量默认情况下被设置为development。<br />`process.env`<br />​<br />
### 如何使用 Node.js REPL
REPL 也被称为**运行评估打印循环**，是一种编程语言环境（主要是控制台窗口），它使用单个表达式作为用户输入，并在执行后将结果返回到控制台。

- 使用tab键自动补全：在编写代码时，如果按下 tab 键，则 REPL 会尝试自动补全所写的内容，以匹配已定义或预定义的变量。
- 探索JavaScript对象，例如在控制台输入`Number`，并添加一个. 然后按下`tab`,REPL会打印可以在该类上访问的所有属性和方法。
- 通过输入 global. 并按下 tab，可以检查可以访问的全局变量。
- 在某些代码之后输入 _，则会打印最后一次操作的结果。
- 点命令
   - .help: 显示点命令的帮助。
   - .editor: 启用编辑器模式，可以轻松地编写多行 JavaScript 代码。当处于此模式时，按下 ctrl-D 可以运行编写的代码。
   - .break: 当输入多行的表达式时，输入 .break 命令可以中止进一步的输入。相当于按下 ctrl-C。
   - .clear: 将 REPL 上下文重置为空对象，并清除当前正在输入的任何多行的表达式。
   - .load: 加载 JavaScript 文件（相对于当前工作目录）。
   - .save: 将在 REPL 会话中输入的所有内容保存到文件（需指定文件名）。
   - .exit: 退出 REPL（相当于按下两次 ctrl-C）。



### Node.js 从命令行接收参数
使用`node xxx.js`可以向xxx.js传入任意数量的参数。参数可以是独立的，也可以是具有键和值。<br />xxx.js通过`process.argv`来获取传入的参数，第一个参数是 node 命令的完整路径，第二个参数是正被执行的文件的完整路径。第三个参数开始才是我们传入的参数。
```javascript
//独立参数
node test.js a

//具有键和值 键和值和= 至今不能有空格，否则会被认为是三个参数
node test.js a=1
```
```javascript
const args = process.argv.slice(2)//获取传入的参数
```
对于有键和值的情况，需要自己解析键和值。也可以使用第三方库[minimist](https://github.com/substack/minimist)来解析。<br />​<br />
### 使用 Node.js 输出到命令行

   - 使用控制台模块的基础输出 `console.log()`，也可以通过传入变量和格式说明符来格式化用语。
      - %s 会格式化变量为字符串
      - %d 会格式化变量为数字
      - %i 会格式化变量为其整数部分
      - %o 会格式化变量为对象



   - `console.clear()`：清除控制台
   - `console.count()`：对打印的字符串的次数进行计数
   - `console.trace()`：打印堆栈踪迹
   - 可以使用 `console.time(str)` 和 `console.timeEnd(str)`， 轻松地计算函数运行所需的时间,其中str要是一样的字符串。可以通过`console.timeLog(str)`输出中间段的时间。
   - `stdout` 和 `stderr` :console.log 非常适合在控制台中打印消息。 这就是所谓的标准输出（或称为 stdout）,console.error 会打印到 stderr 流。它不会出现在控制台中，但是会出现在错误日志中。
   - 为输出着色：可以使用[转义序列](https://gist.github.com/iamnewton/8754917)来输出有颜色的字，但是更推荐使用[chalk](https://github.com/chalk/chalk)插件
   - 创建进度条：[Progress](https://github.com/visionmedia/node-progress#readme) 是一个很棒的软件包，可在控制台中创建进度条。 使用 npm install progress 进行安装。



### 在 Node.js 中从命令行接收输入
如何使Node.js CLI 程序具有交互性？<br />Node.js 提供了 [readline](http://nodejs.cn/api/readline.html) 模块来执行以下操作：每次一行地从可读流（例如 process.stdin 流，在 Node.js 程序执行期间该流就是终端输入）获取输入。
```javascript
const readline = require('readline').createInterface({
  input: process.stdin,
  output: process.stdout
})

readline.question(`你叫什么名字?`, name => {
  console.log(`你好 ${name}!`)
  readline.close()
})
```
推荐使用第三方插件[Inquirer.js](https://github.com/SBoudrias/Inquirer.js).<br />​<br />
### npm 包管理器简介
npm 是 Node.js 标准的软件包管理器。

   - npm install  <package-name> 下载安装特点的软件包
      - --save : 安装到dependencied
      - --save-dev：安装到devDependencies
      - 默认安装在当前文件树的node_modules子文件夹中，-g 安装在全局
   - npm update  更新软件包
      - npm update <package-name> 更新特定的软件包
   - 版本控制
   - 运行任务  运行package.json文件scripts



### 使用和执行npm安装的软件包
npm安装的软件包默认安装在当前文件树的node_modules子文件夹。若要在代码中使用它，只需`require`<br />如果是可执行文件，npm会把它放在`node_modules/.bin/` 文件夹下。使用以下命令执行：<br />`./node_modules/.bin/<package-name>`<br />​

npm > 5.2版本可以使用 `npx <package-name>`执行<br />

### package.json指南

   - 一个json文件
   - 属性分类
      - name
      - author
      - contributors
      - bugs
      - homepage
      - version
      - license
      - keywords
      - description
      - repository
      - main
      - private：防止意外的被发布到npm上
      - scripts
      - dependencies
      - devDependencies
      - engines
      - browserslist
      - 命令特有的属性



### package-lock.json文件
该文件旨在跟踪被安装的每个软件包的确切版本，以便产品可以以相同的方式被 100％ 复制（即使软件包的维护者更新了软件包）。例如：​<br />如果写入的是 〜0.13.0，则只更新补丁版本：即 0.13.1 可以，但 0.14.0 不可以。<br />如果写入的是 ^0.13.0，则要更新补丁版本和次版本：即 0.13.1、0.14.0、依此类推。<br />如果写入的是 0.13.0，则始终使用确切的版本。<br />

### 查看安装的npm版本	

- npm list 获取所有软件包版本，包括嵌套的
   - npm list -depth=0 获取顶层软件包版本
   - npm list -g 获取全局安装的软件包版本
   - npm list <package-name> 查询指定软件包版本
- npm view <package-name> version 查看软件包在npm仓库上最新的可用版本
   - npm view <package-name> versions 查看npm包所有版本



### 安装旧npm版本
npm i <package-name>@version 安装指定版本<br />​<br />
### 升级版本	
npm update<br />

### 使用npm语义控制版本
语义版本控制的概念很简单：所有的版本都有 3 个数字：x.y.z。​

      1. 第一个数字是主版本。
      1. 第二个数字是次版本。
      1. 第三个数字是补丁版本。


<br />当发布新的版本时，不仅仅是随心所欲地增加数字，还要遵循以下规则：<br />当进行不兼容的 API 更改时，则升级主版本。<br />当以向后兼容的方式添加功能时，则升级次版本。<br />当进行向后兼容的缺陷修复时，则升级补丁版本。<br />
<br />npm 设置了一些规则，可用于在 package.json 文件中选择要将软件包更新到的版本（当运行 npm update 时）。规则使用了这些符号：<br />​

`^`<br />`~`<br />`>`<br />`>=`<br />`<`<br />`<=`<br />`=`<br />`-`<br />`||`<br />这些规则的详情如下：<br />​

`^`: 只会执行不更改最左边非零数字的更新。 如果写入的是 `^0.13.0`，则当运行 `npm update` 时，可以更新到 `0.13.1`、`0.13.2` 等，但不能更新到 `0.14.0` 或更高版本。 如果写入的是 `^1.13.0`，则当运行 npm update 时，可以更新到 `1.13.1`、`1.14.0` 等，但不能更新到 2.0.0 或更高版本。<br />`~`: 如果写入的是 〜0.13.0，则当运行 npm update 时，会更新到补丁版本：即 0.13.1 可以，但 0.14.0 不可以。<br />`>`: 接受高于指定版本的任何版本。<br />`>=`: 接受等于或高于指定版本的任何版本。<br />`<=`: 接受等于或低于指定版本的任何版本。<br />`<`: 接受低于指定版本的任何版本。<br />`=`: 接受确切的版本。<br />`-`: 接受一定范围的版本。例如：2.1.0 - 2.6.2。<br />`||`: 组合集合。例如 < 2.1 || > 2.6。<br />可以合并其中的一些符号，例如 1.0.0 || >=1.1.0 <1.2.0，即使用 1.0.0 或从 1.1.0 开始但低于 1.2.0 的版本。<br />还有其他的规则：<br />无符号: 仅接受指定的特定版本（例如 1.2.1）。<br />`latest`: 使用可用的最新版本。<br />​<br />
### 卸载npm包
`npm uninstall <package-name>` 删除node_modules里的文件<br />`npm uninstall -S <package-name>` 此操作还会移除 package.json 文件中的引用<br />`npm uninstall -D <package-name>` 程序包是开发依赖项，使用此操作移除package.json中的引用<br />`npm uninstall -g <package-name>` 删除全局的软件包<br />​<br />
### npm 全局或本地的软件包
本地和全局的软件包之间的主要区别是：​

      - 本地的软件包 安装在运行 `npm install <package-name>` 的目录中，并且放置在此目录下的 `node_modules` 文件夹中。
      - 全局的软件包 放在系统中的单独位置（确切的位置取决于设置），无论在何处运行 `npm install -g <package-name>`。



### Node.js 包运行器 npx
npx 功能

      - 轻松地运行本地命令   

eg:原先执行命令 `node-modules/.bin/mocha --version`<br />    使用npx `npx mocha`  会自动去查找path

      - 无需安装的命令执行（先安装软件包，执行完后擦除软件包）
      - 使用不同的 Node.js 版本运行代码 eg： `npx node@10 -v #v10.18.1`
      - 直接从 URL 运行任意代码片段 

eg：`npx https://gist.github.com/zkat/4bc19503fe9e9309e2bfaa2c58074d32`
