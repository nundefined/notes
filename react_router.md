React-Router
===

Kean.jang

---

<!-- $size: 16:9 -->

# React Router를 쓰지 않는 경우

```js
const App = React.createClass({
  // state에 hash값을 저장하고
  getInitialState() { return { route: window.location.hash.substr(1) } },
  componentDidMount() {
    window.addEventListener('hashchange', () => {
      this.setState({ route: window.location.hash.substr(1) })
    })
  },
  render() {
    // render 시점에 저장된 hash값을 사용하여 적당한 Component를 삽입
    switch (this.state.route) {
      case '/about': Child = About; break;
      case '/inbox': Child = Inbox; break;
      default:      Child = Home;
    }
    return ( <ul>
               <li><a href="#/about">About</a></li>
               <li><a href="#/inbox">Inbox</a></li>
             </ul>)
  }
})
render(<App />, document.body)
```
---

### React Router를 쓰는 경우

```js
// 모듈을 임포트
import { Router, Route, IndexRoute, Link, hashHistory } from 'react-router'

// switch 문은 제거하고 Link 컴포넌트로 변경
const App = React.createClass({
  render() {
    return (
      <div>
        <h1>App</h1>
        {/* <a>는 <Links>로 교체 */}
        <ul>
          <li><Link to="/about">About</Link></li>
          <li><Link to="/inbox">Inbox</Link></li>
        </ul>

        {/* <Child>는 `this.props.children`으로 교체
            router에서 적절한 children을 결정함 
            이 곳에 하위 컴포넌트가 추가된다. */}
        {this.props.children}
      </div>
    )
  }
})
```
---

```js
// 여러 개의 <Route>로 구성된 <Router>를 render 하면 끝
render((
  <Router history={hashHistory}>
    <Route path="/" component={App}>
      <IndexRoute component={Home} />
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox} />
    </Route>
  </Router>
), document.body)
```
---

- React Router는 중첩된 UI도 생성해줌
- 내부적으로 router에서는 `<Route>` 엘리먼트 구조를 route config로 변경함
- 간단하게 일반 객체의 형태로도 사용 가능

```js
const routes = {
  path: '/',
  component: App,
  indexRoute: { component: Home },
  childRoutes: [
    { path: 'about', component: About },
    { path: 'inbox', component: Inbox },
  ]
}

render(<Router history={history} routes={routes} />, document.body)
```
---

## 복잡한 UI의 구성

```
path: /inbox/messages/1234

+---------+------------+------------------------+
| About   |    Inbox   |                        |
+---------+            +------------------------+
| Compose    Reply    Reply All    Archive      |
+-----------------------------------------------+
|Movie tomorrow|                                |
+--------------+   Subject: TPS Report          |
|TPS Report        From:    boss@big.co         |
+--------------+                                |
|New Pull Reque|   So ...                       |
+--------------+                                |
|...           |                                |
+--------------+--------------------------------+
```
---
```
path: /inbox

+---------+------------+------------------------+
| About   |    Inbox   |                        |
+---------+            +------------------------+
| Compose    Reply    Reply All    Archive      |
+-----------------------------------------------+
|Movie tomorrow|                                |
+--------------+   10 Unread Messages           |
|TPS Report    |   22 drafts                    |
+--------------+                                |
|New Pull Reque|                                |
+--------------+                                |
|...           |                                |
+--------------+--------------------------------+
```
---

다음과 같이 구성 가능

```js
// Make a new component to render inside of Inbox
const Message = React.createClass({
  render() {
    return <h3>Message</h3>
  }
})

const Inbox = React.createClass({
  render() {
    return (
      <div>
        <h2>Inbox</h2>
        {/* 하위 route 컴포넌트를 render*/}
        {this.props.children}
      </div>
    )
  }
})
```
---
```js
render((
  <Router history={history}>
    <Route path="/" component={App}>
      <IndexRoute component={Home} />
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox}>
        {/* 중첩해야 할 UI를 둬야 할 곳에 필요한 중첩된 route를 추가 */}
        {/* `/inbox` 일 때의 stats 페이지를 render */}
        <IndexRoute component={InboxStats}/>
        {/* /inbox/messages/123 페이지의 메시지 컴포넌트를 render*/}
        <Route path="messages/:id" component={Message} />
      </Route>
    </Route>
  </Router>
), document.body)
```

---

`inbox/messages/Jkei3c32` 페이지에 접근하면 설정한 route에 해당하므로 다음과 같은 코드가 생성된다.
```js
<App>
  <Inbox>
    <Message params={{ id: 'Jkei3c32' }}/>
  </Inbox>
</App>
```

`/inbox`에 접근하면 다음과 같은 코드가 생성된다.

```js
<App>
  <Inbox>
    <InboxStats/>
  </Inbox>
</App>
```
---

### URL 파라미터 값 얻기

render시에 사용할 수 있도록 path에서 변경되는 부분의 특정 파라미터가 Route 컴포넌트에 추가된다. 

```js
const Message = React.createClass({

  componentDidMount() {
    // `/inbox/messages/:id` 경로에서 얻은 값
    const id = this.props.params.id

    fetchMessage(id, function (err, message) {
      this.setState({ message: message })
    })
  },

  // ...

})
```
---

- query string의 파라미터
  - `/foo?bar=baz`
  - Route 컴포넌트에서 `this.props.location.query.bar`를 확인하면 `"baz"`를 얻을 수 있다.

---
# Route 설정

- URL을 찾는 방법과 각 URL에 따라 실행할 내용을 정리해둔 것

```js
render((
  <Router>
    <Route path="/" component={App}>
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox}>
        <Route path="messages/:id" component={Message} />
      </Route>
    </Route>
  </Router>
), document.body)
```

URL                     | Components
------------------------|-----------
`/`                     | `App`
`/about`                | `App -> About`
`/inbox`                | `App -> Inbox`
`/inbox/messages/:id`   | `App -> Inbox -> Message`
---

## index의 추가
```js
render((
  <Router>
    <Route path="/" component={App}>
      {/* / 에서 Dashboard를 보여준다. */}
      <IndexRoute component={Dashboard} />
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox}>
        <Route path="messages/:id" component={Message} />
      </Route>
    </Route>
  </Router>
), document.body)
```

URL                     | Components
------------------------|-----------
`/`                     | `App -> Dashboard`
`/about`                | `App -> About`
`/inbox`                | `App -> Inbox`
`/inbox/messages/:id`   | `App -> Inbox -> Message`
---

## UI와 URL의 연관 끊기
```js
render((
  <Router>
    <Route path="/" component={App}>
      <IndexRoute component={Dashboard} />
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox} />
      {/* /inbox/messages/:id 대신  /messages/:id 사용 */}
      <Route component={Inbox}>
        <Route path="messages/:id" component={Message} />
      </Route>
    </Route>
  </Router>
), document.body)
```

URL                     | Components               
------------------------|--------------------------
`/`                     | `App -> Dashboard`       
`/about`                | `App -> About`           
`/inbox`                | `App -> Inbox`           
`/messages/:id`         | `App -> Inbox -> Message`

---

## url의 redirect
```js
import { Redirect } from 'react-router'

render((
  <Router>
    <Route path="/" component={App}>
      <IndexRoute component={Dashboard} />
      <Route path="about" component={About} />

      <Route path="inbox" component={Inbox}>
        {/* /inbox/message/:id 는 /message/:id로 redirect된다. */}
        <Redirect from="messages/:id" to="/messages/:id" />
      </Route>

      <Route component={Inbox}>
        <Route path="messages/:id" component={Message} />
      </Route>
    </Route>
  </Router>
), document.body)
```
---
## onEnter, onLeave Hook
- 트랜지션이 확정되면 `onEnter`, `onLeave` hook이 한 번씩 실행됨
- 페이지에서 빠져나갈 때 `onLeave`이 첫 번째 공통 조상 route까지 실행
- 그 후 'onEnter'가 진입하려는 첫 번째 route부터 실행됨
- 위 예제의 경우 `/messages/5` 에서 `/about` 으로 이동하는 링크를 클릭하면
  - `/messages/:id`의 `onLeave` 실행
  - `/inbox`의 `onLeave` 실행
  - `/about`의 `onEnter` 실행

---

## 일반 객체로 Route 설정
- JSX 대신 일반 객체로 설정 가능
- 일반 객체로 설정할 때, `<Redirect>`는 사용 불가. 이 경우에는 `onEnter` hook을 사용하면 됨

---

### Sample
```js
const routes = {
  path: '/',
  component: App,
  indexRoute: { component: Dashboard },
  childRoutes: [
    { path: 'about', component: About },
    {
      path: 'inbox',
      component: Inbox,
      childRoutes: [{
        path: 'messages/:id',
        onEnter: ({ params }, replace) => replace(`/messages/${params.id}`)
      }]
    },
    {
      component: Inbox,
      childRoutes: [{
        path: 'messages/:id', component: Message
      }]
    }
  ]
}

render(<Router routes={routes} />, document.body)
```

---

# Route Matching

- URL이 매치됐는지는 다음의 세 가지 속성에 의해 결정됨
  - 중첩
  - `path`
  - 우선순위

---

## 중첩
- React Router에서는 URL이 매칭됐을 때, 렌더되어야 할 중첩된 view를 선언하는 개념을 도입
- 매칭되는 URL을 찾기 위해 route config를 깊이 우선(depth-first) 탐색 방법으로 검색한다.

---

## `path` 문법
- route path는 문자열 패턴임
- 특수 문자를 제외하고는 문자열 그대로 매칭
- 특수 문자 목록
  - `:paramName`: `/`, `?`, `#`까지의 URL 조각과 매치
  - `()`: URL에서 옵션인 부분을 감쌈
  - `*`: 패턴의 다음 글자가 나타날 때까지 모든 문자에 최소 매칭 (non-greedy). 다음 문자가 없다면 URL의 끝까지 매칭. `splat` param 생성
  - `**`: 패턴에서 다음 `/`, `?`, `#` 을 만날 때까지 모든 글자에 최대 매칭 (greedy). `splat` param 생성

```js
<Route path="/hello/:name">     // /hello/michael, /hello/ryan
<Route path="/hello(/:name)">   // /hello, /hello/michael, /hello/ryan
<Route path="/files/*.*">       // /files/hello.jpg, /files/hello.html
<Route path="/**/*.jpg">        // /files/hello.jpg, /files/path/to/file.jpg
```
---

## 우선순위
- 위에서 아래로 정의된 순서에 따라 매치

---

# History

---

세 가지 종류가 있음
- `browserHistory`
- `hashHistory`
- `createMemoryHistory`

다음과 같이 사용

```js
// JavaScript module import
import { browserHistory } from 'react-router'

...

render(
  <Router history={browserHistory} routes={routes} />,
  document.getElementById('app')
)
```

---
## `browserHistory`

- browser application에 권장
- History API 사용하여 URL을 조작
- 실제 URL을 생성
- IE8, IE9의 경우
  - History API를 탐지하지 못한 경우, 항상 페이지를 리로드함.

---
## `hashHistory`

- URL의 `#` 부분을 사용하는 방법
- 시작하기에는 좋지만 상용 수준이나 서버 사이드 렌더링을 고려한다면 `browserHistory`의 사용을 권장
- 옛날 브라우저에서 전체 페이지의 리로드를 피해야 하는 경우 사용


## `createMemoryHistory`

- 주소창을 활용하지 않음
- 서버 렌더링을 위한 방식
- 테스트나 React Native와 같은 다른 렌더링 환경에서 사용
- 직접 생성하는 것이 다른 두 가지 방식과는 다름
```js
const history = createMemoryHistory(location)
```

---
## 예시
```js
import React from 'react'
import { render } from 'react-dom'
import { browserHistory, Router, Route, IndexRoute } from 'react-router'

import App from '../components/App'
import Home from '../components/Home'
import About from '../components/About'
import Features from '../components/Features'

render(
  <Router history={browserHistory}>
    <Route path='/' component={App}>
      <IndexRoute component={Home} />
      <Route path='about' component={About} />
      <Route path='features' component={Features} />
    </Route>
  </Router>,
  document.getElementById('app')
)
```
---
## 사용자화
- `useRouterHistory` 사용 가능
- `useRouterHistory`는 `useQueries`, `useBasement`를 이용하여 이미 history factory를 개선하고 있음

---

```js
// basename의 지정
import { useRouterHistory } from 'react-router'
import { createHistory } from 'history'

const history = useRouterHistory(createHistory)({
  basename: '/base-path'
})
```

```js
// useBeforeUnload의 사용
import { useRouterHistory } from 'react-router'
import { createHistory, useBeforeUnload } from 'history'

const history = useRouterHistory(useBeforeUnload(createHistory))()

history.listenBeforeUnload(function () {
  return 'Are you sure you want to leave this page?'
})
```
---

# Index Routes 와 Index Links

---
## Index Routes

```js
<Router>
  <Route path="/" component={App}>
    <IndexRoute component={Home}/>
    <Route path="accounts" component={Accounts}/>
    <Route path="statements" component={Statements}/>
  </Route>
</Router>
```

- IndexRoute 를 지정하지 않으면 `this.props.children`이 `null`이 되어 아무런 내용이 표시되지 않음
- 위와 같이 설정하면 first class route component가 되어 routing에 포함됨

---

# Index Redirects

```js
<Route path="/" component={App}>
  <IndexRedirect to="/welcome" />
  <Route path="welcome" component={Welcome} />
  <Route path="about" component={About} />
</Route>
```

- `/`로 접근했을 때 `/welcome`으로 redirect됨

# Index Links
- `<Link to="/">HOME</Link>`로 설정하면 모든 URL은 `/`로 시작하므로 이 링크는 항상 활성화된 상태가 된다.
- 정확히 `/`가 활성화 됐을 때만 활성화 상태로 만들기 위해 `<IndexLink to="/">Home</IndexLink>` 사용한다.

---

## references
- https://github.com/ReactTraining/react-router/tree/master/docs