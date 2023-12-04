## 불필요한 Effect 제거 방법

렌더링을 위해 데이터를 변환하는 경우 Effect는 필요하지 않는다.
컴포넌트의 state를 업데이트할 때 React는 먼저 컴포넌트 함수를 호출해 화면에 표시될 내용을 계산한다.
다음으로 이러한 변경사항을 DOM에 "commit"하여 화면을 업데이트하고, 그 후에 Effect를 실행한다. 만약 Effect 역시 state를 즉시 업데이트한다면, 이로 인해 전체 프로세스가 처음부터 다시 시잘될 것이다.

불필요한 렌더링을 피하려면 모든 데이터 변환을 컴포넌트의 최상위 레벨에서 하자. 그러면 props나 state가 변경될 때마다 해당 코드가 자동으로 다시 실행될 것이다.

## props 또는 state에 따라 state 업데이트하기

```js
function Form() {
  const [firstName, setFirstName] = useState("Taylor");
  const [lastName, setLastName] = useState("Swift");

  // 🔴 Avoid: redundant state and unnecessary Effect
  // 🔴 이러지 마세요: 중복 state 및 불필요한 Effect
  const [fullName, setFullName] = useState("");
  useEffect(() => {
    setFullName(firstName + " " + lastName);
  }, [firstName, lastName]);
  // ...
}
```

```js
function Form() {
  const [firstName, setFirstName] = useState("Taylor");
  const [lastName, setLastName] = useState("Swift");
  // ✅ Good: calculated during rendering
  // ✅ 좋습니다: 렌더링 과정 중에 계산
  const fullName = firstName + " " + lastName;
  // ...
}
```

기존 props나 state에서 계산할 수 있는 것이 있으면 state에 넣지 마라. 대신 렌더링 중에 계산하자.
이렇게 하면 코드가 더 빨라지고(추가적인 "계단식"업데이트를 피함), 더 간단해지고(일부 코드 제거), 오류가 덜 발생한다.(서로 다른 state 변수가 서로 동기화되지 않아 발생하는 버그를 피함)

## 고비용 계산 캐싱하기

```js
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState("");

  // 🔴 Avoid: redundant state and unnecessary Effect
  // 🔴 이러지 마세요: 중복 state 및 불필요한 Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  // ...
}
```

```js
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState("");
  // ✅ This is fine if getFilteredTodos() is not slow.
  // ✅ getFilteredTodos()가 느리지 않다면 괜찮습니다.
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```

getFilteredTodos()가 느리거나 todos가 많을 경우, newTodo와 같이 관련 없는 state 변수가 변경되더라도 getFilteredTodos()를 다시 계산하고 싶지 않을 수 있다.

이럴 땐 값비싼 계산을 useMemo 훅을 감싸서 캐시(또는 메모화)할 수 있다.

```js
import { useMemo, useState } from "react";

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState("");
  // ✅ Does not re-run getFilteredTodos() unless todos or filter change
  // ✅ todos나 filter가 변하지 않는 한 getFilteredTodos()가 재실행되지 않음
  const visibleTodos = useMemo(
    () => getFilteredTodos(todos, filter),
    [todos, filter]
  );
  // ...
}
```

이렇게 하면 todos나 filter가 변경되지 않는 한 내부 함수가 다시 실행되지 않기를 원한다는 것을 React에 알린다.

useMemo로 감싸는 함수는 렌더링 중에 실행되므로, 순수 계산에만 작동한다.

### 계산이 비싼지 어떻게 알까?

콘솔 로그를 추가하여 코드에 소요된 시간을 측정할 수 있다.

```js
console.time("filter array");
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd("filter array");
```

기록된 전체 시간이 상당하다면(예: 1ms 이상) 해당 계산은 메모해 두는 것이 좋을 수 있다.
하지만 useMemo는 첫 번째 렌더링을 더 빠르게 만들지는 않습니다. 업데이트 시 불필요한 작업을 건너뛰는 데에만 도움이 될 뿐이다.

## prop이 변경되면 모든 state 재설정하기

```js
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState("");

  // 🔴 Avoid: Resetting state on prop change in an Effect
  // 🔴 이러지 마세요: prop 변경시 Effect에서 state 재설정 수행
  useEffect(() => {
    setComment("");
  }, [userId]);
  // ...
}
```

이것은 ProfilePage와 그 자식들이 먼저 오래된 값으로 렌더링한 다음 새로운 값으로 다시 렌더링하기 때문에 비효율적이다.

그 대신 명시적인 키를 전달해 각 사용자의 프로필이 개념적으로 다른 프로필이라는 것을 React에 알릴 수 있다. 컴포넌트를 둘로 나누고 **바깥쪽 컴포넌트에서 안쪽 컴포넌트로 key 속성을 전달해라**

```js
export default function ProfilePage({ userId }) {
  return <Profile userId={userId} key={userId} />;
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  // ✅ key가 변하면 이 컴포넌트 및 모든 자식 컴포넌트의 state가 자동으로 재설정됨
  const [comment, setComment] = useState("");
  // ...
}
```

일반적으로 React는 같은 컴포넌트가 같은 위치에서 렌더링될 때 state를 유지한다. userId를 key로 Profile 컴포넌트에 전달하는 것은 곧, userId가 다른 두 Profile 컴포넌트를 state를 공유하지 않는 별개의 컴포넌트들로 취급하도록 React에게 요청하는 것이다.

## props가 변경될 때 일부 state 조정하기

items prop이 다른 배열을 받을 때마다 selection을 null로 재설정하고 싶다.

```js
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 Avoid: Adjusting state on prop change in an Effect
  // 🔴 이러지 마세요: prop 변경시 Effect에서 state 조정
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

items가 변경될 때마다 List와 그 하위 컴포넌트는 처음에는 오래된 selection값으로 렌더링된다. 그런 다음 React는 DOM을 업데이트하고 Effects를 실행한다. 마지막으로setSelection(null)호출은List와 그 자식 컴포넌트를 다시 렌더링하여 이 전체 과정을 재시작하게 된다.

```js
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: Adjust the state while rendering
  // 더 나음: 렌더링 중에 state 조정
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

렌더링 도중 setSelection이 직접 호출된다. React는 return 문과 함께 종료된 직후에 List를 다시 렌더링한다. 이 시점에서는 React는 아직 List의 자식들을 렌더링하거나 DOM을 업데이트하지 않았기 때문에, List의 자식들은 기존의 selection 값에 대한 렌더링을 건너뀌게 된다.

렌더링 도중 컴포넌트를 업데이트하면, React는 반환된 JSX를 버리고 즉시 렌더링을 다시 시도한다.

이 패턴은 Effect보다 효율적이지만, 대부분의 컴포넌트에는 필요하지 않습니다.

항상 key로 모든 state를 재설정하거나 렌더링 중에 모두 계산할 수 있는지를 확인해라. 예를 들어, 선택한 item을 저장(및 재설정)하는 대신, 선택한 item의 ID를 저장할 수 있다.

```js
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: Calculate everything during rendering
  // ✅ 가장 좋음: 렌더링 중에 모든 값을 계산
  const selection = items.find((item) => item.id === selectedId) ?? null;
  // ...
}
```

## state변경을 부모 컴포넌트에 알리기

```js
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 Avoid: The onChange handler runs too late
  // 🔴 이러지 마세요: onChange 핸들러가 너무 늦게 실행됨
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange]);

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }

  // ...
}
```

React는 서로 다른 컴포넌트에서 일괄 업데이트를 함께 처리하므로, 렌더링 과정은 한 번만 발생합니다.

```js
// ✅ Also good: the component is fully controlled by its parent
// ✅ 좋습니다: 부모 컴포넌트에 의해 완전히 제어됨
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }

  // ...
}
```

“state 끌어올리기”는 부모 컴포넌트가 부모 자체의 state를 토글하여 Toggle을 완전히 제어할 수 있게 해줍니다. 이는 부모 컴포넌트가 더 많은 로직을 포함해야 하지만, 전체적으로 걱정해야 할 state가 줄어든다는 것을 의미합니다.

## 데이터 패칭하기

```js
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 Avoid: Fetching without cleanup logic
    // 🔴 이러지 마세요: 클린업 없이 fetch 수행
    fetchResults(query, page).then((json) => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

페치해야 하는 주된 이유가 타이핑 이벤트가 아니라는 점. 검색 입력은 URL에 미리 채워져 있는 경우가 많으며, 사용자는 input을 건드리지 않고도 앞뒤로 탐색할 수 있다.

age와 query가 어디에서 오는지는 중요하지 않다. 이 컴포넌트가 표시되는 동안 현재의 page 및 query에 대한 네트워크의 데이터와 results의 동기화가 유지되면 된다. 이것이 Effect인 이유다.

경쟁 조건을 수정하기 위해서는 오래된 응답을 무시하도록 클린업 함수를 추가해야 한다.

```js
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then((json) => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

프레임워크를 사용하지 않고(또한 직접 만들고 싶지 않고) Effect에서 데이터 페칭을 보다 인체공학적으로 만들고 싶다면, 다음 예시처럼 페칭 로직을 커스텀 훅으로 추출하는 것을 고려해 보자.

```js
function SearchResults({ query }) {
  const [page, setPage] = useState(1);
  const params = new URLSearchParams({ query, page });
  const results = useData(`/api/search?${params}`);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}

function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(url)
      .then((response) => response.json())
      .then((json) => {
        if (!ignore) {
          setData(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [url]);
  return data;
}
```

이 방법만으로는 프레임워크 빌트인 데이터 페칭 메커니즘을 사용하는 것만큼 효율적이지는 않겠지만, 데이터 페칭 로직을 커스텀 훅으로 옮기면 나중에 효율적인 데이터 페칭 전략을 채택하기가 더 쉬워진다.
