---
id: svelte-quick-start
title: Svelte Quick Start
---

# Svelte Quick Start

Get up and running with TanStack Query in Svelte in minutes.

## Installation

```bash
npm install @tanstack/svelte-query
# or
yarn add @tanstack/svelte-query
# or
pnpm add @tanstack/svelte-query
```

## Setup

Create and provide a QueryClient to your application:

```svelte
<script>
  import { QueryClient, QueryClientProvider } from '@tanstack/svelte-query'

  const queryClient = new QueryClient()
</script>

<QueryClientProvider client={queryClient}>
  <YourApp />
</QueryClientProvider>
```

## Your First Query

```svelte
<script>
  import { createQuery } from '@tanstack/svelte-query'

  const query = createQuery({
    queryKey: ['todos'],
    queryFn: async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/todos')
      if (!response.ok) {
        throw new Error('Network response was not ok')
      }
      return response.json()
    }
  })
</script>

<div>
  {#if $query.isLoading}
    <div>Loading...</div>
  {:else if $query.error}
    <div>Error: {$query.error.message}</div>
  {:else}
    <ul>
      {#each $query.data as todo}
        <li>
          {todo.completed ? '✅' : '⬜'} {todo.title}
        </li>
      {/each}
    </ul>
  {/if}
</div>
```

## Your First Mutation

```svelte
<script>
  import { createMutation, useQueryClient } from '@tanstack/svelte-query'

  const queryClient = useQueryClient()

  const mutation = createMutation({
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

  const addTodo = () => {
    $mutation.mutate({
      title: 'New Todo',
      completed: false,
    })
  }
</script>

<div>
  <button on:click={addTodo}>Add Todo</button>
  {#if $mutation.isPending}
    <div>Adding todo...</div>
  {/if}
  {#if $mutation.isError}
    <div>Error: {$mutation.error.message}</div>
  {/if}
  {#if $mutation.isSuccess}
    <div>Todo added!</div>
  {/if}
</div>
```

## Add DevTools (Optional)

Install and add the Svelte Query DevTools for debugging:

```bash
npm install @tanstack/svelte-query-devtools
```

```svelte
<script>
  import { SvelteQueryDevtools } from '@tanstack/svelte-query-devtools'
</script>

<YourApp />
<SvelteQueryDevtools />
```

## Complete Example

Here's a complete working example:

```svelte
<script>
  import { QueryClient, QueryClientProvider } from '@tanstack/svelte-query'
  import { SvelteQueryDevtools } from '@tanstack/svelte-query-devtools'
  import TodoList from './TodoList.svelte'

  const queryClient = new QueryClient()
</script>

<QueryClientProvider client={queryClient}>
  <TodoList />
  <SvelteQueryDevtools />
</QueryClientProvider>
```

TodoList.svelte:

```svelte
<script>
  import { createQuery, createMutation, useQueryClient } from '@tanstack/svelte-query'

  const queryClient = useQueryClient()

  // Fetch todos
  const todosQuery = createQuery({
    queryKey: ['todos'],
    queryFn: async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/todos?_limit=5')
      if (!response.ok) throw new Error('Failed to fetch')
      return response.json()
    }
  })

  // Add todo mutation
  const addTodoMutation = createMutation({
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
  const toggleTodoMutation = createMutation({
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

  const addTodo = () => {
    $addTodoMutation.mutate({
      title: `New Todo ${Date.now()}`,
      completed: false,
    })
  }

  const toggleTodo = (todo) => {
    $toggleTodoMutation.mutate({
      id: todo.id,
      completed: !todo.completed,
    })
  }
</script>

<div style="padding: 20px">
  <h1>Todo List</h1>
  
  <button
    on:click={addTodo}
    disabled={$addTodoMutation.isPending}
  >
    {$addTodoMutation.isPending ? 'Adding...' : 'Add Todo'}
  </button>

  {#if $todosQuery.isLoading}
    <div>Loading...</div>
  {:else if $todosQuery.error}
    <div>Error: {$todosQuery.error.message}</div>
  {:else}
    <ul style="margin-top: 20px">
      {#each $todosQuery.data as todo}
        <li style="margin-bottom: 10px">
          <input
            type="checkbox"
            checked={todo.completed}
            on:change={() => toggleTodo(todo)}
          />
          <span style="margin-left: 10px">{todo.title}</span>
        </li>
      {/each}
    </ul>
  {/if}
</div>
```

## Using with SvelteKit

For SvelteKit, you can set up the QueryClient in a layout:

```svelte
<!-- +layout.svelte -->
<script>
  import { QueryClient, QueryClientProvider } from '@tanstack/svelte-query'
  import { browser } from '$app/environment'

  // Create a client
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        enabled: browser,
        staleTime: 60 * 1000,
      },
    },
  })
</script>

<QueryClientProvider client={queryClient}>
  <slot />
</QueryClientProvider>
```

## Reactive Query Keys

Svelte Query automatically tracks changes to reactive variables:

```svelte
<script>
  import { createQuery } from '@tanstack/svelte-query'

  let userId = 1

  const userQuery = createQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  })

  // When userId changes, the query automatically refetches
</script>

<input type="number" bind:value={userId} />

{#if $userQuery.data}
  <div>{$userQuery.data.name}</div>
{/if}
```

## Working with Stores

Svelte Query returns Svelte stores, so you can use them with the $ prefix:

```svelte
<script>
  import { createQuery } from '@tanstack/svelte-query'

  const query = createQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  })

  // Access as store
  $: console.log($query.data)

  // Or subscribe manually
  query.subscribe(result => {
    console.log(result.data)
  })
</script>
```

## Next Steps

- Learn about [SSR with SvelteKit](../../framework/svelte/ssr.md)
- Explore [Query Keys](../tutorials/getting-started.md)
- Check out [Mutations](../tutorials/mutations-tutorial.md)
- View [Examples](../../framework/svelte/examples/simple.md)

## Additional Resources

- [Svelte Query Reference](../../framework/svelte/reference/index.md)
- [createQuery Documentation](../../framework/svelte/reference/functions/createquery.md)
- [createMutation Documentation](../../framework/svelte/reference/functions/createmutation.md)
