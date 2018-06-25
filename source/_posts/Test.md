---
title: 让前端测试简单高效
date: 2018-06-23 18:50:56
tags: ['Test']
---

前端测试或许被好多人误解，也许大家更加倾向于编写面向后端的测试，逻辑性强，测试方便等

聊到这导致了好多前端从来不写测试（测试全靠手点～～～）

其实没必要达到测试驱动开发的程度，只要写完代码可以补测试，并且补出高效的测试，前端或许真的不需要手点

大前端时代不谈环境不成方圆，本文从下面几个环境一一分析下如何敏捷测试

* node 环境
* vue 环境
* nuxt 服务端渲染环境
* react 环境
* next 服务端渲染环境
* angular 环境

理解测试前需要补充下单元测试（unit）和端到端测试（e2e）的概念，这里不赘述

## node 环境

推荐测试框架 [jest](https://facebook.github.io/jest/)

jest 是 FB 的杰作之一，方便各种场景的 js 代码测试，这里选择 jest 是因为确实方便

使用方法及配置信息可以去官方文档

配置的注意事项

```js
{
  testEnvironment: 'node' // 如不声明默认浏览器环境
}
```

针对 node 只聊一下单元测试，e2e 测试比较少见

当决定写一个 npm 模块时，代码完成后必不可少的就是单元测试，单元测试需要注意的问题比较琐碎

### mock

当引入三方库时，不得不 mock 数据，因为单元测试更多讲求的是局部测试，不要受外界三方引入包的影响

例如：

```js
const { readFileSync } = require('fs')

const getFile = () => {
  try {
    const text = readFileSync('text.txt', 'utf8')
  } catch (err) {
    throw new Error(err)
  }

  console.log(text)
}

module.exports = getFile
```

这时我们并不需要关心 `text.txt` 是否真的存在，也不需要关系 `text` 的内容具体是什么，我们的关注点应该在于读取文件错误时能否及时抛出异常，以及 `console.log()` 是否如预期执行

对应到测试

```js
const getFile = require('./getFile')

describe('readFile', () => {
  const mocks = {
    fs: {
      readFileSync: jest.fn()
    },
    other: {
      text: 'Test text'
    }
  }

  beforeAll(() => {
    jest.mock('fs', () => mocks.fs)
  })

  test('read file success run console.log', () => {
    mocks.fs.readFileSync.mockImplementation(() => this.mocks.other.text)

    getFile()

    expect(console.log).toBeCalled()
  })
})
```

上面代码简单的实现了一个读取文件是否成功的测试，先别急着纠错，这段测试本身是错的，下面慢慢分析

我们在最开始创建了一个 `mocks` 对象，用来模拟数据，由于 `readFileSync` 方法可能存在多种返回结果（成功或报错），所以暂时用 `jest.fn()` 模拟

other 里面则是放一些固定的测试数据（不会随着测试过程而改变）

`beforeAll` 钩子里面执行我们的 mock，把 require 进来的 fs 模块拦截调，也是本测试用例中的关键步骤

在第一个 test 里面我们改写 `mocks.fs.readFileSync` 的返回形式，这里使用的 `mockImplementation` 是直接模拟了一个执行函数，当然也可以模拟返回值，具体可以到 jest 官网

`expect` 用来断言我们的 `console.log` 方法执行了

解释了这么多测试新手们应该也都看的明白了，下面聊一下错在哪，怎么改进

1. `mockImplementation` 最好替换为 `mockReturnValueOnce`，注意这里出现了 Once 结尾，也就是仅模拟一次返回值，`mockImplementation` 最好使用在复杂场景，所谓的复杂就是我们手动实现一个 `readFileSync` 方法使得测试达到我们预期的目的，在这个简单的场景里面我们只需要模拟返回值就好
2. `expect(console.log)` 这里会报错，因为 jest 断言的内容只能是 mock function 或 spy，这里 console 是全局对象 global 上的方法，我们没有 require 将其引入，所以 jest.mock 显然处理上有些吃力，这时候 spy 就派上用场了，`beforeAll` 钩子里直接执行 `jest.spyOn(global.console, 'log')`，接下来我们就能监听到 `console.log` 的执行了 `expect(global.console.log)`
3. 断言的目的是测试 `console.log` 的执行，这是不严谨的测试，我们需要使用 `toBeCalledWith` 来代替 `toBeCalled`，不仅要测试执行了，而且要测试参数正确，简单修改为 `expect(global.console.log).toBeCalledWith(this.mocks.other.text)`

下面补一下 read file 失败的测试

```js
test('read file fail throw error', () => {
  mocks.fs.readFileSync.mockImplementationOnce(() => { throw new Error('readFile error') })

  expect(getFile()).toThrow()
  expect(global.console.log).not.toBeCalled()
})
```

读取文件失败的测试就好理解的多，注意的就是对一个 `jest.fn()` 多次进行修改会导致测试用例之间的相互影响，这里尽量使用 Once 结尾方法，复杂场景可以如下

```js
beforeEach(() => {
  mocks.fs.readFileSync.mockReset()
})
```

每次执行 test 前先清除 mock，避免多个测试用例之间复杂化 mock 导致错误

> 小结：单元测试中的 mock 是个测试思路，我们无需关心外部文件和依赖是什么，只要能模拟出正确的情况程序是否按规则执行，错误的情况程序是否有异常处理，逻辑是否正确等。这样就能排除外界干扰，使得我们测试的当前一小部分是可靠的，稳定的即可。

### 引用外部文件

单拿出一个小结说下 require 的问题，node 9 之前不支持 es6 的 import，这里也不详细说明了。

require 本身并不复杂，但是如果搞不清楚执行时机，那么测试将无法进行，来一个例子

```js
const env = process.env.NODE_ENV

module.export = () => env
```

测试如下

```js
const getEnv = require('./getEnv')

describe('env', () => {
  test('env will be dev', () => {
    process.env.NODE_ENV = 'dev'

    expect(getEnv()).toBe('dev')
  })

  test('env will be pord', () => {
    process.env.NODE_ENV = 'pord'

    expect(getEnv()).toBe('pord')
  })
})
```

十分简单的测试，抛开了 mock 的流程，这里会报测试未通过，原因是 require 同时 env 已经被赋值为 `undefined`，我们再试着改变 `NODE_ENV` 环境变量时，程序不会再次执行，当然了，处理起来也十分简单

```js
let getEnv

test('env will be dev', () => {
  process.env.NODE_ENV = 'dev'
  getEnv = require('./getEnv')

  expect(getEnv()).toBe('dev')
})

test('env will be pord', () => {
  process.env.NODE_ENV = 'pord'
  getEnv = require('./getEnv')

  expect(getEnv()).toBe('pord')
})
```

顺带说了一下，希望大家不要在这种低级错误上浪费时间

其实引用外部文件还有些场景会对测试带来困惑，比如动态路径，场景如下

```js
const packageFile = `${process.cwd()}/package.json`

const package = require(packageFile)
```

读取当前路径下的 `package.json`，当测试真正跑到这段代码时会到当前目录下找 `package.json`，这里尽量 mock 掉 `package.json` 为我们自己的模拟数据，但是 jest 不支持动态路径的 mock，试着这样写 `jest.mock(`${process.cwd()}/package.json`, () => mockFile)` 会报错，所以尽量使用可以 mock 的方案，保证单元测试可以顺利进行，修改如下

```js
const path = require('path')

const filePath = path.join(process.cwd(), 'package.json')
```

这样就可以 mock，`path` 了，和上面 mock 章节，大致思想都差不多

### 覆盖率

单元测试覆盖率不达标等于白测，测试过程尽量覆盖所有判断条件，而不是全部通过了就不管了，在进一阶说，100% 的测试覆盖率并不证明一定覆盖到位了，因为顺带执行的代码也会算进覆盖率，例如

```js
module.export = (list) => list.map(({ id }) => id)
```

我们先不考虑这个 list 类型是不是数组，只是简单的例子，避免过度设计带来复杂化，我们测试可以这样

```js
const getId = require('./getId')
const mocks = {
  list: [{
    id: 1,
    name: 'vue'
  }, {
    id: 2,
    name: 'react'
  }]
}

test('return id', () => {
  expect(getId(mocks.list)).toEqual([1, 2])
})
```

直到有一天代码变成了 `module.export = (list) => [1, 2]`

这时候测试还能通过，并且覆盖率 100%，的确不会有人蠢到把代码改成这样，只是一个例子，实际上逻辑会比这个复杂的多

那就聊一聊解决方案

* mock 数据的随机化，每次测试生成随机的 list 进行测试，现有库 [mockjs](https://github.com/nuysoft/Mock/wiki/Getting-Started)
* 强关联测试，证明 map 方法的确执行了，并且参数正确，先 spy `spyOn(Array.prototype, 'map')` 然后断言

聊了一圈从覆盖率聊到了测试健壮性的问题，可以思考下写过的测试是否真的满足注释或修改任何一行代码都能引起测试的 pass 报错

关于 node 就聊这么多，其实下文主要思想都一样，更多的是介绍些简单可行的方案，以及可能会踩坑的地方

## vue 环境

在 vue 使用场景下，无非就是组件库和业务逻辑，组件库偏向于 unit 测试，业务逻辑偏向于 e2e 测试，当然两者并不冲突

### unit 测试

推荐神器：[vue-test-utils](https://github.com/vuejs/vue-test-utils)

README 给了多个测试库配置的例子，这里还是推荐使用 jest，给个例子

```js
export default {
  props: ['value'],
  data () {
    return {
      currentValue: 0
    }
  },
  watch: {
    value (val) {
      this.currentValue = val
    }
  }
}
```

测试如下

```js
import { mount } from '@vue/test-utils'
import Test from './Test.vue'

test('props value', () => {
  const options = { propsData: { value: 3 } }

  const wrapper = mount(Test)

  expect(wrapper.vm.currentValue).toBe(3)
})
```

十分简单的例子，亮点在测试文件的 wrapper 上，通过 `mount` 方法创建了一个组件实例，创建过程中允许加入一些配置信息，甚至是 mock 组件中的 method 方法

vue 单元测试的范围仅限于数据流动是否正确，逻辑渲染是否正确（v-if v-show v-for），style 和 class 是否正确，我们并不需要关系这个组件在浏览器渲染中的位置，也不需要关系对其它组件会造成什么影响，只要保证组件本身正确即可，前面说的断言，vue-test-utils 都能提供对应的方案，总体上节约很多测试成本

### e2e 测试

也是推荐尤大基于最新脚手架的 [@vue/cli-plugin-e2e-nightwatch](https://www.npmjs.com/package/@vue/cli-plugin-e2e-nightwatch)

e2e 测试的重点在于判断真实 DOM 是否满足预期要求，甚至很少出现 mock 场景，不可或缺的是一个浏览器运行环境，具体细节不赘述，可以看官方文档。


## nuxt 服务端渲染环境

nuxt 官方推荐 [ava](https://github.com/avajs/ava)，顺势带出 ava 的方案

### unit 测试

麻烦在配置上面，先给出需要安装的依赖

```js
"@vue/test-utils",
"ava",
"browser-env",
"require-extension-hooks",
"require-extension-hooks-babel",
"require-extension-hooks-vue",
"sinon"
```

在 package.json 里加几行 ava 配置

```js
"ava": {
  "require": [
    "./tests/helpers/setup.js"
  ]
}
```

下面来写 `./tests/helpers/setup.js`

```js
const hooks = require('require-extension-hooks')

// Setup browser environment
require('browser-env')()

// Setup vue files to be processed by `require-extension-hooks-vue`
hooks('vue').plugin('vue').push()
// Setup vue and js files to be processed by `require-extension-hooks-babel`
hooks(['vue', 'js']).plugin('babel').push()
```

上面的代码唯独没看到 `sinon` 这个库，说到 ava 是没有 mock 功能的，这就给单元测试的 mock 带来巨大困难，不过我们可以通过引入 `sinon` 来解决 mock 数据的问题，在 mock 方面上 `sinon` 做的比 jest 还要优秀，支持沙箱模式，不影响外部数据

给个简单点的例子

```html
<template>
  <el-card v-for="item in topicList" :key="item.id">
    <div class="card-content">
      <span class="link" @click="toMember(item.member.username)">{{ item.member.username }}</span>
    </div>
  </el-card>
</template>

<script>
export default {
  props: {
    topicList: {
      type: Array,
      required: true
    }
  },
  methods: {
    toMember (name) {
      this.$router.push(`/member/${name}`)
    }
  }
}
</script>
```

对应的测试代码如下

```js
import { shallowMount } from '@vue/test-utils'
import test from 'ava'
import sinon from 'sinon'

test('methods: toMember', t => {
  const { topicList } = t.context
  const $router = {
    push: () => {}
  }
  const spy = sinon.spy($router, 'push')

  const wrapper = shallowMount(TopicListChalk, {
    propsData: { topicList },
    mocks: {
      $router
    }
  })

  topicList.forEach((item, index) => {
    const toMemberText = wrapper.findAll('.card-content').at(index).find('.link')

    toMemberText.trigger('click')

    t.true(spy.withArgs(`/member/${item.member.username}`).calledOnce)
  })
})
```

这里直接将 `$router` mock 掉，并且使用 `sinon.spy` 监听执行，至于 `this.$router.push` 后浏览器有没有跳转并不是单元测试需要关心的，这里的写法也比较特别，test 方法在回调里默认参数为 `t`，对应的方法都挂载在 `t` 对象上，上下文可通过 `t.context` 传递

nuxt 单元测试相关就聊这么多

### e2e 测试

这里有个歧义点，nuxt 官网只给出了 e2e 的测试案例 [end-to-end-testing](https://nuxtjs.org/guide/development-tools#end-to-end-testing)

当使用默认脚手架构建的项目，也就是没有 server 端入口文件的项目，这个方案确实可行

但是涉及到其它框架（express|koa）的时候就显得不够用了，很有可能在自定义 server 入口是加入了大量中间件，这对于官网给出的例子是个巨大考验，不可能在每个测试文件里实现一遍 `new Nuxt`，所以需要更高层的封装，也就是忽略 server 启动流程的差异性，直接在浏览器中抓取页面

推荐：[nuxt-jest-puppeteer](https://github.com/studbits/nuxt-jest-puppeteer)

## react 环境

### unit 测试

这一波没得可选，jest 完胜，人家官网就有 [React，RN 的支持文档](https://facebook.github.io/jest/docs/en/tutorial-react.html)

文档的案例也是十分全面，没得讲，不赘述

### e2e 测试

其实上面讲了两个 e2e 的方案选择，大同小异，需要一个能在 node 跑的无头浏览器，官方没有推荐，这里站 vue 一票选择 [nightwatchjs](http://nightwatchjs.org/)

## next 服务端渲染环境

### unit 测试

主要讲一下如何配置，先是依赖包

```js
"babel-core",
"babel-jest",
"enzyme",
"enzyme-adapter-react-16",
"jest",
"react-addons-test-utils",
"react-test-renderer"
```

在 package.json 里面加 script `"test": "NODE_ENV=test jest"`

在跟路径下加 jest.config.js

```js
module.exports = {
  setupFiles: ['<rootDir>/jest.setup.js'],
  testPathIgnorePatterns: ['<rootDir>/.next/', '<rootDir>/node_modules/']
}
```

在跟路径下加 jest.setup.js

```js
import { configure } from 'enzyme'
import Adapter from 'enzyme-adapter-react-16'

configure({
  adapter: new Adapter()
})
```

接下来就可以愉快的写测试了

### e2e 测试

跳过了～～～

## angular 环境

之所以加了这一节，还是因为多少写过一些 angular，angular 作为框架本身就是全面的，cli 新建的项目自身就带有 unit 测试和 e2e 测试

unit 测试默认是 [karma](https://karma-runner.github.io/2.0/index.html) + [jasmine](https://jasmine.github.io/)
e2e 测试默认是 [protractor](https://www.protractortest.org/#/)

也没什么可争辩的，这就是官方解决方案，用起来也方便顺手

## 总结

聊了好多个环境，其实行文目的主要有两方面

* 测试思想，如何写好单元测试，主要集中在前半文
* 测试工具推荐和相应配置

测试本身并不复杂，但是想写出高效测试并不容易，千万不要形成为了测试而测试的想法

> 用谎言去验证谎言得到的还是谎言。。。

大多数情况下都是项目在赶进度没空写测试，抽空把测试补上真的是一件值得去做的事情
