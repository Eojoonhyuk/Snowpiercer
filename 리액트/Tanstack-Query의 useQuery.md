## Tanstack Query 라이브러리의 내부 구조

### QueryClient

```js
export class QueryClient{
  private queryCache: QueryCache

  constructor(config: QueryClientConfig = {}){
    this.queryCache = config.queryCache || new QueryCache()
  }

  setQueryData() {}
  getQueryData() {}
  invalidateQueries() {}
  getQueryCache() {}
  // ...
}
```
QueryClient는 QueryCache를 가지며 이 캐시 인스턴스와 상호작용할 수 있는 몇 가지 메소드를 제공한다.

### QueryCache
```js
export class QueryCache extends Subscribable<> {
  private queries: Query<any, any, any, any>[]
  private queriesMap: QueryHashMap

  constructor(config?: QueryCacheConfig) {
    this.queries = []
    this.queriesMap = {}
  }

  build() {}
  add() {}
  remove() {}
  // ...
}
```
QueryCache는 여러 개의 Query 인스턴스들을 자신의 내부 프로퍼티 필드인 queries, queriesMap에 등록하여 관리하는 역할을 한다. 일반적으로 개발자는 queryClient 인스턴스에 접근하여 캐시와 상호작용할 수 있다. 캐시에 직접 접근하여 상호작용할 필요가 없다.

#### QueryCache가 제공하는 build 메서드
```js
export class QueryCache extends Subscribable<> {
  private queries: Query<any, any, any, any>[]
  private queriesMap: QueryHashMap

  constructor(config?: QueryCacheConfig) {}

  build(client, options, state?) {
    const queryKey = options.queryKey!
    const queryHash = options.queryHash ?? hashQueryKeyByOptions(queryKey, options)
    let query = this.get(queryHash)
    if (!query) {
      query = new Query({
        cache: this,
        logger: client.getLogger(),
        queryKey,
        queryHash,
        options: client.defaultQueryOptions(options),
        state,
        defaultOptions: client.getQueryDefaults(queryKey),
      })
      this.add(query)
    }
    return query
  }

  add(query) {
    if (!this.queriesMap[query.queryHash]) {
      this.queriesMap[query.queryHash] = query
      this.queries.push(query)
    }
  }
}
```
QueryCache의 build
- 옵션 객체의 주어진 queryKey 데이터를 해시한다.
- queryHash를 키값으로 활용하여 queriesMap에 저장된 Query 인스턴스를 조회해본다.
- Query 인스턴스가 존재한다면 바로 반환하고 존재하지 않는다면 새로 생성해서 캐시에 등록 후 반환한다.


### Query
```js
export class Query extends Removable {
  queryKey
  queryHash
  options!: QueryOptions<TQueryFnData, TError, TData, TQueryKey>
  state: QueryState<TData, TError>

  private cache: QueryCache
  private observers: QueryObserver<any, any, any, any, any>[]

  constructor(config: QueryConfig<TQueryFnData, TError, TData, TQueryKey>) {
    super()

    this.observers = []
    this.cache = config.cache
    this.queryKey = config.queryKey
    this.queryHash = config.queryHash
    this.state = this.initialState
  }

  dispatch() {}
  addObserver() {}
  removeObserver() {}
  fetch() {}
  // ...
}
```
Query는 QueryCache가 가진 queries, queriesMap에 저장되는 인스턴스이며 다음과 같은 정보를 가지고 있다.
- Query 인스턴스 자신을 소유하고 관리하고 있는 QueryCache에 대한 참조
- useQuery로 전달한 queryFn의 호출 결과 값 데이터
- Query가 갖고 있는 QueryState 데이터가 변경되는 경우 알림을 전달해야 하는 QueryObserver 목록

#### Query가 가진 QueryState의 업데이트가 발생하는 경우
```js
export class Query extends Removable {
  constructor() {}

  dispatch(action: Action): void {
    const reducer = (state: QueryState): QueryState => {
      switch (action.type) {
        case 'fetch': return {...}
        case 'error': return {...}     
        case 'success': return {
          ...state,
          data: action.data,
          dataUpdateCount: state.dataUpdateCount + 1,
          dataUpdatedAt: action.dataUpdatedAt ?? Date.now(),
          error: null,
          isInvalidated: false,
          status: 'success',
        }  
      }
    }

    this.state = reducer(this.state)

    notifyManager.batch(() => {
      this.observers.forEach((observer) => {
        observer.onQueryUpdate(action)
      })
      /**
       * https://tanstack.com/query/v4/docs/react/guides/migrating-to-react-query-4#streamlined-notifyevents
       * QueryCache 클래스도 Subscribable 클래스를 상속하기 때문에 구독 가능합니다.
       * 개발자가 메뉴얼하게 리스너를 연결하는 것을 요구하기 때문에 내부 동작을 이해하기 위한 이번 포스트 내용과는 큰 연관이 없습니다.
       */
      this.cache.notify({ query: this, type: 'updated', action })
    })
  }
}
```

QueryState 업데이트가 발생하는 경우 QueryObserver에게 지속해서 알림을 주기 위해 Dispatch, Reducer, Action 개념을 사용합니다. 액션의 타입에 따라 리듀서를 통해 QueryState를 새롭게 업데이트한 후, 현재 Query 인스턴스를 구독하고 있는 QueryObserver 인스턴스의 메소드를 실행하는 것을 확인할 수 있다.

### QueryObserver
```js
export class QueryObserver extends Subscribable {

  constructor() {}

  onQueryUpdate(action) {
    const notifyOptions: NotifyOptions = {}

    if (action.type === 'success') {
      notifyOptions.onSuccess = !action.manual
    } else if (action.type === 'error' && !isCancelledError(action.error)) {
      notifyOptions.onError = true
    }

    this.updateResult(notifyOptions)

    if (this.hasListeners()) {
      this.updateTimers()
    }
  }
}
```
QueryObserver는 Query의 변화를 관찰하고 리스너에게 알리는 역할을 한다. 변화가 관찰되었을 때 내부의 최적화 과정을 거치게 되며 최종적으로 변화가 발생했음을 리스너에게 알릴 필요가 있는 경우 그 대상(현재의 문맥에서는 React 컴포넌트)에게 알려준다.

<hr/>
리액트 컴포넌트상에서의 useQuery훅을 호출하게 되면 Query의 변화를 관찰하는 QueryObserver가 생성되어 컴포넌트와 연결을 맺게 되고 Query의 변화에 따라 컴포넌트는 리렌더링이 발생하게 된다.

1. 리액트 컴포넌트에서 useQuery훅을 호출한다.
2. useBaseQuery훅으로 정제된 옵션과 QueryObserver 클래스가 전달된다.
3. useBaseQuery훅의 실행과정에서 리액트 컴포넌트는 useBaseQuery 인스턴스와 연결된다.
4. useBaseQuery가 Query의 변화를 관찰하여 notify 메소드를 실행하게 되면 리액트 컴포넌트의 리렌더링이 발생하게 된다.

참고 자료: https://fe-developers.kakaoent.com/2023/230720-react-query/
