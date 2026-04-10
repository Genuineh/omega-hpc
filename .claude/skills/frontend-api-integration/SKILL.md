---
name: frontend-api-integration
description: Frontend API Integration Guide. Covers type-safe requests, error handling, and request management.
version: 1.0.0
---

# API Integration Guide

Guide for frontend API integration with type safety.

---

## 1. Type-Safe Requests

### Using Zod for Validation

```typescript
import { z } from 'zod';

// Define response schema
const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
  createdAt: z.string().datetime(),
});

type User = z.infer<typeof UserSchema>;

// API response wrapper
const ApiResponseSchema = z.object({
  data: UserSchema,
  meta: z.object({
    requestId: z.string(),
  }),
});

async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  const json = await response.json();

  // Validate response
  const result = ApiResponseSchema.safeParse(json);

  if (!result.success) {
    throw new Error('Invalid response format');
  }

  return result.data.data;
}
```

### Using ts-rest

```typescript
import { initClient, httpBatchLink } from '@ts-rest/core';

const client = initClient(apiContract, {
  baseUrl: 'https://api.example.com',
  links: [httpBatchLink()],
});

// Type-safe calls
const { body, status } = await client.getUser({ params: { id: '123' } });
// body is fully typed
```

---

## 2. Error Handling

### Global Error Handler

```typescript
// lib/api.ts
class ApiError extends Error {
  constructor(
    message: string,
    public code: string,
    public status: number,
    public details?: unknown
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

async function fetchApi<T>(url: string, options?: RequestInit): Promise<T> {
  try {
    const response = await fetch(url, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...options?.headers,
      },
    });

    if (!response.ok) {
      const error = await response.json().catch(() => ({ message: 'Unknown error' }));
      throw new ApiError(
        error.message,
        error.code || 'UNKNOWN',
        response.status,
        error.details
      );
    }

    return response.json();
  } catch (error) {
    if (error instanceof ApiError) throw error;
    throw new ApiError('Network error', 'NETWORK_ERROR', 0);
  }
}
```

### Error Boundary Component

```typescript
// components/ErrorBoundary.tsx
'use client';

import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <DefaultError error={this.state.error} />;
    }
    return this.props.children;
  }
}
```

---

## 3. React Query (TanStack Query)

### Basic Setup

```typescript
// lib/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      retry: 3,
      refetchOnWindowFocus: false,
    },
  },
});
```

### Using React Query

```typescript
'use client';

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function UserList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetchUsers(),
  });

  if (isLoading) return <Skeleton />;
  if (error) return <Error error={error} />;

  return <List items={data} />;
}

function CreateUserButton() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: createUser,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });

  return (
    <button onClick={() => mutation.mutate({ name: 'New User' })}>
      {mutation.isPending ? 'Creating...' : 'Create'}
    </button>
  );
}
```

---

## 4. Request Management

### Prefetching

```typescript
// Prefetch on hover
function UserButton({ userId }: { userId: string }) {
  const queryClient = useQueryClient();

  const handleHover = () => {
    queryClient.prefetchQuery({
      queryKey: ['user', userId],
      queryFn: () => fetchUser(userId),
    });
  };

  return <button onMouseEnter={handleHover}>View User</button>;
}
```

### Optimistic Updates

```typescript
const mutation = useMutation({
  mutationFn: updateTodo,
  onMutate: async (newTodo) => {
    await queryClient.cancelQueries({ queryKey: ['todos'] });

    const previousTodos = queryClient.getQueryData(['todos']);

    queryClient.setQueryData(['todos'], (old: Todo[]) =>
      old.map((todo) => (todo.id === newTodo.id ? newTodo : todo))
    );

    return { previousTodos };
  },
  onError: (err, newTodo, context) => {
    queryClient.setQueryData(['todos'], context?.previousTodos);
  },
});
```

---

## Checklist

- [ ] Type-safe API client
- [ ] Global error handling
- [ ] React Query configured
- [ ] Optimistic updates for mutations
- [ ] Loading states handled
- [ ] Retry logic configured
