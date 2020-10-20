---
id: getQueryData
title: getQueryData
---

`getQueryData` is a synchronous function that can be used to get an existing query's cached data. If the query does not exist, `undefined` will be returned.

```js
import { getQueryData } from 'react-query'

const data = getQueryData(environment, queryKey)
```

**Options**

- `environment: Environment`
- `queryKey?: QueryKey`: [Query Keys](../guides/query-keys)
- `filters?: QueryFilters`: [Query Filters](../guides/query-filters)

**Returns**

- `data: TData | undefined`
  - The data for the cached query, or `undefined` if the query does not exist.
