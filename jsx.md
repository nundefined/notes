# JSX

* [JSX in Depth](https://facebook.github.io/react/docs/jsx-in-depth.html) 정리본

JSX는 XML과 유사한 ECMAScript의 문법 확장. 프리프로세서(트랜스파일러)를 이용하여 표준 ECMAScript로 변환하는 것이 기본 방향

JSX 코드는 React.createElement(component, props, …children) 함수를 편리하게 사용하기 위한 방법이다.

````
// 일반적인 형태
// JSX
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>

// 컴파일 후
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)

// 여는 태그, 닫는 태그가 합쳐진 형태
// JSX
<div className="sidebar" />

// 컴파일 후
React.createElement(
  'div',
  {className: 'sidebar'},
  null
)
````

## React 엘리먼트 타입의 지정
JSX 태그의 첫 부분은 React 엘리먼트의 종류를 지정한다.

대문자로 시작하면 React 컴포넌트를 가리키는 것이다. 이런 태그는 같은 이름의 변수를 바로 참조하므로 변수가 스코프내에 존재해야 한다.

### 스코프내에 React가 있어야 한다.
JSX를 컴파일하면 `React.createElement`를 호출하므로 `React`라이브러리는 항상 JSX 코드의 스코프 내에 존재해야 한다.

### dot 표기법을 사용하여 JSX 타입을 지정
JSX 내에서 dot 표기법을 사용하여 React 컴포넌트를 참조할 수 있다. 모듈 하나에 여러 개의 React 컴포넌트가 포함되어 있는 경우 유용하다.

### 사용자가 정의한 컴포넌트는 대문자로 시작해야 한다.
소문자로 시작하면 내장 컴포넌트를 참조한다.

````
// 내장 컴포넌트
// <div> => React.createElement('div', ...)
// <span> => React.createElement('span', ...)

// 사용자 컴포넌트
// <Foo /> => React.createElement(Foo, ...)
````

### 실행 중 타입 선택
React의 엘리먼트 타입으로 general expression을 사용할 수 없다. 특정한 타입의 엘리먼트를 가리키는 general expression을 사용하려면 먼저 대문자로 시작하는 변수에 할당한 후 사용한다. prop에 따라 다른 컴포넌트를 렌더링할 때 이런 경우를 만나게 된다.

````
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // JSX 타입은 대문자 변수만 사용할 수 있다.
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
````

## JSX에서의 props

### JavaScript 표현식
`{}`로 감싸서 prop에 JavaScript 표현식을 사용할 수 있다.

````
// foo의 값은 10이 된다.
<MyComponent foo={1 + 2 + 3 + 4} />
````

`if` 구문이나 `for` 반복문은 JavaScript 표현식이 아니므로 사용할 수 없다.

````
// JSX 바깥쪽에서 if, for를 사용한다.
function NumberDescriber(props) {
  let description;
  if (props.number % 2 == 0) {
    description = <strong>even</strong>;
  } else {
    description = <i>odd</i>;
  }
  return <div>{props.number} is an {description} number</div>;
}
````

* JavaScript 표현식은 MDN의 [Expressions and operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Expressions)를 참고

### 문자열
문자열을 prop로 전달할 수 있다.
````
// 아래 두 구문은 동일하다.
<MyComponent message="hello world" />
<MyComponent message={'hello world'} />

// 문자열을 prop로 전달할 때, 그 값은 unescape된 값이다.
// 아래 두 구문은 동일하다.
<MyComponent message="&lt;3" />
<MyComponent message={'<3'} />
````

[Q] This behavior is usually not relevant. It's only mentioned here for completeness.

### prop의 기본값은 "True"
prop에 값을 지정하지 않으면 기본 값은 True이다. 하지만 기본값을 생략하는 것을 권장하지 않는다. ES6의 Object shorthand와 혼동할 가능성이 있다.

````
// 아래 두 구문은 동일하다.
<MyTextBox autocomplete />
<MyTextBox autocomplete={true} />
````

### Attribute 펼치기
이미 `props`를 객체로 만들어 뒀다면 `...` 연산자를 사용하여 전체 객체를 넘길 수 있다.

````
// 다음 두 개의 코드는 동일하다.
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
````

범용 컨테이너를 만들 때 유용한 방법이나, 컴포넌트에서 필요로 하지 않는 관계 없는 props를 넘기게 되어 코드가 지저분해질 수 있으므로 필요한 곳에서만 사용하는 것을 권장한다.
