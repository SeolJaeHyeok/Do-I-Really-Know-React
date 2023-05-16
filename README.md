# React Deep Dive

## 목적
React Deep Dive

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

### 4. State란 무엇인가?
컴포넌트의 state는 *컴포넌트의 수명에 따라 변경될 수 있는 일부 정보를 저장하는 객체*다.<br />
중요한 점은 state가 변경되면 리렌더링을 발생시킨다는 것이다. 그렇기 때문에 state를 가능한 단순하게 만들고 Stateful Component를 최소화 하는 것이 좋다.

state는 props와 비슷하지만 private하며 컴포넌트에 의해 완전히 제어된다. 즉, state를 가진 컴포넌트가 전달하기 전까지 다른 컴포넌트가 해당 state에 접근할 수 없다.

### 5. Props란 무엇인가?
Props는 컴포넌트에 대한 입력(값)이다. Props를 HTML의 Attribute와 유사하게 생성 시 컴포넌트에 전달되는 값이다. 여기서 전달되는 데이터의 흐름은 부모 컴포넌트에서 자식 컴포넌트로 이루어진다.

React에서 Props의 주요 목적은 다음과 같은 컴포넌트의 기능을 제공하는 것이다.

1. 컴포넌트에 사용자 정의 데이터를 전달한다.
2. 상태 변경을 일으킨다.
3. 컴포넌트의 `render()` 메서드 내에서 `this.props.whatever`을 통해 사용한다.

이 whatever의 속성 이름은 원래 React 라이브러리를 사용해 생성된 모든 컴포넌트에 이미 존재하는 React의 네이티브 props 객체에 연결된 property가 된다.

함수 컴포넌트에서 props는 아래와 같이 사용 가능하다.
```javascript
import React from "react";
import ReactDOM from "react-dom";

const ChildComponent = (props) => {
  return (
    <div>
      <p>{props.name}</p>
      <p>{props.age}</p>
    </div>
  );
};

const ParentComponent = () => {
  return (
    <div>
      <ChildComponent name="John" age="30" />
      <ChildComponent name="Mary" age="25" />
    </div>
  );
};
```

props 객체 안에 연결된 모든 property는 ES6에서 추가된 destructing을 통해 직접적으로 접근할 수 있다.

```javascript
const ChildComponent = ({name, age}) => {
  return (
    <div>
      <p>{name}</p>
      <p>{age}</p>
    </div>
  );
};
```

### 6. State와 Props의 차이
React에서 state와 props는 모두 일반 자바스크립트 객체이며 컴포넌트의 데이터를 관리하는 데 사용되지만, 서로 다른 방식으로 사용되며 다른 특성을 가지고 있다.

State:

- state는 컴포넌트 내의 내부 데이터 저장 메커니즘이다.
- 컴포넌트의 lifecycle 동안 시간이 지남에 따라 변경될 수 있는 정보를 나타낸다.
- state는 컴포넌트 자체에서 소유하고 제어한다.
- 클래스 컴포넌트의 생성자에서 또는 기능적 컴포넌트의 useState 후크를 사용하여 상태를 초기화할 수 있다.
- state는 setState 메서드(클래스 컴포넌트) 또는 업데이트 기능(funtinoal 컴포넌트)을 사용하여 소유한 컴포넌트 내에서만 수정할 수 있다.
- state가 변경되면 React는 state 업데이트의 영향을 받는 컴포넌트 및 해당 하위 컴포넌트의 다시 렌더링을 자동으로 트리거한다.

Props:

- props(properties의 약자)는 상위 컴포넌트에서 하위 컴포넌트로 데이터를 전달하는 데 사용된다.
- props는 읽기 전용(readonly)이며 자식 컴포넌트에서 수정할 수 없다.
- props는 컴포넌트 트리를 통해 부모 컴포넌트에서 자식 컴포넌트로 전달된다.
- props는 부모 컴포넌트에서 정의되고 값이 할당된 다음 this.props(클래스 컴포넌트)를 사용하거나 매개 변수(기능 컴포넌트)로 자식 컴포넌트 내에서 액세스할 수 있다.
- 상위 컴포넌트의 props을 변경하면 업데이트된 props으로 하위 컴포넌트가 다시 렌더링된다.
- props는 데이터를 전달하고 컴포넌트 간에 통신하는 방법을 제공하여 컴포넌트 구성 및 재사용을 가능하게 한다.

요약하면 state는 *컴포넌트 내에서 변경 가능한 내부 데이터를 관리하는 데 사용*되는 반면 props는 *부모 컴포넌트에서 자식 컴포넌트로 데이터를 전달하는 데 사용*된다.<br />
state는 컴포넌트 자체가 소유하고 제어하는 ​​반면 props은 읽기 전용(readonly)이며 부모 컴포넌트에서 자식 컴포넌트로 전달된다.

### 7. State Update
state를 직접 업데이트하려 시도한다면, 리렌더링이 발생하지 않는다.

```javascript
this.state.message = "Hello world";
```

직접 업데이트 하는 대신 `setState()` 메서드를 사용해 state를 업데이트해야한다.
`setState()` 메서드는 state 객체의 업데이트를 예약(scheduling)한다. 

```javascript
this.setState({ message: "Hello World" });
```

클래스 컴포넌트에서 사용하는 `setState()` 메서드의 두 번째 인자로 콜백 함수를 넘겨줄 수 있다.
콜백 함수는 setState가 완료되고 컴포넌트가 렌더링될 때 호출된다.

> Note:
>
> 하지만 이런 식으로 `setState()` 함수의 콜백 함수를 넘겨주는 것보다 라이프사이클 메서드를 사용하는 것이 낫다.
> ```javascript
>   setState({ name: "John" }, () =>  console.log("The name has updated and component re-rendered"));
> ```

### 8. HTML과 React 사이의 이벤트를 다루는 방법의 차이

HTML과 React 이벤트 핸들링의 주된 차이는 아래와 같다.

1. HTML에서 이벤트는 `lowercase` 컨벤션을 따른다. 반면, React에서는 `camelCase` 컨벤션을 따른다.
   ```javascript
      // HTML
      <button onclick="clickEvent()">Click</button>

      // React
      <button onClick={clickEvent}>Click</button>
   ```
2. HTML에서 이벤트의 기본 동작을 막기 위해 `false` 를 반환할 수 있다. 반면, React에서는 반드시 `preventDefault()` 함수를 명시적으로 호출해야 한다.
   ```javascript
    // HTML
    <a href="#" onclick="clickEvent(); return false;">Move</a>

    // React
    function clickEvent(event) {
      event.preventDefault();
      console.log('Click');
    }
   ```
3. HTML에서는 함수 이름 앞에 `()`를 붙여 함수를 호출해야 하지만, React에서는 붙이면 안된다.

### 9. JSX 콜백에서 메서드 또는 이벤트 핸들러를 바인딩하는 방법

클래스 컴포넌트에서는 3가지의 방법이 존재한다.

1. Binding in Constructor: 자바스크립트 클래스에서 메서드는 기본적으로 바인딩되지 않는다. 클래스 메서드로 선언된 React 이벤트 핸들러 또한 동일한 규칙이 적용된다. 일반적으로 바인딩을 생성자에서 이루어진다.
   ```javascript
   class User extends Component {
     constructor(props) {
       super(props);
       this.handleClick = this.handleClick.bind(this);
     }
     handleClick() {
       console.log("SingOut triggered");
     }
     render() {
       return <button onClick={this.handleClick}>SingOut</button>;
     }
   }
   ```
2. Public class fields syntax: Bind 메서드를 사용하고 싶지 않다면 *public class fields syntax*를 사용하여 콜백을 올바르게 바인딩할 수 있다.
   ```javascript
   handleClick = () => {
     console.log("SingOut triggered", this);
   };

   <button onClick={this.handleClick}>SingOut</button>
   ```
3. Arrow functions in callbacks: 콜백 함수 안에 직접 화살표 함수를 사용하는 것도 가능하다.
   ```javascript
   handleClick() {
       console.log('SingOut triggered');
   }
   render() {
       return <button onClick={() => this.handleClick()}>SignOut</button>;
   }
   ```

### 10. React의 synthetic events

Synthetic Event는 브라우저의 Native Event를 감싼 크로스 브라우저 래퍼다. 이 이벤트는 모든 브라우저에서 동일하게 작동한다는 점을 제외하면 stopPropagation() 및 preventDefault() 등 브라우저의 네이티브 이벤트와 동일한 인터페이스를 가지고 있다.

어떤 이유로 기본 브라우저 이벤트가 필요한 경우 nativeEvent 속성을 사용하여 해당 이벤트를 가져오면 된다. 합성 이벤트는 브라우저의 기본 이벤트와 다르며 직접 매핑되지 않는다.

합성 이벤트 객체에는 이벤트 유형(event type), 대상 요소(target element), 이벤트 좌표(event coordinate)와 같은 네이티브 이벤트 객체와 동일한 프로퍼티와 메서드가 포함되어 있다. 하지만 일관된 동작을 보장하고 automatic event pooling과 같은 추가 기능을 제공하여 성능을 향상시킨다.

SyntheticEvent의 중요한 특징 중 하나는 풀링된 객체라는 점이다. 즉, 이벤트 핸들러가 실행을 완료한 후 SyntheticEvent 객체가 재사용되고 해당 프로퍼티가 무효화된다. 이 풀링은 각 이벤트에 대해 새 객체를 생성하지 않도록 하여 메모리 사용량을 줄이고 성능을 개선하는 데 도움이 된다.

```javascript
// No Pooling
function handleChange(e) {
  // This won't work because the event object gets reused.
  setTimeout(() => {
    console.log(e.target.value); // Too late!
  }, 100);
}

// Pooling
function handleChange(e) {
  // Prevents React from resetting its properties:
  e.persist();

  setTimeout(() => {
    console.log(e.target.value); // Works
  }, 100);
}
```

> Note
>
> React가 예전 브라우저에서 성능을 위해 서로 다른 이벤트 사이에 이벤트 객체를 재사용하고, 그 사이에 모든 이벤트 필드를 null로 설정했었다. React 16 이하에서는 이벤트를 제대로 사용하려면 e.persist()를 호출하거나 필요한 프로퍼티를 먼저 읽어야만 했다.
> 
> 하지만 React 17버전에서는 Pooling이 모던 브라우저에서 성능 개선을 하지 못한다고 판단하여 event pooling이 제거됐다. 따라서 React 17 이후에는 `e.persist()` 가 아무런 동작도 하지 않는다.
