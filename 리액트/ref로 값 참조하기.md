## 컴포넌트에 ref 추가하기

React에서 useRef 훅을 가져와서 컴포넌트에 ref를 추가할 수 있다.

```javascript
import { useRef } from "react";
```

컴포넌트 내부에서 useRef 훅을 호출하고 참조할 초기값을 인자로 전달하는 방법은

```javascript
const ref = useRef(0);

// 값 0은 ref에 대한 초기값이다.
```

useRef는 다음과 같은 **객체**를 반환한다.

```javascript
{
  current: 0; // The value you passed to useRef
}
```

ref.current 속성을 통해 해당 ref의 현재 값에 엑세스할 수 있다. 이 값은 의도적으로 **변이가능하므로 읽기와 쓰기가 모두 가능하다.**

## 컴포넌트는 ref가 증가할 때마다 리렌더링되지 않는다.

state와 마찬가지로 ref는 리렌더링 사이에 React에 의해 유지된다. state를 설정하면 컴포넌트가 리렌더링되지만 ref는 리렌더링되지 않는다.

렌더링에 정보가 사용되는 경우 해당 정보를 state로 유지하고 이벤트 핸들러만 정보를 필요로하고 변경해도 다시 렌더링할 필요가 없는 경우, ref를 사용하는 것이 더 효율적일 수 있다.

## ref와 state의 차이점

ref는 자주 사용하지 않는 "탈출구"이다.

|                          refs                           |                                           state                                           |
| :-----------------------------------------------------: | :---------------------------------------------------------------------------------------: |
| useRef(initialValue)는 { current: initialValue }을 반환 | useState(initialValue)는 state 변수의 현재값과 state 설정자함수([value, setValue])를 반환 |
|            변경 시 리렌더링을 촉발하지 않음             |                                 변경 시 리렌더링을 촉발함                                 |

|
|Mutable— 렌더링 프로세스 외부에서 current 값을 수정하고 업데이트할 수 있음|“Immutable”— state setting 함수를 사용하여 state 변수를 수정해 리렌더링을 대기열에 추가해야함|
|렌더링 중에는 current 값을 읽거나 쓰지 않아야 함|언제든지 state를 읽을 수 있음. 각 렌더링에는 변경되지 않는 자체 state snapshot이 있음|
