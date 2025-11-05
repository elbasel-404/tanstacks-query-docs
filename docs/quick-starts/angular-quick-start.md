---
id: angular-quick-start
title: Angular Quick Start
---

# Angular Quick Start

Get up and running with TanStack Query in Angular in minutes.

## Installation

```bash
npm install @tanstack/angular-query-experimental
# or
yarn add @tanstack/angular-query-experimental
# or
pnpm add @tanstack/angular-query-experimental
```

## Setup

Provide the QueryClient in your app config:

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core'
import {
  provideAngularQuery,
  QueryClient,
} from '@tanstack/angular-query-experimental'

export const appConfig: ApplicationConfig = {
  providers: [
    provideAngularQuery(new QueryClient()),
  ],
}
```

Or in your AppComponent:

```typescript
import { Component } from '@angular/core'
import { QueryClient } from '@tanstack/angular-query-experimental'

@Component({
  selector: 'app-root',
  providers: [
    {
      provide: QueryClient,
      useValue: new QueryClient(),
    },
  ],
  template: `<router-outlet />`,
})
export class AppComponent {}
```

## Your First Query

```typescript
import { Component } from '@angular/core'
import { CommonModule } from '@angular/common'
import { injectQuery } from '@tanstack/angular-query-experimental'

interface Todo {
  id: number
  title: string
  completed: boolean
}

@Component({
  selector: 'app-todos',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div>
      <div *ngIf="query.isLoading()">Loading...</div>
      <div *ngIf="query.error()">Error: {{ query.error().message }}</div>
      <ul *ngIf="query.data()">
        <li *ngFor="let todo of query.data()">
          {{ todo.completed ? '✅' : '⬜' }} {{ todo.title }}
        </li>
      </ul>
    </div>
  `,
})
export class TodosComponent {
  query = injectQuery<Todo[]>(() => ({
    queryKey: ['todos'],
    queryFn: async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/todos')
      if (!response.ok) {
        throw new Error('Network response was not ok')
      }
      return response.json()
    },
  }))
}
```

## Your First Mutation

```typescript
import { Component } from '@angular/core'
import { CommonModule } from '@angular/common'
import {
  injectMutation,
  injectQueryClient,
} from '@tanstack/angular-query-experimental'

interface NewTodo {
  title: string
  completed: boolean
}

@Component({
  selector: 'app-add-todo',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div>
      <button
        (click)="addTodo()"
        [disabled]="mutation.isPending()"
      >
        Add Todo
      </button>
      <div *ngIf="mutation.isPending()">Adding todo...</div>
      <div *ngIf="mutation.isError()">
        Error: {{ mutation.error().message }}
      </div>
      <div *ngIf="mutation.isSuccess()">Todo added!</div>
    </div>
  `,
})
export class AddTodoComponent {
  queryClient = injectQueryClient()

  mutation = injectMutation<Todo, Error, NewTodo>(() => ({
    mutationFn: async (newTodo: NewTodo) => {
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
      this.queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  }))

  addTodo() {
    this.mutation.mutate({
      title: 'New Todo',
      completed: false,
    })
  }
}
```

## Add DevTools (Optional)

Install and add the Angular Query DevTools for debugging:

```bash
npm install @tanstack/angular-query-devtools-experimental
```

```typescript
import { Component } from '@angular/core'
import { AngularQueryDevtools } from '@tanstack/angular-query-devtools-experimental'

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [AngularQueryDevtools],
  template: `
    <router-outlet />
    <angular-query-devtools initialIsOpen />
  `,
})
export class AppComponent {}
```

## Complete Example

Here's a complete working example:

```typescript
import { Component } from '@angular/core'
import { CommonModule } from '@angular/common'
import {
  injectQuery,
  injectMutation,
  injectQueryClient,
} from '@tanstack/angular-query-experimental'
import { AngularQueryDevtools } from '@tanstack/angular-query-devtools-experimental'

interface Todo {
  id: number
  title: string
  completed: boolean
}

@Component({
  selector: 'app-todo-list',
  standalone: true,
  imports: [CommonModule, AngularQueryDevtools],
  template: `
    <div style="padding: 20px">
      <h1>Todo List</h1>
      
      <button
        (click)="addTodo()"
        [disabled]="addTodoMutation.isPending()"
      >
        {{ addTodoMutation.isPending() ? 'Adding...' : 'Add Todo' }}
      </button>

      <div *ngIf="todosQuery.isLoading()">Loading...</div>
      <div *ngIf="todosQuery.error()">
        Error: {{ todosQuery.error().message }}
      </div>
      <ul *ngIf="todosQuery.data()" style="margin-top: 20px">
        <li
          *ngFor="let todo of todosQuery.data()"
          style="margin-bottom: 10px"
        >
          <input
            type="checkbox"
            [checked]="todo.completed"
            (change)="toggleTodo(todo)"
          />
          <span style="margin-left: 10px">{{ todo.title }}</span>
        </li>
      </ul>

      <angular-query-devtools initialIsOpen />
    </div>
  `,
})
export class TodoListComponent {
  queryClient = injectQueryClient()

  // Fetch todos
  todosQuery = injectQuery<Todo[]>(() => ({
    queryKey: ['todos'],
    queryFn: async () => {
      const response = await fetch(
        'https://jsonplaceholder.typicode.com/todos?_limit=5'
      )
      if (!response.ok) throw new Error('Failed to fetch')
      return response.json()
    },
  }))

  // Add todo mutation
  addTodoMutation = injectMutation<Todo, Error, Partial<Todo>>(() => ({
    mutationFn: async (newTodo) => {
      const response = await fetch('https://jsonplaceholder.typicode.com/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newTodo),
      })
      return response.json()
    },
    onSuccess: () => {
      this.queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  }))

  // Toggle todo mutation
  toggleTodoMutation = injectMutation<
    Todo,
    Error,
    { id: number; completed: boolean }
  >(() => ({
    mutationFn: async ({ id, completed }) => {
      const response = await fetch(
        `https://jsonplaceholder.typicode.com/todos/${id}`,
        {
          method: 'PATCH',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ completed }),
        }
      )
      return response.json()
    },
    onSuccess: () => {
      this.queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  }))

  addTodo() {
    this.addTodoMutation.mutate({
      title: `New Todo ${Date.now()}`,
      completed: false,
    })
  }

  toggleTodo(todo: Todo) {
    this.toggleTodoMutation.mutate({
      id: todo.id,
      completed: !todo.completed,
    })
  }
}
```

## Using with Angular Signals

Angular Query integrates seamlessly with Angular signals:

```typescript
import { Component, signal } from '@angular/core'
import { injectQuery } from '@tanstack/angular-query-experimental'

@Component({
  selector: 'app-user',
  template: `
    <div>
      <input
        [value]="userId()"
        (input)="userId.set($event.target.value)"
      />
      <div *ngIf="userQuery.data()">
        {{ userQuery.data().name }}
      </div>
    </div>
  `,
})
export class UserComponent {
  userId = signal(1)

  userQuery = injectQuery(() => ({
    queryKey: ['user', this.userId()],
    queryFn: () => fetchUser(this.userId()),
  }))
}
```

## Using with Angular HttpClient

```typescript
import { Component, inject } from '@angular/core'
import { HttpClient } from '@angular/common/http'
import { injectQuery } from '@tanstack/angular-query-experimental'
import { lastValueFrom } from 'rxjs'

@Component({
  selector: 'app-posts',
  template: `
    <div *ngIf="query.data()">
      <div *ngFor="let post of query.data()">
        {{ post.title }}
      </div>
    </div>
  `,
})
export class PostsComponent {
  http = inject(HttpClient)

  query = injectQuery(() => ({
    queryKey: ['posts'],
    queryFn: () => lastValueFrom(
      this.http.get<Post[]>('https://api.example.com/posts')
    ),
  }))
}
```

## Next Steps

- Learn about [Angular HttpClient Integration](../../framework/angular/angular-httpclient-and-other-data-fetching-clients.md)
- Explore [Zoneless Mode](../../framework/angular/zoneless.md)
- Check out [Query Options](../../framework/angular/guides/query-options.md)
- View [Examples](../../framework/angular/examples/simple.md)

## Additional Resources

- [TypeScript Guide](../../framework/angular/typescript.md)
- [Testing Strategies](../../framework/angular/guides/testing.md)
- [Mutations Guide](../../framework/angular/guides/mutations.md)
