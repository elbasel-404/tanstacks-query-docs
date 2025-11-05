---
id: advanced-patterns
title: Advanced Patterns
---

# Advanced Patterns

Master advanced TanStack Query patterns to build robust and performant applications.

## Dependent Queries

Execute queries that depend on data from previous queries:

```tsx
// Get user first
const { data: user } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
})

// Then fetch user's projects
const { data: projects } = useQuery({
  queryKey: ['projects', user?.id],
  queryFn: () => fetchUserProjects(user.id),
  enabled: !!user?.id, // Only run when user.id exists
})
```

## Parallel Queries

Fetch multiple queries in parallel efficiently:

```tsx
function Dashboard() {
  const userQuery = useQuery({
    queryKey: ['user'],
    queryFn: fetchUser,
  })

  const projectsQuery = useQuery({
    queryKey: ['projects'],
    queryFn: fetchProjects,
  })

  const tasksQuery = useQuery({
    queryKey: ['tasks'],
    queryFn: fetchTasks,
  })

  // All queries run in parallel
  if (userQuery.isLoading || projectsQuery.isLoading || tasksQuery.isLoading) {
    return <div>Loading...</div>
  }

  return (
    <div>
      <UserInfo user={userQuery.data} />
      <ProjectsList projects={projectsQuery.data} />
      <TasksList tasks={tasksQuery.data} />
    </div>
  )
}
```

## useQueries for Dynamic Parallel Queries

When you need to fetch a dynamic number of queries:

```tsx
function UserProjects({ userIds }) {
  const queries = useQueries({
    queries: userIds.map((id) => ({
      queryKey: ['user', id],
      queryFn: () => fetchUser(id),
    })),
  })

  const isLoading = queries.some(query => query.isLoading)
  const allData = queries.map(query => query.data)

  return isLoading ? <div>Loading...</div> : <UserList users={allData} />
}
```

## Infinite Queries

Implement infinite scroll or load more patterns:

```tsx
import { useInfiniteQuery } from '@tanstack/react-query'

function Posts() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
  } = useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: ({ pageParam = 1 }) => fetchPosts(pageParam),
    getNextPageParam: (lastPage, pages) => {
      return lastPage.hasMore ? pages.length + 1 : undefined
    },
    initialPageParam: 1,
  })

  return (
    <div>
      {data?.pages.map((page, i) => (
        <div key={i}>
          {page.posts.map((post) => (
            <Post key={post.id} post={post} />
          ))}
        </div>
      ))}
      
      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage
          ? 'Loading more...'
          : hasNextPage
          ? 'Load More'
          : 'Nothing more to load'}
      </button>
    </div>
  )
}
```

## Paginated Queries

Handle traditional pagination:

```tsx
function PaginatedPosts() {
  const [page, setPage] = useState(1)

  const { data, isLoading, isPlaceholderData } = useQuery({
    queryKey: ['posts', page],
    queryFn: () => fetchPosts(page),
    placeholderData: keepPreviousData, // Keep previous data while fetching
  })

  return (
    <div>
      {isLoading ? (
        <div>Loading...</div>
      ) : (
        <div>
          {data.posts.map(post => (
            <Post key={post.id} post={post} />
          ))}
        </div>
      )}

      <div>
        <button
          onClick={() => setPage(old => Math.max(old - 1, 1))}
          disabled={page === 1}
        >
          Previous
        </button>
        <span>Page {page}</span>
        <button
          onClick={() => setPage(old => old + 1)}
          disabled={isPlaceholderData || !data?.hasMore}
        >
          Next
        </button>
      </div>
    </div>
  )
}
```

## Prefetching

Prefetch data before it's needed for faster navigation:

```tsx
import { useQueryClient } from '@tanstack/react-query'

function PostList({ posts }) {
  const queryClient = useQueryClient()

  return (
    <div>
      {posts.map((post) => (
        <div
          key={post.id}
          onMouseEnter={() => {
            // Prefetch post details on hover
            queryClient.prefetchQuery({
              queryKey: ['post', post.id],
              queryFn: () => fetchPost(post.id),
            })
          }}
        >
          <Link to={`/posts/${post.id}`}>{post.title}</Link>
        </div>
      ))}
    </div>
  )
}
```

## Placeholder Data

Show placeholder data while loading:

```tsx
const { data } = useQuery({
  queryKey: ['post', postId],
  queryFn: () => fetchPost(postId),
  placeholderData: () => {
    // Use data from posts list as placeholder
    const posts = queryClient.getQueryData(['posts'])
    return posts?.find(p => p.id === postId)
  },
})
```

## Initial Data

Provide initial data from another source:

```tsx
const { data } = useQuery({
  queryKey: ['post', postId],
  queryFn: () => fetchPost(postId),
  initialData: () => {
    // Get initial data from posts list cache
    const posts = queryClient.getQueryData(['posts'])
    return posts?.find(p => p.id === postId)
  },
  initialDataUpdatedAt: () => {
    // Return when the initial data was fetched
    return queryClient.getQueryState(['posts'])?.dataUpdatedAt
  },
})
```

## Query Cancellation

Cancel queries when they're no longer needed:

```tsx
const { data } = useQuery({
  queryKey: ['posts', searchTerm],
  queryFn: async ({ signal }) => {
    const response = await fetch(`/api/posts?q=${searchTerm}`, {
      signal, // Pass abort signal to fetch
    })
    return response.json()
  },
})
```

## Retry Logic

Customize retry behavior:

```tsx
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  retry: 3, // Retry failed requests 3 times
  retryDelay: attemptIndex => Math.min(1000 * 2 ** attemptIndex, 30000),
})

// Or conditional retry
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  retry: (failureCount, error) => {
    // Don't retry on 404
    if (error.status === 404) return false
    // Retry up to 3 times for other errors
    return failureCount < 3
  },
})
```

## Suspense Mode

Use React Suspense for cleaner loading states:

```tsx
import { useSuspenseQuery } from '@tanstack/react-query'

function Posts() {
  // This will suspend the component until data is ready
  const { data } = useSuspenseQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  })

  // No need to check isLoading
  return (
    <div>
      {data.map(post => (
        <Post key={post.id} post={post} />
      ))}
    </div>
  )
}

// Wrap with Suspense boundary
function App() {
  return (
    <Suspense fallback={<div>Loading posts...</div>}>
      <Posts />
    </Suspense>
  )
}
```

## Query Options Helper

Create reusable query configurations:

```tsx
import { queryOptions } from '@tanstack/react-query'

const postOptions = (postId) => queryOptions({
  queryKey: ['post', postId],
  queryFn: () => fetchPost(postId),
  staleTime: 5 * 60 * 1000,
})

// Use in components
function Post({ postId }) {
  const { data } = useQuery(postOptions(postId))
  return <div>{data.title}</div>
}

// Also works with prefetching
function prefetchPost(postId) {
  queryClient.prefetchQuery(postOptions(postId))
}
```

## Best Practices

1. **Use query options helpers** for reusable configurations
2. **Enable queries conditionally** when dependent on other data
3. **Implement proper error boundaries** for better error handling
4. **Prefetch on hover** for better perceived performance
5. **Use placeholderData** to reduce loading states
6. **Implement proper retry logic** based on error types
7. **Cancel queries** when components unmount or search terms change

## Next Steps

- Review [Performance Optimization](../framework/react/guides/render-optimizations.md)
- Explore [Server-Side Rendering](../framework/react/guides/ssr.md)
- Check out [Testing Strategies](../framework/react/guides/testing.md)
