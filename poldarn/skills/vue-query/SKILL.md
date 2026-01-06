---
name: Vue Query Patterns
description: This skill should be used when implementing data fetching, API calls, server state management, or mutations in Vue 3 applications. Activates on requests involving "fetch data", "call API", "load data", "submit form", "create/update/delete", "server state", "cache data", "refetch", "polling". Provides guidance on when and how to use @tanstack/vue-query instead of manual data fetching.
version: 1.0.0
model: opus
---

# Vue Query (@tanstack/vue-query) Implementation Guide

## Core Principle: Server State Belongs in Vue Query

**Every interaction with server data should use Vue Query.** Manual `fetch`/`axios` calls with `ref` and `watch` are anti-patterns when Vue Query is available.

## Decision Matrix: When to Use What

### Use `useQuery` When:
| Scenario | Example |
|----------|---------|
| Fetching data on component mount | Load user profile, fetch list items |
| Data that should be cached | Product catalog, user preferences |
| Data shared across components | Current user, feature flags |
| Polling/auto-refresh needed | Live dashboard, notifications |
| Background refetch on focus | Stale data refresh |

### Use `useMutation` When:
| Scenario | Example |
|----------|---------|
| Creating new resources | POST new user, create order |
| Updating existing resources | PUT/PATCH profile, update settings |
| Deleting resources | DELETE item, remove from cart |
| Any non-idempotent operation | Submit form, trigger workflow |
| Operations needing optimistic updates | Like button, toggle favorite |

### NEVER Use Manual Fetching When:
- Vue Query is installed in the project
- The data comes from an API/server
- Multiple components might need the same data
- You need loading/error states
- You need to invalidate/refetch after mutations

## Pattern 1: Basic Query in Composable

**WRONG - Manual fetching:**
```typescript
export function useUsers() {
  const users = ref<User[]>([])
  const loading = ref(false)
  const error = ref<Error | null>(null)

  const fetchUsers = async () => {
    loading.value = true
    try {
      const response = await api.get('/users')
      users.value = response.data
    } catch (e) {
      error.value = e as Error
    } finally {
      loading.value = false
    }
  }

  onMounted(fetchUsers)

  return { users, loading, error, refetch: fetchUsers }
}
```

**CORRECT - Vue Query:**
```typescript
export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: () => api.get('/users').then(res => res.data),
  })
}
```

## Pattern 2: Conditional/Dependent Queries

**WRONG - Manual conditional fetching:**
```typescript
export function useUserPosts(userId: Ref<string | null>) {
  const posts = ref<Post[]>([])

  watch(userId, async (id) => {
    if (id) {
      const response = await api.get(`/users/${id}/posts`)
      posts.value = response.data
    }
  }, { immediate: true })

  return { posts }
}
```

**CORRECT - Vue Query with enabled:**
```typescript
export function useUserPosts(userId: Ref<string | null>) {
  return useQuery({
    queryKey: ['users', userId, 'posts'],
    queryFn: () => api.get(`/users/${userId.value}/posts`).then(res => res.data),
    enabled: computed(() => !!userId.value),
  })
}
```

## Pattern 3: Mutations for Data Modification

**WRONG - Manual mutation:**
```typescript
export function useCreateUser() {
  const loading = ref(false)
  const error = ref<Error | null>(null)

  const createUser = async (userData: CreateUserDto) => {
    loading.value = true
    try {
      const response = await api.post('/users', userData)
      return response.data
    } catch (e) {
      error.value = e as Error
      throw e
    } finally {
      loading.value = false
    }
  }

  return { createUser, loading, error }
}
```

**CORRECT - Vue Query useMutation:**
```typescript
export function useCreateUser() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (userData: CreateUserDto) =>
      api.post('/users', userData).then(res => res.data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] })
    },
  })
}
```

## Pattern 4: Form Submission

**WRONG - Manual form handling:**
```typescript
const handleSubmit = async () => {
  submitting.value = true
  try {
    await api.put(`/users/${userId}`, formData.value)
    router.push('/users')
  } catch (e) {
    errorMessage.value = 'Failed to update'
  } finally {
    submitting.value = false
  }
}
```

**CORRECT - Mutation with callbacks:**
```typescript
const { mutate: updateUser, isPending } = useMutation({
  mutationFn: (data: UpdateUserDto) =>
    api.put(`/users/${userId}`, data).then(res => res.data),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['users', userId] })
    router.push('/users')
  },
  onError: () => {
    errorMessage.value = 'Failed to update'
  },
})

const handleSubmit = () => updateUser(formData.value)
```

## Pattern 5: Optimistic Updates

```typescript
export function useToggleFavorite() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (itemId: string) => api.post(`/items/${itemId}/favorite`),
    onMutate: async (itemId) => {
      await queryClient.cancelQueries({ queryKey: ['items', itemId] })
      const previous = queryClient.getQueryData(['items', itemId])
      queryClient.setQueryData(['items', itemId], (old: Item) => ({
        ...old,
        isFavorite: !old.isFavorite,
      }))
      return { previous }
    },
    onError: (err, itemId, context) => {
      queryClient.setQueryData(['items', itemId], context?.previous)
    },
    onSettled: (data, error, itemId) => {
      queryClient.invalidateQueries({ queryKey: ['items', itemId] })
    },
  })
}
```

## Pattern 6: Composables Wrapping Queries

Always wrap queries in composables for reusability:

```typescript
// features/products/composables/useProduct.ts
export function useProduct(productId: Ref<string>) {
  return useQuery({
    queryKey: ['products', productId],
    queryFn: () => productApi.getById(productId.value),
    staleTime: 5 * 60 * 1000,
  })
}

// features/products/composables/useUpdateProduct.ts
export function useUpdateProduct() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateProductDto }) =>
      productApi.update(id, data),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: ['products', id] })
      queryClient.invalidateQueries({ queryKey: ['products'] })
    },
  })
}
```

## Pattern 7: Query with Select Transform

```typescript
export function useActiveUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: () => api.get('/users').then(res => res.data),
    select: (users) => users.filter(u => u.isActive),
  })
}
```

## Pattern 8: Infinite Queries for Pagination

```typescript
export function useInfiniteProducts() {
  return useInfiniteQuery({
    queryKey: ['products', 'infinite'],
    queryFn: ({ pageParam }) =>
      api.get('/products', { params: { cursor: pageParam } }).then(res => res.data),
    initialPageParam: undefined,
    getNextPageParam: (lastPage) => lastPage.nextCursor,
  })
}
```

## Anti-Pattern Detection Checklist

When reviewing code, flag these patterns for Vue Query refactoring:

| Anti-Pattern | Replace With |
|--------------|--------------|
| `ref` + `onMounted` + `fetch` | `useQuery` |
| `watch` + `fetch` for dependent data | `useQuery` with `enabled` |
| `ref(loading)` + `try/catch` for POST | `useMutation` |
| Manual cache in `ref` or Pinia for server data | Query cache |
| `setInterval` for polling | `refetchInterval` option |
| Re-fetching in `onMounted` across components | Query deduplication |

## Query Key Best Practices

```typescript
// Hierarchical keys for invalidation control
['users']                           // All users
['users', 'list']                   // User list
['users', 'list', { status: 'active' }]  // Filtered list
['users', userId]                   // Single user
['users', userId, 'posts']          // User's posts

// Invalidation examples
queryClient.invalidateQueries({ queryKey: ['users'] })  // All user-related
queryClient.invalidateQueries({ queryKey: ['users', userId] })  // Specific user
```

## Configuration Recommendations

```typescript
// main.ts or app setup
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60,        // 1 minute
      gcTime: 1000 * 60 * 5,       // 5 minutes (formerly cacheTime)
      retry: 1,
      refetchOnWindowFocus: false, // Disable if annoying in dev
    },
  },
})
```

## Quick Reference

| Need | Use |
|------|-----|
| Fetch data | `useQuery` |
| Fetch with parameter | `useQuery` with reactive key |
| Fetch when condition met | `useQuery` with `enabled: computed(() => condition)` |
| Create/Update/Delete | `useMutation` |
| Refresh after mutation | `queryClient.invalidateQueries` in `onSuccess` |
| Paginated/infinite list | `useInfiniteQuery` |
| Polling | `refetchInterval` option |
| Transform response | `select` option |
| Prefetch for navigation | `queryClient.prefetchQuery` |
