## UI 트리

브라우저는 UI를 모델링하기 위해 많은 트리 구조를 사용한다. DOM은 HTML 요소를 나타내고, CSSOM은 CSS에 대해 동일한 역할을 한다. 심지어 접근성 트리도 있다.

React 또한 트리 구조를 사용하여 사용자가 만든 UI를 관리하고 모델링한다. React는 JSX로부터 UI 트리를 만든다. 그런 다음 React DOM은 해당 UI 트리와 일치하도록 브라우저 DOM 엘리먼트를 업데이트 한다.

## state는 트리의 한 위치에 묶인다.

컴포넌트에 state를 부여할 때, state가 컴포넌트 내부에 "존재"한다고 생각할 수 있다. 하지만 **state는 실제로 React 내부에서 유지된다.** React는 UI 트리에서 해당 컴포넌트가 어디에 위치하는지에 따라 보유하고 각 state를 올바른 컴포넌트와 연결한다.

```javascript
export default function App() {
  const counter = <Counter />;
  return (
    <div>
      {counter}
      {counter}
    </div>
  );
}

// 여기에는 <Counter /> JSX 코드가 하나만 있지만 두 개의 다른 위치에서 렌더링한다.
```

이 카운터는 각 트리에서 고유한 위치에 렌더링되기 때문에 두 개의 개별 카운터입니다. 따라서, React에서 화면의 각 컴포넌트는 완전히 분리된 state를 갖는다. React는 같은 컴포넌트를 같은 위치에 렌더링하는 한 그 state를 유지합니다. 또한 컴포넌트가 UI 트리의 해당 위에서 렌더링되는 동안 컴포넌트의 state를 유지한다. 컴포넌트가 제거되거나 같은 위치에 다른 컴포넌트가 렌더링 되면 React는 해당 컴포넌트의 state를 삭제한다.

## 동일한 위치의 동일한 컴포넌트는 state를 유지한다.

항상 같은 컴포넌트가 위치기 동일한 곳에 위치할 경우, state는 유지된다.

## 동일한 위치의 다른 컴포넌트는 state를 초기화한다.

```javascript
import { useState } from "react";

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? <p>See you later!</p> : <Counter />}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={(e) => {
            setIsPaused(e.target.checked);
          }}
        />
        Take a break
      </label>
    </div>
  );
}

// 코드 생략
```

여기서는 같은 위치에서 서로 다른 컴포넌트 유형 사이를 전환한다. 처음에 <div>의 첫 번째 자식에는 Counter가 있다. 하지만 p를 넣었을 때 React는 UI 트리에서 Counter를 제거하고 그 state를 소멸시킨다.

**같은 위치에 다른 컴포넌트를 렌더링 하면 전체 하위 트리의 state가 재설정된다.**

**리렌더링 사이에 state를 유지하려면 트리의 구조가 "일치"해야 한다.** 구조가 다르면 React는 트리에서 컴포넌트를 제거할 때 state를 파괴하기 때문이다.

## 동일한 위치에서 state 재설정하기

동일한 위치에서 컴포넌트의 state를 리셋하고 싶을 때 두 가지 방법이 있다.

```javascript
import { useState } from "react";

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? <Counter person="Taylor" /> : <Counter person="Sarah" />}
      <button
        onClick={() => {
          setIsPlayerA(!isPlayerA);
        }}
      >
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = "counter";
  if (hover) {
    className += " hover";
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>
        {person}'s score: {score}
      </h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

### 컴포넌트를 다른위치에 렌더링하기

```javascript
import { useState } from "react";

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA && <Counter person="Taylor" />}
      {!isPlayerA && <Counter person="Sarah" />}
      <button
        onClick={() => {
          setIsPlayerA(!isPlayerA);
        }}
      >
        Next player!
      </button>
    </div>
  );
}
// 코드 생략
```

### key로 state 재설정하기

key는 목록에만 사용되는 것이 아니다. key를 사용해 React가 모든 컴포넌트를 구분하도록 할 수 있다. 기본적으로 React는 부모 내의 순서를 사용해 컴포넌트를 구분한다. **하지만 key를 사용하면 이것이 첫 번째 컴포넌트나 두 번째 컴포넌트가 아니라 특정 컴포넌트임을 React에 알릴 수 있다.**

## key로 form 재설정하기

키로 state를 재설정하는 것은 form을 다룰 때 특히 유용하다.

다른 버튼을 클릭했을 때 기존 state가 유지된다면 컴포넌트에 key를 부여하면된다. 이렇게 하면 컴포넌트가 그 아래의 트리의 모든 state를 포함하여 처음부터 다시 생선된다. 또한 React는 DOM 엘리먼트를 재사용하는 대신 다시 생성한다.
