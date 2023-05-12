# React Deep Dive

## 목적
React를 사용하면서 아직도 모르는 것들이 넘쳐흐르고 있다.<BR />
이곳을 통해 넘치는걸 막고 흐른걸 주워 담는 프로젝트가 되길 바란다. 

[참조](https://github.com/sudheerj/reactjs-interview-questions)

## 2023.05.11 ~
---
## Core React

### 1. JSX란 무엇인가?
JSX는 JavaScript XML의 약자로, ECMAScript의 XML과 유사한 구문 확장이다.
기본적으로 `React.createElement(type, props, ...children)` 함수의 syntatic sugar(구문적 설탕)을 제공하여 템플릿 구문과 같은 HTML과 함께 JavaScript를 표현할 수 있다.

아래 예시는 텍스트를 가진 `h1` 태그를 반환하는 render function이다. 일반적으로 우리가 익숙하게 쓰는 방식과 유사한 문법으로 JSX를 사용할 수 있다.

```javascript
export default function App() {
    return (
      <h1 className="greeting">{"Hello, this is a JSX Code!"}</h1>
  );
}
```
만약 JSX를 사용하지 않는다면 아래와 같이 작성해야만 React를 사용할 수 있을 것이다.

```javascript
import { createElement } from 'react';

export default function App() {
  return createElement(
    'h1',
    { className: 'greeting' },
    'Hello, this is a JSX Code!'
  );
}
```

### 2. Element와 Component의 차이는 무엇인가?

Element는 화면에 표시할 내용을 DOM Node 또는 기타 컴포넌트로 설명하는 일반 객체다.
Element는 프로퍼티에 다른 Element를 포함(children)할 수 있다. 
React Element를 만드는 것은 값싼 연산이고 일단 생성되면 변경할 수 없다.

React Element를 JavaScript로 표현하면 다음과 같다.

```javascript
const element = React.createElement("div", { id: "login-btn" }, "Login");
```
그리고 이것은 JSX를 사용해 단순화할 수 있다.
```html
 <div id="login-btn">Login</div>
```
`React.createElement()` 함수는 아래와 같은 객체를 반환한다.
```javascript
{
  type: 'div',
  props: {
    children: 'Login',
    id: 'login-btn'
  }
}
```
마지막으로 `ReactDOM.render()` 함수를 사용해 Element를 DOM에 렌더링한다.

컴포넌트는 여러 방식으로 선언될 수 있다. `render()` 함수를 사용해 Class로 만들 수도 있고, 함수로 정의할 수도 있다. 어떤 경우든 props를 입력으로 받고 JSX 트리를 출력으로 반환한다.

```javascript
const Button = ({ handleLogin }) => (
  <div id={"login-btn"} onClick={handleLogin}>
    Login
  </div>
);
```
이후 JSX는 `React.createElement()` 함수 트리로 변환된다.

```javascript
const Button = ({ handleLogin }) =>
  React.createElement(
    "div",
    { id: "login-btn", onClick: handleLogin },
    "Login"
  );
```

### 3. 순수 컴포넌트가 무엇인가?

순수 컴포넌트는 동일한 state와 props에 대해 항상 동일한 출력을 렌더링하는 컴포넌트를 말한다. 
함수 컴포넌트에서는 `React.memo()` 를 통해 순수 컴포넌트를 만들 수 있다. `React.memo()` API는 이전 props와 새로운 props를 얕은 비교를 통해 비교하여 불필요한 리렌더링을 방지한다.

하지만 함수 컴포넌트 자체에서 동일한 상태를 다시 설정할 때 기본적으로 불필요한 렌더링을 방지하기 때문에 이전 상태와 현재 상태를 비교하지 않는다.

Memoized 된 컴포넌트는 아래와 같이 표현된다.

```javascript
const MemoizedComponent = memo(SomeComponent, arePropsEqual?);
```

아래는 `React.memo()` API를 통해 하위 컴포넌트의 불필요한 렌더링을 방지하는 예시다.

```javascript
import { memo, useState } from 'react';

  const EmployeeProfile = memo(function EmployeeProfile({ name, email }) {
    return (<>
          <p>Name:{name}</p>
          <p>Email: {email}</p>
          </>);
  });
  export default function EmployeeRegForm() {
    const [name, setName] = useState('');
    const [email, setEmail] = useState('');
    return (
      <>
        <label>
          Name: <input value={name} onChange={e => setName(e.target.value)} />
        </label>
        <label>
          Email: <input value={email} onChange={e => setEmail(e.target.value)} />
        </label>
        <hr/>
        <EmployeeProfile name={name}/>
      </>
    );
  }
```

위 코드에서는 이메일 props가 하위 컴포넌트로 전달되지 않았다. 따라서 이메일 props 변경에 대한 리렌더링이 발생하지 않는다.

클래스 컴포넌트에서는 React.Component 대신 React.PureComponent를 확장하는 컴포넌트가 순수 컴포넌트가 된다. 
props나 state가 변경되면 PureComponent는 shouldComponentUpdate() 수명 주기가 충족되면 props나 state 모두에 대해 얕은 비교를 수행한다.

> Note: 
> 1. `React.memo()` 는 HOC이다.
> 2. `shouldComponentUpdate` 라이프사이클 메서드를 통해 임의의 조건을 설정해 리렌더링을 방지할 수 있다.
> 3.  `shouldComponentUpdate` 가 반환하는 기본값은 `true`다. 따라서 이 메서드 안에서 로직을 통해 `true` 나 `false`를 반환하면 개발자가 지정한 조건 하에 리렌더링을 발생시킬 수 있다. 


