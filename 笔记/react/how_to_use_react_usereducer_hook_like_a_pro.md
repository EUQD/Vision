---
title: 【译】如何像专家一样使用 React Hook：useReducer
author: 作者 devtrium.com ，译者 Ucely 刘俊
---

> 原文链接：<https://devtrium.com/posts/how-to-use-react-usereducer-hook>
> 
> 【版权信息：著作权归 devtrium.com 所有，如需转载本文，请注明作者 [devtrium.com](https://devtrium.com) 和译者 [Ucely 刘俊](https://Ucely.com)。】

## 简介

React 的管理状态是你在开发 React 网站时要面临的主要问题之一。毫无疑问，`useState` 是在函数式 React 组件中，创建和管理状态的最常见方式。

也有很多库提供了用各自的方式来管理你的整个（或部分）状态，比如 [Redux](https://redux.js.org/)、[Mobx](https://mobx.js.org/README.html)、[Recoil](https://recoiljs.org/) 或 [XState](https://xstate.js.org/docs/)。

但在考虑使用一个库来帮你管理状态之前，你应该知道另一种在 React 中管理状态的原生方法：`useReducer`。当以正确的方式和目的使用时，它可以是非常强大的。事实上，它是如此强大，以至于著名的 Redux 库可以被认为是一个大的、优化的 `useReducer`（文末我们会看到的）。

在这篇文章中，我们将首先解释什么是 `useReducer` 以及如何使用它，给你一个好的心智模型和例子。然后，我们将进行 `useState` 与 `useReducer` 的比较，以了解何时使用哪种方法。

对于 TypeScript 用户来说，我们还将看到如何将 TypeScript 和 `useReducer` 一起使用。

让我们开始行动吧!

## 什么是 React hook useReducer 以及如何使用它

正如简介中提到的，`useState` 和 useReducer 是 React 管理状态的两种本地方式。你可能已经对前者相当熟悉了，所以从那里开始理解 `useReducer` 是很有帮助的。

### useState 和 useReducer：一个快速的比较

乍一看，它们非常相似。让我们来把它们并排做一个对比。

```js
const [state, setState] = useState(initialValue);

const [state, dispatch] = useReducer(reducer, initialValue);
```

正如你所看到的，在这两种情况下， `hook` 钩子函数都返回一个有两个元素的数组。第一个是状态，第二个是让你修改状态的函数：`setState` 用于 `useState` ，`dispatch` 用于 `useReducer`。我们将在后面了解调度器 `dispatch` 是如何工作的。

一个初始状态都分别被提供给 `useState` 和 `useReducer` 。 钩子函数参数的主要区别是提供给 useReducer 的还原器 `reducer` 。

这个 `reducer` 是一个函数，它将处理状态应该如何更新的逻辑。我们也会在文章后面详细了解它。

现在我们来看看如何使用 `setState` 或 `dispatch` 来改变状态。为此，我们将使用一个久经考验的计数器的例子--我们想在按钮被点击时将其递增 1 ：

```js
// 用 `useState`
<button onClick={() => setCount(prevCount => prevCount + 1)}>
  +
</button>

// 用 `useReducer`
<button onClick={() => dispatch({type: 'increment', payload: 1})}>
  +
</button>
```

虽然 `useState` 版本可能对你来说很熟悉（如果不是，可能是因为我们使用的是 `setState` 的函数式更新形式），但 `useReducer` 版本可能看起来有点奇怪。

为什么我们要传递一个带有类型 `type` 和有效载荷 `payload` 属性的对象呢？那个（神奇的）值 `increment` 是从哪里来的？别着急，这些谜团会被一一解释的。

现在，你可以注意到，这两个版本还是很相似的。在任何一种情况下，你都要通过调用更新函数（ `setState` 或 `dispatch` ）来更新状态，并说明你到底要如何更新状态。

现在让我们在高层次上探讨一下 `useReducer` 版本到底是如何工作的。

### useReducer：一个后端的心智模型

在这一节中，我想让你对 `useReducer` 钩子函数 hook 是如何工作的，建立一个良好的心智模型。这一点很重要，因为当我们深陷于实现真正问题的细节之中时，事情可能会变得有点令人不知所措。特别是如果你以前从未接触过类似的结构的时候。

思考 `useReducer` 的一种方式是把它看作是一个后端。这听起来可能有点奇怪，但请容忍我一下 >_<。我对这个比喻非常满意，我认为它很好地解释了什么是 `reducer` （规约函数/简化函数）。

后台的结构通常是用某种方式来保存数据（数据库），还有一个让你修改数据库的API。

该 API 会提供 HTTP 接口让你调用。GET 请求让你访问数据，而 POST 请求让你修改数据。当你做 POST 请求时，你或许可以传入一些参数；例如，如果你想创建一个新的用户，你通常会在 HTTP POST 请求中包括新用户的用户名、电子邮件和密码。

那么，useReducer与后端有什么相似之处？请看：

- 状态 `state` 相当于是数据库。它存储着你的数据；
- `dispatch` 相当于调用 API 接口来修改数据库；
  - 你可以通过指定调用的类型 `type` 来选择调用哪个接口；
  - 你可以用 `payload` 属性提供额外的数据，它对应于 POST 请求的请求体 `body`；
  - `type` 和 `payload` 都是一个对象的属性，这个对象被交给了 `reducer`。这个对象被称为 `action`；
- `reducer` 函数是 API 的逻辑处理函数。当后端收到一个 API 调用（`dispatch` 调用）时，它会被执行，并处理如何基于接口和请求内容（即动作 `action`）来更新数据库。

下面是一个完整的 `useReducer` 使用例子。花点时间来接受它，并将它与上面描述的后端心智模型进行比较。

```jsx
import { useReducer } from 'react';

// 数据库的初始状态
const initialState = { count: 0 };

// API 逻辑: 当 API 接口 'increment' 被调用时，如何更新数据库
const reducer = (state, action) => {
  if (action.type === 'increment') {
    return { count: state.count + action.payload };
  }
};

function App() {
  // 你可以把这步当作是初始化，并于后端建立连接
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      {/* 从数据库读取数据 */}
      Count: {state.count}
      {/* 在按钮被点击的时候调用 API */}
      <button onClick={() => dispatch({ type: 'increment', payload: 1 })}>
        +
      </button>
    </div>
  );
}

export default App;
```

你能看出这两者有什么关系吗？

记住，上面的代码不应该在生产中使用。这是一个最小版本的useReducer钩子，以帮助你将其与后端心智模型进行比较，但它缺少几个重要的东西，你将在这篇文章中了解到。

现在，（希望）你对useReducer是如何在高层次上工作的有了一个很好的概念，让我们进一步探索细节。

### Reducer 是如何工作的

我们将首先解决 `reducer` 的问题，因为它是主要逻辑发生的地方。

正如你在上面的例子中注意到的，`reducer` 是一个需要两个参数的函数。第一个是当前状态 `state` ，第二个是动作 `action`（在我们的后端类比中，它对应于API接口 + 任何请求的主体）。

请记住，你不需要自己提供参数给 `reducer` 。这由 `useReducer` 钩子自动处理：状态是已知的，而动作只是 `dispatch` 的参数，它被作为第二个参数传递给 `reducer` 。

状态有你想要的任何格式（通常是一个对象，但它真的可以是任何东西）。动作也可以是你想要的任何东西，但有一些非常常用的约定，即如何构造它，我建议你遵循这些约定——我们将在后面学习它们。至少是在你熟悉了这些惯例、并确定不遵守这些惯例是你真正想要的之前。

所以按照惯例，动作 `action` 是一个对象，有一个必需的属性和一个可选的属性：

- `type` 是必需的属性（类似于 API 的接口名称）。它告诉 `reducer` 应该使用哪块逻辑来修改状态。
- `payload` 是可选属性（类似于HTTP POST请求的主体，如果有的话）。它向 `reducer` 提供了关于如何修改状态的额外信息。

在我们之前的计数器的例子中，状态是一个具有单一计数 `count` 属性的对象。动作 `action` 是一个对象，其类型 `type` 可以是 “增加” `increment` ，其有效载荷 `payload` 是你想要给计数器的加的数。

```js
// 这个一个 `state` 状态的例子
const state = { count: 0 };

// 这是一个 `action` 动作的例子
const action = { type: 'increment', payload: 2 };
```

例如，`reducer` 函数结构中通常有一个关于 `action` 类型的 `switch` 语句。

```js
const reducer = (state, action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + action.payload };
    case 'decrement':
      return { count: state.count - action.payload };
    case 'reset':
      return { count: 0 };
  }
};
```

在这个例子中，`reducer` 接受三种动作类型。增加 `increment`、减少 `decrement` 和 重置 `reset`。增加和减少都需要一个`action` 的 `payload` ，它将决定计数器增加或减少的数量。相反，`reset` 类型不需要任何有效载荷，因为它把计数器重置为0。

这是一个非常简单的例子，而现实生活中的还原器通常要大得多，也复杂得多。我们将在后面的章节中看到如何改进我们编写 `reducer` 的方法，以及现实生活中的应用程序中的 `reducer` 的例子。

### dispatch 是如何工作的？

如果你已经理解了 `reducer` 的工作原理，那么理解 `dispatch` （调度函数）就非常简单了。

当你调用 `dispatch` 时，无论给它的参数是什么，都将是给你的 `reducer` 函数的第二个参数 `action` 。按照惯例，该参数是一个具有类型 `type` 和 `payload` （可选）的对象 Object ，正如我们在上一节看到的那样。

使用我们上一个 `reducer` 的例子，如果我们想做一个按钮，在点击时将计数器减少2，它将看起来像这样：

```js
<button onClick={() => dispatch({ type: 'decrement', payload: 2 })}>-</button>
```

如果我们想有一个将计数器重置为0的按钮，仍然使用我们上一个例子，你可以省略 `payload`：

```jsx
<button onClick={() => dispatch({ type: 'reset' })}>reset</button>
```

关于 `dispatch` ，需要注意的一件事是 React 保证它的值和内存指向，不会在渲染的过程中发生改变。这意味着你不需要把它放在依赖性数组中（如果你这样做，它永远不会触发依赖性数组）。这与 `useState` 的 `setState` 函数的行为相同。

::: tip
如果你对最后一段话有点模糊，我已经用这篇[关于依赖数组的文章](https://devtrium.com/posts/dependency-arrays)为你做了介绍。
:::

### useReducer 初始状态

到目前为止，我们还没有经常提到它，但是 `useReducer` 也需要一个第二个参数，这是你想给状态的初始值。

它本身不是一个必须的参数，但是如果你不提供它，状态一开始就会是 `undefined` ，这很少是你想要的。

你通常在初始状态中定义还原器状态的完整结构。它通常是一个对象，你不应该在你的还原器中为该对象添加新的属性。

在我们的计数器例子中，初始状态是简单的。

```js
// 数据库的初始状态
const initialState = { count: 0 };
· · ·

// 在组件内使用
const [state, dispatch] = useReducer(reducer, initialState);
```

我们将进一步看到更多这样的例子。

### useReducer的技巧和窍门

我们有几种方法可以改进我们对 `useReducer` 的使用。其中一些是你真正应该做的事情，其他的则更多地是个人品味的问题。

我把它们从重要的到可选的进行了粗略的分类，从最重要的开始。

#### 对于未知的动作类型，还原器应该抛出一个错误

在我们的计数器例子中，我们有一个有三种情况的 `switch` 语句。增加、减少和重置。如果你真的把这个写进了你的代码编辑器，你可能会发现 ESLint 会对你很生气 >_--_< 。

::: tip
你有 [ESLint](https://eslint.org/) 吧？如果你没有，你真的应该把它设置好!
:::

ESLint（正确地）希望switch语句有一个默认的案例。那么，当reducer在处理一个未知的动作类型时，它的默认情况应该是什么？

有些人喜欢简单地返回状态。

```js
const reducer = (state, action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + action.payload };
    case 'decrement':
      return { count: state.count - action.payload };
    case 'reset':
      return { count: 0 };
    default:
      return state;
  }
};
```

但我真的不喜欢这样。要么 `action` 类型是你所期望的，并且应该有一个案例，要么不是，那么返回状态也不是你想要的。这基本上是在提供不正确的动作类型时产生一个[“安静的错误”](https://en.wikipedia.org/wiki/Error_hiding)，而“安静的错误”是很难调试的。

相反，你的默认 `reducer` 案例应该是抛出一个错误。

```js
const reducer = (state, action) => {
  switch (action.type) {
    case 'increment':
      return { count: state.count + action.payload };
    case 'decrement':
      return { count: state.count - action.payload };
    case 'reset':
      return { count: 0 };
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
};

```

这样，你就不会漏掉一个错字或忘记一个案例。

#### 你应该在每个动作中传播状态

到目前为止，我们只看到了一个非常简单的 `useReducer` 例子，其中的状态是一个只有一个属性的对象。但通常情况下，`useReducer` 用例要求状态对象至少要有几个属性。

一个常见的 `useReducer` 用法是处理表单。这里是一个有两个输入字段的例子，但你可以想象有更多字段的情况。

(注意！下面的代码有一个错误，你能发现吗？)

```js
import { useReducer } from 'react';

const initialValue = {
  username: '',
  email: '',
};

const reducer = (state, action) => {
  switch (action.type) {
    case 'username':
      return { username: action.payload };
    case 'email':
      return { email: action.payload };
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
};

const Form = () => {
  const [state, dispatch] = useReducer(reducer, initialValue);
  return (
    <div>
      <input
        type="text"
        value={state.username}
        onChange={(event) =>
          dispatch({ type: 'username', payload: event.target.value })
        }
      />
      <input
        type="email"
        value={state.email}
        onChange={(event) =>
          dispatch({ type: 'email', payload: event.target.value })
        }
      />
    </div>
  );
};

export default Form;
```

问题出在 `reducer` 上：更新用户名 `username` 会完全覆盖之前的状态并删除电子邮件 `email`（而更新电子邮件 `email` 也会对用户名 `username` 做同样的处理）。

解决这个问题的方法是，每次更新一个属性时，要记得保留所有以前的状态。这可以通过[传播语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)轻松实现。

```jsx
import { useReducer } from 'react';

const initialValue = {
  username: '',
  email: '',
};

const reducer = (state, action) => {
  switch (action.type) {
    case 'username':
      return { ...state, username: action.payload };
    case 'email':
      return { ...state, email: action.payload };
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
};

const Form = () => {
  const [state, dispatch] = useReducer(reducer, initialValue);
  return (
    <div>
      <input
        value={state.username}
        onChange={(event) =>
          dispatch({ type: 'username', payload: event.target.value })
        }
      />
      <input
        value={state.email}
        onChange={(event) =>
          dispatch({ type: 'email', payload: event.target.value })
        }
      />
    </div>
  );
};

export default Form;
```

这个例子实际上还可以进一步优化。你可能已经注意到，我们在 `reducer` 中有些重复：用户名 `username` 和电子邮件 `email` 的情况基本上都是同样的逻辑。这对于两个字段来说还不算太坏，但我们如果有更多的字段，不加以优化，代码就会变得庞大而冗余。

有一种方法可以重构代码，让所有的输入都只有一个动作，使用 [`ES2015` 的计算键 `key` 的特性](https://stackoverflow.com/a/19837961/3352975)。

```jsx
import { useReducer } from 'react';

const initialValue = {
  username: '',
  email: '',
};

const reducer = (state, action) => {
  switch (action.type) {
    case 'textInput':
      return {
        ...state,
        [action.payload.key]: action.payload.value,
      };
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
};

const Form = () => {
  const [state, dispatch] = useReducer(reducer, initialValue);
  return (
    <div>
      <input
        value={state.username}
        onChange={(event) =>
          dispatch({
            type: 'textInput',
            payload: { key: 'username', value: event.target.value },
          })
        }
      />
      <input
        value={state.email}
        onChange={(event) =>
          dispatch({
            type: 'textInput',
            payload: { key: 'email', value: event.target.value },
          })
        }
      />
    </div>
  );
};

export default Form;
```

如你所见，我们现在只剩下一个动作类型：`textInput` 。动作的有效载荷 `payload` 也发生了变化——它变成了一个有键（要更新的属性）和值（更新键的值）的对象。

要我说，这种方案很不错 QaQ。

::: tip
你可能会注意到，在这段代码中，我们还有一个地方在重复自己：`onChange` 事件处理程序。唯一改变的是 `payload.key`。
事实上，你可以进一步将其提取为一个可重复使用的动作，你只需要提供 `key` 。
我倾向于只在还原器开始变得非常大的时候，或者在非常相似的动作经常被重复的时候，才使用可重用的动作。
但这是一个非常常见的模式，我们将在文章后面展示一个例子。
:::

#### 坚持传统的动作 `action` 结构

我说的传统的 `action` 结构是指我们在本文中一直使用的结构：动作应该是一个对象，形式上看有一个必需的 `type` 和一个可选的 `payload` 。

这是 `Redux` 的动作结构方式，也是最常用的。它是久经考验的，对你所有的 useReducers 来说是一个非常好的默认选择。

这种结构的主要缺点是，它有时会有点冗长。但除非你对useReducer非常熟悉，否则我建议你坚持使用Redux的方式。

### 语法糖🍬：从动作中解构类型和有效载荷

这是一个提高代码体验的问题。与其在你的 `reducer` 中到处重复 `action.payload`（以及潜在的 `action.type` ），你可以直接解构 `reducer` 的第二个参数，像这样：

```js
const reducer = (state, { type, payload }) => {
  switch (type) {
    case 'increment':
      return { count: state.count + payload };
    case 'decrement':
      return { count: state.count - payload };
    case 'reset':
      return { count: 0 };
    default:
      throw new Error(`Unknown action type: ${type}`);
  }
};
```

你甚至可以更进一步，也可以对状态 `action` 进行解构。这只有在你的 `reducer` 的 状态 `state` 足够小的情况下才方便，但在这些情况下，它也是不错的。

```js
const reducer = ({ count }, { type, payload }) => {
  switch (type) {
    case 'increment':
      return { count: count + payload };
    case 'decrement':
      return { count: count - payload };
    case 'reset':
      return { count: 0 };
    default:
      throw new Error(`Unknown action type: ${type}`);
  }
};
```

技巧和窍门就到此为止!

### useReducer第三个参数：初始化懒加载

很高兴知道useReducer有一个可选的第三个参数。这个参数是一个用于懒惰地（滞后地）初始化状态的函数，如果你需要的话。

这并不经常使用，但当你真正需要它时，它可能相当有用。React文档中有[一个很好的例子](https://reactjs.org/docs/hooks-reference.html#lazy-initialization)，说明如何使用这种懒惰的初始化。

## useState vs useReducer：何时使用哪种方法

现在你知道了 `useReducer` 的工作原理以及如何在你的组件中使用它，我们需要解决一个重要问题。既然 `useState` 和`useReducer` 是两种管理状态的方式，那么你应该在什么时候选择哪一种呢？

这类问题总是一个棘手的话题，因为答案通常会根据你问的人而改变，而且它也是高度依赖于上下文的。然而，仍然有一些准则可以指导你的选择。

首先，我们知道 useState 仍然应该是你管理 React 状态的默认选择。只有当你开始在使用 useState 遇到麻烦时，才切换到useReducer（如果这个麻烦可以通过切换到 `useReducer` 来解决）。至少在你对useReducer有足够的经验，可以提前知道使用哪一个。

我将通过几个例子来说明何时使用useReducer而不是useState。

### 互相依赖的多个状态片段

`useReducer` 的一个很好的用例是当你有多个相互依赖的状态时。

这在你构建表单时很常见。比方说，你有一个文本输入，你想跟踪三件事。

1. 输入的值。
2. 输入是否已经被用户“接触”过。这对了解是否显示错误很有用。例如，如果这个字段是必填的，你想在它是空的时候显示一个错误。然而，当用户以前从未访问过该输入时，你不希望在第一次渲染时显示一个错误。
3. 是否有错误。

使用 `useState` ，你必须使用三次钩子，每次有变化都要分别更新三块状态。

使用 `useReducer` ，逻辑其实很简单。

```jsx
import { useReducer } from 'react';

const initialValue = {
  value: '',
  touched: false,
  error: null,
};

const reducer = (state, { type, payload }) => {
  switch (type) {
    case 'update':
      return {
        value: payload.value,
        touched: true,
        error: payload.error,
      };
    case 'reset':
      return initialValue;
    default:
      throw new Error(`Unknown action type: ${type}`);
  }
};

const Form = () => {
  const [state, dispatch] = useReducer(reducer, initialValue);
  console.log(state);
  return (
    <div>
      <input
        className={state.error ? 'error' : ''}
        value={state.value}
        onChange={(event) =>
          dispatch({
            type: 'update',
            payload: {
              value: event.target.value,
              error: state.touched ? event.target.value.length === 0 : null,
            },
          })
        }
      />
      <button onClick={() => dispatch({ type: 'reset' })}>reset</button>
    </div>
  );
};

export default Form;
```

添加一点基本的CSS来设计错误类，你就有了一个具有良好用户体验和简单逻辑的输入的开始，这要感谢 `useReducer` 。

```css
.error {
  border-color: red;
}

.error:focus {
  outline-color: red;
}
```

![](https://devtrium.com/_next/image?url=%2Fhow-to-use-react-usereducer-hook%2Ftouched-form.gif&w=1080&q=100)

#### 管理复杂的状态

`useReducer`的另一个很好的用例是，当你有很多不同的状态，而把它们都放在useState中会变得非常难以控制。

我们在前面看到一个例子，一个单一的 `action` 用同一个 `action` 管理两个输入。我们可以轻松地将这个例子扩展到4个输入。

当我们这样做的时候，我们也可以把每个输入的动作重构出来。

```jsx
import { useReducer } from 'react';

const initialValue = {
  firstName: '',
  lastName: '',
  username: '',
  email: '',
};

const reducer = (state, action) => {
  switch (action.type) {
    case 'update':
      return {
        ...state,
        [action.payload.key]: action.payload.value,
      };
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
};

const Form = () => {
  const [state, dispatch] = useReducer(reducer, initialValue);

  const inputAction = (event) => {
    dispatch({
      type: 'update',
      payload: { key: event.target.name, value: event.target.value },
    });
  };

  return (
    <div>
      <input
        value={state.firstName}
        type="text"
        name="firstName"
        onChange={inputAction}
      />
      <input
        value={state.lastName}
        type="text"
        name="lastName"
        onChange={inputAction}
      />
      <input
        value={state.username}
        type="text"
        onChange={inputAction}
        name="username"
      />
      <input
        value={state.email}
        type="email"
        name="email"
        onChange={inputAction}
      />
    </div>
  );
};

export default Form;
```

讲真的，这段代码是多么的干净和清晰？想象一下，用4个useState来做这件事吧! 好吧，它不会那么糟糕，但这可以扩展到你想要的输入数量，而不需要添加任何其他东西，除了输入本身。

而且你也可以很容易地在此基础上进一步发展。例如，我们可能想把上一节的 `touched` 和 `error` 属性添加到本节的四个输入中的每一个。

事实上，我建议你自己尝试一下，这是一个很好的练习，可以巩固你到目前为止的学习成果。

#### 如果用useState来代替这个方法呢？

摆脱一打 `useState` 语句的方法之一是把你所有的状态放到一个对象中，存储在一个useState中，然后再更新它。

这个解决方案是可行的，而且有时它是一个好方法。但你经常会发现自己以一种更笨拙的方式重新实现一个 `useReducer` 。还不如马上使用一个 `reducer`。

### 使用 TypeScript 的 useReducer

好了，你现在应该已经掌握了 `useReducer` 的技巧。如果你是 `TypeScript` 的使用者，你可能想知道如何正确地让这两者发挥得更好。

幸好这很容易。在这里，它是这样的：

```jsx
import { useReducer, ChangeEvent } from 'react';

type State = {
  firstName: string;
  lastName: string;
  username: string;
  email: string;
};

type Action =
  | {
      type: 'update';
      payload: {
        key: string;
        value: string;
      };
    }
  | { type: 'reset' };

const initialValue = {
  firstName: '',
  lastName: '',
  username: '',
  email: '',
};

const reducer = (state: State, action: Action) => {
  switch (action.type) {
    case 'update':
      return { ...state, [action.payload.key]: action.payload.value };
    case 'reset':
      return initialValue;
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
};

const Form = () => {
  const [state, dispatch] = useReducer(reducer, initialValue);

  const inputAction = (event: ChangeEvent<HTMLInputElement>) => {
    dispatch({
      type: 'update',
      payload: { key: event.target.name, value: event.target.value },
    });
  };

  return (
    <div>
      <input
        value={state.firstName}
        type="text"
        name="firstName"
        onChange={inputAction}
      />
      <input
        value={state.lastName}
        type="text"
        name="lastName"
        onChange={inputAction}
      />
      <input
        value={state.username}
        type="text"
        onChange={inputAction}
        name="username"
      />
      <input
        value={state.email}
        type="email"
        name="email"
        onChange={inputAction}
      />
    </div>
  );
};

export default Form;
```

如果你不熟悉 `Action` 类型的语法，它是一个 [歧视性的联合](https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html#discriminating-unions)。

## Redux：一个超级强大的 useReducer

我们的 `useReducer` 指南就要结束了（呼，它比我预期的要长！）。还有一件重要的事情要提到：[Redux](https://redux.js.org/)。

你可能听说过 `Redux` 这个非常流行的状态管理库。有些人讨厌它，有些人喜欢它。但事实证明，你所有用于理解 `useReducer` 所绞尽的脑汁对理解 `Redux` 是有用的！

事实上，你可以把 `Redux` 看作是整个应用的一个大的、全局的、管理的和优化的 `useReducer` 。这就是它的全部。

你有一个 “存储”：`store`，其中有你的状态 `state`、你定义的动作 `action`，告诉 `reducer` 如何修改这个存储。是不是听起来很熟悉？↖(^ω^)↗

当然还有一些重要的区别，但如果你已经很好地理解了 `useReducer` ，你就可以很好地理解 Redux 了。

## 总结

这就是文章的结尾! 我希望它能帮助你了解你想要的关于useReducer的一切。

正如你所看到的，它可以成为你的 React 工具箱中一个非常强大的工具。

祝您好运!

【版权信息：著作权归 devtrium.com 所有，如需转载本文，请注明作者 `devtrium.com` 和译者 `Ucely 刘俊`。】