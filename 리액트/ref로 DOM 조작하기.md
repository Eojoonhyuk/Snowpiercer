## ref로 DOM 조작하기

React는 렌더링 출력과 일치하도록 DOM을 자동으로 업데이트하므로 컴포넌트가 자주 조작할 필요가 없다. 하지만 때로는 **노드에 초점**을 맞추거나 **스크롤**하거나 **크기와 위치**를 측정하기 위해 React가 관리하는 DOM 요소에 접근해야 할 수도 있다. React는 이러한 작업을 수행할 수 있는 빌트인된 방법이 없으므로 DOM 노드에 대한 ref가 필요하다.

## 노드에 대한 ref 가져오기

React가 관리하는 DOM 노드에 접근하려면, 먼저 useRef 훅을 불러오고 컴포넌트 내부에서 ref를 선언한다.

마지막으로, DOM 노드를 가져올 JSX 태그에 ref 속성으로 참조를 전달한다.

```javascript
import { useRef } from "react";

const myRef = useRef(null);

<div ref={myRef}>
```

## 텍스트 input에 초점 맞추기

```javascript
import { useRef } from "react";

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

DOM 조작이 ref의 가장 일반적인 사용 사례이지만, useRef 훅은 timer ID와 같은 다른 것들을 React 외부에 저장하는 데 사용될 수 있다.

## 다른 컴포넌트의 DOM 노드에 접근하기

기본적으로 React가 컴포넌트가 다른 컴포넌트의 DOM 노드에 접근하는 것을 허용하지 않는다. 심지어 자신의 자식에게도 말이다. 이것은 의도적인 것이다. ref는 탈출구이기 때문에 아껴서 사용해야 한다. 다른 컴포넌트의 DOM 노드를 수동으로 조작하면 코드가 훨씬 더 취약해진다.

대신, DOM 노드를 노출하길 원하는 컴포넌트에 해당 동작을 설정해야 한다. 컴포넌트는 자신의 ref를 자식 중 하나에 “전달”하도록 지정할 수 있다.

```javascript
import { forwardRef, useRef } from "react";

const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

## React가 ref를 첨부할 때

React에서 모든 업데이트는 두 단계로 나뉜다.

- 런더 단계: React는 컴포넌트를 호출하여 화면에 무엇이 표시되어야 하는지 파악
- 커밋 단계: React는 DOM에 변경 사항을 적용

첫 번째 렌더링 중에는 DOM 노드가 아직 생성되지 않았으므로 ref.current는 null이 된다. 그리고 업데이트를 렌더링하는 동안에는 DOM 노드가 아직 업데이트되지 않았다. 따라서 이를 읽기에는 너무 이르다.

React는 커밋하는 동안에 ref.current를 설정한다. React는 DOM이 업데이트 되기 전에는 ref.current의 값을 null로 설정하였다가, DOM이 업데이트된 직후 해당 DOM 노드로 다시 설정한다.
