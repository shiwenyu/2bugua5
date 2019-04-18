# [译]React函数组件和类组件的差异



**原文**： <https://overreacted.io/how-are-function-components-different-from-classes/> 



在 `React.js` 开发中，*函数组件*(function component) 和 *类组件(class component)* 有什么差异呢？

在以前，通常认为区别是，类组件提供了更多的特性(比如`state`)。随着 [React Hooks](https://reactjs.org/docs/hooks-intro.html) 的到来，这个说法也不成立了(通过hooks，函数组件也可以有`state`和类生命周期回调了)。

或许你也听说过，这两类组件中，有一类的性能更好。哪一类呢？很多这方面的性能测试，都是 [有缺陷的](https://medium.com/@dan_abramov/this-benchmark-is-indeed-flawed-c3d6b5b6f97f) ，因此要从这些测试中 [得出结论](https://github.com/ryardley/hooks-perf-issues/pull/2) ，不得不谨慎一点。性能主要取决于你代码要实现的功能(以及你的具体实现逻辑)，和使用函数组件还是类组件，没什么关系。我们观察发现，尽管函数组件和类组件的性能优化策略 [有些不同](https://reactjs.org/docs/hooks-faq.html#are-hooks-slow-because-of-creating-functions-in-render) ，但是他们性能上的差异是很微小的。

不管是上述哪个原因，我们都 [不建议](https://reactjs.org/docs/hooks-faq.html#should-i-use-hooks-classes-or-a-mix-of-both) 你使用函数组件重写已有的类组件，除非你有别的原因，或者你喜欢当第一个吃螃蟹的人。`React Hooks` 还很新(就像2014年的React一样)，目前还没有使用hooks相关的最佳实践。

除了上面这些，还有什么别的差异么？在函数组件和类组件之间，真的存在根本性的不同么？*"Of course, there are — in the mental model"* (这个实在不知道咋表达，贴下作者原文吧😅) **在这篇文章里，我们将一起看下，这两类组件最大的不同**。这个不同点，在2015年函数组件函数组件 [被引入React](https://reactjs.org/blog/2015/09/10/react-v0.14-rc1.html#stateless-function-components) 时就存在了，但是被大多数人忽略了。



## 函数组件和类组件的差异



**函数组件会捕获render内部的状态** 

让我们一步步来看下，这代表什么意思。

**注意，本文并不是对函数组件和类组件进行价值判断。我只是展示下React生态里，这两种组件编程模型的不同点。要学习如何更好的采用函数组件，推荐官方文档 [Hooks FAQ](https://reactjs.org/docs/hooks-faq.html#adoption-strategy) ** 。



假设我们有如下的函数组件：

```javascript
function ProfilePage(props) {
  const showMessage = () => {
    alert('Followed ' + props.user);
  };

  const handleClick = () => {
    setTimeout(showMessage, 3000);
  };

  return (
    <button onClick={handleClick}>Follow</button>
  );
}
```

组件会渲染一个按钮，当点击按钮的时候，模拟了一个异步请求，并且在请求的回调函数里，显示一个弹窗。比如，如果 `props.user` 的值是 `Dan`，点击按钮3秒之后，我们将看到 `Followed Dan` 这个提示弹窗。非常简单。

(注意，上面代码里，使用箭头函数还是普通的函数，没有什么区别。因为没有 `this` 问题。把箭头函数换成普通函数 `function handleClick()`没有任何问题 )

我们怎样实现同样功能的类组件呢？很简单的翻译一下：

```javascript
class ProfilePage extends React.Component {
  showMessage = () => {
    alert('Followed ' + this.props.user);
  };

  handleClick = () => {
    setTimeout(this.showMessage, 3000);
  };

  render() {
    return <button onClick={this.handleClick}>Follow</button>;
  }
}
```

通常情况下，我们会认为上面两个组件实现，是完全等价的。开发者经常像上面这样，在函数组件和类组件之间重构代码，却没有意识到他们隐含的差异。

![](./1.gif)



**但是，上面两种实现，存在着微妙的差异** 。再仔细看一看，你能看出其中的差异么？讲真，还是花了我一段时间，才看出其中的差异。

**如果你想在线看看源代码，你可以 [点击这里](https://codesandbox.io/s/pjqnl16lm7) **。 本文剩余部分，都是讲解这个不同点，以及为什么不同点会很重要。



在我们继续往下之前，我想再次说明下，本文提到的差异性，和react hooks本身完全没有关系！上面例子里，我都没用到hooks呢。

本文只是讲解，react生态里，函数组件和类组件的差异性。如果你打算在react开发中，大规模的使用函数组件，那么你可能需要了解这个差异。

**我们将使用在react日常开发中，经常遇到的一个bug，来展示这个差异** 。

接下来，我们就来复现下这个bug。打开 [这个在线例子](https://codesandbox.io/s/pjqnl16lm7) ，页面上有一个名字下拉框，下面是两个关注组件，一个是前文的函数组件，另一个是类组件。

对每一个关注按钮，分别进行如下操作：

1. 点击其中1个关注按钮
2. 在3秒之内，重新选择下拉框中的名字
3. 3秒之后，注意看alert弹窗中的文字差异

你应该注意到了两次alert弹窗的差别：

* 在函数组件的测试情况下，下拉框中选中 `Dan`，点击关注按钮，迅速将下拉框切换到`Sophie`，3秒之后，alert弹窗内容仍然是 `Followed Dan` 
* 在类组件的测试情况下，重复相同的动作，3秒之后，alert弹窗将会显示 `Followed Sophie` 

![](./2.gif)



在这个例子里，使用函数组件的实现是正确的，类组件的实现明显有bug。**如果我先关注了一个人，然后切换到了另一个人的页面，关注按钮不应该混淆我实际关注的是哪一个** 。

(PS，我也推荐你真的关注下 [Sophie](https://mobile.twitter.com/sophiebits)) 

那么，为什么我们的类组件，会存在问题呢？

让我们再仔细看看类组件的 `showMessage` 实现：

```javascript
class ProfilePage extends React.Component {
  showMessage = () => {
    alert('Followed ' + this.props.user);
  };
```

这个方法会读取 `this.props.user` 。在React生态里，`props`是不可变数据，永远不会改变。**但是，`this`却始终是可变的** 。

确实，`this`存在的意义，就是可变的。react在执行过程中，会修改`this`上的数据，保证你能够在 `render`和其他的生命周期方法里，读取到最新的数据(props, state)。

因此，如果在网络请求处理过程中，我们的组件重新渲染，`this.props`改变了。在这之后，`showMessage`方法会读取到改变之后的 `this.props`。

这揭示了用户界面渲染的一个有趣的事实。如果我们认为，用户界面(UI)是对当前应用状态的一个可视化表达(`UI=render(state)`)，**那么事件处理函数，同样属于render结果的一部分，正如用户界面一样** 。我们的事件处理函数，是属于事件触发时的`render`，以及那次render相关联的 `props` 和`state`。

然而，我们在按钮点击事件处理函数里，使用定时器(setTimeout)延迟调用 `showMessage`，打破了`showMessage`和`this.props`的关联。`showMessage`回调不再和任何的render绑定，同样丢失了本来关联的props。从 `this` 上读取数据，切断了这种关联。

**假如函数组件不存在**，那我们怎么来解决这个问题呢？

我们需要通过某种方式，修复`showMessage`和它所属的 `render`以及对应props的关联。

一种方式，我们可以在按钮点击处理函数中，读取当前的 props，然后显式的传给 `showMessage`，就像下面这样：

```javascript
class ProfilePage extends React.Component {
  showMessage = (user) => {
    alert('Followed ' + user);
  };

  handleClick = () => {
    const {user} = this.props;
    setTimeout(() => this.showMessage(user), 3000);
  };

  render() {
    return <button onClick={this.handleClick}>Follow</button>;
  }
}
```

这种方式 [可以解决这个问题](https://codesandbox.io/s/3q737pw8lq) 。然而，这个解决方式让我们引入了冗余的代码，随着时间推移，容易引入别的问题。如果我们的 `showMessage`方法要读取更多的props呢？如果`showMessage`还要访问state呢？**如果 `showMessage` 调用了其他的方法，而那个方法读取了别的状态，比如`this.props.something` 或 `this.state.something` ，我们会再次面临同样的问题**。 我们可能需要在 `showMessage` 里显式的传递 `this.props` `this.state` 给其他调用到的方法。

这样做，可能会破坏类组件带给我们的好处。这也很难判断，什么时候需要传递，什么时候不需要，进一步增加了引入bug的风险。

同样，简单地把所有代码都放在 `onClick` 处理函数里，会带给我们其他的问题。为了代码可读性、可维护性等原因，我们通常会把大的函数拆分为一些独立的小的函数。**这个问题不仅仅是react才有，所有在`this`上维护可变数据的UI类库，都很容易遇到这个问题**。

或许，我们可以在类的构造函数里绑定一些方法？

```javascript
class ProfilePage extends React.Component {
  constructor(props) {
    super(props);
    this.showMessage = this.showMessage.bind(this);
    this.handleClick = this.handleClick.bind(this);
  }

  showMessage() {
    alert('Followed ' + this.props.user);
  }

  handleClick() {
    setTimeout(this.showMessage, 3000);
  }

  render() {
    return <button onClick={this.handleClick}>Follow</button>;
  }
}
```

很遗憾，上面的代码 **不能** 解决这个问题！记住，导致这个问题的原因，是我们读取 `this.props`的时机太晚了，和语法没有关系。**然而，如果我们能够完全依赖JavaScript的闭包机制，那么就能彻底解决这个问题** 。

我们大多数情况下，会尽量**避免**使用闭包，因为在闭包的情况下，判断一个可变的变量值，会变得 [有些困难](https://wsvincent.com/javascript-closure-settimeout-for-loop/) 。但是，在react里，props和state是 **不可变**的(严格来说，我们强烈推荐将props和state作为不可变数据)。props和state的不可变特性，完美解决了使用闭包带来的问题。

这意味着，如果在 `render`方法里，通过闭包来访问props和state，我们就能确保，在`showMessage`执行时，访问到的props和state就是render执行时的那份数据：

```javascript
class ProfilePage extends React.Component {
  render() {
    // Capture the props!
    const props = this.props;

    // Note: we are *inside render*.
    // These aren't class methods.
    const showMessage = () => {
      alert('Followed ' + props.user);
    };

    const handleClick = () => {
      setTimeout(showMessage, 3000);
    };

    return <button onClick={handleClick}>Follow</button>;
  }
}
```

**在render执行时，你成功的捕获了当时的props** 。

![](./3.gif)

通过这种方式，render方法里的任何代码，都能访问到render执行时的props，而不是后面被修改过的值。react不会再偷偷挪动我们的奶酪了。

**像上面这样，我们可以在render方法里，根据需要添加任何帮助函数，这些函数都能够正确的访问到render执行时的props**。闭包，这一切的救世主。



[上面的代码](https://codesandbox.io/s/oqxy9m7om5)，功能上没问题，但是看起来有点怪。如果组件逻辑都作为函数定义在render内部，而不是作为类的实例方法，那为什么还要用类呢？

确实，我们剥离掉类的外衣，剩下的就是一个函数组件：

```javascript
function ProfilePage(props) {
  const showMessage = () => {
    alert('Followed ' + props.user);
  };

  const handleClick = () => {
    setTimeout(showMessage, 3000);
  };

  return (
    <button onClick={handleClick}>Follow</button>
  );
}
```

这个函数组件和上面的类组件一样，内部函数捕获了props，react会把props作为函数参数传进去。**和this不同的是，props是不可变的，react不会修改props** 。

如果你在函数参数里，把props解构，代码看起来会更加清晰:

```javascript
function ProfilePage({ user }) {
  const showMessage = () => {
    alert('Followed ' + user);
  };

  const handleClick = () => {
    setTimeout(showMessage, 3000);
  };

  return (
    <button onClick={handleClick}>Follow</button>
  );
}
```

当父组件传入**不同**的props来重新渲染 `ProfilePage`时，react会再次调用 `ProfilePage`。但是在这之前，我们点击关注按钮的事件处理函数，已经捕获了上一次render时的props。

这就是为什么，在 [这个例子](https://codesandbox.io/s/pjqnl16lm7) 的函数组件中，没有问题。

![](./4.gif)

可以看到，功能完全是正确的。(再次PS，建议你也关注下 [Sunil](https://mobile.twitter.com/threepointone)) 

现在我们理解了，在函数组件和类组件之间的这个差异：



**函数组件会捕获render内部的状态** 



## 函数组件配合React Hooks



在有 `Hooks` 的情况下，函数组件同样会捕获render内部的 `state`。看下这个例子：

```javascript
function MessageThread() {
  const [message, setMessage] = useState('');

  const showMessage = () => {
    alert('You said: ' + message);
  };

  const handleSendClick = () => {
    setTimeout(showMessage, 3000);
  };

  const handleMessageChange = (e) => {
    setMessage(e.target.value);
  };

  return (
    <>
      <input value={message} onChange={handleMessageChange} />
      <button onClick={handleSendClick}>Send</button>
    </>
  );
}
```

(在线demo，[点击这里](https://codesandbox.io/s/93m5mz9w24)) 

尽管这是一个很简陋的消息发送组件，它同样展示了和前一个例子相同的问题：如果我点击了发送按钮，这个组件应该发送的是，我点击按钮那一刻输入的信息。



OK，我们现在知道，函数组件会默认捕获props和state。**但是，如果你想读取最新的props、state呢，而不是某一时刻render时捕获的数据**？ 甚至我们想在 [将来某个时刻读取旧的props、state](https://dev.to/scastiel/react-hooks-get-the-current-state-back-to-the-future-3op2) 呢？

在类组件里，我们只需要简单的读取 `this.props` `this.state` 就能访问到最新的数据，因为react会修改`this`。在函数组件里，我们同样可以拥有一个**可变数据**，它可以在每次render里共享同一份数据。这就是hooks里的 `useRef`：

```javascript
function MyComponent() {
  const ref = useRef(null);
  // You can read or write `ref.current`.
  // ...
}
```



但是，你需要自己维护 `ref` 对应的值。

函数组件里的 `ref` 和类组件中的**实例属性** 扮演了[相同的角色](https://reactjs.org/docs/hooks-faq.html#is-there-something-like-instance-variables) 。你或许已经熟悉 `DOM refs`，但是hooks里的 `ref` 更加通用。hooks里的 `ref` 只是一个容器，你可以往容器里放置任何你想放的东东。

甚至看起来，类组件里的 `this.something` 也和hooks里的 `something.current` 相似，他们确实代表了同一个概念。

默认情况下，react不会给函数组件里的props、state创造refs。大多数场景下，你也不需要这样做，这也需要额外的工作来给refs赋值。当然了，你可以手动的实现代码来跟踪最新的state：

```javascript
function MessageThread() {
  const [message, setMessage] = useState('');
  const latestMessage = useRef('');

  const showMessage = () => {
    alert('You said: ' + latestMessage.current);
  };

  const handleSendClick = () => {
    setTimeout(showMessage, 3000);
  };

  const handleMessageChange = (e) => {
    setMessage(e.target.value);
    latestMessage.current = e.target.value;
  };
```

如果我们在 `showMessage`里读取 `message`字段，那么我们会得到我们点击按钮时，输入框的值。但是，如果我们读取的是 `latestMessage.current`，我们会得到输入框里最新的值——甚至我们在点击发送按钮后，不断的输入新的内容。

你可以对比 [这两个demo](https://codesandbox.io/s/ox200vw8k9) 来看看其中的不同。

通常来讲，你应该避免在render函数中，读取或修改 refs ，因为 refs 是可变的。我们希望能保证render的结果可预测。**但是，如果我们想要获取某个props或者state的最新值，每次都手动更新refs的值显得很枯燥** 。这种情况下，我们可以使用  `useEffect` 这个hook：

```javascript
function MessageThread() {
  const [message, setMessage] = useState('');

  // Keep track of the latest value.
  const latestMessage = useRef('');
  useEffect(() => {
    latestMessage.current = message;
  });

  const showMessage = () => {
    alert('You said: ' + latestMessage.current);
  };
```

(demo [在这里](https://codesandbox.io/s/yqmnz7xy8x)) 

我们在 `useEffect` 里去更新 ref 的值，这保证只有在DOM更新之后，ref才会被更新。这确保我们对 ref 的修改，不会破坏react中的一些新特性，比如 [时间切分和中断](https://reactjs.org/blog/2018/03/01/sneak-peek-beyond-react-16.html) ，这些特性都依赖 可被中断的render。

像上面这样使用 ref 不会太常见。**大多数情况下，我们需要捕获props和state** 。但是，在处理[命令式API](https://overreacted.io/making-setinterval-declarative-with-react-hooks/)的情况下，使用 ref 会非常简便，比如设置定时器，订阅事件等。记住，你可以使用 ref 来跟踪任何值——一个prop，一个state，整个props，或者是某个函数。

使用 ref 在某些性能优化的场景下，同样适用。比如在使用 `useCallback` 时。但是，[使用useReducer](https://reactjs.org/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down) 在大多数场景下是一个 [更好的解决方案](https://github.com/ryardley/hooks-perf-issues/pull/3) 。



## 总结



在这篇文章里，我们回顾了类组件中常见的一个问题，以及怎样使用闭包来解决这个问题。但是，你可能已经经历过了，如果你尝试通过指定hooks的依赖项，来优化hooks的性能，那么你很可能会在hooks里，访问到旧的props或state。这是否意味着闭包会带来问题呢？我想不是的。

正如我们上面看到的，在一些不易察觉的场景下，闭包帮助我们解决掉这些微妙的问题。不仅如此，闭包也让我们在 [并行模式](https://reactjs.org/blog/2018/03/01/sneak-peek-beyond-react-16.html) 下更加容易写出没有bug的代码。因为闭包捕获了我们render函数运行时的props和state，使得并行模式成为可能。

到目前为止，我经历的所有情况下，访问到旧的props、state，通常是由于我们错误的认为"函数不会变"，或者"props始终是一样的"。实时上不是这样的，我希望在本文里，能够帮助你了解到这一点。

在我们使用函数来开发大部分react组件时，需要更正我们对于 [代码优化](https://github.com/ryardley/hooks-perf-issues/pull/3) 和 [哪些状态会改变](https://github.com/facebook/react/issues/14920) 的认知。

正如 [Fredrik说的](https://mobile.twitter.com/EphemeralCircle/status/1099095063223812096): 

> 在使用react hooks过程中，我学习到的最重要规则就是，"任何变量，都可以在任何时间被改变"

函数同样遵守这条规则。

**react函数始终会捕获props、state**，最后再强调一下。



**译注**： 有些地方不明白怎么翻译，有删减，建议阅读原文！



​     ———时2019年4月17日 18:47 竣工于帝都五道口清华科技园