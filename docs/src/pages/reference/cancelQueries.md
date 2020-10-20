---
id: cancelQueries
title: cancelQueries
---

The `cancelQueries` function can be used to cancel outgoing queries based on their query keys or any other functionally accessible property/state of the query.

This is most useful when performing optimistic updates since you will likely need to cancel any outgoing query refetches so they don't clobber your optimistic update when they resolve.

```js
import { cancelQueries } from 'react-query'

await cancelQueries(environment, 'posts', { exact: true })
```

**Options**

- `environment: Environment`
- `queryKey?: QueryKey`: [Query Keys](../guides/query-keys)
- `filters?: QueryFilters`: [Query Filters](../guides/query-filters)

**Returns**

This method does not return anything
