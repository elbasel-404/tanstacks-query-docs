---
id: react-quick-start
title: React Quick Start
---

# React Quick Start

Get up and running with TanStack Query in React in minutes.

## Installation

```bash
npm install @tanstack/react-query
# or
yarn add @tanstack/react-query
# or
pnpm add @tanstack/react-query
```

## Setup QueryClient

Create and provide a QueryClient to your application:

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

// Create a client
const queryClient = new QueryClient()

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
    </QueryClientProvider>
  )
}
```

## Your First Query

```tsx
import { useQuery } from '@tanstack/react-query'

function Example() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['todos'],
    queryFn: async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/todos')
      if (!response.ok) {
        throw new Error('Network response was not ok')
      }
      return response.json()
    }
  })

  if (isLoading) return <div>Loading...</div>
  if (error) return <div>Error: {error.message}</div>

  return (
    <ul>
      {data.map(todo => (
        <li key={todo.id}>
          {todo.completed ? '✅' : '⬜'} {todo.title}
        </li>
      ))}
    </ul>
  )
}
```

## Your First Mutation

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query'

function AddTodo() {
  const queryClient = useQueryClient()

  const mutation = useMutation({
    mutationFn: async (newTodo) => {
      const response = await fetch('https://jsonplaceholder.typicode.com/todos', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(newTodo),
      })
      return response.json()
    },
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  })

  return (
    <div>
      <button
        onClick={() => {
          mutation.mutate({
            title: 'New Todo',
            completed: false,
          })
        }}
      >
        Add Todo
      </button>
      {mutation.isPending && <div>Adding todo...</div>}
      {mutation.isError && <div>Error: {mutation.error.message}</div>}
      {mutation.isSuccess && <div>Todo added!</div>}
    </div>
  )
}
```

## Add DevTools (Optional)

Install and add the React Query DevTools for debugging:

```bash
npm install @tanstack/react-query-devtools
```

```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```

## Complete Example

Here's a complete working example:

```tsx
import {
  QueryClient,
  QueryClientProvider,
  useQuery,
  useMutation,
  useQueryClient,
} from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

const queryClient = new QueryClient()

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <TodoApp />
      <ReactQueryDevtools />
    </QueryClientProvider>
  )
}

function TodoApp() {
  const queryClient = useQueryClient()

  // Fetch todos
  const { data: todos, isLoading, error } = useQuery({
    queryKey: ['todos'],
    queryFn: async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/todos?_limit=5')
      return response.json()
    }
  })

  // Add todo mutation
  const addTodoMutation = useMutation({
    mutationFn: async (newTodo) => {
      const response = await fetch('https://jsonplaceholder.typicode.com/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newTodo),
      })
      return response.json()
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  })

  // Toggle todo mutation
  const toggleTodoMutation = useMutation({
    mutationFn: async ({ id, completed }) => {
      const response = await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ completed }),
      })
      return response.json()
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  })

  if (isLoading) return <div>Loading...</div>
  if (error) return <div>Error: {error.message}</div>

  return (
    <div style={{ padding: '20px' }}>
      <h1>Todo List</h1>
      
      <button
        onClick={() => {
          addTodoMutation.mutate({
            title: `New Todo ${Date.now()}`,
            completed: false,
          })
        }}
        disabled={addTodoMutation.isPending}
      >
        {addTodoMutation.isPending ? 'Adding...' : 'Add Todo'}
      </button>

      <ul style={{ marginTop: '20px' }}>
        {todos?.map(todo => (
          <li key={todo.id} style={{ marginBottom: '10px' }}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => {
                toggleTodoMutation.mutate({
                  id: todo.id,
                  completed: !todo.completed,
                })
              }}
            />
            <span style={{ marginLeft: '10px' }}>{todo.title}</span>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

## Next Steps

- Learn about [Query Keys](../../framework/react/guides/query-keys.md)
- Explore [Mutations](../../framework/react/guides/mutations.md)
- Check out [Caching](../../framework/react/guides/caching.md)
- View [Examples](../../framework/react/examples/simple.md)

## Additional Resources

- [TypeScript Guide](../../framework/react/typescript.md)
- [React Native Support](../../framework/react/react-native.md)
- [Server-Side Rendering](../../framework/react/guides/ssr.md)
