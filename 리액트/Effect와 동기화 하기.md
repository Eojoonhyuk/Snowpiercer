## Effect란 무엇이며 이벤트와는 어떤게 다른가요?

**렌더링 코드**는 컴포넌트의 최상위 레벨에 있다. 여기서 props와 state를 가져와 변환하고 표시할 JSX를 반환한다. **렌더링 코드는 순수해야한다.** 수학 공식처럼 결과만 계산할 뿐 다른 작업은 수행하지 않는다.

**이벤트 핸들러**는 컴포넌트 내부에 있는 중첩된 함수로, **계산만 하는 것이 아니라 별도의 작업도 수행한다.** 이벤트 핸들러에서는 입력 필드를 업데이트하거나, HTTP POST요청을 제출하여 제품을 구매하거나, 사용자를 다른 화면으로 이동할 수 있다. 이벤트 핸들러에는 특정 사용자 작업(예: 버튼 클릭 또는 입력)으로 인해 발생하는 **사이드 이펙트(프로그램의 state를 변경함)**가 포함되어 있다.

**Effect를 사용하면 특정 이벤트가 아닌 렌더링 자체로 인해 발생하는 사이드 이펙트를 명시할 수 있다.** 서버 연결을 설정하는 것은 컴포넌트를 표시하게 만든 상호작용에 관계없이 발생해야 하기 때문에 하나의 Effect이다. Effect는 화면 업데이트 후 커밋이 끝날 때 실행된다. 이 때가 React 컴포넌트를 일부 외부 시스템(네트워크 또는 서드파티 라이브러리와 같은)과 동기화하기에 좋은 시기다.

## Effect가 필요하지 않을 수 있다.

Effect를 작성하려면 다음 세 단계를 따를 것

1. Effect를 선언한다. 기본적으로 Effect는 모든 렌더링 후에 실행된다.

2. Effect의 의존성을 명시한다.

3. 필요한 경우 클린업을 추가한다.

## 개발 환경에서 두 번 씩 실행되는 Effect를 처리하는 방법은 무엇일까

React는 개발 환경에서 버그를 찾기 위해 컴포넌트를 의도적으로 다시 마운트한다. 올바른 질문은 "어떻게 하면 Effect를 한 번만 실행할 수 있는가"가 아니라 "어떻게 다시 마운트한 후에도 Effect가 잘 작동하도록 수정하는가"이다.

일반적으로 정답은 클린업 함수를 구현하는 것이다. 클린업 함수는 Effect가 수행 중이던 작업을 중지하거나 취소해야 한다. 한 번만 실행되는 Effect와 (개발 환경에서의) 설정 -> 정리 -> 설정 시퀸스를 사용자가 구분할 수 없어야 한다.

작성하게 될 대부분의 Effect는 아래의 일반적인 패턴 중 하나에 해당한다.

## React가 아닌 위젯 제어하기

```javascript
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

위에 코드는 React로 작성하지 않은 UI 위젯을 추가해야 하는 경우이다. 이 경우 클린업이 필요하지 않다. 개발 환경에서 React는 Effect를 두 번 호출하지만 동일한 값으로 setZommLevel을 두 번 호출해도 아무 작업도 수행하지 않기 때문에 문제가 되지 않는다. 약간 느릴 수는 있지만 상용 환경에서는 불필요하게 다시 마운트되지 않으므로 문제가 되지 않는다.

일부 API는 연속으로 두 번 호출하는 것을 허용하지 않을 수 있다.

```javascript
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);

// 대화 상자가 이미 열려 있거나 대화 상자가 이미 표시되고 있는 팝오버인 경우 InvalidStateError가 발생
// 클린업함수를 추가해서 해결
```

## 이벤트 구독하기

Effect가 무언가를 구독하는 경우, 클린업 함수는 구독을 취소해야 한다.

```javascript
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener("scroll", handleScroll);
  return () => window.removeEventListener("scroll", handleScroll);
}, []);
```

개발 중에 Effect는 addEventListener()를 호출한 다음 즉시 removeEventListener()를 호출한다. 그런 다음 동일한 핸들러를 사용하여 다시 addEventListener()를 사용함으로써, 한 번에 하나의 구독만 활성화 되도록 한다.

## 애니메이션 촉발하기

```javascript
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
  // 애니메이션 촉발
  return () => {
    node.style.opacity = 0; // Reset to the initial value
    // 초기값으로 재설정
  };
}, []);
```

## 데이터 페칭하기

Effect가 무언가를 페치하면 클린업 함수는 페치를 중단하거나 그 결과를 무시해야 한다:

```javascript
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```

이미 발생한 네트워크 요청을 "실행 취소"할 수는 없으므로, 대신 클린업 함수에서 더 이상 관련이 없는 페치가 애플리케이션에 계속 영향을 미치지 않도록 해야한다.

만약 userId가 'Alice'에서 'Bob'으로 변경되면 클린업은 'Alice' 응답이 'Bob' 이후에 도착하더라도 이를 무시하도록 합니다.

## 분석 보내기

```javascript
useEffect(() => {
  logVisit(url); // Sends a POST request
}, [url]);
```

이 코드를 그대로 유지하는 것이 좋다. 어차피 개발 환경에서는 파일을 저장할 때마다 컴포넌트가 다시 마운트 될 것이고, 따라서 추가 방문이 기록된다.

**상용 환경에서는 중복 방문 로그가 없다.**

## Effect가 아님: 애플리케이션 초기화 하기

```javascript
if (typeof window !== "undefined") {
  // Check if we're running in the browser.
  // 실행환경이 브라우저인지 여부 확인
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

일부 로직은 애플리케이션이 시작될 때 한 번만 실행되어야 합니다. 이런 로직은 컴포넌트 외부에 넣을 수 있다.
이렇게 하면 위 로직은 브라우저가 페이지를 로드한 후 한 번만 실행된다.

## Effect가 아님: 제품 구매하기

클린업 함수를 작성하더라도 Effect를 두 번 실행함으로써 체감상 결과가 달라지는 것을 막을 방법이 없는 경우도 잇다. 예를 들어, Effect가 제품 구매와 같은 POST 요청을 보낸다고 해보자.

```javascript
useEffect(() => {
  // 🔴 Wrong: This Effect fires twice in development, exposing a problem in the code.
  // 🔴 틀렸습니다: 이 Effect는 개발모드에서 두 번 실행되며, 문제를 일으킵니다.
  fetch("/api/buy", { method: "POST" });
}, []);
```

구매는 렌더링으로 인한 것이 아니므로 특정 상호 작용에 의해 발생한다. 따라서 이벤트 핸들러로 작성해야 한다.
