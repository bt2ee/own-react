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

```js
const element = <h1 title="foo">Hello</h1>
const container = document.getElementById("root")
ReactDOM.render(element, container)
```

> **Let’s remove all the React specific code and replace it with vanilla JavaScript.**

**让我们移除 React 具体代码并且替换为原始 JavaScript**

> On the first line we have the element, defined with JSX. It isn’t even valid JavaScript, so in order to replace it with vanilla JS, first we need to replace it with valid JS.
> JSX is transformed to JS by build tools like Babel. The transformation is usually simple: replace the code inside the tags with a call to createElement, passing the tag name, the props and the children as parameters.

```js
const element = <h1 title="foo">Hello</h1>
```

在第一行我们有了 JSX 定义的元素。它甚至不是有效的JavaScript，因此，为了用原始 JavaScript 替代它，首先我们需要用有效的 JS 替代它。
JSX 通过 Babel 这样的构建工具被转换成 JS。转换通常很简单：调用 createElement 来替换标签内的代码，传递名称，props 和 children 作为参数。

> React.createElement creates an object from its arguments. Besides some validations, that’s all it does. So we can safely replace the function call with its output.

```js
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

```js
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

> The other piece of React code we need to replace is the call to ReactDOM.render.
>
> render is where React changes the DOM, so let’s do the updates ourselves.

```js
ReactDOM.render(element, container)
```

另一段我们需要替换的 React 代码是 ReactDOM.render 的调用。
渲染是 React 更改 DOM 的地方，因此我们自己进行更新。

> First we create a node* using the element type, in this case h1.
>
> Then we assign all the element props to that node. Here it’s just the title.
>
> * To avoid confusion, I’ll use “element” to refer to React elements and “node” for DOM elements.

首先我们使用 type 元素创建一个节点，在本例中的 h1。
然后我们把所有元素 props 分配给这个节点。在这里就是 title。

> 为了避免混淆，我会使用 “element” 来代表 React 元素，用 “node” 代表 DOM 元素。

```js
const element = {
  type: "h1",
  props: {
    title: "foo",
    ...
  },
}
​
...
​
const node = document.createElement(element.type)
node["title"] = element.props.title

...

```

> Then we create the nodes for the children. We only have a string as a child so we create a text node.
>
> Using textNode instead of setting innerText will allow us to treat all elements in the same way later. Note also how we set the nodeValue like we did it with the h1 title, it’s almost as if the string had props: {nodeValue: "hello"}.

然后我们为 children 创建节点。我们仅需要一个字符串当作 child 因此我们创建一个文本节点。
使用 textNode 而不是设置 innerText 使我们以后用同一方式处理所有元素。同时注意，我们如何像设置 h1 标题一样设置 nodeValue ，就像是字符串有了 props：{nodeValue: "hello"}。

```js
const element = {
  ...
  props: {
    title: "foo",
    children: "Hello",
  },
}
​
...
​
const text = document.createTextNode("")
text["nodeValue"] = element.props.children

...

```

> Finally, we append the textNode to the h1 and the h1 to the container.

最终，我们添加 textNode 到 h1 并且添加 h1 到容器中。

```js

...
​
const container = document.getElementById("root")

...
​
node.appendChild(text)
container.appendChild(node)
```

> And now we have the same app as before, but without using React.

现在，我们有了一个和之前一样到应用，但是没有使用 React。

```js
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
}
​
const container = document.getElementById("root")
​
const node = document.createElement(element.type)
node["title"] = element.props.title
​
const text = document.createTextNode("")
text["nodeValue"] = element.props.children
​
node.appendChild(text)
container.appendChild(node)
```

## 第 1 步：`createElement` 函数

> Let’s start again with another app. This time we’ll replace React code with our own version of React.
> We’ll start by writing our own createElement.
> Let’s transform the JSX to JS so we can see the createElement calls.

让我们从另一个应用开始。这次我们将会使用我们自己版本的 React 替换掉 React 代码。
我们将会从编写我们的自己的 createElement 开始。
让我们把 JSX 转换成 JS 以便于我们可以看出 createElement 如何调用。

```js
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
const container = document.getElementById("root")
ReactDOM.render(element, container)
```

> As we saw in the previous step, an element is an object with type and props. The only thing that our function needs to do is create that object.

就像我们看到的前一步，一个元素是一个拥有 type 和 props 的对象。我们函数需要的唯一的事情就是创建一个对象。

```js
const element = React.createElement(
  "div",
  { id: "foo" },
  React.createElement("a", null, "bar"),
  React.createElement("b")
)
...
```

> We use the spread operator for the props and the rest parameter syntax for the children, this way the children prop will always be an array.
> For example, createElement("div") returns:
> {
>   "type": "div",
>   "props": { "children": [] }
> }
> createElement("div", null, a) returns:
> {
>   "type": "div",
>   "props": { "children": [a] }
> }
> and createElement("div", null, a, b) returns:
> {
>  "type": "div",
>  "props": { "children": [a, b] }
> }

我们对 props 使用展开操作符并且对 children 使用 rest 参数语法，这样 children prop 将始终是一个数组。
例如，`createElement("div")` 返回：

```js
{
  "type": "div",
  "props": { "children": [] }
}
```

`createElement("div", null, a)` 返回：

```js
{
  "type": "div",
  "props": { "children": [a] }
}
```

`createElement("div", null, a, b)` 返回：


```js
{
  "type": "div",
  "props": { "children": [a, b] }
}
```

```js
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children,
    },
  }
}
...
```

> The children array could also contain primitive values like strings or numbers. So we’ll wrap everything that isn’t an object inside its own element and create a special type for them: TEXT_ELEMENT.
> React doesn’t wrap primitive values or create empty arrays when there aren’t children, but we do it because it will simplify our code, and for our library we prefer simple code than performant code.

children 数组同样能包含原始值，例如 string 数组或者 number 数组。因此我们会将不是对象的所有内容包含在自身元素中并且为他们创建一个特殊类型：`TEXT_ELEMENT`。
当没有 children 时，React 没有包裹原始值或者创建一个新的空数组，但是我们不这样做因为它会简化我们的代码，对于我们的库，我们更喜欢简单代码而不是高性能代码。

```js
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
       children: children.map(child =>
        typeof child === "object"
          ? child
          : createTextElement(child)
      ),
    },
  }
}
function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  }
}
...
```

> We are still using React’s createElement.
> In order to replace it, let’s give a name to our library. We need a name that sounds like React but also hints its didactic purpose.

我们仍然使用 React 的 createElement 方法。
为了替换它，让我们给我们的库取一个名字。我们需要一个名字听起来像 React 但是也暗示它的教学目的。

```js
const element = React.createElement(
  "div",
  { id: "foo" },
  React.createElement("a", null, "bar"),
  React.createElement("b")
)
...
```

> We’ll call it Didact.
> But we still want to use JSX here. How do we tell babel to use Didact’s createElement instead of React’s?

我们叫它 Didact
但是我们仍然希望使用 JSX。我们如果告诉 babel 来使用 Didact 的 createElement 代替 React 的呢？

```js
const Didact = {
  createElement,
}
​
const element = Didact.createElement(
  "div",
  { id: "foo" },
  Didact.createElement("a", null, "bar"),
  Didact.createElement("b")
)
```

> If we have a comment like this one, when babel transpiles the JSX it will use the function we define.

如果我们有这样的评论，当babel转译JSX时，它将使用我们定义的功能。

```js
...
/** @jsx Didact.createElement */
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
...
```

## 第 2 步：`render` 函数

> Next, we need to write our version of the ReactDOM.render function.

下一步，我们需要编写我们版本的 ReactDOM.render 函数

```jsx
ReactDOM.render(element, container)
```

> For now, we only care about adding stuff to the DOM. We’ll handle updating and deleting later.

现在，我们只关注在 DOM 中添加内容。我们会在稍后处理更新和删除。

```jsx
function render(element, container) {
  // TODO create dom nodes
}
​
const Didact = {
  createElement,
  render,
}
​
...
Didact.render(element, container)
```

> We start by creating the DOM node using the element type, and then append the new node to the container.

我们从使用元素类型创建 DOM 节点开始，然后添加新节点到容器中。

```jsx
function render(element, container) {
  const dom = document.createElement(element.type)
​
  container.appendChild(dom)
}
```

> We recursively do the same for each child.

我们为每一个 child 做相同的递归。

```jsx
function render(element, container) {
  const dom = document.createElement(element.type)
​
  element.props.children.forEach(child =>
    render(child, dom)
  )
​
  container.appendChild(dom)
}
```

> We also need to handle text elements, if the element type is TEXT_ELEMENT we create a text node instead of a regular node.

我们同样需要处理文本元素，如果元素类型是 `TEXT_ELEMENT` 我们创建文本节点而不是常规节点。

```jsx
function render(element, container) {
  const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)
​
  element.props.children.forEach(child =>
    render(child, dom)
  )
​
  container.appendChild(dom)
}
```

> The last thing we need to do here is assign the element props to the node.

这里最后一件需要做的事情是将元素 props 分配给节点。

```jsx
function render(element, container) {
  const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)
​
  const isProperty = key => key !== "children"
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })
​
  element.props.children.forEach(child =>
    render(child, dom)
  )
​
  container.appendChild(dom)
}
```

> And that’s it. We now have a library that can render JSX to the DOM.
> Give it a try on codesandbox.

就是这样，现在，我们有了一个可以渲染 JSX 到 DOM 的库了。
在 [codesandbox](https://codesandbox.io/s/didact-2-k6rbj) 尝试。

```
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child =>
        typeof child === "object"
          ? child
          : createTextElement(child)
      ),
    },
  }
}
​
function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  }
}
​
function render(element, container) {
  const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)
​
  const isProperty = key => key !== "children"
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })
​
  element.props.children.forEach(child =>
    render(child, dom)
  )
​
  container.appendChild(dom)
}
​
const Didact = {
  createElement,
  render,
}
​
/** @jsx Didact.createElement */
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
const container = document.getElementById("root")
Didact.render(element, container)
```
