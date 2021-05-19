> 来源于：https://pomb.us/build-your-own-react/

## Build your own React

> We are going to rewrite React from scratch. Step by step. Following the architecture from the real React code but without all the optimizations and non-essential features.

我们准备从头一步步重写 React。跟随真实 React 代码架构，但没有所有的优化和没必要的功能。

> If you’ve read any of [my previous “build your own React” posts](https://engineering.hexacta.com/didact-learning-how-react-works-by-building-it-from-scratch-51007984e5c5), the difference is that this post is based on React 16.8, so we can now use hooks and drop all the code related to classes.

如果你阅读过任何[我以前“创建你自己的 React”文章](https://engineering.hexacta.com/didact-learning-how-react-works-by-building-it-from-scratch-51007984e5c5)，区别于前者基于 React 16.8，因此我们现在可以使用 hooks 并且下掉基于类的所有代码。

> Starting from scratch, these are all the things we’ll add to our version of React one by one:
>
> - Step I: The createElement Function
> - Step II: The render Function
> - Step III: Concurrent Mode
> - Step IV: Fibers
> - Step V: Render and Commit Phases
> - Step VI: Reconciliation
> - Step VII: Function Components
> - Step VIII: Hooks

从头开始，所有这些都是我们将一一添加到我们的 React 版本中的所有内容：

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
const element = <h1 title="foo">Hello</h1>;
const container = document.getElementById("root");
ReactDOM.render(element, container);
```

> **Let’s remove all the React specific code and replace it with vanilla JavaScript.**

**让我们移除 React 具体代码并且替换为原始 JavaScript**

> On the first line we have the element, defined with JSX. It isn’t even valid JavaScript, so in order to replace it with vanilla JS, first we need to replace it with valid JS.
> JSX is transformed to JS by build tools like Babel. The transformation is usually simple: replace the code inside the tags with a call to createElement, passing the tag name, the props and the children as parameters.

```js
const element = <h1 title="foo">Hello</h1>;
```

在第一行我们有了 JSX 定义的元素。它甚至不是有效的 JavaScript，因此，为了用原始 JavaScript 替代它，首先我们需要用有效的 JS 替代它。
JSX 通过 Babel 这样的构建工具被转换成 JS。转换通常很简单：调用 createElement 来替换标签内的代码，传递名称，props 和 children 作为参数。

> React.createElement creates an object from its arguments. Besides some validations, that’s all it does. So we can safely replace the function call with its output.

```js
const element = React.createElement("h1", { title: "foo" }, "Hello");
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
};
```

这就是一个元素，一个具有两个属性的元素：type 和 props（好吧，它还有[更多属性](https://github.com/facebook/react/blob/f4cc45ce962adc9f307690e1d5cfa28a288418eb/packages/react/src/ReactElement.js#L111)，但是我们只关心这两个）

type 参数是一个 string，用于我们想要创建的 DOM 节点的类型，这是当你想要创建一个 HTML 元素时传递给 document.createElement 的 tagName。它也可以是个函数，我们会在第 VII 讲到。

props 是另一个对象，它具有 JSX 属性中的所有键和值。 它还有一个特殊的属性：children。

在这种情况下，children 是一个字符串，但通常是一个包含更多元素的数组。 这就是为什么元素也是树的原因。

> The other piece of React code we need to replace is the call to ReactDOM.render.
>
> render is where React changes the DOM, so let’s do the updates ourselves.

```js
ReactDOM.render(element, container);
```

另一段我们需要替换的 React 代码是 ReactDOM.render 的调用。
渲染是 React 更改 DOM 的地方，因此我们自己进行更新。

> First we create a node\* using the element type, in this case h1.
>
> Then we assign all the element props to that node. Here it’s just the title.
>
> - To avoid confusion, I’ll use “element” to refer to React elements and “node” for DOM elements.

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
);
const container = document.getElementById("root");
ReactDOM.render(element, container);
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
> "type": "div",
> "props": { "children": [] }
> }
> createElement("div", null, a) returns:
> {
> "type": "div",
> "props": { "children": [a] }
> }
> and createElement("div", null, a, b) returns:
> {
> "type": "div",
> "props": { "children": [a, b] }
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

如果我们有这样的评论，当 babel 转译 JSX 时，它将使用我们定义的功能。

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
ReactDOM.render(element, container);
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

## 第 3 步：并发模式

> But… before we start adding more code we need a refactor.

但是，在我们开始添加更多代码前，我们需要重构。

> There’s a problem with this recursive call.
> Once we start rendering, we won’t stop until we have rendered the complete element tree. If the element tree is big, it may block the main thread for too long. And if the browser needs to do high priority stuff like handling user input or keeping an animation smooth, it will have to wait until the render finishes.

递归调用存在一个问题。
一旦我们开始渲染，我们不会停止知道我们渲染了整个元素树。如果元素树太大，可能会长时间阻塞主线程。如果浏览器需要处理高优先级任务，例如处理用户输入或者保持动画流畅，则它不得不等待到渲染结束。

> So we are going to break the work into small units, and after we finish each unit we’ll let the browser interrupt the rendering if there’s anything else that needs to be done.

因此我们将工作拆分成几个小部分，然后在我们结束每个部分后，如果有其他任务需要执行，我们会让浏览器中断渲染。

```jsx
let nextUnitOfWork = null
​
function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }
  requestIdleCallback(workLoop)
}
​
requestIdleCallback(workLoop)
​
function performUnitOfWork(nextUnitOfWork) {
  // TODO
}
```

> We use requestIdleCallback to make a loop. You can think of requestIdleCallback as a setTimeout, but instead of us telling it when to run, the browser will run the callback when the main thread is idle.
> React doesn’t use requestIdleCallback anymore. Now it uses the scheduler package. But for this use case it’s conceptually the same.

我们使用 `requestIdleCallback` 进行循环。你可以把 `requestIdleCallback` 视为 `setTimeout`，但是区别于我们告诉它什么时候运行，浏览器会运行回调当主线程空闲时。
React 不再使用 `requestIdleCallback`。现在它使用调度包。但是对于这个用例，它在概念上是相同的。

> requestIdleCallback also gives us a deadline parameter. We can use it to check how much time we have until the browser needs to take control again.
> As of November 2019, Concurrent Mode isn’t stable in React yet. The stable version of the loop looks more like this:

> ```
> while (nextUnitOfWork) {
>    nextUnitOfWork = performUnitOfWork(
>    nextUnitOfWork
>  )
> }
> ```

`requestIdleCallback` 同样给我们一个截止日期参数。我们可以使用它来检查浏览器再次控制还有多长时间。
截至 2019 年 11 月，并发模式在 React 中还不稳定。 循环的稳定版本看起来像这样：

```jsx
while (nextUnitOfWork) {
  nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
}
```

> To start using the loop we’ll need to set the first unit of work, and then write a performUnitOfWork function that not only performs the work but also returns the next unit of work.

为了开始使用循环首先我们需要设置第一个工作单元，之后编写 `performUnitOfWork` 函数，该函数不仅执行工作，还返回下一个工作单元。

## 第 4 步：Fiber

> To organize the units of work we’ll need a data structure: a fiber tree.
> We’ll have one fiber for each element and each fiber will be a unit of work.
> Let me show you with an example.
> Suppose we want to render an element tree like this one:

为了组织工作单元我们会需要一个数据结构：fiber 树。
我们每个元素会有一个 fiber 并且每个 fiber 会成为一个工作单元。
让我们看一下示例。

![image](https://user-images.githubusercontent.com/32665965/118453832-3e7fbb80-b72a-11eb-9e9d-6c0999c19397.png)

假设我们需要渲染像下面的一个元素树：

```jsx
Didact.render(
  <div>
    <h1>
      <p />
      <a />
    </h1>
    <h2 />
  </div>,
  container
);
```

> In the render we’ll create the root fiber and set it as the nextUnitOfWork. The rest of the work will happen on the performUnitOfWork function, there we will do three things for each fiber:
>
> 1. add the element to the DOM
> 2. create the fibers for the element’s children
> 3. select the next unit of work

渲染中我们会创建根 fiber 并且设置它作为 `nextUnitOfWork`。剩余的工作将会在 `performUnitOfWork` 中进行，我们将会为每个 fiber 做三件事：

1. 向 DOM 添加元素
2. 为元素的子节点创建 fiber
3. 选择下一个工作单元。

> One of the goals of this data structure is to make it easy to find the next unit of work. That’s why each fiber has a link to its first child, its next sibling and its parent.

这个数据结构的目标之一就是能够更好的找到下一个工作单元。这就是为什么每个 fiber 都要连接链接第一个子节点，下一个兄弟节点和父节点。

![image](https://user-images.githubusercontent.com/32665965/118455098-91a63e00-b72b-11eb-8f36-55ed303aa9e5.png)

> When we finish performing work on a fiber, if it has a child that fiber will be the next unit of work.
> From our example, when we finish working on the div fiber the next unit of work will be the h1 fiber.

当我们完成对 fiber 的工作，如果它有子节点，那么 fiber 将会是下一个工作单元。
从我们对示例中，当我们结束 div fiber 工作，一个工作单元将会是 h1 fiber。

> If the fiber doesn’t have a child, we use the sibling as the next unit of work.
> For example, the p fiber doesn’t have a child so we move to the a fiber after finishing it.

如果 fiber 没有子节点，我们会使用兄弟节点作为下一个工作单元。
例如，p fiber 没有子节点因此当结束时我们移动至 a fiber。

> And if the fiber doesn’t have a child nor a sibling we go to the “uncle”: the sibling of the parent. Like a and h2 fibers from the example.
> Also, if the parent doesn’t have a sibling, we keep going up through the parents until we find one with a sibling or until we reach the root. If we have reached the root, it means we have finished performing all the work for this render.

并且如果 fiber 没有子节点或者兄弟节点，我们会返回至“叔叔”：兄弟节点的父节点。就像示例中的 a 和 h2 fiber。
另外，如果父节点没有兄弟，我们会不断检查父节点直到找到有兄弟节点的父节点或者根节点。如果达到根节点，意味着我们以及完成了渲染的全部工作。

> Now let’s put it into code.

现在，将其放入代码中。

> First, let’s remove this code from the render function.

首先，让我们从 render 函数中移动代码。

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
​
let nextUnitOfWork = null
```

> We keep the part that creates a DOM node in its own function, we are going to use it later.

我们将创建 DOM 节点的部分保留在其自身的功能中，稍后将使用它。

```jsx
function createDom(fiber) {
  const dom =
    fiber.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(fiber.type)
​
  const isProperty = key => key !== "children"
  Object.keys(fiber.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = fiber.props[name]
    })
​
  return dom
}
​
function render(element, container) {
  // TODO set next unit of work
}
​
let nextUnitOfWork = null
```

> In the render function we set nextUnitOfWork to the root of the fiber tree.

在 render 函数中我们设置 `nextUnitOfWork` fiber 树的根。

```jsx
function render(element, container) {
  nextUnitOfWork = {
    dom: container,
    props: {
      children: [element],
    },
  }
}
​
let nextUnitOfWork = null
```

> Then, when the browser is ready,it will call our workLoop and we’ll start working on the root.

之后，当浏览器准备好，它会运行我们的 `workLoop`，我们将在根目录上开始工作。

```jsx
function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }
  requestIdleCallback(workLoop)
}
​
requestIdleCallback(workLoop)
​
function performUnitOfWork(fiber) {
  // TODO add dom node
  // TODO create new fibers
  // TODO return next unit of work
}
```

> First, we create a new node and append it to the DOM.
> We keep track of the DOM node in the fiber.dom property.

首先，我们创建一个新节点并且添加到 DOM 中。
我们在 fiber.dom 属性中跟踪 DOM 节点。

```jsx
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
​
  // TODO create new fibers
  // TODO return next unit of work
}
```

> Then for each child we create a new fiber.

之后，每一个子节点我们创建一个新的 fiber 节点。

```jsx
function performUnitOfWork(fiber) {
  ...

  const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​
  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
  }
​
  // TODO return next unit of work
}
```

> And we add it to the fiber tree setting it either as a child or as a sibling, depending on whether it’s the first child or not.

我们添加它到 fiber 树，设置它为子节点或者兄弟节点，依赖于它是不是第一个子节点。

```jsx
function performUnitOfWork(fiber) {
  ...

  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }

     if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }
​
    prevSibling = newFiber
    index++
  }
​
  // TODO return next unit of work
}
```

> Finally we search for the next unit of work. We first try with the child, then with the sibling, then with the uncle, and so on.

最后我们搜索下一个工作单元。我们首先搜索子节点，然后是兄弟节点，然后是叔叔，以此类推...

```jsx
function performUnitOfWork(fiber) {
  ...

  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }

     if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }
​
    prevSibling = newFiber
    index++
  }
​
  if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}
```

> And that’s our performUnitOfWork.

这就是我们的 `performUnitOfWork`。

```jsx
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
​
  const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​
  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
​
    if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }
​
    prevSibling = newFiber
    index++
  }
​
  if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}
```

## 第 5 步：渲染和提交阶段

> We have another problem here.
> We are adding a new node to the DOM each time we work on an element. And, remember, the browser could interrupt our work before we finish rendering the whole tree. In that case, the user will see an incomplete UI. And we don’t want that.

我们有另外一个问题。
每次处理元素时，我们都要添加一个新节点到 DOM 中，浏览器能在我们渲染整个树前打断我们的工作。在这种情况下，用户会看见不完整的 UI。但这不是我们想要的。

```jsx
function performUnitOfWork(fiber) {
  ...
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
​
  ...
}
```

> So we need to remove the part that mutates the DOM from here.

因此我们需要移除这里转变 DOM 的部分。

```jsx
function performUnitOfWork(fiber) {
  ...
​
  // if (fiber.parent) {
  //  fiber.parent.dom.appendChild(fiber.dom)
  // }
​
  ...
}
```

> Instead, we’ll keep track of the root of the fiber tree. We call it the work in progress root or wipRoot.

相反，我们将跟踪 fiber 树的根。 我们称为进行中的工作根或 `wipRoot`。

```
function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
  }
  nextUnitOfWork = wipRoot
}
​
let nextUnitOfWork = null
let wipRoot = null
```

> And once we finish all the work (we know it because there isn’t a next unit of work) we commit the whole fiber tree to the DOM.

一旦我们结束所有工作（我们知道工作结束是因为没有下一个工作单元），我们向 DOM 提交整个 fiber 树。

```jsx
function commitRoot() {
  // TODO add nodes to dom
}
​
...

let nextUnitOfWork = null
let wipRoot = null
​
function workLoop(deadline) {
  ...
​
  if (!nextUnitOfWork && wipRoot) {
    commitRoot()
  }
​
  requestIdleCallback(workLoop)
}
```

> We do it in the commitRoot function. Here we recursively append all the nodes to the dom.

我们在 commitRoot 函数中处理。在这里，我们将所有节点递归加入 dom 中。

```jsx
function commitRoot() {
  commitWork(wipRoot.child)
  wipRoot = null
}
​
function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  domParent.appendChild(fiber.dom)
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

## 第 6 步：协调(Reconciliation)

> So far we only added stuff to the DOM, but what about updating or deleting nodes?
> That’s what we are going to do now, we need to compare the elements we receive on the `render` function to the last fiber tree we committed to the DOM.

目前为止我们仅仅向 DOM 中添加了东西，但是更新和删除节点应该怎么办呢？
这是我们现在需要解决的，我们需要比较在 `render` 函数中获得的元素和上一次向 DOM 提交的 fiber 树。

> So we need to save a reference to that “last fiber tree we committed to the DOM” after we finish the commit. We call it currentRoot.
> We also add the alternate property to every fiber. This property is a link to the old fiber, the fiber that we committed to the DOM in the previous commit phase.

因此在我们结束提交后我们需要减少和 “上一次向 DOM 提交的 fiber 树” 联系。我们称它为 `currentRoot`。
我们还添加 `alternate` 属性到每个 fiber 节点。这个属性用来和我们在上一个提交阶段向 DOM 提交的 fiber 产生连接。

```jsx
function commitRoot() {
  commitWork(wipRoot.child)
  currentRoot = wipRoot
  wipRoot = null
}
​
...
​
function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
    alternate: currentRoot,
  }
  nextUnitOfWork = wipRoot
}
​
let nextUnitOfWork = null
let currentRoot = null
let wipRoot = null
```

> Now let’s extract the code from performUnitOfWork that creates the new fibers…

现在，让我们从 `performUnitOfWork` 中提取代码创建新的 fiber...

```jsx
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​
  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
​
    if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }
​
    prevSibling = newFiber
    index++
  }
​
  if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}
```

> …to a new reconcileChildren function.

新的 `reconcileChildren` 函数。

```jsx

function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  const elements = fiber.props.children
  reconcileChildren(fiber, elements)
​
  ...

  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}
​
function reconcileChildren(wipFiber, elements) {
  let index = 0
  let prevSibling = null
```

> Here we will reconcile the old fibers with the new elements.

现在我们会将旧 fiber 和新元素进行协调。

> We iterate at the same time over the children of the old fiber (wipFiber.alternate) and the array of elements we want to reconcile.
> If we ignore all the boilerplate needed to iterate over an array and a linked list at the same time, we are left with what matters most inside this while: oldFiber and element. The element is the thing we want to render to the DOM and the oldFiber is what we rendered the last time.
> We need to compare them to see if there’s any change we need to apply to the DOM.

我们同时遍历旧 fiber 的子级（`wipFiber.alternate`）和要协调的元素数组。
如果我们忽略同时遍历数组和链接列表的所有样板，那么在此期间剩下的最重要的是：oldFiber 和 element。元素是我们想要渲染到 DOM 中的东西，oldFiber 是我们上次渲染的东西。
我们需要比较他们然后查看是否有我们需要对 DOM 进行任何更改。

```jsx
function reconcileChildren(wipFiber, elements) {
  let index = 0
  let oldFiber =
    wipFiber.alternate && wipFiber.alternate.child
  let prevSibling = null
​
  while (
    index < elements.length ||
    oldFiber != null
  ) {
    const element = elements[index]
    let newFiber = null
​
    // TODO compare oldFiber to element
​
    if (oldFiber) {
      oldFiber = oldFiber.sibling
    }
```

> To compare them we use the type:
> - if the old fiber and the new element have the same type, we can keep the DOM node and just update it with the new props
> - if the type is different and there is a new element, it means we need to create a new DOM node
> - and if the types are different and there is an old fiber, we need to remove the old node
>
> Here React also uses keys, that makes a better reconciliation. For example, it detects when children change places in the element array.

为了比较他们我们使用类型：
- 如果旧 fiber 和新元素有一样对类型，我们可以保持 DOM 节点并且仅仅更新新的 props。
- 如果类型和新元素不一样，意味着我们需要创建一个新的 DOM 节点。
- 如果类型不一样并且有旧 fiber，我们需需要移除旧的节点。

这里 React 也是使用的 keys 来更好的协调。例如，它检测子元素何时更改元素数组中的位置。

```jsx
  const element = elements[index]
  let newFiber = null
​
  const sameType =
    oldFiber &&
    element &&
    element.type == oldFiber.type
​
  if (sameType) {
    // TODO update the node
  }
  if (element && !sameType) {



    // TODO add this node
  }
  if (oldFiber && !sameType) {
    // TODO delete the oldFiber's node
  }
​
  if (oldFiber) {
    oldFiber = oldFiber.sibling
  }
```

> When the old fiber and the element have the same type, we create a new fiber keeping the DOM node from the old fiber and the props from the element.
> We also add a new property to the fiber: the effectTag. We’ll use this property later, during the commit phase.

当旧 fiber 和元素有一样当类型时，我们创建一个新的 fiber，以使 DOM 节点与旧 fiber 和元素的 props 保持一致。
我们同样给 fiber 添加一个属性：`effectTag`，我们会在稍后提交阶段使用这个属性。

```jsx
  const sameType =
    oldFiber &&
    element &&
    element.type == oldFiber.type
​
  if (sameType) {
    newFiber = {
      type: oldFiber.type,
      props: element.props,
      dom: oldFiber.dom,
      parent: wipFiber,
      alternate: oldFiber,
      effectTag: "UPDATE",
    }
  }
  if (element && !sameType) {
    // TODO add this node
  }
```

> Then for the case where the element needs a new DOM node we tag the new fiber with the PLACEMENT effect tag.

然后，对于元素需要新 DOM 节点的情况，我们使用 `PLACEMENT`效果标签来标记新的 fiber。

```jsx
if (element && !sameType) {
  newFiber = {
    type: element.type,
    props: element.props,
    dom: null,
    parent: wipFiber,
    alternate: null,
    effectTag: "PLACEMENT",
  }
}
```

> And for the case where we need to delete the node, we don’t have a new fiber so we add the effect tag to the old fiber.
> But when we commit the fiber tree to the DOM we do it from the work in progress root, which doesn’t have the old fibers.

对于需要删除节点的情况，我们没有新 fiber 所以我们向旧 fiber 添加效果标签。
但是当我们向 DOM 提交 fiber 树，我们是从正在进行的根目录开始的，他没有旧 fiber。

```jsx
  if (oldFiber && !sameType) {
    oldFiber.effectTag = "DELETION"
    deletions.push(oldFiber)
  }
```

> So we need an array to keep track of the nodes we want to remove.

所以我们需要一个数组追踪我们想要移除的节点。

```jsx
function render(element, container) {
  ...
  deletions = []
  nextUnitOfWork = wipRoot
}
​
let nextUnitOfWork = null
let currentRoot = null
let wipRoot = null
let deletions = null
```

> And then, when we are commiting the changes to the DOM, we also use the fibers from that array.

之后，当我们向 DOM 提交更改，我们也要使用数组中的 fiber。

```jsx
function commitRoot() {
  deletions.forEach(commitWork)
  commitWork(wipRoot.child)
  currentRoot = wipRoot
  wipRoot = null
}
```

> Now, let’s change the commitWork function to handle the new effectTags.

现在，我们更改 `commitWork` 函数来处理 `effectTags`。

```jsx
function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  domParent.appendChild(fiber.dom)
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

> If the fiber has a PLACEMENT effect tag we do the same as before, append the DOM node to the node from the parent fiber.

如果 fiber 有 `PLACEMENT` 效果标签我们和之前一样操作，添加 DOM 节点到父 fiber 节点上。

```jsx
function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  if (
    fiber.effectTag === "PLACEMENT" &&
    fiber.dom != null
  ) {
    domParent.appendChild(fiber.dom)
  }
​
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

> If it’s a DELETION, we do the opposite, remove the child.

如果是 `DELETION`，我们做相反的事情，删除子节点。

```jsx
function commitWork(fiber) {
  ...
  if (
    fiber.effectTag === "PLACEMENT" &&
    fiber.dom != null
  ) {
    domParent.appendChild(fiber.dom)
  } else if (fiber.effectTag === "DELETION") {
    domParent.removeChild(fiber.dom)
  }
​
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

> And if it’s an UPDATE, we need to update the existing DOM node with the props that changed.

如果是 `UPDATE`，我们需要更改 props 来更新已经存在的 DOM 节点。

```jsx
function commitWork(fiber) {
  ...
  if (
    fiber.effectTag === "PLACEMENT" &&
    fiber.dom != null
  ) {
    domParent.appendChild(fiber.dom)
  } else if (
    fiber.effectTag === "UPDATE" &&
    fiber.dom != null
  ) {
    updateDom(
      fiber.dom,
      fiber.alternate.props,
      fiber.props
    )
  } else if (fiber.effectTag === "DELETION") {
    domParent.removeChild(fiber.dom)
  }
​
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

 > We’ll do it in this updateDom function.

 我们将在此 `updateDom` 函数中进行操作。

 ```jsx
function updateDom(dom, prevProps, nextProps) {
  // TODO
}
```

> We compare the props from the old fiber to the props of the new fiber, remove the props that are gone, and set the props that are new or changed.

我们比较旧 fiber 节点和新 fiber 节点的 props，删除消失的 props，设置新的或者更改 props。

```jsx
const isProperty = key => key !== "children"
const isNew = (prev, next) => key =>
  prev[key] !== next[key]
const isGone = (prev, next) => key => !(key in next)
function updateDom(dom, prevProps, nextProps) {
  // Remove old properties
  Object.keys(prevProps)
    .filter(isProperty)
    .filter(isGone(prevProps, nextProps))
    .forEach(name => {
      dom[name] = ""
    })
​
  // Set new or changed properties
  Object.keys(nextProps)
    .filter(isProperty)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      dom[name] = nextProps[name]
    })
}
```

> One special kind of prop that we need to update are event listeners, so if the prop name starts with the “on” prefix we’ll handle them differently.

我们需要更新中有一个特别的 prop 是时间监听器，因此如果 prop 名字以 “on” 为前缀我们会用不同方式处理。

```jsx
const isEvent = key => key.startsWith("on")
const isProperty = key =>
  key !== "children" && !isEvent(key)
```

> If the event handler changed we remove it from the node.

如果事件处理改变了我们从节点中将它删除。

```jsx
function updateDom(dom, prevProps, nextProps) {
  //Remove old or changed event listeners
  Object.keys(prevProps)
    .filter(isEvent)
    .filter(
      key =>
        !(key in nextProps) ||
        isNew(prevProps, nextProps)(key)
    )
    .forEach(name => {
      const eventType = name
        .toLowerCase()
        .substring(2)
      dom.removeEventListener(
        eventType,
        prevProps[name]
      )
    })
​
  // Remove old properties
```

> And then we add the new handler.

然后，我们添加新的处理程序。

```jsx
  .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      dom[name] = nextProps[name]
    })
​
  // Add event listeners
  Object.keys(nextProps)
    .filter(isEvent)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      const eventType = name
        .toLowerCase()
        .substring(2)
      dom.addEventListener(
        eventType,
        nextProps[name]
      )
    })
}
```

> Try the version with reconciliation on codesandbox.

在 [codesandbox](https://codesandbox.io/s/didact-6-96533) 上尝试
