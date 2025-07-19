# Angular 19 Development Guide

## Introduction

Angular 19, released in 2024, builds upon the foundation established by Angular 17 and 18, introducing significant improvements in developer experience, performance, and feature set. This guide provides comprehensive information about the evolution from Angular 17 to Angular 19, highlighting key changes, new features, and best practices to help you effectively generate modern web applications.

## Evolution from Angular 17 to Angular 19

### Angular 18 Key Improvements (May 2024)

Angular 18 introduced several important enhancements:

1. **Improved Reactivity System**: Enhanced signals API with additional operators and better integration with RxJS
2. **Server-Side Rendering (SSR) Optimizations**: Reduced hydration mismatches and improved performance
3. **Angular CLI Enhancements**: Faster build times and improved developer experience
4. **Zone.js Optional**: Further progress toward making Zone.js optional for change detection
5. **Improved Standalone API**: Refinements to the standalone component architecture

### Angular 19 Key Improvements (November 2024)

Angular 19 builds on these changes with:

1. **Fully Matured Signals API**: Complete reactivity system with comprehensive tooling
2. **Zone.js-less Applications**: Full support for applications without Zone.js
3. **Enhanced Build Performance**: Significant improvements in build and compilation speed
4. **Advanced Server-Side Rendering**: New rendering strategies and hydration improvements
5. **Improved Developer Tooling**: Enhanced debugging and performance profiling

## Key Features and Changes from Angular 17 to Angular 19

### 1. Reactivity System Evolution

#### Angular 17 (Initial Signals)

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <h2>Counter: {{ count() }}</h2>
    <p>Doubled: {{ doubled() }}</p>
    <button (click)="increment()">Increment</button>
  `
})
export class CounterComponent {
  count = signal(0);
  doubled = computed(() => this.count() * 2);

  increment() {
    this.count.update(value => value + 1);
  }
}
```

#### Angular 19 (Enhanced Signals)

```typescript
import { Component, signal, computed, effect, untracked } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <h2>Counter: {{ count() }}</h2>
    <p>Doubled: {{ doubled() }}</p>
    <p>Last updated: {{ lastUpdated() }}</p>
    <button (click)="increment()">Increment</button>
  `
})
export class CounterComponent {
  count = signal(0);
  doubled = computed(() => this.count() * 2);
  lastUpdated = signal(new Date().toLocaleTimeString());

  constructor() {
    // New effect API with improved cleanup
    effect((onCleanup) => {
      console.log(`Count changed to: ${this.count()}`);

      // Access a signal without creating a dependency
      const currentTime = untracked(() => new Date().toLocaleTimeString());
      this.lastUpdated.set(currentTime);

      // Cleanup function
      onCleanup(() => {
        console.log('Cleaning up previous effect');
      });
    });
  }

  increment() {
    this.count.update(value => value + 1);
  }
}
```

### 2. Zone.js-less Applications

#### Angular 17 (Zone.js Required)

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch(err => console.error(err));
```

#### Angular 19 (Zone.js Optional)

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

// Zone.js is now completely optional
bootstrapApplication(AppComponent, {
  ...appConfig,
  ngZoneEventCoalescing: true,
  ngZoneRunCoalescing: true
})
.catch(err => console.error(err));
```

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { withViewTransitions } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { provideClientHydration } from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withViewTransitions()
    ),
    provideHttpClient(),
    provideClientHydration()
  ]
};
```

### 3. Enhanced Control Flow

#### Angular 17 (Initial Control Flow)

```html
@if (condition) {
  <p>Content to render when condition is true</p>
} @else {
  <p>Content to render otherwise</p>
}

@for (item of items; track item.id) {
  <p>{{ item.name }}</p>
} @empty {
  <p>No items found</p>
}
```

#### Angular 19 (Enhanced Control Flow)

```html
@if (condition) {
  <p>Content to render when condition is true</p>
} @else {
  <p>Content to render otherwise</p>
}

@for (item of items(); track item.id) {
  @if (item.isSpecial) {
    <p class="special">{{ item.name }}</p>
  } @else {
    <p>{{ item.name }}</p>
  }
} @empty {
  <p>No items found</p>
}

<!-- New defer options -->
@defer (on viewport; prefetch: true) {
  <heavy-component />
} @loading (minimum: 500ms) {
  <p>Loading...</p>
} @error {
  <p>Error loading component</p>
} @placeholder (minimum: 0ms) {
  <p>Placeholder content</p>
}
```

### 4. Improved Server-Side Rendering

#### Angular 17 (Basic Hydration)

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideClientHydration } from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [
    provideClientHydration()
  ]
};
```

#### Angular 19 (Advanced Hydration Strategies)

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideClientHydration, withHttpTransferCache } from '@angular/platform-browser';
import { provideRouter, withComponentInputBinding, withViewTransitions } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideClientHydration(
      withHttpTransferCache(),
      withComponentInputBinding()
    ),
    provideRouter(
      routes,
      withViewTransitions()
    )
  ]
};
```

### 5. Enhanced Standalone API

#### Angular 17 (Basic Standalone)

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-feature',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './feature.component.html'
})
export class FeatureComponent {}
```

#### Angular 19 (Enhanced Standalone)

```typescript
import { Component, input, output, model } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-feature',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './feature.component.html'
})
export class FeatureComponent {
  // New input() syntax
  name = input<string>('Default Name');
  config = input.required<{type: string, enabled: boolean}>();

  // New output() syntax
  selected = output<string>();

  // Two-way binding with model()
  value = model<number>(0);

  // Methods
  selectItem(item: string) {
    this.selected.emit(item);
  }

  updateValue(newValue: number) {
    this.value.set(newValue);
  }
}
```

## Required Configuration Changes

### 1. Project Setup

#### Package.json Dependencies for Angular 19

```json
{
  "dependencies": {
    "@angular/animations": "^19.0.0",
    "@angular/common": "^19.0.0",
    "@angular/compiler": "^19.0.0",
    "@angular/core": "^19.0.0",
    "@angular/forms": "^19.0.0",
    "@angular/platform-browser": "^19.0.0",
    "@angular/platform-browser-dynamic": "^19.0.0",
    "@angular/router": "^19.0.0",
    "rxjs": "~7.8.0",
    "tslib": "^2.6.0",
    "zone.js": "~0.14.3"
  },
  "devDependencies": {
    "@angular-devkit/build-angular": "^19.0.0",
    "@angular/cli": "^19.0.0",
    "@angular/compiler-cli": "^19.0.0",
    "@types/jasmine": "~5.1.0",
    "jasmine-core": "~5.1.0",
    "karma": "~6.4.0",
    "karma-chrome-launcher": "~3.2.0",
    "karma-coverage": "~2.2.0",
    "karma-jasmine": "~5.1.0",
    "karma-jasmine-html-reporter": "~2.1.0",
    "typescript": "~5.3.0"
  }
}
```

### 2. Application Bootstrap

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch(err => console.error(err));
```

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter, withComponentInputBinding, withViewTransitions } from '@angular/router';
import { provideHttpClient, withFetch } from '@angular/common/http';
import { provideClientHydration, withHttpTransferCache } from '@angular/platform-browser';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withComponentInputBinding(),
      withViewTransitions()
    ),
    provideHttpClient(withFetch()),
    provideClientHydration(withHttpTransferCache()),
    provideAnimationsAsync()
  ]
};
```

## Optional Configuration Sets

### 1. State Management with NgRx

```typescript
// app.config.ts with NgRx 19
import { ApplicationConfig, isDevMode } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { provideStore } from '@ngrx/store';
import { provideEffects } from '@ngrx/effects';
import { provideStoreDevtools } from '@ngrx/store-devtools';
import { provideRouterStore, routerReducer } from '@ngrx/router-store';

import { routes } from './app.routes';
import { reducers, metaReducers } from './reducers';
import { UserEffects } from './effects/user.effects';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    provideStore(
      { router: routerReducer, ...reducers }, 
      { metaReducers }
    ),
    provideEffects([UserEffects]),
    provideRouterStore(),
    provideStoreDevtools({
      maxAge: 25,
      logOnly: !isDevMode(),
      autoPause: true,
      trace: false,
      traceLimit: 75,
    })
  ]
};
```

### 2. Advanced Signals for State Management

```typescript
// user.service.ts
import { Injectable, computed, signal } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { toSignal, toObservable } from '@angular/core/rxjs-interop';
import { catchError, map, switchMap } from 'rxjs/operators';
import { of } from 'rxjs';

export interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

@Injectable({
  providedIn: 'root'
})
export class UserService {
  // State
  private usersSignal = signal<User[]>([]);
  private selectedUserIdSignal = signal<number | null>(null);
  private loadingSignal = signal(false);
  private errorSignal = signal<string | null>(null);

  // Computed values
  readonly users = this.usersSignal.asReadonly();
  readonly selectedUserId = this.selectedUserIdSignal.asReadonly();
  readonly loading = this.loadingSignal.asReadonly();
  readonly error = this.errorSignal.asReadonly();

  readonly selectedUser = computed(() => {
    const userId = this.selectedUserIdSignal();
    if (userId === null) return null;
    return this.usersSignal().find(user => user.id === userId) || null;
  });

  readonly activeUsers = computed(() => 
    this.usersSignal().filter(user => user.isActive)
  );

  readonly totalUsers = computed(() => this.usersSignal().length);
  readonly activeUserCount = computed(() => this.activeUsers().length);

  // Convert signal to observable for use with RxJS
  readonly selectedUser$ = toObservable(this.selectedUser);

  constructor(private http: HttpClient) {}

  loadUsers() {
    this.loadingSignal.set(true);
    this.errorSignal.set(null);

    this.http.get<User[]>('/api/users')
      .pipe(
        catchError(err => {
          this.errorSignal.set(err.message || 'Failed to load users');
          return of([]);
        })
      )
      .subscribe(users => {
        this.usersSignal.set(users);
        this.loadingSignal.set(false);
      });
  }

  selectUser(userId: number) {
    this.selectedUserIdSignal.set(userId);
  }

  updateUser(userId: number, updates: Partial<User>) {
    this.loadingSignal.set(true);

    this.http.patch<User>(`/api/users/${userId}`, updates)
      .pipe(
        catchError(err => {
          this.errorSignal.set(err.message || 'Failed to update user');
          return of(null);
        })
      )
      .subscribe(updatedUser => {
        if (updatedUser) {
          this.usersSignal.update(users => 
            users.map(user => user.id === userId ? { ...user, ...updates } : user)
          );
        }
        this.loadingSignal.set(false);
      });
  }

  addUser(newUser: Omit<User, 'id'>) {
    this.loadingSignal.set(true);

    this.http.post<User>('/api/users', newUser)
      .pipe(
        catchError(err => {
          this.errorSignal.set(err.message || 'Failed to add user');
          return of(null);
        })
      )
      .subscribe(user => {
        if (user) {
          this.usersSignal.update(users => [...users, user]);
        }
        this.loadingSignal.set(false);
      });
  }

  deleteUser(userId: number) {
    this.loadingSignal.set(true);

    this.http.delete<void>(`/api/users/${userId}`)
      .pipe(
        catchError(err => {
          this.errorSignal.set(err.message || 'Failed to delete user');
          return of(null);
        })
      )
      .subscribe(() => {
        this.usersSignal.update(users => users.filter(user => user.id !== userId));
        if (this.selectedUserIdSignal() === userId) {
          this.selectedUserIdSignal.set(null);
        }
        this.loadingSignal.set(false);
      });
  }
}
```

### 3. Enhanced Server-Side Rendering and Static Site Generation

```typescript
// app.config.server.ts
import { mergeApplicationConfig, ApplicationConfig } from '@angular/core';
import { provideServerRendering } from '@angular/platform-server';
import { appConfig } from './app.config';

const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering()
  ]
};

export const config = mergeApplicationConfig(appConfig, serverConfig);
```

```typescript
// server.ts
import { APP_BASE_HREF } from '@angular/common';
import { CommonEngine } from '@angular/ssr';
import express from 'express';
import { fileURLToPath } from 'node:url';
import { dirname, join, resolve } from 'node:path';
import bootstrap from './src/main.server';

// The Express app is exported so that it can be used by serverless Functions.
export function app(): express.Express {
  const server = express();
  const serverDistFolder = dirname(fileURLToPath(import.meta.url));
  const browserDistFolder = resolve(serverDistFolder, '../browser');
  const indexHtml = join(serverDistFolder, 'index.server.html');

  const commonEngine = new CommonEngine();

  server.set('view engine', 'html');
  server.set('views', browserDistFolder);

  // Serve static files from /browser
  server.get('*.*', express.static(browserDistFolder, {
    maxAge: '1y'
  }));

  // All regular routes use the Angular engine
  server.get('*', (req, res, next) => {
    const { protocol, originalUrl, baseUrl, headers } = req;

    commonEngine
      .render({
        bootstrap,
        documentFilePath: indexHtml,
        url: `${protocol}://${headers.host}${originalUrl}`,
        publicPath: browserDistFolder,
        providers: [{ provide: APP_BASE_HREF, useValue: baseUrl }],
      })
      .then((html) => res.send(html))
      .catch((err) => next(err));
  });

  return server;
}

function run(): void {
  const port = process.env['PORT'] || 4000;

  // Start up the Node server
  const server = app();
  server.listen(port, () => {
    console.log(`Node Express server listening on http://localhost:${port}`);
  });
}

run();
```

## Migration Guide from Angular 17 to Angular 19

### 1. Update Angular CLI and Core Packages

```bash
# Update to Angular 18 first
ng update @angular/cli@18 @angular/core@18

# Then update to Angular 19
ng update @angular/cli@19 @angular/core@19
```

### 2. Adopt New Input/Output Syntax

```typescript
// Before (Angular 17)
@Component({
  selector: 'app-user-card',
  standalone: true,
  // ...
})
export class UserCardComponent {
  @Input() user!: User;
  @Input({ required: true }) role!: string;
  @Output() userSelected = new EventEmitter<User>();
}

// After (Angular 19)
@Component({
  selector: 'app-user-card',
  standalone: true,
  // ...
})
export class UserCardComponent {
  user = input<User>();
  role = input.required<string>();
  userSelected = output<User>();
}
```

### 3. Migrate to Enhanced Control Flow

```html
<!-- Before (Angular 17) -->
@for (item of items; track item.id) {
  <div>{{ item.name }}</div>
}

<!-- After (Angular 19) -->
@for (item of items(); track item.id) {
  <div>{{ item.name }}</div>
}
```

### 4. Adopt Signal-Based Reactivity

```typescript
// Before (Angular 17)
@Component({
  // ...
})
export class CounterComponent {
  count = signal(0);

  increment() {
    this.count.update(value => value + 1);
  }
}

// After (Angular 19)
@Component({
  // ...
})
export class CounterComponent {
  count = signal(0);

  constructor() {
    // Use new effect API
    effect(() => {
      console.log(`Count changed to: ${this.count()}`);
    });
  }

  increment() {
    this.count.update(value => value + 1);
  }
}
```

### 5. Update Router Configuration

```typescript
// Before (Angular 17)
export const appConfigAngular17: ApplicationConfig = {
  providers: [
    provideRouter(routes)
  ]
};

// After (Angular 19)
export const appConfigAngular19: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withComponentInputBinding(),
      withViewTransitions()
    )
  ]
};
```

## Best Practices for Angular 19 Applications

### 1. Embrace Signal-Based Architecture

```typescript
// user-list.component.ts
import { Component, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { UserService } from '../../services/user.service';
import { UserCardComponent } from '../user-card/user-card.component';

@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule, UserCardComponent],
  template: `
    <div class="user-list">
      <h2>Users ({{ userService.totalUsers() }})</h2>
      <p>Active Users: {{ userService.activeUserCount() }}</p>

      @if (userService.loading()) {
        <div class="loading">Loading users...</div>
      } @else if (userService.error()) {
        <div class="error">
          <p>{{ userService.error() }}</p>
          <button (click)="userService.loadUsers()">Try Again</button>
        </div>
      } @else {
        <div class="user-grid">
          @for (user of userService.users(); track user.id) {
            <app-user-card 
              [user]="user" 
              (userSelected)="userService.selectUser(user.id)">
            </app-user-card>
          } @empty {
            <p>No users found.</p>
          }
        </div>
      }

      @if (userService.selectedUser()) {
        <div class="selected-user">
          <h3>Selected User: {{ userService.selectedUser()?.name }}</h3>
          <button (click)="userService.selectUser(null)">Clear Selection</button>
        </div>
      }
    </div>
  `
})
export class UserListComponent {
  userService = inject(UserService);

  constructor() {
    this.userService.loadUsers();
  }
}
```

### 2. Optimize for Zone.js-less Applications

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

// Bootstrap without Zone.js
bootstrapApplication(AppComponent, appConfig)
  .catch(err => console.error(err));
```

```typescript
// counter.component.ts
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <h2>Counter: {{ count() }}</h2>
    <button (click)="increment()">Increment</button>
  `
})
export class CounterComponent {
  count = signal(0);

  // Manual change detection with signals
  increment() {
    this.count.update(value => value + 1);
    // No need to call detectChanges() or markForCheck()
  }
}
```

### 3. Leverage Advanced Defer Loading

```html
<!-- Defer with multiple triggers -->
@defer (
  on viewport;
  when isReady();
  prefetch on idle
) {
  <app-data-visualization [data]="chartData()" />
} @loading (minimum: 300ms) {
  <app-skeleton-loader type="chart" />
} @error {
  <app-error-display message="Failed to load visualization" />
} @placeholder {
  <div class="placeholder-box">Chart data will appear here</div>
}
```

### 4. Use Model for Two-Way Binding

```typescript
// slider.component.ts
import { Component, model } from '@angular/core';

@Component({
  selector: 'app-slider',
  standalone: true,
  template: `
    <div class="slider-container">
      <input 
        type="range" 
        min="0" 
        max="100" 
        [value]="value()" 
        (input)="onInput($event)"
      >
      <span>{{ value() }}</span>
    </div>
  `
})
export class SliderComponent {
  value = model<number>(0);

  onInput(event: Event) {
    const input = event.target as HTMLInputElement;
    this.value.set(parseInt(input.value, 10));
  }
}
```

Usage:

```html
<app-slider [(value)]="brightness"></app-slider>
```

## Implementation Patterns with Angular 19

### 1. Advanced Data Table with Signals and Server-Side Operations

```typescript
// data-table.component.ts
import { Component, inject, input, output, signal, computed } from '@angular/core';
import { CommonModule } from '@angular/common';
import { HttpClient } from '@angular/common/http';
import { ReactiveFormsModule, FormControl } from '@angular/forms';
import { debounceTime, distinctUntilChanged, switchMap, catchError } from 'rxjs/operators';
import { of } from 'rxjs';

interface TableColumn<T> {
  key: keyof T;
  header: string;
  sortable?: boolean;
  formatter?: (value: any) => string;
}

interface SortState {
  column: string;
  direction: 'asc' | 'desc';
}

interface PaginationState {
  page: number;
  pageSize: number;
  total: number;
}

@Component({
  selector: 'app-data-table',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `
    <div class="data-table-container">
      <!-- Search -->
      <div class="search-container">
        <input 
          type="text" 
          [formControl]="searchControl"
          placeholder="Search..."
        >
      </div>

      <!-- Table -->
      <table class="data-table">
        <thead>
          <tr>
            @for (column of columns(); track column.key) {
              <th 
                [class.sortable]="column.sortable"
                (click)="column.sortable ? sort(column.key.toString()) : null"
              >
                {{ column.header }}
                @if (column.sortable && sortState().column === column.key.toString()) {
                  <span class="sort-indicator">
                    {{ sortState().direction === 'asc' ? '↑' : '↓' }}
                  </span>
                }
              </th>
            }
            @if (showActions()) {
              <th>Actions</th>
            }
          </tr>
        </thead>
        <tbody>
          @if (loading()) {
            <tr>
              <td [attr.colspan]="showActions() ? columns().length + 1 : columns().length" class="loading-cell">
                Loading data...
              </td>
            </tr>
          } @else {
            @for (item of data(); track trackBy(item)) {
              <tr>
                @for (column of columns(); track column.key) {
                  <td>
                    {{ column.formatter ? column.formatter(item[column.key]) : item[column.key] }}
                  </td>
                }
                @if (showActions()) {
                  <td class="actions-cell">
                    <button (click)="viewItem(item)">View</button>
                    <button (click)="editItem(item)">Edit</button>
                    <button (click)="deleteItem(item)">Delete</button>
                  </td>
                }
              </tr>
            } @empty {
              <tr>
                <td [attr.colspan]="showActions() ? columns().length + 1 : columns().length" class="empty-cell">
                  No data available
                </td>
              </tr>
            }
          }
        </tbody>
      </table>

      <!-- Pagination -->
      @if (pagination().total > 0) {
        <div class="pagination">
          <button 
            [disabled]="pagination().page === 1"
            (click)="changePage(pagination().page - 1)"
          >
            Previous
          </button>

          <span class="page-info">
            Page {{ pagination().page }} of {{ totalPages() }}
          </span>

          <button 
            [disabled]="pagination().page >= totalPages()"
            (click)="changePage(pagination().page + 1)"
          >
            Next
          </button>

          <select [value]="pagination().pageSize" (change)="changePageSize($event)">
            <option value="10">10 per page</option>
            <option value="25">25 per page</option>
            <option value="50">50 per page</option>
            <option value="100">100 per page</option>
          </select>
        </div>
      }

      @if (error()) {
        <div class="error-message">
          {{ error() }}
          <button (click)="loadData()">Retry</button>
        </div>
      }
    </div>
  `,
  styleUrl: './data-table.component.css'
})
export class DataTableComponent<T extends { id: string | number }> {
  private http = inject(HttpClient);

  // Inputs
  apiUrl = input.required<string>();
  columns = input.required<TableColumn<T>[]>();
  showActions = input<boolean>(true);

  // Outputs
  rowSelected = output<T>();
  rowDeleted = output<T>();
  rowEdited = output<T>();

  // State
  data = signal<T[]>([]);
  loading = signal(false);
  error = signal<string | null>(null);
  sortState = signal<SortState>({ column: 'id', direction: 'asc' });
  pagination = signal<PaginationState>({ page: 1, pageSize: 10, total: 0 });
  searchControl = new FormControl('');

  // Computed
  totalPages = computed(() => 
    Math.ceil(this.pagination().total / this.pagination().pageSize)
  );

  constructor() {
    // Handle search with RxJS
    this.searchControl.valueChanges.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(searchTerm => {
        this.loading.set(true);
        return this.fetchData(searchTerm || '');
      }),
      catchError(err => {
        this.error.set(err.message || 'An error occurred while searching');
        this.loading.set(false);
        return of([]);
      })
    ).subscribe(response => {
      this.data.set(response.data);
      this.pagination.update(p => ({ ...p, total: response.total }));
      this.loading.set(false);
    });

    // Initial data load
    this.loadData();
  }

  loadData() {
    this.loading.set(true);
    this.error.set(null);

    this.fetchData(this.searchControl.value || '')
      .pipe(
        catchError(err => {
          this.error.set(err.message || 'Failed to load data');
          this.loading.set(false);
          return of({ data: [], total: 0 });
        })
      )
      .subscribe(response => {
        this.data.set(response.data);
        this.pagination.update(p => ({ ...p, total: response.total }));
        this.loading.set(false);
      });
  }

  private fetchData(searchTerm: string) {
    const { page, pageSize } = this.pagination();
    const { column, direction } = this.sortState();

    return this.http.get<{ data: T[], total: number }>(
      `${this.apiUrl()}?page=${page}&pageSize=${pageSize}&sortBy=${column}&sortDirection=${direction}&search=${searchTerm}`
    );
  }

  sort(column: string) {
    this.sortState.update(state => {
      if (state.column === column) {
        return {
          column,
          direction: state.direction === 'asc' ? 'desc' : 'asc'
        };
      }
      return { column, direction: 'asc' };
    });

    this.loadData();
  }

  changePage(page: number) {
    this.pagination.update(p => ({ ...p, page }));
    this.loadData();
  }

  changePageSize(event: Event) {
    const select = event.target as HTMLSelectElement;
    const pageSize = parseInt(select.value, 10);

    this.pagination.update(p => ({ ...p, pageSize, page: 1 }));
    this.loadData();
  }

  viewItem(item: T) {
    this.rowSelected.emit(item);
  }

  editItem(item: T) {
    this.rowEdited.emit(item);
  }

  deleteItem(item: T) {
    if (confirm('Are you sure you want to delete this item?')) {
      this.http.delete(`${this.apiUrl()}/${item.id}`)
        .subscribe({
          next: () => {
            this.data.update(items => items.filter(i => i.id !== item.id));
            this.pagination.update(p => ({ ...p, total: p.total - 1 }));
            this.rowDeleted.emit(item);
          },
          error: (err) => {
            this.error.set(err.message || 'Failed to delete item');
          }
        });
    }
  }

  trackBy(item: T) {
    return item.id;
  }
}
```

Usage:

```typescript
// products-page.component.ts
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { DataTableComponent } from '../../shared/components/data-table/data-table.component';

interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
  inStock: boolean;
  createdAt: string;
}

@Component({
  selector: 'app-products-page',
  standalone: true,
  imports: [CommonModule, DataTableComponent],
  template: `
    <div class="products-page">
      <h1>Products</h1>

      <app-data-table
        [apiUrl]="apiUrl"
        [columns]="columns"
        (rowSelected)="onProductSelected($event)"
        (rowEdited)="onProductEdit($event)"
        (rowDeleted)="onProductDeleted($event)"
      ></app-data-table>
    </div>
  `
})
export class ProductsPageComponent {
  apiUrl = '/api/products';

  columns = [
    { key: 'name', header: 'Product Name', sortable: true },
    { key: 'price', header: 'Price', sortable: true, formatter: (value: number) => `$${value.toFixed(2)}` },
    { key: 'category', header: 'Category', sortable: true },
    { key: 'inStock', header: 'In Stock', formatter: (value: boolean) => value ? 'Yes' : 'No' },
    { key: 'createdAt', header: 'Created', sortable: true, formatter: (value: string) => new Date(value).toLocaleDateString() }
  ];

  onProductSelected(product: Product) {
    console.log('Product selected:', product);
  }

  onProductEdit(product: Product) {
    console.log('Edit product:', product);
  }

  onProductDeleted(product: Product) {
    console.log('Product deleted:', product);
  }
}
```

### 2. Advanced Form with Dynamic Validation and Conditional Fields

```typescript
// dynamic-form.component.ts
import { Component, input, output, signal, computed } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ReactiveFormsModule, FormBuilder, FormGroup, Validators, ValidatorFn } from '@angular/forms';

export interface FormFieldConfig {
  key: string;
  label: string;
  type: 'text' | 'number' | 'email' | 'password' | 'select' | 'checkbox' | 'radio' | 'textarea' | 'date';
  defaultValue?: any;
  options?: { value: any, label: string }[];
  validators?: {
    required?: boolean;
    minLength?: number;
    maxLength?: number;
    min?: number;
    max?: number;
    email?: boolean;
    pattern?: string;
    custom?: (value: any) => boolean;
  };
  errorMessages?: { [key: string]: string };
  dependsOn?: {
    field: string;
    value: any;
    operator?: '==' | '!=' | '>' | '<' | '>=' | '<=';
  };
}

export interface FormConfig {
  fields: FormFieldConfig[];
  submitLabel?: string;
  cancelLabel?: string;
}

@Component({
  selector: 'app-dynamic-form',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      @for (field of visibleFields(); track field.key) {
        <div class="form-field">
          <label [for]="field.key">{{ field.label }}</label>

          @switch (field.type) {
            @case ('text') {
              <input 
                [id]="field.key" 
                type="text" 
                [formControlName]="field.key">
            }
            @case ('number') {
              <input 
                [id]="field.key" 
                type="number" 
                [formControlName]="field.key">
            }
            @case ('email') {
              <input 
                [id]="field.key" 
                type="email" 
                [formControlName]="field.key">
            }
            @case ('password') {
              <input 
                [id]="field.key" 
                type="password" 
                [formControlName]="field.key">
            }
            @case ('date') {
              <input 
                [id]="field.key" 
                type="date" 
                [formControlName]="field.key">
            }
            @case ('select') {
              <select [id]="field.key" [formControlName]="field.key">
                <option value="">-- Select --</option>
                @for (option of field.options; track option.value) {
                  <option [value]="option.value">{{ option.label }}</option>
                }
              </select>
            }
            @case ('checkbox') {
              <input 
                [id]="field.key" 
                type="checkbox" 
                [formControlName]="field.key">
            }
            @case ('radio') {
              <div class="radio-group">
                @for (option of field.options; track option.value) {
                  <label class="radio-label">
                    <input 
                      type="radio" 
                      [formControlName]="field.key" 
                      [value]="option.value"> 
                    {{ option.label }}
                  </label>
                }
              </div>
            }
            @case ('textarea') {
              <textarea 
                [id]="field.key" 
                [formControlName]="field.key" 
                rows="4">
              </textarea>
            }
          }

          @if (hasErrors(field.key)) {
            <div class="error-messages">
              @for (error of getErrorMessages(field); track error) {
                <p class="error">{{ error }}</p>
              }
            </div>
          }
        </div>
      }

      <div class="form-actions">
        @if (showCancelButton()) {
          <button type="button" (click)="onCancel()">
            {{ formConfig().cancelLabel || 'Cancel' }}
          </button>
        }
        <button type="submit" [disabled]="form.invalid">
          {{ formConfig().submitLabel || 'Submit' }}
        </button>
      </div>
    </form>
  `,
  styleUrl: './dynamic-form.component.css'
})
export class DynamicFormComponent {
  // Inputs
  formConfig = input.required<FormConfig>();
  initialValues = input<Record<string, any>>({});
  showCancelButton = input<boolean>(true);

  // Outputs
  formSubmit = output<Record<string, any>>();
  formCancel = output<void>();

  // Form
  form!: FormGroup;

  // Computed
  visibleFields = computed(() => {
    return this.formConfig().fields.filter(field => {
      if (!field.dependsOn) return true;

      const { dependsOn } = field;
      const dependentValue = this.form?.get(dependsOn.field)?.value;

      if (dependentValue === undefined) return false;

      switch (dependsOn.operator) {
        case '!=': return dependentValue !== dependsOn.value;
        case '>': return dependentValue > dependsOn.value;
        case '<': return dependentValue < dependsOn.value;
        case '>=': return dependentValue >= dependsOn.value;
        case '<=': return dependentValue <= dependsOn.value;
        default: return dependentValue === dependsOn.value;
      }
    });
  });

  constructor(private fb: FormBuilder) {}

  ngOnInit() {
    this.buildForm();

    // Listen for changes in fields with dependencies
    const fieldsWithDependents = new Set(
      this.formConfig().fields
        .filter(f => f.dependsOn)
        .map(f => f.dependsOn!.field)
    );

    fieldsWithDependents.forEach(fieldKey => {
      this.form.get(fieldKey)?.valueChanges.subscribe(() => {
        // Recalculate visible fields (handled by the computed property)
        // Reset values for fields that become invisible
        this.formConfig().fields.forEach(field => {
          if (field.dependsOn?.field === fieldKey && !this.isFieldVisible(field)) {
            this.form.get(field.key)?.reset(field.defaultValue || null);
          }
        });
      });
    });
  }

  private buildForm() {
    const formControls: Record<string, any> = {};

    this.formConfig().fields.forEach(field => {
      const validators: ValidatorFn[] = [];

      if (field.validators?.required) {
        validators.push(Validators.required);
      }

      if (field.validators?.minLength) {
        validators.push(Validators.minLength(field.validators.minLength));
      }

      if (field.validators?.maxLength) {
        validators.push(Validators.maxLength(field.validators.maxLength));
      }

      if (field.validators?.min) {
        validators.push(Validators.min(field.validators.min));
      }

      if (field.validators?.max) {
        validators.push(Validators.max(field.validators.max));
      }

      if (field.validators?.email) {
        validators.push(Validators.email);
      }

      if (field.validators?.pattern) {
        validators.push(Validators.pattern(field.validators.pattern));
      }

      if (field.validators?.custom) {
        validators.push(control => {
          return field.validators!.custom!(control.value) ? null : { custom: true };
        });
      }

      const initialValue = this.initialValues()[field.key] !== undefined 
        ? this.initialValues()[field.key] 
        : field.defaultValue !== undefined 
          ? field.defaultValue 
          : '';

      formControls[field.key] = [initialValue, validators];
    });

    this.form = this.fb.group(formControls);
  }

  hasErrors(fieldKey: string): boolean {
    const control = this.form.get(fieldKey);
    return !!control && control.invalid && (control.dirty || control.touched);
  }

  getErrorMessages(field: FormFieldConfig): string[] {
    const control = this.form.get(field.key);
    if (!control || !control.errors) return [];

    const errorMessages: string[] = [];
    const errors = control.errors;
    const defaultMessages: Record<string, string> = {
      required: `${field.label} is required`,
      minlength: `${field.label} must be at least ${field.validators?.minLength} characters`,
      maxlength: `${field.label} cannot exceed ${field.validators?.maxLength} characters`,
      min: `${field.label} must be at least ${field.validators?.min}`,
      max: `${field.label} cannot exceed ${field.validators?.max}`,
      email: `Please enter a valid email address`,
      pattern: `${field.label} has an invalid format`,
      custom: `${field.label} is invalid`
    };

    Object.keys(errors).forEach(errorKey => {
      const customMessage = field.errorMessages?.[errorKey];
      errorMessages.push(customMessage || defaultMessages[errorKey] || `${field.label} is invalid`);
    });

    return errorMessages;
  }

  isFieldVisible(field: FormFieldConfig): boolean {
    if (!field.dependsOn) return true;

    const { dependsOn } = field;
    const dependentValue = this.form?.get(dependsOn.field)?.value;

    if (dependentValue === undefined) return false;

    switch (dependsOn.operator) {
      case '!=': return dependentValue !== dependsOn.value;
      case '>': return dependentValue > dependsOn.value;
      case '<': return dependentValue < dependsOn.value;
      case '>=': return dependentValue >= dependsOn.value;
      case '<=': return dependentValue <= dependsOn.value;
      default: return dependentValue === dependsOn.value;
    }
  }

  onSubmit() {
    if (this.form.valid) {
      this.formSubmit.emit(this.form.value);
    } else {
      this.markFormGroupTouched(this.form);
    }
  }

  onCancel() {
    this.formCancel.emit();
  }

  private markFormGroupTouched(formGroup: FormGroup) {
    Object.values(formGroup.controls).forEach(control => {
      control.markAsTouched();

      if ((control as any).controls) {
        this.markFormGroupTouched(control as FormGroup);
      }
    });
  }
}
```

Usage:

```typescript
// registration-page.component.ts
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { DynamicFormComponent, FormConfig } from '../../shared/components/dynamic-form/dynamic-form.component';

@Component({
  selector: 'app-registration-page',
  standalone: true,
  imports: [CommonModule, DynamicFormComponent],
  template: `
    <div class="registration-page">
      <h1>Create Account</h1>

      <app-dynamic-form
        [formConfig]="formConfig"
        (formSubmit)="onFormSubmit($event)"
        (formCancel)="onCancel()">
      </app-dynamic-form>
    </div>
  `
})
export class RegistrationPageComponent {
  formConfig: FormConfig = {
    fields: [
      {
        key: 'name',
        label: 'Full Name',
        type: 'text',
        validators: {
          required: true,
          minLength: 3,
          maxLength: 50
        },
        errorMessages: {
          required: 'Please enter your full name',
          minLength: 'Name must be at least 3 characters long'
        }
      },
      {
        key: 'email',
        label: 'Email Address',
        type: 'email',
        validators: {
          required: true,
          email: true
        },
        errorMessages: {
          required: 'Please enter your email address',
          email: 'Please enter a valid email address'
        }
      },
      {
        key: 'password',
        label: 'Password',
        type: 'password',
        validators: {
          required: true,
          minLength: 8,
          pattern: '^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)[a-zA-Z\\d]{8,}$'
        },
        errorMessages: {
          required: 'Please enter a password',
          minLength: 'Password must be at least 8 characters long',
          pattern: 'Password must contain at least one uppercase letter, one lowercase letter, and one number'
        }
      },
      {
        key: 'accountType',
        label: 'Account Type',
        type: 'select',
        options: [
          { value: 'personal', label: 'Personal Account' },
          { value: 'business', label: 'Business Account' }
        ],
        validators: {
          required: true
        }
      },
      {
        key: 'companyName',
        label: 'Company Name',
        type: 'text',
        validators: {
          required: true
        },
        dependsOn: {
          field: 'accountType',
          value: 'business'
        }
      },
      {
        key: 'taxId',
        label: 'Tax ID / VAT Number',
        type: 'text',
        dependsOn: {
          field: 'accountType',
          value: 'business'
        }
      },
      {
        key: 'subscriptionPlan',
        label: 'Subscription Plan',
        type: 'radio',
        options: [
          { value: 'free', label: 'Free' },
          { value: 'basic', label: 'Basic ($9.99/month)' },
          { value: 'premium', label: 'Premium ($19.99/month)' }
        ],
        defaultValue: 'free',
        validators: {
          required: true
        }
      },
      {
        key: 'paymentMethod',
        label: 'Payment Method',
        type: 'select',
        options: [
          { value: 'creditCard', label: 'Credit Card' },
          { value: 'paypal', label: 'PayPal' },
          { value: 'bankTransfer', label: 'Bank Transfer' }
        ],
        validators: {
          required: true
        },
        dependsOn: {
          field: 'subscriptionPlan',
          value: 'free',
          operator: '!='
        }
      },
      {
        key: 'termsAccepted',
        label: 'I accept the Terms and Conditions',
        type: 'checkbox',
        validators: {
          required: true
        },
        errorMessages: {
          required: 'You must accept the Terms and Conditions to continue'
        }
      }
    ],
    submitLabel: 'Create Account',
    cancelLabel: 'Cancel'
  };

  onFormSubmit(formData: Record<string, any>) {
    console.log('Form submitted:', formData);
    // Handle form submission (API call, etc.)
  }

  onCancel() {
    console.log('Registration cancelled');
    // Handle cancellation (navigation, etc.)
  }
}
```

## Integration with Other Technologies

### 1. Tailwind CSS Integration

```bash
# Install Tailwind CSS
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init
```

Configure `tailwind.config.js`:

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./src/**/*.{html,ts}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0f9ff',
          100: '#e0f2fe',
          200: '#bae6fd',
          300: '#7dd3fc',
          400: '#38bdf8',
          500: '#0ea5e9',
          600: '#0284c7',
          700: '#0369a1',
          800: '#075985',
          900: '#0c4a6e',
          950: '#082f49',
        },
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
  ],
}
```

### 2. Firebase Integration with Angular 19

```typescript
// app.config.ts
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { provideFirebaseApp, initializeApp } from '@angular/fire/app';
import { provideAuth, getAuth } from '@angular/fire/auth';
import { provideFirestore, getFirestore } from '@angular/fire/firestore';
import { provideStorage, getStorage } from '@angular/fire/storage';
import { provideFunctions, getFunctions } from '@angular/fire/functions';

import { routes } from './app.routes';

const firebaseConfig = {
  apiKey: "your-api-key",
  authDomain: "your-auth-domain",
  projectId: "your-project-id",
  storageBucket: "your-storage-bucket",
  messagingSenderId: "your-messaging-sender-id",
  appId: "your-app-id"
};

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    importProvidersFrom(
      provideFirebaseApp(() => initializeApp(firebaseConfig)),
      provideAuth(() => getAuth()),
      provideFirestore(() => getFirestore()),
      provideStorage(() => getStorage()),
      provideFunctions(() => getFunctions())
    )
  ]
};
```

### 3. GraphQL with Apollo Client and Angular 19

```typescript
// app.config.ts
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { APOLLO_OPTIONS, ApolloModule } from 'apollo-angular';
import { HttpLink } from 'apollo-angular/http';
import { InMemoryCache, ApolloLink } from '@apollo/client/core';
import { setContext } from '@apollo/client/link/context';

import { routes } from './app.routes';

export function createApollo(httpLink: HttpLink) {
  const basic = setContext((operation, context) => ({
    headers: {
      Accept: 'charset=utf-8'
    }
  }));

  const auth = setContext((operation, context) => {
    const token = localStorage.getItem('token');

    if (token === null) {
      return {};
    } else {
      return {
        headers: {
          Authorization: `Bearer ${token}`
        }
      };
    }
  });

  const link = ApolloLink.from([
    basic,
    auth,
    httpLink.create({ uri: 'https://your-graphql-endpoint' })
  ]);

  return {
    link,
    cache: new InMemoryCache(),
    defaultOptions: {
      watchQuery: {
        fetchPolicy: 'cache-and-network',
        errorPolicy: 'ignore',
      },
      query: {
        fetchPolicy: 'network-only',
        errorPolicy: 'all',
      },
      mutate: {
        errorPolicy: 'all',
      },
    },
  };
}

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    importProvidersFrom(ApolloModule),
    {
      provide: APOLLO_OPTIONS,
      useFactory: createApollo,
      deps: [HttpLink],
    }
  ]
};
```

## Conclusion

Angular 19 represents a significant evolution from Angular 17, with major improvements in reactivity, performance, and developer experience. The introduction of a fully matured signals API, zone.js-less applications, enhanced build performance, and advanced server-side rendering capabilities make Angular 19 a powerful framework for building modern web applications.

By adopting the new features and best practices outlined in this guide, you can leverage the full potential of Angular 19 to create efficient, maintainable, and scalable applications. The migration path from Angular 17 to Angular 19 is well-defined, allowing for incremental updates and adoption of new features at your own pace.

As Angular continues to evolve, staying up-to-date with the latest features and best practices will ensure that your applications remain modern, efficient, and aligned with the framework's recommended patterns.
