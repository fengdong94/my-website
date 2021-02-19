---
id: react
title: 前端面试宝典之 React 篇
---

## 01｜React是什么/谈一谈对React的理解

概念题通解：

- 讲概念：用简洁的话说清楚该技术是什么。最好能用一句话描述。
- 说用途：描述该技术的用途。能够具体结合适合场景，拓展性的描述。
- 理思路：梳理该技术的核心思路或运作流程。这个地方可深可浅，如果对其有足够深入的了解，建议详细地展开说明。
- 优缺点：对该技术栈的优缺点进行列举。适当与其他技术方案横向对比。

答题：

> React 是一个网页 UI 框架，通过组件化的方式解决视图层开发复用的问题，本质是一个组件化框架。
>> 它的核心设计思路有三点，分别是声明式、组件化与通用性。  
声明式的优势在于直观与组合。  
组件化的优势在于视图的拆分与模块复用，可以更容易做到高内聚低耦合。  
通用性在于一次学习，随处编写。比如 React Native，React 360 等，这里主要靠虚拟 DOM 来保证实现。这使得 React 的适用范围变得足够广，无论是 Web、Native、VR，甚至 Shell 应用都可以进行开发。这也是 React 的优势。  
> 但作为一个视图层的框架，React 的劣势也十分明显。它并没有提供完整的一揽子解决方案，在开发大型前端应用时，需要向社区寻找并整合解决方案。虽然一定程度上促进了社区的繁荣，但也为开发者在技术选型和学习适用上造成了一定的成本。

补充：

- 承接在优势后，可以再谈一下自己对于 React 优化的看法、对虚拟 DOM 的看法。
- 向自己主导过的 React 项目上引导，谈一谈 React 相关的工程架构与设计模式。

## 02｜为什么 React 要用 JSX？

解题思路：一句话解释，核心概念，方案对比。

答题：

> 在回答问题之前，我首先解释下什么是 JSX 吧。JSX 是一个 JavaScript 的语法扩展，结构类似 XML。
>> JSX 主要用于声明 React 元素，但 React 中并不强制使用 JSX。即使使用了 JSX，也会在构建过程中，通过 Babel 插件编译为 React.createElement。所以 JSX 更像是 React.createElement 的一种语法糖。
>
> 从这里可以看出，React 团队并不想引入 JavaScript 本身以外的开发体系。而是希望通过合理的关注点分离保持组件开发的纯粹性。
>
> 接下来与 JSX 以外的三种技术方案进行对比。
>
> 首先是模板，React 团队认为模板不应该是开发过程中的关注点，因为引入了模板语法、模板指令等概念，是一种不佳的实现方案。
>
> 其次是模板字符串，模板字符串编写的结构会造成多次内部嵌套，使整个结构变得复杂，并且优化代码提示也会变得困难重重。
>
> 最后是 JXON，同样因为代码提示困难的原因而被放弃。
>
> 所以 React 最后选用了 JSX，因为 JSX 与其设计思想贴合，不需要引入过多新的概念，对编辑器的代码提示也极为友好。

进阶：Babel 插件如何实现 JSX 到 JS 的编译？

Babel 读取代码并解析，生成 AST，再将 AST 传入插件层进行转换，在转换时就可以将 JSX 的结构转换为 React.createElement 的函数。如下代码所示：

```js
module.exports = function (babel) {
  var t = babel.types;
  return {
    name: "custom-jsx-plugin",
    visitor: {
      JSXElement(path) {
        var openingElement = path.node.openingElement;
        var tagName = openingElement.name.name;
        var args = []; 
        args.push(t.stringLiteral(tagName)); 
        var attribs = t.nullLiteral(); 
        args.push(attribs); 
        var reactIdentifier = t.identifier("React"); //object
        var createElementIdentifier = t.identifier("createElement"); 
        var callee = t.memberExpression(reactIdentifier, createElementIdentifier)
        var callExpression = t.callExpression(callee, args);
        callExpression.arguments = callExpression.arguments.concat(path.node.children);
        path.replaceWith(callExpression, path.node); 
      },
    },
  };
};
```

## 03｜如何避免生命周期中的坑？

![lifeCircle](imgs/lifeCircle.png)

答题：

> 避免生命周期中的坑需要做好两件事：  
> 不在恰当的时候调用了不该调用的代码；  
> 在需要调用时，不要忘了调用。
>
> 那么主要有这么 7 种情况容易造成生命周期的坑。
>
> getDerivedStateFromProps 容易编写反模式代码，使受控组件与非受控组件区分模糊。
>
> componentWillMount 在 React 中已被标记弃用，不推荐使用，主要原因是新的异步渲染架构会导致它被多次调用。所以网络请求及事件绑定代码应移至 componentDidMount 中。
>
> componentWillReceiveProps 同样被标记弃用，被 getDerivedStateFromProps 所取代，主要原因是性能问题。
>
> shouldComponentUpdate 通过返回 true 或者 false 来确定是否需要触发新的渲染。主要用于性能优化。
>
> componentWillUpdate 同样是由于新的异步渲染机制，而被标记废弃，不推荐使用，原先的逻辑可结合 getSnapshotBeforeUpdate 与 componentDidUpdate 改造使用。
>
> 如果在 componentWillUnmount 函数中忘记解除事件绑定，取消定时器等清理操作，容易引发 bug。
>
> 如果没有添加错误边界处理，当渲染发生异常时，用户将会看到一个无法操作的白屏，所以一定要添加。

进阶：React 的请求应该放在哪里，为什么?

对于异步请求，应该放在 componentDidMount 中去操作。从时间顺序来看，除了 componentDidMount 还可以有以下选择：

- constructor：可以放，但从设计上而言不推荐。constructor 主要用于初始化 state 与函数绑定，并不承载业务逻辑。而且随着类属性的流行，constructor 已经很少使用了。
- componentWillMount：已被标记废弃，在新的异步渲染架构下会触发多次渲染，容易引发 Bug，不利于未来 React 升级后的代码维护。

所以React 的请求放在 componentDidMount 里是最好的选择。

## 04 | 类组件与函数组件有什么区别呢？

类组件的模式并不能很好地适应未来的趋势：

- this 的模糊性；
- 业务逻辑散落在生命周期中；
- React 的组件代码缺乏标准的拆分方式。

答题：

> 作为组件而言，类组件与函数组件在使用与呈现上没有任何不同，性能上在现代浏览器中也不会有明显差异。
>
> 它们在开发时的心智模型上却存在巨大的差异。类组件是基于面向对象编程的，它主打的是继承、生命周期等核心概念；而函数组件内核是函数式编程，主打的是 immutable、没有副作用、引用透明等特点。
>
> 之前，在使用场景上，如果存在需要使用生命周期的组件，那么主推类组件；设计模式上，如果需要使用继承，那么主推类组件。
>
> 但现在由于 React Hooks 的推出，生命周期概念的淡出，函数组件可以完全取代类组件。
>
> 其次继承并不是组件最佳的设计模式，官方更推崇“组合优于继承”的设计概念，所以类组件在这方面的优势也在淡出。
>
> 性能优化上，类组件主要依靠 shouldComponentUpdate 阻断渲染来提升性能，而函数组件依靠 React.memo 缓存渲染结果来提升性能。
>
> 从上手程度而言，类组件更容易上手，从未来趋势上看，由于React Hooks 的推出，函数组件成了社区未来主推的方案。
>
> 类组件在未来时间切片与并发模式中，由于生命周期带来的复杂度，并不易于优化。而函数组件本身轻量简单，且在 Hooks 的基础上提供了比原先更细粒度的逻辑组织与复用，更能适应 React 的未来发展。

## 05 | 如何设计 React 组件？

解法：围绕“如何组合”这一核心主题，通过列举场景的方式展现设计模式的分类及用途。

答题：

> React 组件应从设计与工程实践两个方向进行探讨。
>
> 从设计上而言，社区主流分类的方案是展示组件与灵巧组件。
>
> 展示组件内部没有状态管理，仅仅用于最简单的展示表达。展示组件中最基础的一类组件称作代理组件。代理组件常用于封装常用属性、减少重复代码。很经典的场景就是引入 Antd 的 Button 时，你再自己封一层。如果未来需要替换掉 Antd 或者需要在所有的 Button 上添加一个属性，都会非常方便。基于代理组件的思想还可以继续分类，分为样式组件与布局组件两种，分别是将样式与布局内聚在自己组件内部。
>
> 灵巧组件由于面向业务，其功能更为丰富，复杂性更高，复用度低于展示组件。最经典的灵巧组件是容器组件。在开发中，我们经常会将网络请求与事件处理放在容器组件中进行。容器组件也为组合其他组件预留了一个恰当的空间。还有一类灵巧组件是高阶组件。高阶组件被 React 官方称为 React 中复用组件逻辑的高级技术，它常用于抽取公共业务逻辑或者提供某些公用能力。常用的场景包括检查登录态，或者为埋点提供封装，减少样板代码量。高阶组件可以组合完成链式调用，如果基于装饰器使用，就更为方便了。高阶组件中还有一个经典用法就是反向劫持，通过重写渲染函数的方式实现某些功能，比如场景的页面加载圈等。但高阶组件也有两个缺陷，第一个是静态方法不能被外部直接调用，需要通过向上层组件复制的方式调用，社区有提供解决方案，使用 hoist-non-react-statics 可以解决；第二个是 refs 不能透传，使用 React.forwardRef API 可以解决。
>
> 从工程实践而言，通过文件夹划分的方式切分代码。我初步常用的分割方式是将页面单独建立一个目录，将复用性略高的 components 建立一个目录，在下面分别建立 basic、container 和 hoc 三类。这样可以保证无法复用的业务逻辑代码尽量留在 Page 中，而可以抽象复用的部分放入 components 中。其中 basic 文件夹放展示组件，由于展示组件本身与业务关联性较低，所以可以使用 Storybook 进行组件的开发管理，提升项目的工程化管理能力。

进阶：如何在渲染劫持中为原本的渲染结果添加新的样式？

```js
function withLoading(WrappedComponent) {
    return class extends WrappedComponent {
        render() {
            if(this.props.isLoading) {
                return <Loading />;
            } else {
                const elementsTree = super.render();
                return (
                    <div className={xxx}>
                        {elementsTree}
                    </div>
                );
            }
        }
    };
}
```

## 06 | setState 是同步更新还是异步更新？

回答：

> setState 并非真异步，只是看上去像异步。在源码中，通过 isBatchingUpdates 来判断 setState 是先存进 state 队列还是直接更新，如果值为 true 则执行异步操作，为 false 则直接更新。
>
> 那么什么情况下 isBatchingUpdates 会为 true 呢？在 React 可以控制的地方，就为 true，比如在 React 生命周期事件和合成事件中，都会走合并操作，延迟更新的策略。
>
> 但在 React 无法控制的地方，比如原生事件，具体就是在 addEventListener 、setTimeout、setInterval 等事件中，就只能同步更新。
>
> 一般认为，做异步设计是为了性能优化、减少渲染次数，React 团队还补充了两点：
>
> 1. 保持内部一致性。如果将 state 改为同步更新，那尽管 state 的更新是同步的，但是 props不是。
> 2. 启用并发更新，完成异步渲染。

进阶：下面的代码输出什么呢？

```js
class Test extends React.Component {
  state  = {
      count: 0
  };

  componentDidMount() {
    this.setState({count: this.state.count + 1});
    console.log(this.state.count);

    this.setState({count: this.state.count + 1});
    console.log(this.state.count);

    setTimeout(() => {
      this.setState({count: this.state.count + 1});
      console.log(this.state.count);

      this.setState({count: this.state.count + 1});
      console.log(this.state.count);
    }, 0);
  }
 
  render() {
    return null;
  }
};
```

0 0 2 3  
虽然 count 在前面经过了两次的 this.state.count + 1，但是每次获取的 this.state.count 都是初始化时的值，也就是 0；所以此时 count 是 1，那么后续在 setTimeout 中的输出则是 2 和 3。

## 07 | 如何面向组件跨层级通信？

答题：

> 在跨层级通信中，主要分为一层或多层的情况。
>
> 如果只有一层，那么按照 React 的树形结构进行分类的话，主要有以下三种情况：父组件向子组件通信，子组件向父组件通信以及平级的兄弟组件间互相通信。
>
> 在父与子的情况下，因为 React 的设计实际上就是传递 Props 即可。那么场景体现在容器组件与展示组件之间，通过 Props 传递 state，让展示组件受控。
>
> 在子与父的情况下，有两种方式，分别是回调函数与实例函数。回调函数，比如输入框向父级组件返回输入内容，按钮向父级组件传递点击事件等。实例函数的情况有些特别，主要是在父组件中通过 React 的 ref API 获取子组件的实例，然后是通过实例调用子组件的实例函数。这种方式在过去常见于 Modal 框的显示与隐藏。这样的代码风格有着明显的 jQuery 时代特征，在现在的 React 社区中已经很少见了，因为流行的做法是希望组件的所有能力都可以通过 Props 控制。
>
> 多层级间的数据通信，有两种情况。第一种是一个容器中包含了多层子组件，需要最底部的子组件与顶部组件进行通信。在这种情况下，如果不断透传 Props 或回调函数，不仅代码层级太深，后续也很不好维护。第二种是两个组件不相关，在整个 React 的组件树的两侧，完全不相交。那么基于多层级间的通信一般有三个方案。
>
> 第一个是使用 React 的 Context API，最常见的用途是做语言包国际化。
>
> 第二个是使用全局变量与事件。全局变量通过在 Windows 上挂载新对象的方式实现，这种方式一般用于临时存储值，这种值用于计算或者上报，缺点是渲染显示时容易引发错误。全局事件就是使用 document 的自定义事件，因为绑定事件的操作一般会放在组件的 componentDidMount 中，所以一般要求两个组件都已经在页面中加载显示，这就导致了一定的时序依赖。如果加载时机存在差异，那么很有可能导致两者都没能对应响应事件。
>
> 第三个是使用状态管理框架，比如 Flux、Redux 及 Mobx。优点是由于引入了状态管理，使得项目的开发模式与代码结构得以约束，缺点是学习成本相对较高。

## 08 | 列举一种你了解的 React 状态管理框架

回答：

> 首先介绍 Flux，Flux 是一种使用单向数据流的形式来组合 React 组件的应用架构。
>
> Flux 包含了 4 个部分，分别是 Dispatcher、 Store、View、Action。Store 存储了视图层所有的数据，当 Store 变化后会引起 View 层的更新。如果在视图层触发一个 Action，就会使当前的页面数据值发生变化。Action 会被 Dispatcher 进行统一的收发处理，传递给 Store 层，Store 层已经注册过相关 Action 的处理逻辑，处理对应的内部状态变化后，触发 View 层更新。
>
> Flux 的优点是单向数据流，解决了 MVC 中数据流向不清的问题，使开发者可以快速了解应用行为。从项目结构上简化了视图层设计，明确了分工，数据与业务逻辑也统一存放管理，使在大型架构的项目中更容易管理、维护代码。
>
> 其次是 Redux，Redux 本身是一个 JavaScript 状态容器，提供可预测化状态的管理。社区通常认为 Redux 是 Flux 的一个简化设计版本，但它吸收了 Elm 的架构思想，更像一个混合产物。它提供的状态管理，简化了一些高级特性的实现成本，比如撤销、重做、实时编辑、时间旅行、服务端同构等。
>
> Redux 的核心设计包含了三大原则：单一数据源、纯函数 Reducer、State 是只读的。
>
> Redux 中整个数据流的方案与 Flux 大同小异。
>
> Redux 中的另一大核心点是处理“副作用”，AJAX 请求等异步工作，或不是纯函数产生的第三方的交互都被认为是 “副作用”。这就造成在纯函数设计的 Redux 中，处理副作用变成了一件至关重要的事情。社区通常有两种解决方案：
>
> 第一类是在 Dispatch 的时候会有一个 middleware 中间件层，拦截分发的 Action 并添加额外的复杂行为，还可以添加副作用。第一类方案的流行框架有 Redux-thunk、Redux-Promise、Redux-Observable、Redux-Saga 等。
>
> 第二类是允许 Reducer 层中直接处理副作用，采取该方案的有 React Loop，React Loop 在实现中采用了 Elm 中分形的思想，使代码具备更强的组合能力。
>
> 除此以外，社区还提供了更为工程化的方案，比如 rematch 或 dva，提供了更详细的模块架构能力，提供了拓展插件以支持更多功能。
>
> Redux 的优点很多：结果可预测；代码结构严格易维护；模块分离清晰且小函数结构容易编写单元测试；Action 触发的方式，可以在调试器中使用时间回溯，定位问题更简单快捷；单一数据源使服务端同构变得更为容易；社区方案多，生态也更为繁荣。
>
> 最后是 Mobx，Mobx 通过监听数据的属性变化，可以直接在数据上更改触发UI 的渲染。在使用上更接近 Vue，比起 Flux 与 Redux 的手动挡的体验，更像开自动挡的汽车。Mobx 的响应式实现原理与 Vue 相同，以 Mobx 5 为分界点，5 以前采用 Object.defineProperty 的方案，5 及以后使用 Proxy 的方案。它的优点是样板代码少、简单粗暴、用户学习快、响应式自动更新数据让开发者的心智负担更低。

个人观点部分：

> 我认为 Flux 的设计更偏向 Facebook 内部的应用场景，Facebook 的方案略显臃肿，拓展能力欠佳，所以在社区中热度不够。而 Redux 因为纯函数的原因，碰上了社区热点，简洁不简单的 API 设计使社区疯狂贡献发展，短短数年方案层出不穷。但从工程角度而言，不是每一个项目都适用单一数据源。因为很多项目的数据是按页面级别切分的，页面之间相对隔绝，并不需要共享数据源，是否需要 Redux 应该视具体情况而定。Mobx 在开发项目时简单快速，但应用 Mobx 的场景 ，其实完全可以用 Vue 取代。如果纯用 Vue，体积还会更小巧。

进阶：徒手实现一个 Redux

- createStore。即通过 createStore，注入 Reducer 与 middleware，生成 Store 对象。
- Store 对象的 getState、subscribe 与 dispatch 函数。getState 获取当前状态树，subscribe 函数订阅状态树变更，dispatch 发送 Action。

## 09 | Virtual DOM 的工作原理是什么？

回答：

> 虚拟 DOM 的工作原理是通过 JS 对象模拟 DOM 的节点。在 Facebook 构建 React 初期时，考虑到要提升代码抽象能力、避免人为的 DOM 操作、降低代码整体风险等因素，所以引入了虚拟 DOM。
>
> 虚拟 DOM 在实现上通常是 Plain Object，以 React 为例，在 render 函数中写的 JSX 会在 Babel 插件的作用下，编译为 React.createElement 执行 JSX 中的属性参数。
>
> React.createElement 执行后会返回一个 Plain Object，它会描述自己的 tag 类型、props 属性以及 children 情况等。这些 Plain Object 通过树形结构组成一棵虚拟 DOM 树。当状态发生变更时，将变更前后的虚拟 DOM 树进行差异比较，这个过程称为 diff，生成的结果称为 patch。计算之后，会渲染 Patch 完成对真实 DOM 的操作。
>
> 虚拟 DOM 的优点主要有三点：改善大规模 DOM 操作的性能、规避 XSS 风险、能以较低的成本实现跨平台开发。
>
> 虚拟 DOM 的缺点在社区中主要有两点。
>
> 内存占用较高，因为需要模拟整个网页的真实 DOM。
>
> 高性能应用场景存在难以优化的情况，类似像 Google Earth 一类的高性能前端应用在技术选型上往往不会选择 React。

进阶：除了渲染页面，虚拟 DOM 还有哪些应用场景？

这个问题考验面试者的想象力。通常而言，我们只是将虚拟 DOM 与渲染绑定在一起，但实际上虚拟 DOM 的应用更为广阔。比如，只要你记录了真实 DOM 变更，它甚至可以应用于埋点统计与数据记录等。可以往这个方向回答，具体案例可以参考 [rrweb](https://github.com/rrweb-io/rrweb)。

## 10 | 与其他框架相比，React 的 diff 算法有何不同？

答题：

> 在回答有何不同之前，首先需要说明下什么是 diff 算法。
>
> diff 算法是指生成更新补丁的方式，主要应用于虚拟 DOM 树变化后，更新真实 DOM。所以 diff 算法一定存在这样一个过程：触发更新 → 生成补丁 → 应用补丁。
>
> React 的 diff 算法，触发更新的时机主要在 state 变化与 hooks 调用之后。此时触发虚拟 DOM 树变更遍历，采用了深度优先遍历算法。但传统的遍历方式，效率较低。为了优化效率，使用了分治的方式。将单一节点比对转化为了 3 种类型节点的比对，分别是树、组件及元素，以此提升效率。
>
> 树比对：由于网页视图中较少有跨层级节点移动，两株虚拟 DOM 树只对同一层次的节点进行比较。
>
> 组件比对：如果组件是同一类型，则进行树比对，如果不是，则直接放入到补丁中。
>
> 元素比对：主要发生在同层级中，通过标记节点操作生成补丁，节点操作对应真实的 DOM 剪裁操作。
>
> 以上是经典的 React diff 算法内容。自 React 16 起，引入了 Fiber 架构。为了使整个更新过程可随时暂停恢复，节点与树分别采用了 FiberNode 与 FiberTree 进行重构。fiberNode 使用了双链表的结构，可以直接找到兄弟节点与子节点。
>
> 整个更新过程由 current 与 workInProgress 两株树双缓冲完成。workInProgress 更新完成后，再通过修改 current 相关指针指向新节点。
>
> 然后拿 Vue 和 Preact 与 React 的 diff 算法进行对比。
>
> Preact 的 Diff 算法相较于 React，整体设计思路相似，但最底层的元素采用了真实 DOM 对比操作，也没有采用 Fiber 设计。Vue 的 Diff 算法整体也与 React 相似，同样未实现 Fiber 设计。
>
> 然后进行横向比较，React 拥有完整的 Diff 算法策略，且拥有随时中断更新的时间切片能力，在大批量节点更新的极端情况下，拥有更友好的交互体验。
>
> Preact 可以在一些对性能要求不高，仅需要渲染框架的简单场景下应用。
>
> Vue 的整体 diff 策略与 React 对齐，虽然缺乏时间切片能力，但这并不意味着 Vue 的性能更差，因为在 Vue 3 初期引入过，后期因为收益不高移除掉了。除了高帧率动画，在 Vue 中其他的场景几乎都可以使用防抖和节流去提高响应性能。

进阶：如何根据 React diff 算法原理优化代码？

- 根据 diff 算法的设计原则，应尽量避免跨层级节点移动。
- 通过设置唯一 key 进行优化，尽量减少组件层级深度。因为过深的层级会加深遍历深度，带来性能问题。
- 设置 shouldComponentUpdate 或者 React.pureComponet 减少 diff 次数。

## 11 | 如何解释 React 的渲染流程？
