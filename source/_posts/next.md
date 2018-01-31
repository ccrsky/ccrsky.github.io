---
title: next快速入门
date: 2017-08-17 20:16:53
tags:
    - js

category: 
    - js
---



Next是react服务端渲染框架。

### 特点
- 默认服务端渲染
- 自动切分代码
- 基于webpack开发，支持HMR
- 可以集成Express或其他Node server
- 可以自定义Babel和Webpack配置


### 快速入门

#### 初始化项目

```bash
mkdir hello-next
cd hello-next
npm init -y
npm install --save react react-dom next
# pages是页面路由目录，必须有，路由默认是基于该目录下的文件
mkdir pages
```
npm script 添加如下代码

``` js
{
  "scripts": {
    "dev": "next"
  }
}

// npm run dev 即可运行开发服务器 http://localhost:3000
```
目前没有路由，访问会出现404页面，接下来添加路由。

#### 添加路由
在pages目录下添加index.js(pages/index.js),内容如下：

``` js
const Index = () => (
  <div>
    <p>Hello Next.js</p>
  </div>
)
export default Index
```
>React组件要 **默认** 导出；默认情况下`pages`为路由目录，`static`为静态文件目录。

#### 页面导航
添加about页面

```js
export default () => (
  <div>
    <p>This is the about page</p>
  </div>
)
```
通过 `a` 标签可以很容易做到页面跳转，但那是通过服务端导航，不是我们想要的，我们可以通过 `next/link`支持客户端导航。

```js
import Link from 'next/link'

const Index = () => (
  <div>
    <Link href="/about">
      <a>About Page</a>
    </Link>
    <p>Hello Next.js</p>
  </div>
)

export default Index
```
或者

```js
// 包裹按钮
<Link href="/about">
  <button>Go to About Page</button>
</Link>
```

>Link组件包裹a标签，处理页面跳转，无论是a，button还是其他组件(比如div)，只要Link子组件接收onClick属性，都可以跳转。

链接添加样式

```js
// 生效
<Link href="/about">
  <a style={{ fontSize: 20 }}>About Page</a>
</Link>

// 不生效
<Link href="/about" style={{ fontSize: 20 }}>
  <a>About Page</a>
</Link>
```
>Link是高阶组件，直接收 href等一些其他属性，不支持style。


#### 共享组件
多个页面经常会有公用部分（比如Head），我们可以新建`components/Header.js`

```js
import Link from 'next/link'

const linkStyle = {
  marginRight: 15
}

const Header = () => (
    <div>
        <Link href="/">
          <a style={linkStyle}>Home</a>
        </Link>
        <Link href="/about">
          <a style={linkStyle}>About</a>
        </Link>
    </div>
)
export default Header
```
在首页中导入`Head`组件

```js
import Header from '../components/Header'

export default () => (
  <div>
    <Header />
    <p>Hello Next.js</p>
  </div>
)
```
>组件可以放在任何目录中，除了 **pages** 目录，pages下的文件跟服务端路由相关。

#### Layout组件
可以将整个页面框架做成layout组件(components/MyLayout.js)，各个页面复用

Layout组件

```js
import Header from './Header'

const layoutStyle = {
  margin: 20,
  padding: 20,
  border: '1px solid #DDD'
}

const Layout = (props) => (
  <div style={layoutStyle}>
    <Header />
    {props.children}
  </div>
)

export default Layout

```

index页面

```js
// pages/index.js

import Layout from '../components/MyLayout.js'

export default () => (
    <Layout>
       <p>Hello Next.js</p>
    </Layout>
)
```

about页面

```js
pages/about.js

import Layout from '../components/MyLayout.js'

export default () => (
    <Layout>
       <p>about</p>
    </Layout>
)
```
layout除了使用`{props.children}`还可以使用属性传值,高级组件的方式。

```js
import withLayout from '../lib/layout'

const Page = () => (
  <p>This is the about page</p>
)
export default withLayout(Page)
```

```js
// pages/index.js
const Page = () => (
  <p>This is the about page</p>
)

export default () => (<Layout page={Page}/>)
```

#### 动态页面
前边我们实现了静态页面，现在我们通过 query string 实现动态页面。

```js
// pages/index.js

import Layout from '../components/MyLayout.js'
import Link from 'next/link'

const PostLink = (props) => (
  <li>
    <Link href={`/post?title=${props.title}`}>
      <a>{props.title}</a>
    </Link>
  </li>
)
// 通过Link组件href属性向下一个页面传参

export default () => (
  <Layout>
    <h1>My Blog</h1>
    <ul>
      <PostLink title="Hello Next.js"/>
      <PostLink title="Learn Next.js is awesome"/>
      <PostLink title="Deploy apps with Zeit"/>
    </ul>
  </Layout>
)

```
处理查询参数

```js
import Layout from '../components/MyLayout.js'

export default (props) => (
    <Layout>
       <h1>{props.url.query.title}</h1>
       <p>This is the blog post content.</p>
    </Layout>
)
```

只有default导出的组件才有`url`属性

```js
import Layout from '../components/MyLayout.js'

const Content = (props) => (
  <div>
    <h1>{props.url.query.title}</h1>
    <p>This is the blog post content.</p>
  </div>
)

export default () => (
    <Layout>
       <Content />
    </Layout>
)

// 页面会抛出异常

// 可以通过组件传值的方式将url参数带到子组件中去
<Layout>
   <Content url={props.url} />
</Layout>

```
>每个页面都会获取`url`属性，可以`props.url.query.title`拿到相关的数据。

#### 自定义路由
Link组件有个`as`属性，我们可以通过它自定义路由格式，它会显示在浏览器的地址栏中，但实际下个页面接收的数据还是href属性上的参数。

```js
const PostLink = (props) => (
  <li>
    <Link as={`/p/${props.id}`} href={`/post?title=${props.title}`}>
      <a>{props.title}</a>
    </Link>
  </li>
)
```
>虽然可以自定义客户端路由显示，但是刷新页面可能会404,服务端没有对应的路由。

#### 自定义服务器

安装`Express`

```js
npm install --save express
```
根目录下创建`server.js`

```js
const express = require('express')
const next = require('next')

const dev = process.env.NODE_ENV !== 'production'
const app = next({ dev })
const handle = app.getRequestHandler()

app.prepare()
.then(() => {
  const server = express()

  // 处理对应路由
  server.get('/p/:id', (req, res) => {
    const actualPage = '/post'
    // 这里拿到的title是路由中的id,而客户端路由拿到的是Link href中的title，有点区别，但实际生产环境中都会使用资源id去请求数据。
    const queryParams = { title: req.params.id } 
    // 定义了一个路由映射到已存在页面，同时附带查询参数
    app.render(req, res, actualPage, queryParams)
  })
  
  server.get('*', (req, res) => {
    return handle(req, res)
  })

  server.listen(3000, (err) => {
    if (err) throw err
    console.log('> Ready on http://localhost:3000')
  })
})
.catch((ex) => {
  console.error(ex.stack)
  process.exit(1)
})
```

更新下npm脚本

```js
{
  "scripts": {
    "dev": "node server.js"
  }
}
```

#### 远程获取数据
可以通过一个异步方法`getInitialProps`（同时支持客户端和服务端）来获取远程数据。

安装 `isomorphic-unfetch` 请求模块

```bash
npm install --save isomorphic-unfetch
```

修改首页内容

```js
import Layout from '../components/MyLayout.js'
import Link from 'next/link'
import fetch from 'isomorphic-unfetch'

const Index = (props) => (
  <Layout>
    <h1>Batman TV Shows</h1>
    <ul>
      {props.shows.map(({show}) => (
        <li key={show.id}>
          <Link as={`/p/${show.id}`} href={`/post?id=${show.id}`}>
            <a>{show.name}</a>
          </Link>
        </li>
      ))}
    </ul>
  </Layout>
)

Post.getInitialProps = async function (context) {
  // 可以通过context上的query拿到查询参数
  const { id } = context.query
  const res = await fetch(`https://api.tvmaze.com/shows/${id}`)
  const show = await res.json()

  console.log(`Fetched show: ${show.name}`)

  return { show }
}

export default Index
```

getInitialProps receives a context object with the following properties:

```
pathname - path section of URL
query - query string section of URL parsed as an object
asPath - the actual url path
req - HTTP request object (server only)
res - HTTP response object (server only)
jsonPageRes - Fetch Response object (client only)
err - Error object if any error is encountered during the rendering
```

#### 组件样式
给组件添加样式有如下两种方法：

- 外部引入css文件
- css in js （next推荐）

next.js使用 [styled-jxs](https://github.com/zeit/styled-jsx)控制局部样式。

```js
import Layout from '../components/MyLayout.js'
import Link from 'next/link'

function getPosts () {
  return [
    { id: 'hello-nextjs', title: 'Hello Next.js'},
    { id: 'learn-nextjs', title: 'Learn Next.js is awesome'},
    { id: 'deploy-nextjs', title: 'Deploy apps with ZEIT'},
  ]
}

export default () => (
  <Layout>
    <h1>My Blog</h1>
    <ul>
      {getPosts().map((post) => (
        <li key={post.id}>
          <Link as={`/p/${post.id}`} href={`/post?title=${post.title}`}>
            <a>{post.title}</a>
          </Link>
          <!-- prefetch 会预加载下个页面的脚本，不会加载数据-->
          <li><Link prefetch href='/'><a>Home</a></Link></li>
        </li>
      ))}
    </ul>
    <!-- 除了 Link 添加prefech属性，还可以通过脚本预加载-->
    <a onClick={ () => setTimeout(() => url.pushTo('/dynamic'), 100) }>
      A route transition will happen after 100ms
    </a>
    {
      // but we can prefetch it!
      Router.prefetch('/dynamic')
    }
    // 样式必须用反引号包裹，必须放在模板字符串中，Styled jsx仅仅作为一个babel插件
    <style jsx>{`
      h1, a {
        font-family: "Arial";
      }

      ul {
        padding: 0;
      }

      li {
        list-style: none;
        margin: 5px 0;
      }

      a {
        text-decoration: none;
        color: blue;
      }

      a:hover {
        opacity: 0.6;
      }
    `}</style>
  </Layout>
)
```

>styled-jsx 样式默认不作用于子组件，如果要子组件生效，可以单独定义子组件样式，或者使用全局样式。

`styled-jsx` 全局样式

```js
<style jsx global>{`
     .markdown {
       font-family: 'Arial';
     }

     .markdown a {
       text-decoration: none;
       color: blue;
     }

     .markdown a:hover {
       opacity: 0.6;
     }

     .markdown h3 {
       margin: 0;
       padding: 0;
       text-transform: uppercase;
     }
  `}</style>

```

### 应用部署
#### 部署步骤

1. build代码
2. 启动服务

修改下npm脚本

```js
"scripts": {
  "build": "next build",
  "start": "next start"
}
```

#### 启动多个服务
为了水平扩展，通常会使用多个app实例

```js
//$PORT 为环境变量
"scripts": {
  "start": "next start -p $PORT"
}

//window下用 "next start -p %PORT%"
```

调用
```bash
PORT=8000 npm start
PORT=9000 npm start
```
可以通过 [cross-env](https://www.npmjs.com/package/cross-env) 设置环境变量。



### 高阶使用

#### 导出静态网站

在根目录下创建文件`next.config.js`

```js
module.exports = {
  exportPathMap: function () {
    return {
      '/': { page: '/' },
      '/about': { page: '/about' },
      '/p/hello-nextjs': { page: '/post', query: { title: "Hello Next.js" } },
      '/p/learn-nextjs': { page: '/post', query: { title: "Learn Next.js is awesome" } }
    }
  }
}
```

修改npm脚本

```js
{
  "scripts": {
    "build": "next build",
    "export": "next export"
  }
}
```
执行导出命令

```bash
npm run export
```
导入内容会放在`out`目录中，可放在任何静态服务器上使用。


☞[官网](https://github.com/zeit/next.js#custom-server-and-routing)

