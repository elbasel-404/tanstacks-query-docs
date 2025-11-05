---
id: query-invalidation-tutorial
title: Query Invalidation Tutorial
---

# Query Invalidation Tutorial

Learn how to keep your data fresh by invalidating and refetching queries in TanStack Query.

## What is Query Invalidation?

Query invalidation is the process of marking cached data as stale, which triggers a refetch. This ensures your UI always shows the most up-to-date data from the server.

## When to Invalidate

Common scenarios for query invalidation:

- After creating a new item (invalidate the list query)
- After updating an item (invalidate both the item and list queries)
- After deleting an item (invalidate the list query)
- On user action (like clicking a refresh button)
- After a certain time period

## Basic Invalidation

### React Example

```tsx
import { useQueryClient } from '@tanstack/react-query'

function RefreshButton() {
  const queryClient = useQueryClient()

  return (
    <button
      onClick={() => {
        // Invalidate all queries with 'users' key
        queryClient.invalidateQueries({ queryKey: ['users'] })
      }}
    >
      Refresh Users
    </button>
  )
}
```

## Invalidation After Mutations

The most common pattern is to invalidate queries after a successful mutation:

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query'

function CreatePost() {
  const queryClient = useQueryClient()

  const mutation = useMutation({
    mutationFn: async (newPost) => {
      const response = await fetch('/api/posts', {
        method: 'POST',
        body: JSON.stringify(newPost),
      })
      return response.json()
    },
    onSuccess: () => {
      // Invalidate the posts list to refetch
      queryClient.invalidateQueries({ queryKey: ['posts'] })
    },
  })

  return (
    <button onClick={() => mutation.mutate({ title: 'New Post' })}>
      Create Post
    </button>
  )
}
```

## Selective Invalidation

You can be more specific about which queries to invalidate:

```tsx
const queryClient = useQueryClient()

// Invalidate all queries
queryClient.invalidateQueries()

// Invalidate all queries with 'posts' key
queryClient.invalidateQueries({ queryKey: ['posts'] })

// Invalidate specific post
queryClient.invalidateQueries({ queryKey: ['posts', postId] })

// Invalidate all queries that start with 'posts'
queryClient.invalidateQueries({ queryKey: ['posts'] })

// Exact match only
queryClient.invalidateQueries({ 
  queryKey: ['posts'],
  exact: true 
})
```

## Invalidation with Filters

Use filters for more advanced invalidation patterns:

```tsx
// Invalidate all active queries
queryClient.invalidateQueries({
  predicate: (query) => query.state.status === 'success'
})

// Invalidate queries by type
queryClient.invalidateQueries({
  queryKey: ['posts'],
  refetchType: 'active' // or 'inactive' or 'all'
})
```

## Refetch vs Invalidation

There's a subtle difference:

```tsx
// Invalidate: marks as stale and refetches if currently in use
queryClient.invalidateQueries({ queryKey: ['users'] })

// Refetch: immediately refetches regardless of staleness
queryClient.refetchQueries({ queryKey: ['users'] })
```

## Multiple Query Invalidation

Invalidate multiple related queries at once:

```tsx
const mutation = useMutation({
  mutationFn: updateUser,
  onSuccess: (data, userId) => {
    // Invalidate multiple related queries
    queryClient.invalidateQueries({ queryKey: ['users'] })
    queryClient.invalidateQueries({ queryKey: ['user', userId] })
    queryClient.invalidateQueries({ queryKey: ['user-stats', userId] })
  },
})
```

## Automatic Invalidation

Set up automatic invalidation based on time:

```tsx
const { data } = useQuery({
  queryKey: ['news'],
  queryFn: fetchNews,
  staleTime: 5 * 60 * 1000, // 5 minutes
  refetchInterval: 30 * 1000, // Refetch every 30 seconds
})
```

## Preventing Unnecessary Refetches

Sometimes you want to invalidate without refetching:

```tsx
queryClient.invalidateQueries({
  queryKey: ['posts'],
  refetchType: 'none' // Don't refetch, just mark as stale
})
```

## Practical Example: Todo App

Here's a complete example showing invalidation patterns in a Todo app:

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'

function TodoApp() {
  const queryClient = useQueryClient()

  // Fetch todos
  const { data: todos } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  })

  // Create todo
  const createTodo = useMutation({
    mutationFn: addTodo,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  })

  // Update todo
  const updateTodo = useMutation({
    mutationFn: ({ id, completed }) => 
      fetch(`/api/todos/${id}`, {
        method: 'PATCH',
        body: JSON.stringify({ completed }),
      }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  })

  // Delete todo
  const deleteTodo = useMutation({
    mutationFn: (id) => 
      fetch(`/api/todos/${id}`, { method: 'DELETE' }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  })

  return (
    <div>
      <h1>Todos</h1>
      <ul>
        {todos?.map((todo) => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => 
                updateTodo.mutate({ id: todo.id, completed: !todo.completed })
              }
            />
            {todo.text}
            <button onClick={() => deleteTodo.mutate(todo.id)}>
              Delete
            </button>
          </li>
        ))}
      </ul>
      <button onClick={() => createTodo.mutate({ text: 'New Todo' })}>
        Add Todo
      </button>
    </div>
  )
}
```

## Best Practices

1. **Invalidate after mutations** to keep data synchronized
2. **Use specific query keys** to avoid unnecessary refetches
3. **Consider optimistic updates** for better UX
4. **Be strategic** about refetch timing
5. **Use predicate functions** for complex invalidation logic

## Next Steps

- Learn about [Optimistic Updates](../framework/react/guides/optimistic-updates.md)
- Explore [Advanced Caching Strategies](../framework/react/guides/caching.md)
- Check out [Mutation Examples](../framework/react/examples/optimistic-updates-ui)
