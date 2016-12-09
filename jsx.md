# JSX

## JSX specification

* [Draft: JSX Specification](http://facebook.github.io/jsx/) 정리

JSX는 ECMAScript 6th Edition의 PrimaryExpression을 확장함

````
PrimaryExpression:
  JSXElement

// Elements
JSXElement:
  JSXSelfClosing
  JSXOpeningElemnt JSXChildren(opt) JSXClosingElement // element 이름이 같아야 함

JSXSelfClosingElement:
  < JSXElementName JSXAttributes(opt) />

JSXOpeningElement:
  < JSXElementName JSXAttributes(opt) >

JSXClosingElement:
  </ JSXElementName >

JSXElementName:
  JSXIdentifier
  JSXNamedspacedName
  JSXMemberExpression

JSXIdentifier:
  IdentifierStart
  JSXIdentifier IdentifierPart
  JSXIdentifier NO WHITESPACE OR COMMENT -

JSXNamespacedName :
  JSXIdentifier : JSXIdentifier

JSXMemberExpression :
  JSXIdentifier . JSXIdentifier
  JSXMemberExpression . JSXIdentifier

// Attributes
JSXAttributes :
  JSXSpreadAttribute JSXAttributes(opt)
  JSXAttribute JSXAttributes(opt)

JSXSpreadAttribute :
  { ... AssignmentExpression }

JSXAttribute :
  JSXAttributeName = JSXAttributeValue

JSXAttributeName :
  JSXIdentifier
  JSXNamespacedName

JSXAttributeValue :
  " JSXDoubleStringCharacters(opt) "
  ' JSXSingleStringCharacters(opt) '
  { AssignmentExpression }
  JSXElement

JSXDoubleStringCharacters :
  JSXDoubleStringCharacter JSXDoubleStringCharacters(opt)

JSXDoubleStringCharacter :
  SourceCharacter (" 제외)

JSXSingleStringCharacters :
  JSXSingleStringCharacter JSXSingleStringCharactersopt

JSXSingleStringCharacter :
  SourceCharacter (' 제외)

// Children
JSXChildren :
  JSXChild JSXChildren(opt)

JSXChild :
  JSXText
  JSXElement
  { AssignmentExpression(opt) }

JSXText :
  JSXTextCharacter JSXText(opt)

JSXTextCharacter :
  SourceCharacter ({, <, >, } 제외)

// 공백과 주석
JSX는 ECMAScript와 같은 구두법과 중괄호를 사용한다.
공백, 줄마침(line terminators), 주석은 어떤 구두법에서도 쓸 수 있다.
````

## JSX in Depth (React에서 사용하는 JSX)

* [JSX in Depth](https://facebook.github.io/react/docs/jsx-in-depth.html) 정리

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


## JSX에서의 Children
여는 태그와 닫는 태그로 구성된 JSX 표현식에서 두 태그 사이의 내용은 특별한 prop인 `props.children` 으로 처리된다. 이 값을 처리하는 방법은 여러 가지가 있다.

### 문자열
두 태그 사이에 문자열을 넣을 수 있으며 `props.children`은 해당 문자열이 된다.

````
// MyComponent의 props.children은 "Hello world!"가 된다.
<MyComponent>Hello world!</MyComponent>

// HTML은 unescape이므로 HTML을 사용하듯 작성하면 된다.
<div>This is valid HTML &amp; JSX at the same time.</div>

// JSX에서는 한 줄의 양 끝단의 공백과 빈 줄을 제거한다.
// 태그 근처의 줄바꿈은 제거된다.
// 문자열 사이의 줄바꿈은 한 칸의 공백으로 변경된다.
// 다음은 모두 동일하게 표시된다.
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
````

### JSX Children
JSX 엘리먼트를 으로 사용할 수 있다. 중첩된 컴포넌트를 표시할 때 유용하다.

````
<MyContainer>
  <MyFirstComponent />
  <MySecondComponent />
</MyContainer>
````

서로 다른 종류의 children을 섞어 사용할 수 있다. 문자열과 children을 함께 사용할 수 있다. JSX가 HTML과 유사한 측면이다.

````
<div>
  Here is a list: // 문자열 children
  <ul>            // 컴포넌트 children
    <li>Item 1</li>
    <li>Item 2</li>
  </ul>
</div>
````

React 컴포넌트는 다중 React 엘리먼트를 반환하지 못하나, 하나의 JSX 표현식에서는 여러 개의 children을 가질 수 있으므로 여러 가지를 렌더링할 컴포넌트가 필요할 때는 `div`와 같은 것으로 컴포넌트를 감싸면 된다.

### JavaScript 표현식
JavaScript 표현식도 `{}`으로 감싸 chidlren으로 사용할 수 있다. 다른 종류의 children과도 섞어 사용할 수 있다.

````
// 다음 두 개는 동일하다.
<MyComponent>foo</MyComponent>
<MyComponent>{'foo'}</MyComponent>

// 문자열과 JavaScript 표현식을 같이 사용할 수 있다.
function Hello(props) {
  return <div>Hello {props.addressee}!</div>;
}
````

이런 기능은 특정한 길이의 JSX 표현식 목록을 렌더링할 때 유용하다.

````
function Item(props) {
  return <li>{props.message}</li>;
}

function TodoList() {
  const todos = ['finish doc', 'submit pr', 'nag dan to review'];
  return (
    <ul>
      {todos.map((message) => <Item key={message} message={message} />)}
    </ul>
  );
}
````

### Children으로 사용하는 함수
보통 JSX에 추가된 JavaScript 표현식의 결과는 문자열, React 엘리먼트나 문자열과, React 엘리먼트의 목록이 된다. 하지만 `props.children`은 React가 렌더링하는 방법을 아는 것 뿐만 아니라 어떤 종류의 데이터도 전달할 수 있다는 점에서 다른 prop와 마찬가지로 동작한다. 사용자 컴포넌트가 있는 경우 `props.children`과 같은 형태로 콜백을 사용할 수 있다.

````
function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}
    </Repeat>
  );
}

// children의 callback인 numTimes를 호출하여 반복되는 컴포넌트를 생성한다.
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}
````

React가 렌더링 하기 전에 이해할 수 있게 변환만 된다면 어떤 사용자 컴포넌트라도 children으로 전달할 수 있다. 이런 사용법이 평범하지는 않지만 JSX의 가능성을 높이려고 한다면 잘 동작할 것이다.

### Boolean, Null, Undefined는 무시된다.
`false`, `null`, `undefined`, `true`는 유효한 children이며, 렌더링되지 않는다.

````
// 5가지 JSX 표현식은 모두 동일한 결과가 나온다.
<div />
<div></div>
<div>{false}</div>
<div>{null}</div>
<div>{true}</div>
````

이런 특징은 조건에 따라 React 엘리먼트를 렌더링할 때 유용하다.

````
// showHeader가 true인 경우에만 <Header />를 렌더링한다.
<div>
  {showHeader && <Header />}
  <Content />
</div>
````

한 가지 조심할 것은 `0`과 같은 일부 "거짓" 값일 때다. 이 값들은 React에 의해 렌더링된다.

````
// props.messages가 없을 때 화면에 0이 표시된다.
<div>
  {props.messages.length &&
    <MessageList messages={props.messages} />
  }
</div>

// 다음과 같이 작성해야 정상적으로 동작한다.
<div>
  {props.messages.length > 0 &&
    <MessageList messages={props.messages} />
  }
</div>
````

`false`, `true`, `null`, `undefined`와 같은 값을 화면에 표시하려면 문자열로 변환하여 출력한다.

````
<div>
  My JavaScript variable is {String(myVariable)}.
</div>
````




















