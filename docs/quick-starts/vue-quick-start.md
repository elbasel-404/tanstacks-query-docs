---
id: vue-quick-start
title: Vue Quick Start
---

# Vue Quick Start

Get up and running with TanStack Query in Vue in minutes.

## Installation

```bash
npm install @tanstack/vue-query
# or
yarn add @tanstack/vue-query
# or
pnpm add @tanstack/vue-query
```

## Setup (Vue 3)

Create and provide a QueryClient to your application:

```vue
<script setup>
import { VueQueryPlugin } from '@tanstack/vue-query'
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

app.use(VueQueryPlugin)
app.mount('#app')
</script>
```

Or with options:

```typescript
import { VueQueryPlugin, QueryClient } from '@tanstack/vue-query'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,
    },
  },
})

app.use(VueQueryPlugin, {
  queryClient,
})
```

## Your First Query

```vue
<script setup>
import { useQuery } from '@tanstack/vue-query'

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
</script>

<template>
  <div>
    <div v-if="isLoading">Loading...</div>
    <div v-else-if="error">Error: {{ error.message }}</div>
    <ul v-else>
      <li v-for="todo in data" :key="todo.id">
        {{ todo.completed ? '✅' : '⬜' }} {{ todo.title }}
      </li>
    </ul>
  </div>
</template>
```

## Your First Mutation

```vue
<script setup>
import { useMutation, useQueryClient } from '@tanstack/vue-query'

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

const addTodo = () => {
  mutation.mutate({
    title: 'New Todo',
    completed: false,
  })
}
</script>

<template>
  <div>
    <button @click="addTodo">Add Todo</button>
    <div v-if="mutation.isPending.value">Adding todo...</div>
    <div v-if="mutation.isError.value">Error: {{ mutation.error.value.message }}</div>
    <div v-if="mutation.isSuccess.value">Todo added!</div>
  </div>
</template>
```

## Add DevTools (Optional)

Install and add the Vue Query DevTools for debugging:

```bash
npm install @tanstack/vue-query-devtools
```

```vue
<script setup>
import { VueQueryDevtools } from '@tanstack/vue-query-devtools'
</script>

<template>
  <div>
    <YourApp />
    <VueQueryDevtools />
  </div>
</template>
```

## Complete Example

Here's a complete working example:

```vue
<script setup>
import { useQuery, useMutation, useQueryClient } from '@tanstack/vue-query'
import { VueQueryDevtools } from '@tanstack/vue-query-devtools'

const queryClient = useQueryClient()

// Fetch todos
const { data: todos, isLoading, error } = useQuery({
  queryKey: ['todos'],
  queryFn: async () => {
    const response = await fetch('https://jsonplaceholder.typicode.com/todos?_limit=5')
    if (!response.ok) throw new Error('Failed to fetch')
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
</script>

<template>
  <div style="padding: 20px">
    <h1>Todo List</h1>
    
    <button
      @click="addTodo"
      :disabled="addTodoMutation.isPending.value"
    >
      {{ addTodoMutation.isPending.value ? 'Adding...' : 'Add Todo' }}
    </button>

    <div v-if="isLoading">Loading...</div>
    <div v-else-if="error">Error: {{ error.message }}</div>
    <ul v-else style="margin-top: 20px">
      <li
        v-for="todo in todos"
        :key="todo.id"
        style="margin-bottom: 10px"
      >
        <input
          type="checkbox"
          :checked="todo.completed"
          @change="toggleTodo(todo)"
        />
        <span style="margin-left: 10px">{{ todo.title }}</span>
      </li>
    </ul>

    <VueQueryDevtools />
  </div>
</template>
```

## Composition API

TanStack Vue Query is built for the Composition API. Here are some tips:

```vue
<script setup>
import { computed, ref } from 'vue'
import { useQuery } from '@tanstack/vue-query'

const userId = ref(1)

// Reactive query key
const { data } = useQuery({
  queryKey: computed(() => ['user', userId.value]),
  queryFn: () => fetchUser(userId.value),
})

// You can also use reactive function
const { data: posts } = useQuery({
  queryKey: () => ['posts', userId.value],
  queryFn: () => fetchUserPosts(userId.value),
})
</script>
```

## Using with Nuxt

For Nuxt 3, you can create a plugin:

```typescript
// plugins/vue-query.ts
import { VueQueryPlugin, QueryClient } from '@tanstack/vue-query'

export default defineNuxtPlugin((nuxt) => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 5 * 60 * 1000,
      },
    },
  })

  nuxt.vueApp.use(VueQueryPlugin, { queryClient })
})
```

## Next Steps

- Learn about [Reactivity](../../framework/vue/reactivity.md)
- Explore [Mutations](../../framework/vue/guides/mutations.md)
- Check out [SSR with Nuxt](../../framework/vue/guides/ssr.md)
- View [Examples](../../framework/vue/examples/basic.md)

## Additional Resources

- [TypeScript Guide](../../framework/vue/typescript.md)
- [GraphQL Support](../../framework/vue/graphql.md)
- [Custom Query Client](../../framework/vue/guides/custom-client.md)
