---
title: 开发 eslint 规则
date: 2018-09-29 17:22:32
tags: ['eslint']
---

![](/uploads/eslint-rule1.png)

<!--more-->

前端的日常开发离不开各种 lint 的支持，使用 lint 的一种误解是：个人能力不足，必须 lint 规范才能写出规范的代码，实际上规范的定义主要取决于开源项目作者的习惯，或者公司团队编码的习惯，即使两个前端专家，写出的代码规范也会有差别。

今天主题聊聊 eslint，作为最主流的 JavaScript lint 工具深受大家喜爱，而 JSHint 却逐渐淡出了大家的视线，使用的比较少了

常用的 eslint 扩展有 standard，airbnb 等

## 剖析 eslint 扩展

扩展无非就作两个事情

* 在原有的 eslint 的基础上配置些 config（具体规则参数，全局变量，运行环境等）
* 自定义些自己的 rule，以满足需求

原理就是利用 eslint 的继承模式，理论上可以无限继承并覆盖上一级的规则

第一条不详细介绍了，eslint 官网说的十分详细，基本每一条规则都支持自定义参数，覆盖面也特别广，基本上所有的语法都有 rule

第二条的自定义 rule 才是本文的重头戏，因为特殊的业务场景靠 eslint 自身配置已经无法满足业务需求了，如：

* eslint-plugin-vue
* eslint-plugin-react
* eslint-plugin-jest

一般特殊场景的自定义规则都使用 `eslint-plugin-*` 的命名，使用时可以方便的写成

```js
{
  plugins: [
    'vue',
    'react',
    'jest'
  ]
}
```

当然 eslint-config-* 同理，不过配置时需要写成

```js
{
  extends: 'standard'
}
```

下面介绍下开发流程

## 创建 eslint plugin 工程

官方推荐使用 yeoman 生成项目，感觉生成的项目比较守旧，推荐下习惯我的项目结构

```
eslint-plugin-skr
  |- __tests__
  |  |- rules
  |  |- utils
  |
  |- lib
  |  |- rules
  |  |- utils
  |  |- index.js
  |
  |- jest.config.js
  |
  |- package.json
  |
  |- README.md
```

整体看下来发现多了 jest 配置文件，是的 yeoman 生成的项目默认采用 Mocha 作为测试框架，个人感觉调试起来麻烦，没有 jest 灵活，vscode 轻松搞定调试

教程一搜一大把哈，给伸手党一个链接 [debugging-jest-tests](https://github.com/Microsoft/vscode-recipes/tree/master/debugging-jest-tests)

关于 jest 的 config 文件也po出来一下，都是些基本的配置，复杂的用不到，下面会详细介绍测试部分

```js
module.exports = {
  testEnvironment: 'node',
  roots: ['__tests__'],
  resetModules: true,
  clearMocks: true,
  verbose: true
}
```

自定义的规则全部在 lib/rules 下面，每条规则单独一个文件足以

下面一个简单的例子打通任督二脉

## 开发一个规则

前期准备

* [官方开发文档](https://eslint.org/docs/developer-guide/working-with-rules)
* AST 抽象语法树

这个官方文档写的密密麻麻，好几十个属性，其实只是冰山一角，有很多复杂场景需要考虑

有人疑问：一定需要精通 AST？

我的回答是当然不需要，简单了解便是，最起码知道解析出来的语法树大体结构长什么样子

那就随便给自己一个命题写吧！写个超级简单的

```js
module.exports = {
  meta: {
    docs: {
      description: '禁止块级注释',
      category: 'Stylistic Issues',
      recommended: true
    }
  },

  create (context) {
    const sourceCode = context.getSourceCode()

    return {
      Program () {
        const comments = sourceCode.getAllComments()

        const blockComments = comments.filter(({ type }) => type === 'Block')

        blockComments.length && context.report({
          message: 'No block comments'
        })
      }
    }
  }
}
```

具体写法官方文档有介绍哈，就不赘述了，例子也十分简单，调用了环境变量 context 中的方法获取全部注释

## 稍微复杂点的场景

如需要 lint `bar` 对象中属性的顺序，如下假设一个规则

```js
// good
const bar = {
  meta: {},
  double: num => num * 2
}

// bed
const bar = {
  double: num => num * 2,
  meta: {},
}
```

这个第一次些会有些蒙，官网没有提供具体的例子，解决办法很简单，推荐一个利器 [astexplorer](https://astexplorer.net/)

点进去别着急复制代码查看 AST 结果，首先选择 espree（eslint 使用的语法解析库），如下

![](/uploads/eslint-rule2.png)

这短短的四行代码会对应着一个抽象语法树，如下图：

![](/uploads/eslint-rule3.png)

由于全展开太长了哈，感兴趣的自行尝试，会发现层级嵌套的特别深，找到 `bar` 的属性需要 `Program.body[0].declarations[0].init.properties`

当然不至于每次都从最顶级的 `Program` 找下来，从上面的例子可以看出 `create` 方法的 `return` 返回的是一个 object，里面可以定义很多检测类型，如官网的例子：

```js
function checkLastSegment (node) {
  // report problem for function if last code path segment is reachable
}

module.exports = {
  meta: { ... },
  create: function(context) {
    // declare the state of the rule
    return {
      ReturnStatement: function(node) {
        // at a ReturnStatement node while going down
      },
      // at a function expression node while going up:
      "FunctionExpression:exit": checkLastSegment,
      "ArrowFunctionExpression:exit": checkLastSegment,
      onCodePathStart: function (codePath, node) {
        // at the start of analyzing a code path
      },
      onCodePathEnd: function(codePath, node) {
        // at the end of analyzing a code path
      }
    }
  }
}
```

这里可以使用 `VariableDeclarator` 类型作为检察目标，从下面的解析树可以分析出筛选条件

![](/uploads/eslint-rule4.png)

以 `VariableDeclarator` 对象作为当前的 `node`

当变量名为 `bar`，即 `node.id.name === 'bar'`，且值为对象，即 `node.init.type === 'ObjectExpression'`，代码如下：

```js
module.exports = {
  meta: { ... },
  create (context) {
    return {
      VariableDeclarator (node) {
        const isBarObj = node.id.name === 'bar' &&
          node.init.type === 'ObjectExpression'

        if (!isBarObj) return

        // checker
      }
    }
  }
}
```

就这样成功取到 `bar` 对象后就可以检测属性的顺序了，排序算法一大把，挑一个喜欢的用就行了，这里不啰嗦了，直接上结果：

```js
const ORDER = ['meta', 'double']

function getOrderMap () {
  const orderMap = new Map()

  ORDER.forEach((name, i) => {
    orderMap.set(name, i)
  })

  return orderMap
}

module.exports = {
  create (context) {
    const orderMap = getOrderMap()

    function checkOrder (propertiesNodes) {
      const properties = propertiesNodes
        .filter(property => property.type === 'Property')
        .map(property => property.key)

      properties.forEach((property, i) => {
        const propertiesAbove = properties.slice(0, i)
        const unorderedProperties = propertiesAbove
          .filter(p => orderMap.get(p.name) > orderMap.get(property.name))
          .sort((p1, p2) => orderMap.get(p1.name) > orderMap.get(p2.name))

        const firstUnorderedProperty = unorderedProperties[0]

        if (firstUnorderedProperty) {
          const line = firstUnorderedProperty.loc.start.line

          context.report({
            node: property,
            message: `The "{{name}}" property should be above the "{{firstUnorderedPropertyName}}" property on line {{line}}.`,
            data: {
              name: property.name,
              firstUnorderedPropertyName: firstUnorderedProperty.name,
              line
            }
          })
        }
      })
    }

    return {
      VariableDeclarator (node) {
        const isBarObj = node.id.name === 'bar' &&
          node.init.type === 'ObjectExpression'

        if (!isBarObj) return

        checkOrder(node.init.properties)
      }
    }
  }
}
```

这里代码有点多，耐心看完其实挺简单的，大致解释下

`getOrderMap` 方法将数组转成 Map 类型，方面通过 `get` 获取下标，这里也可以处理多纬数组，例如两个 `key` 希望在相同的排序等级，不分上下，可以写成：

```js
const order = [
  'meta'
  ['double', 'treble']
]

function getOrderMap () {
  const orderMap = new Map()

  ORDER.forEach((name, i) => {
    if (Array.isArray(property)) {
      property.forEach(p => orderMap.set(p, i))
    } else {
      orderMap.set(property, i)
    }
  })

  return orderMap
}
```

这样 `double` 和 `treble` 就拥有相同的等级了，方便后面扩展，当然实际情况会有 n 个属性的排序规则，也可以在这个规则上轻松扩展，内部的 sort 逻辑就不赘述了。

开发就介绍到这里，通过上面安利的在线语法解析工具可以轻松反推出 lint 逻辑。

如果 rule 比较复杂，就需要大量的 utils 支持，不然每个 rule 都会显得一团糟，比较考验公共代码提取的能力

## 测试

如前面所讲建议使用 jest 进行测试，这里的测试和普通的单元测试还不太一样，eslint 是基于结果的测试，什么意思呢？

lint 只有两种情况，通过与不通过，只需要把通过和不通过的情况整理成两个数组，剩下的工作交给 eslint 的 `RuleTester` 处理就行了

上面的属性排序 rule，测试如下：

```js
const RuleTester = require('eslint').RuleTester
const rule = require('../../lib/rules/test')

const ruleTester = new RuleTester({
  parserOptions: {
    ecmaVersion: 6
  }
})

ruleTester.run('test rule', rule, {
  valid: [
    `const bar = {
      meta: {},
      double: num => num * 2
    }`
  ],
  invalid: [
    {
      code: `const bar = {
        double: num => num * 2,
        meta: {},
      }`,
      errors: [{
        message: 'The "meta" property should be above the "double" property on line 2.'
      }]
    }
  ]
})
```

`valid` 中是希望通过的代码，`invalid` 中是不希望通过的代码和错误信息，到这里一个 rule 算是真正完成了。

## 打包输出

最后写好的 rules 需要发一个 npm 包，以便于在项目中使用，这里就不赘述怎么发包了，简单聊聊怎么优雅的把 rules 导出来。

直接上代码：

```js
const requireIndex = require('requireindex')

// import all rules in lib/rules
module.exports.rules = requireIndex(`${__dirname}/rules`)
```

这里使用了三方依赖 `requireindex`，对于批量的导出一个文件夹内的所有文件显得简洁很多。

当然前提是保证 rules 文件夹下都是 rule 文件，不要把 utils 写进去哈

## 总结

行文目的是国内外对于自定义 eslint rule 的相关资源较少，希望分享一些写自定义规则的经验。

千万不要在学习 AST 上浪费时间，不同的库对 AST 的实现是不同的，下次写 babel 插件又要学其它的 AST 规则，再次安利一下 AST 神器 [astexplorer](https://astexplorer.net/)，只要把需要验证的代码放到 `astexplorer` 里跑一遍，然后总结出规律，逻辑其实十分简单，对 AST 结果进行判断就行了。

从团队层面讲，希望所有的团队都有自己的 eslint 规则库，可以大大降低 code review 的成本，又能保证代码的一致性，一劳永逸的事情。
