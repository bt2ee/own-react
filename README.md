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

> On the first line we have the element, defined with JSX. It isn’t even valid JavaScript, so in order to replace it with vanilla JS, first we need to replace it with valid JS.
> JSX is transformed to JS by build tools like Babel. The transformation is usually simple: replace the code inside the tags with a call to createElement, passing the tag name, the props and the children as parameters.

```
const element = <h1 title="foo">Hello</h1>
```

在第一行我们有了 JSX 定义的元素。它甚至不是有效的JavaScript，因此，为了用原始 JavaScript 替代它，首先我们需要用有效的 JS 替代它。
JSX 通过 Babel 这样的构建工具被转换成 JS。转换通常很简单：调用 createElement 来替换标签内的代码，传递名称，props 和 children 作为参数。

> React.createElement creates an object from its arguments. Besides some validations, that’s all it does. So we can safely replace the function call with its output.

```
const element = React.createElement(
  "h1",
  { title: "foo" },
  "Hello"
)
```

React.createElement 根据参数创建一个对象。除了一些验证外，这就是它全部的工作。因此，我们可以安全地将函数调用替换为其输出。

> And this is what an element is, an object with two properties: type and props (well, it has more, but we only care about these two).
>
> The type is a string that specifies the type of the DOM node we want to create, it’s the tagName you pass to document.createElement when you want to create an HTML element. It can also be a function, but we’ll leave that for Step VII.
>
> props is another object, it has all the keys and values from the JSX attributes. It also has a special property: children.
>
> children in this case is a string, but it’s usually an array with more elements. That’s why elements are also trees.

```
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
}
```

这就是一个元素，一个具有两个属性的元素：type 和 props（好吧，它还有[更多属性](https://github.com/facebook/react/blob/f4cc45ce962adc9f307690e1d5cfa28a288418eb/packages/react/src/ReactElement.js#L111)，但是我们只关心这两个）

type 参数是一个 string，用于我们想要创建的 DOM 节点的类型，这是当你想要创建一个 HTML 元素时传递给 document.createElement 的 tagName。它也可以是个函数，我们会在第 VII 讲到。

props 是另一个对象，它具有 JSX 属性中的所有键和值。 它还有一个特殊的属性：children。

在这种情况下，children 是一个字符串，但通常是一个包含更多元素的数组。 这就是为什么元素也是树的原因。
