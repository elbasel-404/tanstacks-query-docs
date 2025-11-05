---
id: getting-started
title: Getting Started with TanStack Query
---

# Getting Started with TanStack Query

Welcome to TanStack Query! This tutorial will guide you through the basics of setting up and using TanStack Query in your application.

## What is TanStack Query?

TanStack Query is a powerful data-fetching and state management library that makes working with server state simple and efficient. It handles caching, synchronization, and updates automatically, so you can focus on building great user experiences.

## Prerequisites

Before you begin, make sure you have:

- Node.js installed (version 14 or higher)
- Basic knowledge of React, Vue, Svelte, Angular, or Solid (depending on your framework of choice)
- A text editor or IDE

## Installation

First, install TanStack Query for your framework:

### React
```bash
npm install @tanstack/react-query
```

### Vue
```bash
npm install @tanstack/vue-query
```

### Svelte
```bash
npm install @tanstack/svelte-query
```

### Solid
```bash
npm install @tanstack/solid-query
```

### Angular
```bash
npm install @tanstack/angular-query-experimental
```

## Basic Setup

### React Example

1. **Create a QueryClient**

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

const queryClient = new QueryClient()

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
    </QueryClientProvider>
  )
}
```

2. **Fetch Data with useQuery**

```tsx
import { useQuery } from '@tanstack/react-query'

function Users() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const response = await fetch('https://api.example.com/users')
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
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

## Core Concepts

### Query Keys

Query keys uniquely identify your queries. They can be simple strings or arrays:

```tsx
// Simple key
queryKey: ['users']

// Key with parameters
queryKey: ['user', userId]

// Complex key
queryKey: ['users', { status: 'active', page: 1 }]
```

### Query Functions

Query functions are async functions that return your data:

```tsx
queryFn: async () => {
  const response = await fetch('https://api.example.com/data')
  return response.json()
}
```

### Automatic Caching

TanStack Query automatically caches your data. When you use the same query key again, it returns the cached data immediately and refetches in the background.

## Next Steps

Now that you understand the basics:

1. Explore [mutations](./mutations-tutorial.md) for updating server data
2. Learn about [query invalidation](./query-invalidation-tutorial.md)
3. Dive into [advanced patterns](./advanced-patterns.md)

## Additional Resources

- [Official Documentation](../framework/react/overview.md)
- [API Reference](../framework/react/reference/useQuery.md)
- [Examples](../framework/react/examples/simple)
