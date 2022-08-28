---
title: vue源码阅读笔记
---
## vue源码阅读笔记

### 1.new Vue发生了什么

### 2.Vue实例的挂载实现

### 3.render

​	用来把实例渲染成一个虚拟Node。 最终调用了createElement方法，并返回一个vnode。

### 4.虚拟DOM

参考https://github.com/snabbdom/snabbdom

###5.createElement

> 创建虚拟DOM的方法

_createElement方法的封装

_createElement(context, tag, data, children, normalizationType)

五个参数：

context: component类型

tag：标签，字符串或者component

data：VNodeData类型

children： 任意类型

normalizationType：子节点规范的类型

这个方法主要的流程：

- children的规范化，涉及以下两个方法

​	normalizeChildren(children) 调用场景有两个：render是用户自己手写的、编译slot和v-for时

simpleNormalizeChildren(children) 调用场景是render函数生成的，理论上编译生成的children都是VNode类型，但这里有一个例外，就是functional component函数式组件返回的是一个数组而不是一个根节点，

- VNode的创建

​	先判断tag，tag是字符串类型，继续看是否为内置的一些节点，是的话直接创建一个普通的VNode，如果是已注册的组件名则通过createComponent创建一个组件类型的VNode，否则创建一个未知的标签的VNode节点。

### 6.update

> 把VNode渲染成真实的DOM

​	Vue的_update是实例的一个私有方法，它被调用的时机有两个：

- 一个是首次渲染，

- 一个是数据更新的时候

​	核心是调用vm.\_\_patch\_\_方法。这个方法在不同的平台定义是不一样的。

### 7.createComponent

> createElement对tag判断，当tag不是普通html标签时，以createComponent方法创建一个组件VNode

- 构造子类构造函数
- 安装组件钩子函数，暴露patch流程中的钩子函数，方便做一些额外操作
- 实例化VNode

### 8.patch



