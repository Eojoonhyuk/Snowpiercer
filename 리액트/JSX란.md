## JSX란?

- JSX는 리액트가 등장하면서 페이스북에서 소개한 새로운 구문
- JSX는 흔히 개발자들이 알고 있는 XML 과 유사한 내장형 구문이며, 리액트에 종속적이지 않는 독자적인 문법으로 보는 것이 옳다.
- JSX는 ECMAScript라고 불리는 자바스크립트 표준의 일부는 아니다.
- JSX가 포함된 코드를 아무런 처리 없이 그대로 실행하면 에러가 발생한다. 반드시 트랜스파일러를 거쳐야 비로소 자바스크립트 런타임이 이해할 수 있는 의미 있는 자바스크립트 코드로 변환된다.
- **jSX는 자바스크립트 내부에서 표현하기 까다로웠던 XML 스타일의 트리 구문을 작성하는 데 많은 도움을 주는 새로운 문법이라고 볼 수 있다.**

```jsx
// JSX를 사용하지 않은 경우
const element = React.createElement(
  "div",
  { className: "myClass" },
  "Hello, React!"
);

// JSX를 사용한 경우
const element = <div className="myClass">Hello, React!</div>;
```

## JSX의 정의

### JSXElement

jSX를 구성하는 가장 기본 요소로, HTML의 요소와 비슷한 역할을 한다.

- JSXOpeningElement: 일반적으로 볼 수 있는 요소다. JSXOpeningElement로 시작했다면 JSXClosingElement가 동일한 요소로 같은 단계에서 선언돼 있어야 올바른 JSX 문법으로 간주된다.

> 예: <JSXElement JSXAttributes(optional)>

- JSXClosingElement: JSXOpeningElement가 종료됐음을 알리는 요소로, 반드시 JSXOpeningElement와 쌍으로 사용돼야 한다.

> 예: <JSXElement/>

- JSXSelfClosingElement: 요소가 시작되고, 스스로 종료되는 형태를 의미한다. <script/>와 동일한 모습을 띠고 있다. 이는 내부적으로 자식을 포함할 수 없는 형태를 의미한다.

> 예: <JSXElement JSXAttribute(optional) />

- JSXFragment: 아무런 요소가 없는 형태로, JSXSelfClosingElement 형태를 띌 수는 없다. </>는 불가능하다. 단, <></>는 가능하다.

> 예: <>JSXChildren(optional)</>

### JSXAttributes

JSXElement에 부여할 수 있는 속성을 의미한다. 단순히 속성을 의미하기 때문에 모든 경우에서 필수값이 아니고, 존재하지 않아도 에러가 나지 않는다.

### JSXChildren

JSXElement의 자식 값을 나타낸다. JSX는 앞서 언급했듯 속석을 가진 트리 구조를 나타내기 위해 만들어졌기 때문에 JSX로 부모와 자식 관계를 나타낼 수 있으며, 그 자식을 JSXChildren이라고 한다.
