## state 구조화 원칙

- 관련 state를 그룹화 한다.
- state의 모순을 피한다.
- 불피요한 state를 피한다.
- state의 중복을 피한다.
- 깊게 중첩된 state를 피한다.

## 관련 state 그룹화 하기

```javascript
const [x, setX] = useState(0);
const [y, setY] = useState(0);
```

```javascript
const [positon, setPosition] = useState({ x: 0, y: 0 });
```

두 개의 state 변수가 항상 함께 변경되는 경우에는 하나의 state 변수로 통합하는 것이 좋다.

### 함정

state 변수가 객체인 경우 다른 필드를 명시적으로 복사하지 않고는 그 안의 한 필드만 업데이트할 수 없다는 점을 기억하자.

## 불필요한 state 피하기

렌더링 중에 컴포넌트의 props나 기존 state 변수에서 일부 정보를 계산할 수 있는 경우 해당 정보를 컴포넌트의 state에 넣지 않아야 한다.

```javascript
import { useState } from "react";

export default function Form() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");
  const [fullName, setFullName] = useState("");

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + " " + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + " " + e.target.value);
  }

  return (
    <>
      <h2>Let’s check you in</h2>
      <label>
        First name: <input value={firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name: <input value={lastName} onChange={handleLastNameChange} />
      </label>
      <p>
        Your ticket will be issued to: <b>{fullName}</b>
      </p>
    </>
  );
}
```

위 코드에서 fullName은 불필요하다. 렌더링 중에 언제든지 firstName과 lastName에서 fullName을 계산할 수 있으므로 state에서 제거해야한다.

```javascript
//코드 생략
const fullName = firstName + " " + lastName;

function handleFirstNameChange(e) {
  setFirstName(e.target.value);
}

function handleLastNameChange(e) {
  setLastName(e.target.value);
}
```

변경 핸들러는 이를 업데이트하기 위해 특별한 작업을 수행할 필요가 없다. setFirstName 또는 setLastName을 호출하면 **다시 렌더링을 촉발하고 다음 fullName이 새 데이터에서 계산된다.**

## state 중복을 피하기

```javascript
import { useState } from "react";

const initialItems = [
  { title: "pretzels", id: 0 },
  { title: "crispy seaweed", id: 1 },
  { title: "granola bar", id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(items[0]);

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map((item) => (
          <li key={item.id}>
            {item.title}{" "}
            <button
              onClick={() => {
                setSelectedItem(item);
              }}
            >
              Choose
            </button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

현재 선택된 항목은 selectedItem state 변수에 객체로 저장된다. 그러나 이것은 좋지 않다. selectedItem의 내용은 items 목록 내에 항목 중 하나와 동일한 객체이다. **즉, 항복 자체에 대한 정보가 두 곳에 중복된다.**

```javascript
export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedId, setSelectedId] = useState(0);

  const selectedItem = items.find((item) => item.id === selectedId);

  function handleItemChange(id, e) {
    setItems(
      items.map((item) => {
        if (item.id === id) {
          return {
            ...item,
            title: e.target.value,
          };
        } else {
          return item;
        }
      })
    );
  }

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={(e) => {
                handleItemChange(item.id, e);
              }}
            />{" "}
            <button
              onClick={() => {
                setSelectedId(item.id);
              }}
            >
              Choose
            </button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

## 깊게 중첩된 state를 피한다.

중첩된 state를 업데이트하려면 변경된 부분부터 위쪽까지 객체의 복사본을 만들어야 한다. 이러한 코드는 매우 장황할 수 있다,.

깊게 중첩된 state를 업데이트하는 것이 복잡하다면 state를 평평하게 만들어 보자.
