---
id: chapter3
title: 第三章：项目开发篇
---

## 使用 ES6 模板字符串实现模板引擎

模板渲染

通过 vm 模块编译 JavaScript 形成函数（vm 模块可在 V8 虚拟机上下文中编译和运行代码）

- xss 过滤、模板 helper 函数
- include 子模板

## 极客时间播放页需求解构

API 服务 - RESTful

- 简单易懂
- 可以快速搭建
- 在数据的聚合方面有很大劣势

API 服务 - GraphQL

- 专注数据聚合，前端要什么就返回什么

使用 koa - GraphQL 中间件

## 极客时间课程列表页的需求解构

后端需要渲染列表

- 首屏加速
- SEO

前端也需要渲染列表

- 无刷新过滤、排序

前后端同构

- 同一个模板/组件，可在浏览器渲染，也可在 Node.js 渲染

## 前后端同构

ReactDomServer.renderToString() 

VueServerRenderer.renderToString()

React/Vue 同构的最大难题其实是数据部分（redux）

- 使用 Next

```js
// browser/index.jsx

const Container = require('../component/container.jsx');
const React = require('react');
const ReactDOM = require('react-dom');

class App extends React.Component {

    constructor() {
        super();
        this.state = {
						// 数据挂在 window 上
            columns: reactInitData,
            filtType: reactInitFiltType,
            sortType: reactInitSortType
        }
    }

    render() {
        return (
            <Container
                columns={this.state.columns}
                filt={(filtType) => {
                    fetch(`./data?sort=${this.state.sortType}&filt=${filtType}`)
                        .then(res => res.json())
                        .then(json => {
                            this.setState({
                                columns: json,
                                filtType: filtType
                            })
                        })
                }}
                sort={(sortType) => {
                    fetch(`./data?sort=${sortType}&filt=${this.state.filtType}`)
                        .then(res => res.json())
                        .then(json => {
                            this.setState({
                                columns: json,
                                sortType: sortType
                            })
                        })
                }}
            />
        )
    }
}
ReactDOM.render(
    <App />,
    document.getElementById('reactapp')
)
```

```js
// component/container.jsx

const React = require('react');
const ColumnItem = require('./column_item.jsx')

module.exports = class Container extends React.Component {

    render() {
        return (
            <div className="_2lx4a-CP_0">
                <div className="_3KjZQbwk_0">
                    <div className="kcMABq6U_0">
                        <span>课程：</span>
                        <a className="_2TWCBjxa_0" onClick={this.props.filt.bind(this, 0)}>全部</a>
                        <a className="_2TWCBjxa_0" onClick={this.props.filt.bind(this, 1)}>专栏</a>
                        <a className="_2TWCBjxa_0" onClick={this.props.filt.bind(this, 2)}>视频课程</a>
                        <a className="_2TWCBjxa_0" onClick={this.props.filt.bind(this, 3)}>微课</a>
                    </div>
                </div>
                <div className="_3hVBef3W_0">
                    <div className="_3S9KmBtG_0">
                        <div className="_1o6EOwiF_0">
                            <div className="_3HUryTHs_0">
                                <a className="_1kRLIDSR_0" onClick={this.props.sort.bind(this, 1)}>上新</a>
                                <a className="_1kRLIDSR_0" onClick={this.props.sort.bind(this, 2)}>订阅数</a>
                                <a className="_1kRLIDSR_0" onClick={this.props.sort.bind(this, 3)}>价格
                                    <span className="_1Yk9PA11_0">
                                        <i className="iconfont _2jewjGCJ_0"></i> <i className="iconfont _38FM8KCt_0"></i>
                                    </span>
                                </a>
                            </div>
                            <span className="JfgzzksA_0">{this.props.columns.length}个课程</span>
                        </div>
                        <div>
                            {this.props.columns.map(column => {
                                return (
                                    <ColumnItem column={column} key={column.id} />
                                )
                            })}
                        </div>
                        <div className="OjL5wNoM_0"><span>— 没有更多了 —</span></div>
                    </div>
                </div>
            </div>

        )
    }
}
```

```js
// node/app.jsx

const React = require('react')
const Container = require('../component/container')

module.exports = function (reactData) {
    return <Container
        columns={reactData}
        filt={() => { }}
        sort={() => { }}
    />
}

// node/index.js

const app = new (require('koa'));
const mount = require('koa-mount');
const static = require('koa-static');
const getData = require('./get-data')
const ReactDOMServer = require('react-dom/server');
require('babel-register')({
    presets: ['react']
});
const App = require('./app.jsx')
const template = require('./template')(__dirname + '/index.htm')

app.use(mount('/static', static(__dirname + '/source')))

app.use(mount('/data', async (ctx) => {
    ctx.body = await getData(+(ctx.query.sort || 0), +(ctx.query.filt || 0));
}));

app.use(async (ctx) => {
    ctx.status = 200;
    const filtType = +(ctx.query.filt || 0)
    const sortType = +(ctx.query.sort || 0);
    const reactData = await getData(sortType, filtType);
    // console.log(ReactDOMServer.renderToString(ReactRoot)); 
    ctx.body = template({
        reactString: ReactDOMServer.renderToString(
            App(reactData)
        ),
        reactData,
        filtType,
        sortType
    })
})

// app.listen(3000)

module.exports = app;

// index.html

...
<script>
  window.reactInitData = ${reactData ? JSON.stringify(reactData) : ''};
  window.reactInitFiltType = ${filtType};
  window.reactInitSortType = ${sortType}
</script>
<script src="./static/main.js"></script>
...
```