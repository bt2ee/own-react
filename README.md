> 来源于：https://pomb.us/build-your-own-react/
## Build your own React

> We are going to rewrite React from scratch. Step by step. Following the architecture from the real React code but without all the optimizations and non-essential features.

我们准备从头一步步重写 React。跟随真实 React 代码架构，但没有所有的优化和没必要的功能。

> If you’ve read any of [my previous “build your own React” posts](https://engineering.hexacta.com/didact-learning-how-react-works-by-building-it-from-scratch-51007984e5c5), the difference is that this post is based on React 16.8, so we can now use hooks and drop all the code related to classes.

如果你阅读过任何[我以前“创建你自己的 React”文章](https://engineering.hexacta.com/didact-learning-how-react-works-by-building-it-from-scratch-51007984e5c5)，区别于前者基于 React 16.8，因此我们现在可以使用 hooks 并且下掉基于类的所有代码。

> Starting from scratch, these are all the things we’ll add to our version of React one by one:
> - Step I: The createElement Function
> - Step II: The render Function
> - Step III: Concurrent Mode
> - Step IV: Fibers
> - Step V: Render and Commit Phases
> - Step VI: Reconciliation
> - Step VII: Function Components
> - Step VIII: Hooks

从头开始，所有这些都是我们将一一添加到我们的React版本中的所有内容：

- I、createElement 函数
- II、render 函数
- III、并发模式
- IV、fibers
- V、渲染和提交阶段
- VI、协调器
- VII、函数组件
- VIII、Hooks

> ## Step Zero: Review

## 第 0 步：审查

> But first let’s review some basic concepts. You can skip this step if you already have a good idea of how React, JSX and DOM elements work.

首先让我们来审查一些基础概念。如果你已经对 React、JSX 和 DOM 的工作方式有很好的理解你可以跳过这步。

> We’ll use this React app, just three lines of code. The first one defines a React element. The next one gets a node from the DOM. The last one renders the React element into the container.

我们将使用 React 应用，仅仅只有三行代码，第一行定义了一个 React 元素。下一行通过 DOM 获取了一个节点。最后一行将 React 节点渲染到容器中

```
const element = <h1 title="foo">Hello</h1>
const container = document.getElementById("root")
ReactDOM.render(element, container)
```

> **Let’s remove all the React specific code and replace it with vanilla JavaScript.**

**让我们移除 React 具体代码并且替换为原始 JavaScript**
