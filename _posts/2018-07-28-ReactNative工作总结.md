---
layout:     post
title:      "ReactNative 工作总结"
subtitle:   " \"基础知识\""
date:       2018-07-28 12:00:00
author:     "Undervoid"
header-img: "img/post-bg-react-native.png"
catalog: true
tags:
    - 磨砻淬砺集
    - React Native
    - 工作总结
---

# React Native 前言

最近工作开始接触React Native（以下简称RN），这应该是最近近两年来最火的框架之一。它的推出也伴随着一个名词“全栈式前端”，这是个什么概念呢？大概意思就是一个人可以同时写Android,iOS,Web程序。虽然个人本身仍然是个Android程序员，但是接触一点RN我也相信对自己的职业生涯会有好处，目前我使用的仍然是Javascript进行RN的撰写，虽然本人我对 `javascript` 这门语言真的是没什么好感，感觉写代码就像光着身子在路上走路一样，当然也可以尝试使用 `Typescript` 去替代可毕竟自己不是RN的很感冒所以目前也只是尝试而已分享下自己的见解，如有不足敬请谅解。


## React Native 基本知识

关`于React Native` 的 `Component` 等网上有一系列的文章和教程，我就不一一阐述了，本文也适合一些已经对RN框架已经有所掌握的人阅读.我主要在以下几点来解析下RN的特性：

### 高阶组件Higher-Order Component 

#### 为何使用
关于高阶组件解决的问题可以简单概括成以下几个方面：代码复用：这是高阶组件最基本的功能。组件是React中最小单元，两个相似度很高的组件通过将组件重复部分抽取出来，再通过高阶组件扩展，增删改props，可达到组件可复用的目的；条件渲染：控制组件的渲染逻辑，常见case：鉴权；生命周期捕获/劫持：借助父组件子组件生命周期规则捕获子组件的生命周期，常见case：打点。

#### 使用方法

- 1、直接返回一个stateless component，如：
 ```javascript
 function EnhanceWrapper(WrappedComponent) {
   const newProps = {
        source: 'app',
    };
    return props => <WrappedComponent {...props} {...newProps} />;
}
 ```
 - 2、在新组件的render函数中返回一个新的class component，如：
 ```javascript
function EnhanceWrapper(WrappedComponent) {
    return class WrappedComponent extends React.Component {
        render() {
           return <WrappedComponent {...this.props} />;
        }
    }
}
 ```
 - 3、继承（extends）原组件后返回一个新的class component，如
 ```javascript
 function EnhanceWrapper(WrappedComponent) {
    return class WrappedComponent extends WrappedComponent {
        render() {
            return super.render();
        }
    }
}
 ```
 ### Function as Child Components
 ```javascript
 class StudentWithAge extends React.Component {
    componentWillMount() {
        this.setState({
            name: '小红',
            age: 25,
        });
    }
    render() {
        return (
            <div>
                {this.props.children(this.state.name, this.state.age)}
            </div>
        );
    }
}
```
使用的时候可以这样：
```javascript
<StudentWithAge>
    {
        (name, age) => {
            let studentName = name;
            if (age > 22) {
                studentName = `大学毕业的${studentName}`;
            }
            return <Student name={studentName} />;
        }
    }
</StudentWithAge>

```
* 比起高阶组件，这种方式有一些优势：
   * 1、代码结构上少掉了一层（返回高阶组件的）函数封装。
   * 2、调试时组件结构更加清晰；
   * 3、从组件复用角度来看，父组件和子组件之间通过children连接，两个组件其实又完全可以单独使用，内部耦合较小。当然单独使用意义并不大，而且高阶组件也可以通过组合两个组件来做到。
* 同时也有一些劣势：
   * 1、（返回子组件）函数占用了父组件原本的props.children；
   * 2、（返回子组件）函数只能进行调用，无法劫持劫持原组件生命周期方法或取到static方法；
   * 3、（返回子组件）函数作为子组件包裹在父组件中的方式看起来虽灵活但不够优雅；
   * 4、由于子组件的渲染控制完全通过在父组件render方法中调用（返回子组件）函数，无法通过shouldComponentUpdate来做性能优化。

### Getter,Setter方法

与其说是 `RN` 的 `Getter Setter` 方法， 不如说是 `ES6`的`Getter Setter` 在 `RN` 中的组件的应用。

通常我们会怎么样计算一个组件的Props并显示出来呢？ 代码如下 

```javascript
constructor (props) {
    super()
    this.state = {};
  }

    
  render () {
    // 常规写法，在render中直接计算 
    const fullName = `${this.props.firstName} ${this.props.lastName}`;

    return (
      <View>
        <Text>{fullName}</Text>
      </Viw>
    );
  }
```

后来我又了解到我们可以写的更好比如独立化计算的方法，然后直接替代变量,这种写法的好处就是可以不用多余创造一个变量

```javascript
constructor (props) {
    super()
    this.state = {};
  }
    // better写法，独立计算方法
_getFullName = ()=>`${this.props.firstName} ${this.props.lastName}`
    
  render () {
    return (
      <View>
        <Text>{this._getFullName()}</Text>
      </Viw>
    );
  }
```

使用 `Getter` 替代这个方法,这个方法我是在阅读开源代码里面看到的，虽然在我看来只是另一种写法而已，但是我在 `Github` 上看到了这片 [RN Pattern](https://github.com/planningcenter/react-patterns#component-organization) 的规定 一个关于标准组件的组织结构

* class definition
  * constructor
    * event handlers
  * 'component' lifecycle events
  * getters
  * render
* defaultProps
* proptypes

```javascript
class Person extends React.Component {
  constructor (props) {
    super(props);

    this.state = { smiling: false };

    this.handleClick = () => {
      this.setState({smiling: !this.state.smiling});
    };
  }

  componentWillMount () {
    // add event listeners (Flux Store, WebSocket, document, etc.)
  }

  componentDidMount () {
    // React.getDOMNode()
  }

  componentWillUnmount () {
    // remove event listeners (Flux Store, WebSocket, document, etc.)
  }

  get smilingMessage () {
    return (this.state.smiling) ? "is smiling" : "";
  }

  render () {
    return (
      <div onClick={this.handleClick}>
        {this.props.name} {this.smilingMessage}
      </div>
    );
  }
}

Person.defaultProps = {
  name: 'Guest'
};

Person.propTypes = {
  name: React.PropTypes.string
};
```

当然也看到了一篇文章关于 `Vue` 的 `computed` 也使用了类似的方法但是更好的一点是因为它还有个缓存机制。之后我会研究下看能不能使用同样的方法在 `RN` 上实现。


### Saga MiddleWare
 redux-saga 通过创建 Sagas 将所有的异步操作逻辑收集在一个地方集中处理，可以用来代替 redux-thunk 中间件。
	 这意味着应用的逻辑会存在两个地方：
		- Reducers 负责处理 action 的 state 更新。
		- Sagas 负责协调那些复杂或异步的操作
	Sagas 不同于 Thunks，Thunks 是在 action 被创建时调用，而 Sagas 只会在应用启动时调用（但初始启动的 Sagas 可能会动态调用其他 Sagas）。
    Sagas 可以被看作是在后台运行的进程。Sagas 监听发起的 action，然后决定基于这个 action 来做什么：是发起一个异步调用（比如一个 Ajax 请求），
    还是发起其他的 action 到 Store，甚至是调用其他的 Sagas。

		
 * Sage 就像个AsyncAction的Call

#### Saga辅助函数
 * takeEvery 就是每次执行某个Action时然后进行的操作，允许多次执行
 * takeLatest 只允许执行一个 fetchData 任务	
 
#### 声明式 Effects

#### 同时执行多个任务

```javascript
 // 错误写法，effects 将按照顺序执行
const users = yield call(fetch, '/users'),
      repos = yield call(fetch, '/repos')
```

```javascript
import { call } from 'redux-saga/effects'

// 正确写法, effects 将会同步执行
const [users, repos] = yield [
  call(fetch, '/users'),
  call(fetch, '/repos')
]
```
当我们需要 yield 一个包含 effects 的数组， generator 会被阻塞直到所有的 effects 都执行完毕，
或者当一个 effect 被拒绝 （就像 Promise.all 的行为）。


#### Sagas API参考用法
 * take =>
 * put => dispatch(用于调用Redux Action方法)
 * call =>就是为了调用一个 普通的非Redux Action方法
 * select(selector, ...args)=>来获取当前state的状态
 创建一条 Effect 描述信息，指示 middleware 调用提供的选择器获取 Store state 上的数据（例如，返回 selector(getState(), ...args) 的结果）。
 selector: Function - 一个 (state, ...args) => args 函数. 通过当前 state 和一些可选参数，返回当前 Store state 上的部分数据。
 args: Array - 可选参数，传递给选择器（附加在 getState 后）如果 select 调用时参数为空（即 yield select()），那 effect 会取得整个的 state（和   调用 getState() 的结果一样）。
 > 重要提醒：在发起 action 到 store 时，middleware 首先会转发 action 到 reducers 然后通知 Sagas。这意味着，当你查询 Store 的 state， 你获取的是 action 被处理之后的 state
 *** 这里需要注意到selector 的用法

* 使用 Channel
到目前為止，我們使用了 take 和 put effect 來和 Redux Store 溝通。Channels 概括了這些 Effect 與外部事件來源或 Saga 它們之間的溝通。它們也可以從 Store 指定隊列的 action。

在這個部份，我們將會看到：

如何從 Store 使用 yield actionChannel Effect 來緩衝指定的 action。

如何使用 eventChannel factory function 連結 take Effect 到外部的事件來源。

如何使用通用的 channel factory function 建立一個 channel，並在兩個 Saga 之間使用 take 和 put Effect 做溝通。

actionChannel => 用来一次一次处理action的关系
eventChannel => factory 連結外部的事件為 Redux Store 以外的事件來源建立一個 Channel。


### Redux-Connect
#### Redux -- Connect
React 和 redux 本身
对于某些人来说，也许很难相信 redux 和 react 实际上是两个独立的库这件事情，它们完全可以独立使用。
我们先来看一下 redux 的状态（state）管理机制:

                                            Dispatch 
                                     //       ||         \\
                                    //       ||           \\
                                  Reducer1 Reducer2  Reducer3    
                                    \\       ||          //
                                     \\      ||         //
                                            Store
                                             ||
                                             ||
                                            State
                                            

#### redux flow

如果你之前曾经使用过 redux，你就知道 redux 主要是围绕着 "store" 来运转的，它的 store 就是用来保存应用里所有的 state 的地方。任何人都不能直接修改 store，只能通过 reducers 来修改 store 里的数据，而想要触发 reducers 我们需要派发 actions。所以最终：

> To change data, we need to dispatch an action

想要改变数据，我们需要派发一个 action

另一方面，如果我们想要获取数据，也不能直接从 store 里取。而是通过 store.getState() 来得到当时 store 里所有数据的 “快照”，也即是获取了我们调用 getState 时整个程序的状态

> To obtain data we need to get the current state of the store

想要获取数据，我们需要得到 store 里的状态

接下来我们看一下一个标准 React todo 应用的组件结构。

                                             App 
                                     //               \\
                                    //                 \\
                                  TodoFilter             TodoList    
                                                            //    \\
                                                           //      \\
                                                        TodoItem    TodoItem
                                           



#### 把 React 和 Redux 结合起来
如果想把 redux store 和我们的 react 应用链接起来，那么首先需要让应用知道 store 的存在。这也即是 react-redux 库的第一个主要部分 Provider。

Provider 其实是 react-redux 提供的一个 React 组件，它只有一个作用，就是把 store “提供 provide” 给它的子组件。

```javascript
//This is the store we create with redux's createStore method
const store = createStore(todoApp,{})

// Provider is given the store as a prop
render(  
  <Provider store={store}>
    <App/>
  </Provider>, document.getElementById('app-node'))
  
 ```
既然只有 provider 的子组件能获取到 store，而我们一般又希望能让整个应用获取到 store，那么最明智的做法就是把 App 组件放到 Provider 里。

按照上面两幅图的思路，我们应该把 Provider 放在 App 上面表示是 App 的父组件，然而考虑到 Provider 提供的功能，把它表示成包裹着整个应用树的形式更为合适，如下图： react flow provider

Connect
既然已经把 redux store “provide” 给了我们的应用，接下来就可以把应用里的组件和它 connect 起来了。还记得前面强调的不能直接和 store 交互吗？我们只能通过获取应用的状态来获取 store 里的数据，或者通过派发 action 来改变应用的状态：我们只能获取到上面 redux 流程图里最顶部（dispatch）和最底部（State）的元素。

这些正是 connect 的功能：把 redux 的 dispatch 和 state 映射为 react 组件的 props！（看下面这段代码）

```javascript
import {connect} from 'react-redux'

const TodoItem = ({todo, destroyTodo}) => {  
  return (
    <div>
      {todo.text}
      <span onClick={destroyTodo}> x </span>
    </div>
  )
}

const mapStateToProps = state => {  
  return {
    todo : state.todos[0]
  }
}

const mapDispatchToProps = dispatch => {  
  return {
    destroyTodo : () => dispatch({
      type : 'DESTROY_TODO'
    })
  }
}

export default connect(  
  mapStateToProps,
  mapDispatchToProps
)(TodoItem)

```
mapStateToProps 和 mapDispatchToProps 是两个春函数，分别用来获取 store “state” 和 “dispatch” 。具体来说，这两个函数各返回一个对象，对象的 keys 将作为 props 传递给它们所连接的组件。

这上面这个例子中，mapStateToProps 返回了一个对象只有一个 key: todo。mapDispatchToProps 返回了一个拥有 key destroyTodo 的对象。

被连结的组件(也就是最后被 export 的那个组件)，把 todo 和 destroyTodo 传递给 TodoItem 做 props。 final connect flow

尤其需要注意的是，只有在 Provider 内的组件才能被连结（connect）。从上面的图表可以看到，连结是通过 Provider 建立的。

Redux 是一个非常强大的工具，尤其是当与 React 组合在一起使用时。明白 react-redux 的各个部分是如何使用以及为什么使用会很有帮助，希望阅读完这篇文章，你能清楚 Provider 和 'connect' 的具体功能。

## 参考

[文章0](https://juejin.im/post/59b36b416fb9a00a636a207e#heading-4)
[文章1](https://juejin.im/entry/59b105585188252430288a34)
[文章2](http://varnull.cn/react-redux-connect-explain/)





