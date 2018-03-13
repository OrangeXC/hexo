---
title: React 和 Redux 快速开发实战
date: 2017-03-02 18:50:56
tags: ['React', 'Redux']
---

<!--
![](/uploads/React 和 Redux 快速开发实战1.jpg)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9knkbdvij30rs0cm43q.jpg)

今天聊一聊 react + redux 环境快速搭建，以及实战一个 TodoList，可能是有史以来最简洁的方法哦，是不是很期待，当时橙子也是很吃惊这样的搭建速度。

本文适合有 react 和 redux 基础的同学看，当然你也可以看完本文去学基础。下面进入正题

<!-- more -->

## 快速搭建环境

没错，看过前面博客的同学猜到，我会选择各种 cli 工具，当然你可能猜错了，这里用的不是 create-react-app 这个 star 最多的构建工具。

这里我们用 [react-redux-starter-kit](https://github.com/davezuko/react-redux-starter-kit)

需要 `node 4.5.0+ & npm 3.0.0+` 即可，首先

```
git clone https://github.com/davezuko/react-redux-starter-kit.git <my-project-name>
cd <my-project-name>
npm install
npm start
```

没错你已经搭建环境完毕。好快的样子！

此时你可以看到 `localhost：3000` 的首屏了，welcome 然后一个鸭子，奇怪的首屏，作者说 duck 是 redux 的谐音，好吧。。。

此时可能你已经兴奋的点完了，这个 cli 的全部内容。

先不急看效果，既然是 redux，就要有对应的 chrome 插件，让我们直观的看到 store 的变化。

推荐 [redux-devtool](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd)，点开链接，添加至 chrome 即可。

此时开启控制台，找到 Redux 的 tab，会看到如下界面

<!--
![](/uploads/React 和 Redux 快速开发实战2.jpg)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9knkb0m1j314m14sgoe.jpg)

功能强大，想深入学习的可以看文档，这里我们只关心 Action 和 State 的变化

此时项目在 `localhost:3000/counter` 这个路由下有个累加器，Increment 每次同步加 1，Double(async) 每次异步乘 2。

异步处理速度慢？不是的看代码会发现这样一段

```js
export const doubleAsync = () => {
  return (dispatch, getState) => {
    return new Promise((resolve) => {
      setTimeout(() => {
        dispatch({
          type    : COUNTER_DOUBLE_ASYNC,
          payload : getState().counter
        })
        resolve()
      }, 200)
    })
  }
}
```

没错，是一个 200ms 的 setTimeout。

大家印象里，官方文档有说 action 是立即执行的同步函数，我们可以借助中间件的形式实现异步，这里用的是 `redux-thunk`，这种方式的确解决了问题，之后的文章还会单独介绍更好的办法。

代码简单过一遍，没什么难点。接下来我们就可以根据这个项目渐进开发了。

## 实战 TodoList

好，我们首先添加一个路由

src/compoments/Header/Header.js

在 div 标签内最下面添加如下代码

```html
{' · '}
<Link to='/todo' activeClassName='route--active'>
  Todo
</Link>
```

src/routes/index.js

对应位置如下修改，添加一个子路由。

```js
import CoreLayout from '../layouts/CoreLayout'
import Home from './Home'
import CounterRoute from './Counter'
import TodoRoute from './Todo'

export const createRoutes = (store) => ({
  path        : '/',
  component   : CoreLayout,
  indexRoute  : Home,
  childRoutes : [
    CounterRoute(store),
    TodoRoute(store)
  ]
})
```

我们现在还没有 Todo 对应路由的组件，下面在 routes 下创建 Todo 文件夹，然后文件夹结构如下

```
Todo
  |-components
  |  |-Todo.js
  |
  |-containers
  |  |-TodoContainer.js
  |
  |-modules
  |  |-todo.js
  |
  |-index.js
```

简单解释一下这个目录结构，index.js 是 Todo 作为路由页面的入口文件，components 下放的是模块的视图部分，containers 作为一个容器来绑定组件的 event、 state 和 prop，modules 主要是负责从 action->reducer->newState 的过程。

条例清晰，梳理完结构，看看每个文件都怎么完成的

index.js

```js
import { injectReducer } from '../../store/reducers'

export default (store) => ({
  path : 'todo',
  getComponent (nextState, cb) {
    require.ensure([], (require) => {
      const Todo = require('./containers/TodoContainer').default
      const { itemReducer } = require('./modules/todo')
      const { todoListReducer } = require('./modules/todo')

      injectReducer(store, { key: 'itemData', reducer: itemReducer })
      injectReducer(store, { key: 'todoList', reducer: todoListReducer })

      cb(null, Todo)
    }, 'todo')
  }
})
```

这里将 store 和 reducer 绑定，将 store 注入到 /todo 的路由下的 props 中去

components/Todo.js

```js
import React, { Component } from 'react'

export default class Todo extends Component {
  render () {
    return (
      <div>
        <input
          className='form-control'
          style={{ marginBottom: '10px' }}
          type='text' ref='input'
          value={this.props.itemData}
          onChange={(e) => this.handleChange(e)}
        />
        <button
          className='btn btn-primary btn-lg btn-block'
          style={{ marginBottom: '10px' }}
          onClick={(e) => this.handleSubmit(e)}
        >
          添加
        </button>
        <ul className='list-group'>
          {
            this.props.todoList.map((item, index) => {
              return <li
                className='list-group-item'
                key={index}
              >
                {item}
                <button
                  className='btn btn-default'
                  data-index={index}
                  onClick={(e) => this.handleDel(e)}
                >
                  ❌
                </button>
              </li>
            })
          }
        </ul>
      </div>
    )
  }

  handleChange (e) {
    const node = this.refs.input
    const text = node.value.trim()
    this.props.updateItem(text)
  }

  handleSubmit (e) {
    const node = this.refs.input
    const text = node.value.trim()
    this.props.addItem(text)
    this.props.updateItem('')
  }

  handleDel (e) {
    const index = e.target.getAttribute('data-index')
    this.props.delItem(Number(index))
  }
}

Todo.propTypes = {
  addItem: React.PropTypes.func.isRequired,
  itemData: React.PropTypes.string.isRequired,
  updateItem: React.PropTypes.func.isRequired,
  delItem: React.PropTypes.func.isRequired,
  todoList: React.PropTypes.array.isRequired
}
```

就是个简单的 jsx，这里后缀没有写成 .jsx 文件，因为 webpack 配置了解析规则，.js 文件也可以正常解析，这里的 onClick 事件写成了 `=>` 箭头函数的形式，目的是为了锁定 this 的作用域，在 handle 函数内可以拿到 this。

containers/TodoContainer.js

```js
import { connect } from 'react-redux'
import { updateItem, addItem, delItem } from '../modules/todo'
import Todo from '../components/Todo'

const mapDispatchToProps = {
  updateItem,
  addItem,
  delItem
}

const mapStateToProps = (state) => ({
  itemData: state.itemData,
  todoList: state.todoList
})

export default connect(mapStateToProps, mapDispatchToProps)(Todo)
```

这里通过一个 connect 方法，将组件的 props 进行绑定。

modules/todo.js

```js
export const UPDATE_ITEM = 'UPDATE_ITEM'
export const ADD_ITEM = 'ADD_ITEM'
export const DELETE_ITEM = 'DELETE_ITEM'

export function updateItem (item) {
  return {
    type: UPDATE_ITEM,
    payload: item
  }
}

export function addItem (item) {
  return {
    type: ADD_ITEM,
    payload: item
  }
}

export function delItem (index) {
  return {
    type: DELETE_ITEM,
    payload: index
  }
}

export const actions = {
  updateItem,
  addItem,
  delItem
}

const ITEM_ACTION_HANDLERS = {
  [UPDATE_ITEM]: (state, action) => action.payload
}

const LIST_ACTION_HANDLERS = {
  [ADD_ITEM]: (state, action) => state.concat(action.payload),
  [DELETE_ITEM]: (state, action) => state.filter((item, index) => {
    return index !== action.payload
  })
}

export function itemReducer (state = '', action) {
  const handler = ITEM_ACTION_HANDLERS[action.type]

  return handler ? handler(state, action) : state
}

export function todoListReducer (state = [], action) {
  const handler = LIST_ACTION_HANDLERS[action.type]

  return handler ? handler(state, action) : state
}
```

这里 reducer 运用两个数组方法 concat 和 filter，分别做拼接和删除操作，这里意遵循 redux 的官方文档在不修改原 state，而是返回新的 state。在底部将两个 reducer 导出，绑定到 index.js 中。完成一个闭环，数据更新完成。

到这里我们的 todoList 完成了。

一边看界面一边看控制台 redux store 的变化吧！代码没有晦涩难懂的点，一个 todoList 也足以说明列表操作的流程，这也是各大框架都以 todoList 为 demo 的原因。

## 总结

对于复杂的应用 redux 带来的优势不赘述，本文的目的是讲述如何快速开发 react + redux 应用，让大家在配置环境上少踩坑，也通过一个简单的例子，说明我们应该怎么更简单的去使用 redux。动手写起来吧，橙子的文章一般都看不出效果的，亲手实现一遍会有意外收获哦，上面代码也是橙子自己踩过坑后写出来的，各大神有什么建议的地方尽管提出来！感谢阅读。
