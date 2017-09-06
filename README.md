# Intro
软件开发没有银弹。没有最好的，只有最合适的。本文从前端软件开发的各个角度来描述开发过程，并提供最佳实践（长期更新）。 更多介绍：https://my.oschina.net/wanjubang/blog/1480214

> 注： 由于本文是基于钉钉人力窝项目本身的，本文的一切关注点都是基于react+ redux 架构的。

# 前端开发过程的核心关注点

这里只考虑开发过程，不讨论产品出设计，设计出图，和测试撕逼等一系列过程。

## 处理路由

现在的页面由于多种原因，都在做spa，因此router 必不可少。 
项目中使用的react-router + react-router-redux: 前者是业界标准，后者可以同步 route 信息到 state，
这样你可以在 view 根据 route 信息调整展现，以及通过 action 来修改 route 。

## 组织数据

数据管理使用大名鼎鼎的redux，这里着重讲一下， 一切外部依赖都放在redux store中进行管理。 其他都放在component state 中管理。 为什么
这么做呢？一方面方便测试，我们知道redux是函数式的，方便测试的，而且我们的外部依赖存放在store，因此我们可以通过模拟store，从而模拟外部依赖。第二，
由于react是数据驱动的，我们的目标就是给定的数据，输出的页面是一致的，这样我们可以将核心关注点放在数据本身上。第三，不要将组件自身状态存在store中，什么是组件自身状态？ 组件自身状态包括不限于从props中计算的state，引发自身状态改变的state（是否被选中，是否被点击过等等）。 有人有这个疑问，
如果其他组件需要获取我的组件自身状态（是否被选中，是否点击过等等）呢？ 这里分两种情况，一种是父子组件，这种直接通过父子组件数据共享的方式解决。第二
种情况，组件没有父子关系，可能完全是两个页面。这种直接转化为外部依赖，你可以传到后端，然后再从后端取。或者存到任何其他的外部依赖（localsstorage等）

## 管理外部依赖
外部依赖有哪些？ 后端接口，本地db，用户操作，其他库。 如果管理这些依赖非常重要。我们挨个讨论。
### 后端接口
后端接口，我们统一放到action中处理，这样有一个集中处理和查看的地方，不好的地方在于频繁切换文件，打断思路。
dispatch通过外部注入，我们可以在任意地方调用dispacth 发送action 进而改变store。

### 本地db
本地db的操作都封装在了utils/db.js . 但是这个过程是不纯粹的，它永远是一个噩梦，你永远无法保证他是正常可用的。可以将db操作封装到高阶函数中
减少被噩梦惊醒的风险🤦‍。

### 用户操作

用户有哪些操作，正常异常操作是否考虑周全。是否对异常操作进行了处理。

### 其他库

外部库选择上我们通常会选择使用人数多，维护好，更新频繁的库，这种库有保障，使用质量地下的库，会带来很多隐患，这种需要引入外部库的
是否小心谨慎。
另一方面，版本变更有时候会到处功能异常，这种可以通过锁定版本的方式降低风险。


参考：

https://github.com/acdlite/redux-promise

https://github.com/acdlite/redux-actions

https://www.zhihu.com/question/36431501/answer/67560434

## 视图
视图就应该有视图的样子。视图应该保证他的纯粹性。 什么意思呢？ 就是说尽可能纯粹。给定外部依赖，输出页面是一致的，给定操作（操作也是一种外部依赖），
页面输出也是一致的（输出不仅仅是展示哦，还包括他影响的其他东西，譬如说受它影响的组件）。 组件最好只有render函数，这样有利于维护测试，而且还有利于代码重用。 试想谁会用你组件里面的方法？

参考：

https://stackoverflow.com/questions/37793936/eslint-react-with-airbnb

https://github.com/missive/functional-components-benchmark

# 代码风格

良好的代码风格体现了一个人的信仰，风度，甚至态度。
遵循一定的业界标准是很重要的，代码风格尽量不要特立独行。
写出易读易理解很困难，不然我们也不会都骂前任的代码。
为了我们不被接手的人骂，我们应该多注意下自己的代码风格。
[想看更多？](https://my.oschina.net/wanjubang/blog/1494513)

参考：

https://github.com/airbnb/javascript

http://google.github.io/styleguide/jsguide.html

https://github.com/prettier/prettier

上面是对普通的js语法进行讲解的最佳实践。针对react的最佳实践[可参考](https://github.com/renliwo/react-best-practice)。


# git 工作流
目前比较推荐的工作流程（workflow）是这样的：

```
Make changes
Commit those changes
Make sure your tests pass if exsit
Bump version in package.json
conventionalChangelog
Commit package.json and CHANGELOG.md files
Tag
Push
```

下面是钉钉具体项目的开发步骤：

```
1、git clone xxxxxx
2、从master建立一个本地分支 `git checkout -b daily/xxx`【xxx比当前publish/0.0.2下的版本号大，比如0.0.3】
3、修改完之后添加/提交 `git add -A /git commit -m '修改日志'`
4、推送到远程分支 `git push origin daily/xxx`
5、打tag版本跟daily版本一致   `git tag publish/xxx`
6、将daily/xxx推送到publish/xxx  `git push origin pubish/xxx`
7、这个时候已经将资源发布到cdn上了，检查内容是否发布成功https://g.alicdn.com/dingding/项目名/xxx/index.css
8、资源发布成功后，下次开发需要更新下master，再继续上面的步骤
9、修改diamond配置http://diamond.alibaba-inc.com/diamond-ops/static/pages/config/index.html?serverId=pre&=undefined
```

开启新release
```
1. 切换到主分支 `git checkout master`
2. 同步代码 `git pull`
3. 从master建立一个本地分支 `git checkout -b daily/xxx`【xxx比当前publish/0.0.2下的版本号大，比如0.0.3】
4. 构建代码 npm run build
5. 推送分支 git push --set-upstream origin daily/x.x.x
```

关于commit 方面，业界比较流行的做法是angular 的  [AngularJS Git Commit Message Conventions](https://gist.github.com/stephenparish/9941e89d80e2bc58a153)
目前mobile端的规范就是遵循的上述规范，这里有具体的介绍。 目前我在项目中使用的是[conventional-changelog](https://marketplace.visualstudio.com/items?itemName=KnisterPeter.vscode-commitizen)这个 vscode
的插件来规范提交的。如果大家使用的不是vscode，可以自行搜索应用市场下载，如果没有的化，可以使用[cli 工具](https://github.com/conventional-changelog/conventional-changelog/tree/master/packages/conventional-changelog-cli)继承

> 目前项目中并没有添加强制使用angular风格提交的hooks。计划大家都理解习惯之后再添加。

# 关于文档

## README
文档一律采用markdown 来写。 项目总体介绍在根目录下的README.md  
大家可以自行在自己的模块下添加README

## CHANGELOG

关于changelog， 目前的做法是直接根据conventional commit message 生成。如果看来，规范的commit message 还是
很有必要的，大家加油。一个版本完成之后，执行：

```js
npm run changelog
``` 
系统会自动根据semver生成增量的changelog信息。 如此看来，语义化版本还是很重要的。
好在项目版本是semver

# 基本原则

1. 尽量不要手动bind

2.  不要写console代码

3. 关键路径加上monitor

4. component 尽量就是一个render（see also： prefer-stateless-function）

5. 不要写for 这种命令式的代码（see alse： functional-programing）

6. eslint 尽量不要留warning（1.墨菲定律）（2.牺牲代码质量换取的结果，通常是得不偿失）


# 一些准则
1. import 中间   外部函数库和 内部库， 样式  组件 。以及 action 等分组（中间空一行，并在上方加行注释）

2. 函数内部  正常逻辑  和 异常逻辑中间空一行。 先异常return  然后正常逻辑。 变量声明和其他内容空一行

3. 尽量不要给变量命名，因为要命名你就要想一个reasonable 的 名字。

4. think before coding。 写代码前注释写下你的想法。

想好外部依赖，分析好组件划分，划分好函数功能块以及是否可以通用。 正常逻辑是怎样，异常逻辑是怎样。 处理不了的意外输入是否明确抛出。

