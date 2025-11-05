---
id: mutations-tutorial
title: Working with Mutations
---

# Working with Mutations

In this tutorial, you'll learn how to use mutations to create, update, and delete data with TanStack Query.

## What are Mutations?

While queries are used to fetch data, mutations are used to modify data on the server. Common use cases include:

- Creating new records
- Updating existing records
- Deleting records
- Any other side-effect operations

## Basic Mutation

### React Example

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query'

function CreateUser() {
  const queryClient = useQueryClient()

  const mutation = useMutation({
    mutationFn: async (newUser) => {
      const response = await fetch('https://api.example.com/users', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(newUser),
      })
      return response.json()
    },
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['users'] })
    },
  })

  return (
    <button
      onClick={() => {
        mutation.mutate({
          name: 'John Doe',
          email: 'john@example.com',
        })
      }}
    >
      Create User
    </button>
  )
}
```

## Mutation States

Mutations provide several states to track the operation:

```tsx
const mutation = useMutation({ mutationFn: updateUser })

// States available
mutation.isIdle      // Mutation hasn't been called yet
mutation.isPending   // Mutation is currently running
mutation.isError     // Mutation encountered an error
mutation.isSuccess   // Mutation completed successfully
mutation.data        // Data returned from successful mutation
mutation.error       // Error object if mutation failed
```

## Handling Loading and Error States

```tsx
function UpdateUser({ userId }) {
  const mutation = useMutation({
    mutationFn: async (updatedData) => {
      const response = await fetch(`https://api.example.com/users/${userId}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(updatedData),
      })
      if (!response.ok) {
        throw new Error('Update failed')
      }
      return response.json()
    },
  })

  if (mutation.isPending) {
    return <div>Updating...</div>
  }

  if (mutation.isError) {
    return <div>Error: {mutation.error.message}</div>
  }

  if (mutation.isSuccess) {
    return <div>User updated successfully!</div>
  }

  return (
    <button onClick={() => mutation.mutate({ name: 'Updated Name' })}>
      Update User
    </button>
  )
}
```

## Optimistic Updates

Optimistic updates improve user experience by updating the UI immediately, before the server responds:

```tsx
const mutation = useMutation({
  mutationFn: updateTodo,
  onMutate: async (newTodo) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['todos'] })

    // Snapshot the previous value
    const previousTodos = queryClient.getQueryData(['todos'])

    // Optimistically update to the new value
    queryClient.setQueryData(['todos'], (old) => [...old, newTodo])

    // Return context with the snapshotted value
    return { previousTodos }
  },
  onError: (err, newTodo, context) => {
    // Rollback on error
    queryClient.setQueryData(['todos'], context.previousTodos)
  },
  onSettled: () => {
    // Always refetch after error or success
    queryClient.invalidateQueries({ queryKey: ['todos'] })
  },
})
```

## Mutation Callbacks

Mutations support several lifecycle callbacks:

```tsx
const mutation = useMutation({
  mutationFn: createUser,
  onMutate: (variables) => {
    // Called before mutation function
    console.log('Creating user...', variables)
  },
  onSuccess: (data, variables, context) => {
    // Called on success
    console.log('User created!', data)
  },
  onError: (error, variables, context) => {
    // Called on error
    console.error('Failed to create user:', error)
  },
  onSettled: (data, error, variables, context) => {
    // Called after success or error
    console.log('Mutation completed')
  },
})
```

## Resetting Mutations

You can reset a mutation to its initial state:

```tsx
const mutation = useMutation({ mutationFn: updateUser })

// Reset the mutation
mutation.reset()
```

## Best Practices

1. **Always invalidate queries** after successful mutations to keep data fresh
2. **Use optimistic updates** for better UX in fast-paced applications
3. **Handle errors gracefully** and provide meaningful feedback to users
4. **Consider retry logic** for network-related errors

## Next Steps

- Learn about [Query Invalidation](./query-invalidation-tutorial.md)
- Explore [Advanced Patterns](./advanced-patterns.md)
- Check out [Optimistic Updates Guide](../framework/react/guides/optimistic-updates.md)
