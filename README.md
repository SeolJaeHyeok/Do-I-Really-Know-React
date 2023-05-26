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


### 11. React key props

key는 element 배열을 만들 때 포함해야 하는 특별한 attribute다. key 프로퍼티는 React가 어떤 항목이 변경, 추가 또는 제거되었는지 식별하는 데 도움이 된다.
key는 형제들 사이에서 고유해야 하므로 대부분의 경우 데이터의 ID를 key로 사용한다.

```javascript
const todoItems = todos.map((todo) => <li key={todo.id}>{todo.text}</li>);
```

key는 React가 Element를 만들 때 각 컴포넌트의 인스턴스를 식별하는 고유한 식별자로서 역할을 한다. key는 단지 React를 위한 prop으로 동작하기 때문에 `props.key` 와 같이 접근할 수 없다.

key는 인스턴스를 식별하는 고유한 식별자로 동작하기 때문에 해당 key가 변경되면 이전 컴포넌트의 인스턴스와 DOM을 파괴하고 새롭게 컴포넌트를 생성한다.

만약 `<Form key={selectedItem.key} />` 와 같이 렌더링한다고 하면 선택된 아이템이 변경될 때마다 이전에 생성된 `Form` 컴포넌트의 인스턴스와 DOM을 파괴하고 새롭게 생성한다. 따라서 폼 내에 오래된 상태로 인한 문제가 발생하지 않는다.

> Note
> 
> 배열 요소의 key로 인덱스를 사용하는 경우도 있다. 실제로 React는 key를 지정하지 않으면 index를 사용한다. 하지만 요소가 삽입되거나 삭제되는 경우 또는 배열의 순서가 바뀌는 경우에는 시간이 지남에 따라 항목을 렌더링하는 순서가 변경된다. 이럴 경우 index를 key로 사용할 때 예상치 못한 버그가 발생할 수 있다.
> 
> 또한 `Math.random()`과 같이 즉석에서 key를 생성하는 것도 지양해야 한다. 이렇게 하게 되면 렌더링 간의 key가 일치하지 않아 모든 컴포넌트의 인스턴스와 DOM을 파괴하고 새로 생성하게 된다. 이는 속도가 느려질뿐만 아니라 만약 이를 의도하지 않았다면 요소 내부의 유저의 입력도 잃게 된다.

### 12. ref
ref는 요소에 대한 참조를 반환하는 데 사용된다. 대부분의 경우 사용하지 않는 것이 좋지만, DOM 요소나 컴포넌트의 인스턴스에 직접 액세스해야 할 때 유용할 수 있다.

> ref를 사용하지 않아야 하는 이유
>
> 1. 렌더 트리에 불필요한 상태를 도입할 수 있다. useRef를 사용하여 DOM 요소에 대한 참조를 유지하는 경우 해당 요소가 렌더 트리에 있는 한 메모리에 유지된다.
> 2. 예기치 않은 동작을 일으킬 수 있다. 예를 들어, useRef를 사용하여 컴포넌트의 현재 상태를 유지하는 경우 사용자가 상태를 변경하면 컴포넌트가 다시 렌더링되지 않는다.
> 3. 1번과 같은 이유로 성능 저하의 원인이 될 수 있다.

### 13. forward refs란?
Ref forwarding(참조 전달)은 컴포넌트가 받은 ref를 가져와 하위 컴포넌트로 전달할 수 있는 기능이다.
`forwardRef` 로 감싼 컴포넌트는 두 번째 매개 변수를 갖게 되는데, 이를 통해 ref를 전달받을 수 있다.


```javascript
import React, { forwardRef, useRef } from "react";

const Input = forwardRef((props, ref) => {
  return <input type="text" ref={ref} />;
});

function Field() {
  const inputRef = useRef(null);

  function handleFocus() {
    inputRef.current.focus();
  }

  return (
    <>
      <Input ref={inputRef} />
      <button onClick={handleFocus}>입력란 포커스</button>
    </>
  );
}
```

### 14. Virtual DOM은 어떻게 동작하는가?
Virtual DOM은 세 가지의 간단한 과정으로 동작한다.

1. 데이터가 변경될 때마다 전체 UI가 가상 DOM 표현으로 다시 렌더링된다.
2. 그 후, 이전 DOM과 새로운 DOM의 차이를 계산한다.(reconciliation)
3. 계산이 완료되면 실제 DOM에는 변경된 내용만 업데이트된다.

> *Virtual DOM이라는 용어에 대한 Dan Abramov의 입장*
> 
> 저는 우리가 "가상 DOM"이라는 용어를 폐기할 수 있길 바랍니다.
>  
> 2013년에는 지금과 달리 이렇게 이야기 하지 않으면 사람들은 리액트가 모든 렌더에서 DOM 노드를 생성한다고 생각했기 때문에 의미가 있었습니다. 
> 
> 그러나 오늘날 사람들은 이런 가정을 거의 하지 않습니다. "가상 DOM"은 일부 DOM 문제에 대한 해결 방법처럼 들리지만 리액트는 그런 것이 아닙니다.
>
> 리액트는 "값 UI"입니다. 그것의 핵심 원칙은 UI가 문자열이나 배열과 마찬가지로 값이라는 것입니다. 변수에 저장하고, 전달하고, 자바스크립트 제어 흐름에서 사용할 수 있습니다. 그 표현가능함이 요점입니다. DOM에 변경 사항을 적용하는 것을 피하기 위한 비교 같은 것이 아닙니다.
>
> 항상 DOM을 나타내는 것도 아닙니다. 예를 들어 `<Message recipientId={10} />`는 DOM이 아닙니다. 개념적으로 이는 지연 함수 호출 `Message.bind(null, { recipientId: 10 })`를 나타냅니다.


리액트는 기존 컴포넌트 트리와 DOM 구조를 가능한 많이 재활용하여 효율적인 리렌더링을 위해 재조정(Reconciliation) 과정을 거친다.

- **재조정(Reconciliation)이란?**
    
    
    만약 `ReactDOM.render()` 가 같은 컨테이너에서 두 번 호출된다면 어떻게 될까?
    
    ```jsx
    ReactDOM.render(
      <button className="blue" />,
      document.getElementById('container')
    );
    
    // ... 나중에 ...
    
    // 호스트 객체를 교체해야 할까
    // 아니면 기존 객체에 속성만 교체하면 될까?
    ReactDOM.render(
      <button className="red" />,
      document.getElementById('container')
    );
    ```
    
    - React의 목표는 주어진 **React 엘리먼트 트리와 호스트 트리를 일치시키는 것**이다.
        
        새로운 정보를 가지고 **호스트 객체 트리에 어떤 작업을 해야 하는지를 파악하는 과정**이 조정(****Reconciliation)****이라고 한다.
        
    - 여기에는 두 가지 방법이 존재한다.
        - 기존 정보를 날리고 새로운 트리를 생성
            
            ```jsx
            let domContainer = document.getElementById('container');
            // 트리를 날린다.
            domContainer.innerHTML = '';
            // 새로운 객체 트리를 만든다.
            let domNode = document.createElement('button');
            domNode.className = 'red';
            domContainer.appendChild(domNode);
            ```
            
            위 방법의 단점은 느리고 포커스, 선택, 스크롤 정보 등 중요한 정보를 잃게 된다. 대신 아래와 같이 우리가 원하는 방향으로 작업할 수도 있다.
            
            ```jsx
            let domNode = domContainer.firstChild;
            // 기존 호스트 객체를 변경한다.
            domNode.className = 'red';
            ```
            
            같은 위치에 같은 호스트 객체를 다시 렌더링 하는 것이기 때문에 다시 만들 필요 없이 재사용하면 된다.
            
        - **트리의 같은 위치에 있는 엘리먼트 타입이 이전 렌더링과 다음 렌더링 사이에 일치하면 React는 기존 호스트 객체를 다시 사용한다.**
            
            ```jsx
            // let domNode = document.createElement('button');
            // domNode.className = 'blue';
            // domContainer.appendChild(domNode);

            ReactDOM.render(
              <button className="blue" />,
              document.getElementById('container')
            );
            
            // 호스트 객체를 다시 사용할 수 있을까? 네! (button → button)
            // domNode.className = 'red';

            ReactDOM.render(
              <button className="red" />,
              document.getElementById('container')
            );
            
            // 호스트 객체를 다시 사용할 수 있을까? 아뇨! (button → p)
            // domContainer.removeChild(domNode);
            // domNode = document.createElement('p');
            // domNode.textContent = 'Hello';
            // domContainer.appendChild(domNode);

            ReactDOM.render(
              <p>Hello</p>,
              document.getElementById('container')
            );
            
            // 호스트 객체를 다시 사용할 수 있을까? 네! (p → p)
            // domNode.textContent = 'Goodbye';
            ReactDOM.render(
              <p>Goodbye</p>,
              document.getElementById('container')
            );
            ```
            
    
    ### 조건(Conditions)
    
    - 매 갱신마다 엘리먼트의 타입이 일치할 때만 호스트 객체를 재활용한다면 조건부 렌더링의 경우는 어떻게 동작할까?
        
        ```jsx
        // 첫 렌더링
        ReactDOM.render(
          <dialog>
            <input />
          </dialog>,
          domContainer
        );
        
        // 두 번째 렌더링
        ReactDOM.render(
          <dialog>
            <p>I was just added here!</p>
            <input />
          </dialog>,
          domContainer
        );
        ```
        
    - 위 예제에서 `input` 호스트 객체는 재생성될 것이다. 이미 존재하는 `input` 호스트 객체를 재활용하지 않고 재생성하는 과정은 아래와 같다.
        - `dialog → dialog`: 호스트 객체를 다시 사용할 수 있나요? **네, 타입이 일치합니다.**
        - `input → p`: 호스트 객체를 다시 사용할 수 있나요? **아뇨, 타입이 다릅니다.**
        - `input`을 삭제하고 `p`를 추가해야 합니다.
        - `(없음) → input`: 새로운 `input` 호스트 객체를 만들어야 합니다.
    - 따라서 React가 실행하는 코드는 다음과 같다.
        
        ```jsx
        let oldInputNode = dialogNode.firstChild;
        dialogNode.removeChild(oldInputNode);
        
        let pNode = document.createElement('p');
        pNode.textContent = 'I was just added here!';
        dialogNode.appendChild(pNode);
        
        let newInputNode = document.createElement('input');
        dialogNode.appendChild(newInputNode);
        ```
        
    - 새로 생성하게 되면 선택, 포커스, 내용을 잃기 때문에 재생성은 불필요한 과정이 된다.

### 15. Fiber 객체는 무엇인가?
Fiber 객체는 React 16에 처음 적용된 새로운 재조정 엔진이다. React Fiber는 애니메이션, 레이아웃, 제스처, 작업 일시정지, 중단, 재사용 기능 등 다양한 유형의 업데이트에 우선순위를 할당하고 새로운 concurrency primitives와 같은 영역에 대한 적합성을 높이는 것을 목표로 한다.

대표적인 기능은 증분 렌더링(incremental rendering)으로, 렌더링 작업을 청크로 분할아여 여러 프레임에 걸쳐 분산할 수 있는 기능이다.

Fiber 객체의 주된 목표는 아래와 같다.

1. 중단 가능한 작업을 청크로 분할하는 기능
2. 진행 중인 작업의 우선순위를 정하고, rebase하고 reuse할 수 있는 기능
3. React에서 레이아웃을 지원하기 위해 부모와 자식 간에 양보할 수 있는 기능
4. `render()`에서 여러 요소를 반환하는 기능
5. Error boundaries에 대한 더 나은 지원


> ### concurrency primitives
> 동시성 프리미티브(concurrency primitives)는 동시 스레드 또는 프로세스 간에 동기화 및 조정을 제공하는 프로그래밍 구조 또는 메커니즘이다.
>
> 이러한 프리미티브는 공유 리소스를 관리하고 동시 프로그램에서 정확하고 예측 가능한 동작을 보장하는 데 필수적이다.

Fiber 객체 타입을 간단히 나타내면 다음과 같다.
```javascript
export type Fiber = {
  // 파이버 타입을 식별하기 위한 태그
  tag: WorkTag;

  // 해당 요소의 고유 식별자
  key: null | string;

  // 파이버와 관련된 것으로 확인된 함수/클래스
  type: any;

  // 단일 연결 리스트 트리 구조
  child: Fiber | null;
  sibling: Fiber | null;
  index: number;

  // 파이버로 입력되는 데이터 (arguments/props)
  pendingProps: any;
  memoizedProps: any; // 출력을 만드는데 사용되는 props입니다.

  // 상태 업데이트 및 콜백 큐
  updateQueue: Array<State | StateUpdaters>;

  // 출력을 만드는데 사용되는 상태
  memoizedState: any;

  // 파이버에 대한 종속성(컨텍스트, 이벤트)(존재하는 경우)
  dependencies: Dependencies | null;
};
```

렌더링 패스 동안 React는 이 Fiber 객체 트리를 순회하고 새 렌더링 결과를 계산할 때 업데이트된 트리를 구성한다. 

중요한 점은 Fiber 객체는 실제 컴포넌트의 props와 state 값을 저장한다는 것이다. 다시 말해, 컴포넌트에서 props와 state를 사용할 때 **React는 Fiber 객체에 저장된 값에 대한 접근을 제공한다**는 것이다.

### 16. controlled component and uncontrolled component - [examples](https://github.com/SeolJaeHyeok/dirkr-examples/tree/main/app/controlled-uncontrolled)
제어 컴포넌트(controlled component)는 프로퍼티를 통해 현재 값과 업데이트 콜백 함수를 전달 받고, 부모 컴포넌트는 컴포넌트의 상태를 관리한다. 사용자가 컴포넌트와 상호작용하면 부모 컴포넌트가 상태를 업데이트하고, 이는 다시 컴포넌트의 값을 업데이트한다.

비제어 컴포넌트(uncontrolled component)는 자체 내부 상태를 유지하며, 사용자가 컴포넌트와 상호작용하면 자체 상태를 업데이트하고, 이는 다시 컴포넌트의 값을 업데이트 한다.

> ### 사용 예시
> 
> 결제 시스템의 경우 사용자가 카드 정보를 입력하는 양식을 처리하는 데 제어 컴포넌트를 사용하면 양식 데이터가 상위 컴포넌트에서 관리되고 이를 바로 서버에 요청할 수 있다.
>  
> 반면, 비제어 컴포넌트는 검색창 아래에 검색 결과가 표시되고 검색창의 상태가 컴포넌트 내부에서 관리되는 검색창과 같은 용도로 사용할 수 있다.