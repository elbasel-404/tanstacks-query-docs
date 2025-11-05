---
id: solid-quick-start
title: Solid Quick Start
---

# Solid Quick Start

Get up and running with TanStack Query in Solid in minutes.

## Installation

```bash
npm install @tanstack/solid-query
# or
yarn add @tanstack/solid-query
# or
pnpm add @tanstack/solid-query
```

## Setup

Create and provide a QueryClient to your application:

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/solid-query'
import { render } from 'solid-js/web'

const queryClient = new QueryClient()

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
    </QueryClientProvider>
  )
}

render(() => <App />, document.getElementById('root'))
```

## Your First Query

```tsx
import { createQuery } from '@tanstack/solid-query'
import { For, Show } from 'solid-js'

function Todos() {
  const query = createQuery(() => ({
    queryKey: ['todos'],
    queryFn: async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/todos')
      if (!response.ok) {
        throw new Error('Network response was not ok')
      }
      return response.json()
    }
  }))

  return (
    <div>
      <Show when={query.isLoading}>
        <div>Loading...</div>
      </Show>

      <Show when={query.error}>
        <div>Error: {query.error.message}</div>
      </Show>

      <Show when={query.data}>
        <ul>
          <For each={query.data}>
            {(todo) => (
              <li>
                {todo.completed ? '✅' : '⬜'} {todo.title}
              </li>
            )}
          </For>
        </ul>
      </Show>
    </div>
  )
}
```

## Your First Mutation

```tsx
import { createMutation, useQueryClient } from '@tanstack/solid-query'
import { Show } from 'solid-js'

function AddTodo() {
  const queryClient = useQueryClient()

  const mutation = createMutation(() => ({
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
  }))

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
      
      <Show when={mutation.isPending}>
        <div>Adding todo...</div>
      </Show>

      <Show when={mutation.isError}>
        <div>Error: {mutation.error.message}</div>
      </Show>

      <Show when={mutation.isSuccess}>
        <div>Todo added!</div>
      </Show>
    </div>
  )
}
```

## Add DevTools (Optional)

Install and add the Solid Query DevTools for debugging:

```bash
npm install @tanstack/solid-query-devtools
```

```tsx
import { SolidQueryDevtools } from '@tanstack/solid-query-devtools'

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
      <SolidQueryDevtools />
    </QueryClientProvider>
  )
}
```

## Complete Example

Here's a complete working example:

```tsx
import { render } from 'solid-js/web'
import { For, Show } from 'solid-js'
import {
  QueryClient,
  QueryClientProvider,
  createQuery,
  createMutation,
  useQueryClient,
} from '@tanstack/solid-query'
import { SolidQueryDevtools } from '@tanstack/solid-query-devtools'

const queryClient = new QueryClient()

function TodoList() {
  const queryClient = useQueryClient()

  // Fetch todos
  const todosQuery = createQuery(() => ({
    queryKey: ['todos'],
    queryFn: async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/todos?_limit=5')
      if (!response.ok) throw new Error('Failed to fetch')
      return response.json()
    }
  }))

  // Add todo mutation
  const addTodoMutation = createMutation(() => ({
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
  }))

  // Toggle todo mutation
  const toggleTodoMutation = createMutation(() => ({
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
  }))

  const addTodo = () => {
    addTodoMutation.mutate({
      title: `New Todo ${Date.now()}`,
      completed: false,
    })
  }

  const toggleTodo = (todo) => {
    toggleTodoMutation.mutate({
      id: todo.id,
      completed: !todo.completed,
    })
  }

  return (
    <div style={{ padding: '20px' }}>
      <h1>Todo List</h1>
      
      <button
        onClick={addTodo}
        disabled={addTodoMutation.isPending}
      >
        {addTodoMutation.isPending ? 'Adding...' : 'Add Todo'}
      </button>

      <Show when={todosQuery.isLoading}>
        <div>Loading...</div>
      </Show>

      <Show when={todosQuery.error}>
        <div>Error: {todosQuery.error.message}</div>
      </Show>

      <Show when={todosQuery.data}>
        <ul style={{ 'margin-top': '20px' }}>
          <For each={todosQuery.data}>
            {(todo) => (
              <li style={{ 'margin-bottom': '10px' }}>
                <input
                  type="checkbox"
                  checked={todo.completed}
                  onChange={() => toggleTodo(todo)}
                />
                <span style={{ 'margin-left': '10px' }}>{todo.title}</span>
              </li>
            )}
          </For>
        </ul>
      </Show>
    </div>
  )
}

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <TodoList />
      <SolidQueryDevtools />
    </QueryClientProvider>
  )
}

render(() => <App />, document.getElementById('root')!)
```

## Reactive Query Keys

Solid Query automatically tracks reactive dependencies:

```tsx
import { createSignal } from 'solid-js'
import { createQuery } from '@tanstack/solid-query'

function User() {
  const [userId, setUserId] = createSignal(1)

  // Query automatically refetches when userId changes
  const userQuery = createQuery(() => ({
    queryKey: ['user', userId()],
    queryFn: () => fetchUser(userId()),
  }))

  return (
    <div>
      <input
        type="number"
        value={userId()}
        onInput={(e) => setUserId(parseInt(e.currentTarget.value))}
      />
      <Show when={userQuery.data}>
        <div>{userQuery.data.name}</div>
      </Show>
    </div>
  )
}
```

## Using with SolidStart

For SolidStart, set up the QueryClient in your root:

```tsx
// root.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/solid-query'

export default function Root() {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 60 * 1000,
      },
    },
  })

  return (
    <Html>
      <Head>
        <Title>SolidStart App</Title>
      </Head>
      <Body>
        <QueryClientProvider client={queryClient}>
          <Routes>
            <FileRoutes />
          </Routes>
        </QueryClientProvider>
        <Scripts />
      </Body>
    </Html>
  )
}
```

## Suspense Mode

Solid Query works great with Suspense:

```tsx
import { Suspense } from 'solid-js'
import { createQuery } from '@tanstack/solid-query'

function Posts() {
  const query = createQuery(() => ({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  }))

  // Access data directly - will suspend if not ready
  const posts = () => query.data

  return (
    <For each={posts()}>
      {(post) => <Post post={post} />}
    </For>
  )
}

function App() {
  return (
    <Suspense fallback={<div>Loading posts...</div>}>
      <Posts />
    </Suspense>
  )
}
```

## Dependent Queries

Execute queries that depend on previous data:

```tsx
function UserProjects(props) {
  // Get user first
  const userQuery = createQuery(() => ({
    queryKey: ['user', props.userId],
    queryFn: () => fetchUser(props.userId),
  }))

  // Then fetch user's projects
  const projectsQuery = createQuery(() => ({
    queryKey: ['projects', userQuery.data?.id],
    queryFn: () => fetchUserProjects(userQuery.data.id),
    enabled: !!userQuery.data?.id, // Only run when user data exists
  }))

  return (
    <Show when={projectsQuery.data}>
      <ProjectList projects={projectsQuery.data} />
    </Show>
  )
}
```

## Next Steps

- Learn about [SSR with SolidStart](../../framework/solid/guides/ssr.md)
- Explore [Suspense Integration](../../framework/solid/guides/suspense.md)
- Check out [Query Options](../../framework/solid/guides/query-options.md)
- View [Examples](../../framework/solid/examples/simple.md)

## Additional Resources

- [TypeScript Guide](../../framework/solid/typescript.md)
- [Testing Strategies](../../framework/solid/guides/testing.md)
- [Advanced SSR](../../framework/solid/guides/advanced-ssr.md)
