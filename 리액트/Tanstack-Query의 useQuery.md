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
