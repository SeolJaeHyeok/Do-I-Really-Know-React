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
