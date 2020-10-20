---
id: migrating-to-react-query-3
title: Migrating to React Query 3
---

## V3 migration

This article explains how to migrate your application to React Query 3.

### Environment

The original `QueryCache` has been split into an `Environment`, `QueryCache`, and `MutationCache`.
The `QueryCache` contains all queries, the `MutationCache` contains all mutations, and the `Environment` is used to bundle together configuration and caches.

This has some benefits:

- Allows for different type of caches.
- Multiple environments with different configurations can use the same caches.
- Environments can be used to track queries, which can be used for shared caches on SSR.

Use the `EnvironmentProvider` component to connect a `Environment` to your application:

```js
import { Environment, EnvironmentProvider, QueryCache } from 'react-query'

const environment = new Environment({
  queryCache: new QueryCache(),
  mutationCache: new MutationCache(), // can be omitted to reduce file size when not using mutations
})

function App() {
  return (
    <EnvironmentProvider environment={environment}>...</EnvironmentProvider>
  )
}
```

### useQueryCache()

The `useQueryCache()` hook has been replaced by the `useEnvironment()` hook:

```js
import { useCallback } from 'react'
import { useEnvironment } from 'react-query'

function Todo() {
  const environment = useEnvironment()

  const onClickButton = useCallback(() => {
    invalidateQueries(environment, 'posts')
  }, [client])

  return <button onClick={onClickButton}>Refetch</button>
}
```

### ReactQueryConfigProvider

The `ReactQueryConfigProvider` component has been removed. Default options for queries and mutations can now be specified in `Environment`:

```js
const environment = new Environment({
  queryCache: new QueryCache(),
  defaultOptions: {
    queries: {
      staleTime: Infinity,
    },
  },
})
```

### usePaginatedQuery()

The `usePaginatedQuery()` hook has been replaced by the `keepPreviousData` option on `useQuery`:

```js
import { useQuery } from 'react-query'

function Page({ page }) {
  const { data } = useQuery(['page', page], fetchPage, {
    keepPreviousData: true,
  })
}
```

### useInfiniteQuery()

The `useInfiniteQuery()` interface has changed to fully support bi-directional infinite lists and manual updates.

The `data` of an infinite query is now an object containing the `pages` and the `pageParams` used to fetch the pages.

One direction:

```js
const {
  data,
  fetchNextPage,
  hasNextPage,
  isFetchingNextPage,
} = useInfiniteQuery('projects', fetchProjects, {
  getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
})
```

Both directions:

```js
const {
  data,
  fetchNextPage,
  fetchPreviousPage,
  hasNextPage,
  hasPreviousPage,
  isFetchingNextPage,
  isFetchingPreviousPage,
} = useInfiniteQuery('projects', fetchProjects, {
  getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
  getPreviousPageParam: (firstPage, pages) => firstPage.prevCursor,
})
```

One direction reversed:

```js
const {
  data,
  fetchNextPage,
  hasNextPage,
  isFetchingNextPage,
} = useInfiniteQuery('projects', fetchProjects, {
  select: data => ({
    pages: [...data.pages].reverse(),
    pageParams: [...data.pageParams].reverse(),
  }),
  getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
})
```

Manually removing the first page:

```js
setQueryData(environment, 'projects', data => ({
  pages: data.pages.slice(1),
  pageParams: data.pageParams.slice(1),
}))
```

### useMutation()

The `useMutation()` hook now returns an object instead of an array:

```js
// Old:
const [mutate, { status, reset }] = useMutation()

// New:
const { mutate, status, reset } = useMutation()
```

Previously the `mutate` function returned a promise which resolved to `undefined` if a mutation failed instead of throwing.
We got a lot of questions regarding this behavior as users expected the promise to behave like a regular promise.
Because of this the `mutate` function is now split into a `mutate` and `mutateAsync` function.

The `mutate` function can be used when using callbacks:

```js
const { mutate } = useMutation(addTodo)

mutate('todo', {
  onSuccess: data => {
    console.log(data)
  },
  onError: error => {
    console.error(error)
  },
  onSettled: () => {
    console.log('settled)
  },
})
```

The `mutateAsync` function can be used when using async/await:

```js
const { mutateAsync } = useMutation(addTodo)

try {
  const data = await mutateAsync('todo')
  console.log(data)
} catch (error) {
  console.error(error)
} finally {
  console.log('settled)
}
```

Callbacks passed to the `mutate` or `mutateAsync` functions will now override the callbacks defined on `useMutation`.
The `mutateAsync` function can be used to compose side effects.

### Query object syntax

The object syntax has been collapsed:

```js
// Old:
useQuery({
  queryKey: 'posts',
  queryFn: fetchPosts,
  config: { staleTime: Infinity },
})

// New:
useQuery({
  queryKey: 'posts',
  queryFn: fetchPosts,
  staleTime: Infinity,
})
```

### queryCache.prefetchQuery()

The `prefetchQuery` function should now only be used for prefetching scenarios where the result is not relevant.

Use the `fetchQuery` method to get the query data or error:

```js
// Prefetch a query:
await prefetchQuery(environment, {
  queryKey: 'posts',
  queryFn: fetchPosts,
})

// Fetch a query:
try {
  const data = await fetchQuery(environment, {
    queryKey: 'posts',
    queryFn: fetchPosts,
  })
} catch (error) {
  // Error handling
}
```

### ReactQueryCacheProvider

The `ReactQueryCacheProvider` component has been replaced by the `EnvironmentProvider` component.

### makeQueryCache()

The `makeQueryCache()` function has replaced by `new QueryCache()`.

### ReactQueryErrorResetBoundary

The `ReactQueryErrorResetBoundary` component has been renamed to `QueryErrorResetBoundary`.

### queryCache.resetErrorBoundaries()

The `queryCache.resetErrorBoundaries()` method has been replaced by the `QueryErrorResetBoundary` component.

### queryCache.fetchQuery()

The `queryCache.fetchQuery()` method has been replaced by `fetchQuery()`.

### queryCache.prefetchQuery()

The `queryCache.prefetchQuery()` method has been replaced by `prefetchQuery()`.

### queryCache.invalidateQueries()

The `queryCache.invalidateQueries()` method has been replaced by `invalidateQueries()`.

### queryCache.refetchQueries()

The `queryCache.refetchQueries()` method has been replaced by `refetchQueries()`.

### queryCache.cancelQueries()

The `queryCache.cancelQueries()` method has been replaced by `cancelQueries()`.

### queryCache.removeQueries()

The `queryCache.removeQueries()` method has been replaced by `removeQueries()`.

### queryCache.setQueryData()

The `queryCache.setQueryData()` method has been replaced by `setQueryData()`.

### queryCache.getQueryData()

The `queryCache.getQueryData()` method has been replaced by `getQueryData()`.

### queryCache.getQuery()

The `queryCache.getQuery()` method has been replaced by `findQuery()`.

### queryCache.getQueries()

The `queryCache.getQueries()` method has been replaced by `findQueries()`.

### queryCache.isFetching

The `queryCache.isFetching` property has been replaced by `isFetching()`.

### QueryOptions.enabled

The `enabled` query option will now only disable a query when the value is `false`.
If needed, values can be casted with `!!userId` or `Boolean(userId)`.

### QueryOptions.initialStale

The `initialStale` query option has been removed and initial data is now treated as regular data.
Which means that if `initialData` is provided, the query will refetch on mount by default.
If you do not want to refetch immediately, you can define a `staleTime`.

### QueryOptions.forceFetchOnMount

The `forceFetchOnMount` query option has been replaced by `refetchOnMount: 'always'`.

### QueryOptions.refetchOnMount

When `refetchOnMount` was set to `false` any additional components were prevented from refetching on mount.
In version 3 only the component where the option has been set will not refetch on mount.

### QueryResult.clear()

The `QueryResult.clear()` method has been renamed to `QueryResult.remove()`.

### setConsole

The `setConsole` function has been replaced by `setLogger`:

```js
import { setLogger } from 'react-query'

// Log with Sentry
setLogger({
  error: error => {
    Sentry.captureException(error)
  },
})

// Log with Winston
setLogger(winston.createLogger())
```

To prevent showing error screens in React Native when a query fails it was necessary to manually change the Console:

```js
import { setConsole } from 'react-query'

setConsole({
  log: console.log,
  warn: console.warn,
  error: console.warn,
})
```

In version 3 this is done automatically when React Query is used in React Native.

### New features

Some new features have also been added besides the API changes, performance improvements and file size reduction.

#### Selectors

The `useQuery` and `useInfiniteQuery` hooks now have a `select` option to select or transform parts of the query result.

```js
import { useQuery } from 'react-query'

function User() {
  const { data } = useQuery('user', fetchUser, {
    select: user => user.username,
  })
  return <div>Username: {data}</div>
}
```

Set the `notifyOnStatusChange` option to `false` to only re-render when the selected data changes.

#### useQueries()

The `useQueries()` hook can be used to fetch a variable number of queries:

```js
import { useQueries } from 'react-query'

function Overview() {
  const results = useQueries([
    { queryKey: ['post', 1], queryFn: fetchPost },
    { queryKey: ['post', 2], queryFn: fetchPost },
  ])
  return (
    <ul>
      {results.map(({ data }) => data && <li key={data.id}>{data.title})</li>)}
    </ul>
  )
}
```

#### watchQuery

The `wachtQuery` function can be used to create and/or watch a query:

```js
const observer = watchQuery(environment, { queryKey: 'posts' })

const unsubscribe = observer.subscribe(result => {
  console.log(result)
  unsubscribe()
})
```

#### watchQueries

The `watchQueries` function can be used to create and/or watch multiple queries:

```js
const observer = watchQueries(environment, [
  { queryKey: ['post', 1], queryFn: fetchPost },
  { queryKey: ['post', 2], queryFn: fetchPost },
])

const unsubscribe = observer.subscribe(result => {
  console.log(result)
  unsubscribe()
})
```

## `environment.setQueryDefaults`

The `environment.setQueryDefaults()` method can be used to set default options for specific queries:

```js
environment.setQueryDefaults('posts', { queryFn: fetchPosts })

function Component() {
  const { data } = useQuery('posts')
}
```

## `environment.setMutationDefaults`

The `environment.setMutationDefaults()` method can be used to set default options for specific mutations:

```js
environment.setMutationDefaults('addPost', { mutationFn: addPost })

function Component() {
  const { mutate } = useMutation('addPost')
}
```

#### useIsFetching()

The `useIsFetching()` hook now accepts filters which can be used to for example only show a spinner for certain type of queries:

```js
const fetches = useIsFetching(['posts'])
```

#### Mutations offline support

Mutations now accept the `retry` and `retryDelay` options. When set and mutations fail, they will be retried in the same order when the client reconnects:

```js
const mutation = useMutation(addTodo, {
  retry: 3,
})
```

#### Core separation

The core of React Query is now fully separated from React, which means it can also be used standalone or in other frameworks. Use the `react-query/core` entrypoint to only import the core functionality:

```js
import { Environment } from 'react-query/core'
```
