# Angular 20 Development Guide

## Introduction

Angular 20, released in May 2025, represents the next major evolution in the Angular framework, building upon the foundations established in Angular 19. This guide provides comprehensive information about the evolution from Angular 19 to Angular 20, highlighting key changes, new features, and best practices to help you effectively generate modern web applications with the latest Angular capabilities.

## Evolution from Angular 19 to Angular 20

### Angular 19 Key Improvements (November 2024)

Angular 19 introduced several important enhancements:

1. **Fully Matured Signals API**: Complete reactivity system with comprehensive tooling
2. **Zone.js-less Applications**: Full support for applications without Zone.js
3. **Enhanced Build Performance**: Significant improvements in build and compilation speed
4. **Advanced Server-Side Rendering**: New rendering strategies and hydration improvements
5. **Improved Developer Tooling**: Enhanced debugging and performance profiling

### Angular 20 Key Improvements (May 2025)

Angular 20 builds on these changes with:

1. **Reactive State Management Framework**: Built-in state management solution based on signals
2. **Hydration Improvements**: Zero-flicker hydration with intelligent state reconciliation
3. **Micro-Frontend Architecture Support**: Native tooling for module federation and micro-frontends
4. **Enhanced Developer Experience**: Improved error messages, debugging tools, and performance insights
5. **Streamlined Dependency Injection**: Simplified DI system with improved tree-shaking
6. **Web Components Integration**: Better support for creating and consuming web components

## Key Features and Changes from Angular 19 to Angular 20

### 1. Reactive State Management

#### Angular 19 (Signal-based State)

```typescript
import { Component, signal, computed, effect } from '@angular/core';

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

  constructor() {
    effect(() => {
      console.log(`Count changed to: ${this.count()}`);
    });
  }

  increment() {
    this.count.update(value => value + 1);
  }
}
```

#### Angular 20 (Built-in State Management)

```typescript
import { Component, inject } from '@angular/core';
import { state, select, action } from '@angular/core/state';

// Define state interface
interface CounterState {
  count: number;
  lastUpdated: Date;
}

// Create a state store with initial values
const counterState = state<CounterState>({
  count: 0,
  lastUpdated: new Date()
});

// Define actions
const increment = action(counterState, 'increment', (state) => ({
  ...state,
  count: state.count + 1,
  lastUpdated: new Date()
}));

const reset = action(counterState, 'reset', () => ({
  count: 0,
  lastUpdated: new Date()
}));

// Create selectors
const selectCount = select(counterState, state => state.count);
const selectDoubled = select(counterState, state => state.count * 2);
const selectLastUpdated = select(counterState, state => state.lastUpdated);

@Component({
  selector: 'app-counter',
  template: `
    <h2>Counter: {{ count() }}</h2>
    <p>Doubled: {{ doubled() }}</p>
    <p>Last updated: {{ lastUpdated() | date:'medium' }}</p>
    <button (click)="increment()">Increment</button>
    <button (click)="reset()">Reset</button>
  `
})
export class CounterComponent {
  // Inject selectors as signals
  count = inject(selectCount);
  doubled = inject(selectDoubled);
  lastUpdated = inject(selectLastUpdated);

  // Inject actions
  increment = inject(increment);
  reset = inject(reset);
}
```

### 2. Hydration Improvements

#### Angular 19 (Basic Hydration)

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
import { provideRouter } from '@angular/router';
import { provideClientHydration } from '@angular/platform-browser';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideClientHydration()
  ]
};
```

#### Angular 20 (Enhanced Hydration)

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
import { provideRouter } from '@angular/router';
import { provideClientHydration, withAdvancedHydration, HydrationFeatures } from '@angular/platform-browser';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideClientHydration(
      withAdvancedHydration({
        // New hydration features
        features: [
          HydrationFeatures.ZeroFlicker,
          HydrationFeatures.StateReconciliation,
          HydrationFeatures.ProgressiveHydration
        ],
        // Prioritize critical components
        criticalComponents: ['app-header', 'app-navigation', 'app-hero'],
        // Defer hydration for non-visible components
        deferNonVisibleHydration: true
      })
    )
  ]
};
```

### 3. Micro-Frontend Architecture Support

#### Angular 19 (Manual Module Federation)

```typescript
// webpack.config.js
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

module.exports = {
  // ...
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      filename: 'remoteEntry.js',
      remotes: {
        mfe1: 'mfe1@http://localhost:3001/remoteEntry.js',
      },
      shared: {
        '@angular/core': { singleton: true },
        '@angular/common': { singleton: true },
        '@angular/router': { singleton: true }
      }
    })
  ]
};
```

```typescript
// Remote component loading
import { loadRemoteModule } from '@angular-architects/module-federation';

const routes: Routes = [
  {
    path: 'mfe1',
    loadChildren: () => loadRemoteModule({
      type: 'module',
      remoteEntry: 'http://localhost:3001/remoteEntry.js',
      exposedModule: './Module'
    }).then(m => m.RemoteModule)
  }
];
```

#### Angular 20 (Native Micro-Frontend Support)

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideMicroFrontends, MicroFrontendConfig } from '@angular/core/micro-frontends';

import { routes } from './app.routes';

const microFrontends: MicroFrontendConfig[] = [
  {
    name: 'dashboard',
    remoteUrl: 'http://localhost:3001/remoteEntry.js',
    exposedModule: './Dashboard',
    sharedDependencies: ['@angular/core', '@angular/common', '@angular/router']
  },
  {
    name: 'user-profile',
    remoteUrl: 'http://localhost:3002/remoteEntry.js',
    exposedModule: './UserProfile',
    sharedDependencies: ['@angular/core', '@angular/common', '@angular/router']
  }
];

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideMicroFrontends(microFrontends)
  ]
};
```

```typescript
// app.routes.ts
import { Routes } from '@angular/router';
import { microFrontendRoute } from '@angular/core/micro-frontends';

export const routes: Routes = [
  {
    path: 'dashboard',
    ...microFrontendRoute('dashboard')
  },
  {
    path: 'profile',
    ...microFrontendRoute('user-profile')
  }
];
```

```typescript
// Shell component
import { Component } from '@angular/core';
import { MicroFrontendLoader } from '@angular/core/micro-frontends';

@Component({
  selector: 'app-shell',
  template: `
    <header>Main Application Shell</header>
    <nav>
      <a routerLink="/">Home</a>
      <a routerLink="/dashboard">Dashboard</a>
      <a routerLink="/profile">Profile</a>
    </nav>

    <!-- Directly load micro-frontend without routing -->
    <micro-frontend name="notifications"></micro-frontend>

    <main>
      <router-outlet></router-outlet>
    </main>
  `
})
export class ShellComponent {}
```

### 4. Streamlined Dependency Injection

#### Angular 19 (Current DI System)

```typescript
import { Component, Injectable, inject } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  getUser() {
    return { id: 1, name: 'John Doe' };
  }
}

@Component({
  selector: 'app-user',
  template: `<h2>User: {{ user.name }}</h2>`
})
export class UserComponent {
  private userService = inject(UserService);
  user = this.userService.getUser();
}
```

#### Angular 20 (Enhanced DI System)

```typescript
import { Component, injectable, inject } from '@angular/core';

// Simplified injectable decorator with improved tree-shaking
@injectable()
export class UserService {
  getUser() {
    return { id: 1, name: 'John Doe' };
  }

  // Scoped providers
  static scoped() {
    return {
      provide: UserService,
      useFactory: () => new UserService()
    };
  }
}

@Component({
  selector: 'app-user',
  template: `<h2>User: {{ user.name }}</h2>`,
  // Component-scoped providers with simplified syntax
  providers: [UserService.scoped()]
})
export class UserComponent {
  // Automatic type inference
  user = inject(UserService).getUser();
}
```

### 5. Web Components Integration

#### Angular 19 (Basic Web Components)

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-rating',
  standalone: true,
  template: `
    <div class="rating">
      @for (star of stars; track $index) {
        <span 
          class="star" 
          [class.filled]="star <= value"
          (click)="setValue(star)">
          ★
        </span>
      }
    </div>
  `,
  styles: [`
    .star { cursor: pointer; font-size: 24px; color: #ccc; }
    .filled { color: gold; }
  `]
})
export class RatingComponent {
  @Input() value = 0;
  @Output() valueChange = new EventEmitter<number>();

  stars = [1, 2, 3, 4, 5];

  setValue(value: number) {
    this.value = value;
    this.valueChange.emit(value);
  }
}
```

#### Angular 20 (Enhanced Web Components)

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { withWebComponent } from '@angular/elements';

@Component({
  selector: 'app-rating',
  standalone: true,
  template: `
    <div class="rating" part="container">
      @for (star of stars; track $index) {
        <span 
          class="star" 
          part="star"
          [attr.data-value]="star"
          [class.filled]="star <= value"
          (click)="setValue(star)">
          ★
        </span>
      }
    </div>
  `,
  styles: [`
    :host { display: block; }
    .star { cursor: pointer; font-size: 24px; color: #ccc; }
    .filled { color: gold; }
  `],
  // New decorator for enhanced web component support
  providers: [
    withWebComponent({
      // Define custom element name
      name: 'rating-element',
      // Expose CSS custom properties
      cssProperties: ['--star-color', '--star-size', '--star-filled-color'],
      // Expose CSS shadow parts
      cssParts: ['container', 'star'],
      // Expose form association
      formAssociated: true
    })
  ]
})
export class RatingComponent {
  @Input() value = 0;
  @Output() valueChange = new EventEmitter<number>();

  stars = [1, 2, 3, 4, 5];

  setValue(value: number) {
    this.value = value;
    this.valueChange.emit(value);
  }
}
```

Usage in HTML:

```html
<!-- Using the web component in any framework or vanilla HTML -->
<rating-element value="3"></rating-element>

<!-- Styling using CSS custom properties -->
<style>
  rating-element {
    --star-color: #ddd;
    --star-filled-color: #ff9900;
    --star-size: 32px;
  }

  /* Styling using CSS shadow parts */
  rating-element::part(star) {
    transition: transform 0.2s;
  }

  rating-element::part(star):hover {
    transform: scale(1.2);
  }
</style>
```

## Required Configuration

### 1. Project Setup

#### Using Angular CLI (Recommended)

```bash
# Install the latest Angular CLI
npm install -g @angular/cli

# Create a new Angular 20 project
ng new my-angular-app --standalone

# Navigate to the project directory
cd my-angular-app

# Start the development server
ng serve
```

#### package.json Dependencies

Ensure your package.json includes these core dependencies:

```json
{
  "dependencies": {
    "@angular/animations": "^20.0.0",
    "@angular/common": "^20.0.0",
    "@angular/compiler": "^20.0.0",
    "@angular/core": "^20.0.0",
    "@angular/forms": "^20.0.0",
    "@angular/platform-browser": "^20.0.0",
    "@angular/platform-browser-dynamic": "^20.0.0",
    "@angular/router": "^20.0.0",
    "rxjs": "~7.8.1",
    "tslib": "^2.6.0",
    "zone.js": "~0.15.0"
  },
  "devDependencies": {
    "@angular-devkit/build-angular": "^20.0.0",
    "@angular/cli": "^20.0.0",
    "@angular/compiler-cli": "^20.0.0",
    "@types/jasmine": "~4.3.5",
    "jasmine-core": "~5.1.0",
    "karma": "~6.4.2",
    "karma-chrome-launcher": "~3.2.0",
    "karma-coverage": "~2.2.1",
    "karma-jasmine": "~5.1.0",
    "karma-jasmine-html-reporter": "~2.1.0",
    "typescript": "~5.8.3"
  }
}
```

### 2. Application Bootstrap

#### main.ts

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch((err) => console.error(err));
```

#### app.config.ts

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideRouter, withComponentInputBinding } from '@angular/router';
import { provideHttpClient, withFetch } from '@angular/common/http';
import { provideClientHydration, withAdvancedHydration, HydrationFeatures } from '@angular/platform-browser';
import { provideAnimations } from '@angular/platform-browser/animations';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withComponentInputBinding()
    ),
    provideHttpClient(withFetch()),
    provideClientHydration(
      withAdvancedHydration({
        features: [HydrationFeatures.ZeroFlicker]
      })
    ),
    provideAnimations()
  ]
};
```

### 3. Routing Configuration

#### app.routes.ts

```typescript
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    loadComponent: () => import('./home/home.component').then(m => m.HomeComponent)
  },
  {
    path: 'about',
    loadComponent: () => import('./about/about.component').then(m => m.AboutComponent)
  },
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.routes').then(m => m.DASHBOARD_ROUTES)
  },
  {
    path: '**',
    loadComponent: () => import('./not-found/not-found.component').then(m => m.NotFoundComponent)
  }
];
```

## Optional Configuration Sets

### 1. State Management

#### Built-in State Management

Angular 20 introduces a built-in state management solution based on signals:

```typescript
// state/counter.state.ts
import { state, select, action } from '@angular/core/state';

export interface CounterState {
  count: number;
  isLoading: boolean;
  error: string | null;
}

// Create state with initial values
export const counterState = state<CounterState>({
  count: 0,
  isLoading: false,
  error: null
});

// Define selectors
export const selectCount = select(counterState, state => state.count);
export const selectIsLoading = select(counterState, state => state.isLoading);
export const selectError = select(counterState, state => state.error);
export const selectDoubledCount = select(counterState, state => state.count * 2);

// Define actions
export const increment = action(counterState, 'increment', (state) => ({
  ...state,
  count: state.count + 1
}));

export const decrement = action(counterState, 'decrement', (state) => ({
  ...state,
  count: state.count - 1
}));

export const reset = action(counterState, 'reset', (state) => ({
  ...state,
  count: 0
}));

export const setLoading = action(counterState, 'setLoading', (state, isLoading: boolean) => ({
  ...state,
  isLoading
}));

export const setError = action(counterState, 'setError', (state, error: string | null) => ({
  ...state,
  error
}));
```

```typescript
// counter.component.ts
import { Component, inject } from '@angular/core';
import { 
  selectCount, 
  selectDoubledCount, 
  selectIsLoading, 
  selectError,
  increment,
  decrement,
  reset,
  setLoading,
  setError
} from '../state/counter.state';

@Component({
  selector: 'app-counter',
  template: `
    <div class="counter">
      <h2>Counter: {{ count() }}</h2>
      <p>Doubled: {{ doubledCount() }}</p>

      @if (isLoading()) {
        <p>Loading...</p>
      }

      @if (error()) {
        <p class="error">{{ error() }}</p>
      }

      <div class="actions">
        <button (click)="decrement()">-</button>
        <button (click)="increment()">+</button>
        <button (click)="reset()">Reset</button>
        <button (click)="loadFromServer()">Load from Server</button>
      </div>
    </div>
  `
})
export class CounterComponent {
  // Inject selectors as signals
  count = inject(selectCount);
  doubledCount = inject(selectDoubledCount);
  isLoading = inject(selectIsLoading);
  error = inject(selectError);

  // Inject actions
  increment = inject(increment);
  decrement = inject(decrement);
  reset = inject(reset);
  setLoading = inject(setLoading);
  setError = inject(setError);

  // Additional methods
  loadFromServer() {
    this.setLoading(true);
    this.setError(null);

    // Simulate API call
    setTimeout(() => {
      if (Math.random() > 0.7) {
        this.setError('Failed to load from server');
      } else {
        // Update state with new count
        this.increment();
        this.increment();
      }
      this.setLoading(false);
    }, 1000);
  }
}
```

#### NgRx Integration

NgRx can still be used with Angular 20 for more complex state management needs:

```bash
# Install NgRx packages
npm install @ngrx/store @ngrx/effects @ngrx/entity @ngrx/store-devtools
```

```typescript
// app.config.ts
import { ApplicationConfig, isDevMode } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { provideStore } from '@ngrx/store';
import { provideEffects } from '@ngrx/effects';
import { provideStoreDevtools } from '@ngrx/store-devtools';

import { routes } from './app.routes';
import { reducers, metaReducers } from './reducers';
import { UserEffects } from './effects/user.effects';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    provideStore(reducers, { metaReducers }),
    provideEffects([UserEffects]),
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

### 2. Form Handling

#### Reactive Forms with Signals

Angular 20 enhances reactive forms with signals integration:

```typescript
import { Component } from '@angular/core';
import { FormBuilder, FormGroup, ReactiveFormsModule, Validators } from '@angular/forms';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-user-form',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <div>
        <label for="name">Name</label>
        <input id="name" type="text" formControlName="name">
        @if (userForm.controls.name.invalid && userForm.controls.name.touched) {
          <div class="error">
            @if (userForm.controls.name.errors?.['required']) {
              <p>Name is required</p>
            }
            @if (userForm.controls.name.errors?.['minlength']) {
              <p>Name must be at least 3 characters</p>
            }
          </div>
        }
      </div>

      <div>
        <label for="email">Email</label>
        <input id="email" type="email" formControlName="email">
        @if (userForm.controls.email.invalid && userForm.controls.email.touched) {
          <div class="error">
            @if (userForm.controls.email.errors?.['required']) {
              <p>Email is required</p>
            }
            @if (userForm.controls.email.errors?.['email']) {
              <p>Please enter a valid email</p>
            }
          </div>
        }
      </div>

      <div>
        <label for="role">Role</label>
        <select id="role" formControlName="role">
          <option value="">Select a role</option>
          <option value="user">User</option>
          <option value="admin">Admin</option>
          <option value="editor">Editor</option>
        </select>
      </div>

      <button type="submit" [disabled]="userForm.invalid || isSubmitting()">
        @if (isSubmitting()) {
          Submitting...
        } @else {
          Submit
        }
      </button>
    </form>

    @if (formValues()) {
      <div class="preview">
        <h3>Form Values:</h3>
        <pre>{{ formValues() | json }}</pre>
      </div>
    }
  `
})
export class UserFormComponent {
  userForm: FormGroup;
  isSubmitting = signal(false);
  formValues = signal<any>(null);

  constructor(private fb: FormBuilder) {
    this.userForm = this.fb.group({
      name: ['', [Validators.required, Validators.minLength(3)]],
      email: ['', [Validators.required, Validators.email]],
      role: ['', Validators.required]
    });

    // Form value changes are automatically tracked with signals
    effect(() => {
      console.log('Form value changed:', this.userForm.value);
    });
  }

  onSubmit() {
    if (this.userForm.valid) {
      this.isSubmitting.set(true);

      // Simulate API call
      setTimeout(() => {
        this.formValues.set(this.userForm.value);
        this.isSubmitting.set(false);
        this.userForm.reset();
      }, 1500);
    } else {
      this.userForm.markAllAsTouched();
    }
  }
}
```

### 3. Server-Side Rendering (SSR) and Static Site Generation (SSG)

Angular 20 enhances SSR and SSG capabilities:

```bash
# Add SSR to an existing project
ng add @angular/ssr
```

#### Enhanced SSR Configuration

```typescript
// app.config.server.ts
import { mergeApplicationConfig, ApplicationConfig } from '@angular/core';
import { provideServerRendering } from '@angular/platform-server';
import { appConfig } from './app.config';

const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering({
      // New options for enhanced SSR
      prerender: {
        // Routes to prerender at build time
        routes: ['/', '/about', '/features'],
        // Dynamically generate routes from data
        routeGenerator: async () => {
          // Could fetch from API or read from file
          return ['/products/1', '/products/2', '/products/3'];
        }
      },
      // Streaming SSR options
      streaming: {
        enabled: true,
        // Prioritize critical content
        criticalChunks: ['header', 'navigation', 'hero']
      }
    })
  ]
};

export const config = mergeApplicationConfig(appConfig, serverConfig);
```

### 4. Progressive Web App (PWA) Support

```bash
# Add PWA support
ng add @angular/pwa
```

Enhanced PWA configuration in Angular 20:

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideServiceWorker, ServiceWorkerConfig } from '@angular/service-worker';

import { routes } from './app.routes';

const serviceWorkerConfig: ServiceWorkerConfig = {
  enabled: true,
  // New options in Angular 20
  registration: {
    scope: './',
    registrationStrategy: 'registerWhenStable:30000'
  },
  caching: {
    dataGroups: [
      {
        name: 'api',
        urls: ['/api/**'],
        cacheConfig: {
          maxSize: 100,
          maxAge: '1d',
          strategy: 'freshness'
        }
      }
    ]
  },
  notifications: {
    enabled: true,
    defaultIcon: '/assets/icons/icon-128x128.png'
  },
  offline: {
    fallbackRoute: '/offline'
  }
};

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideServiceWorker('ngsw-worker.js', serviceWorkerConfig)
  ]
};
```

### 5. Internationalization (i18n)

Angular 20 enhances i18n support:

```bash
# Add i18n support
ng add @angular/localize
```

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { provideI18n, withTranslations, withLocale } from '@angular/localize';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    provideI18n(
      // Load translations from JSON files
      withTranslations({
        'en': () => import('./assets/i18n/en.json'),
        'es': () => import('./assets/i18n/es.json'),
        'fr': () => import('./assets/i18n/fr.json')
      }),
      // Set default locale
      withLocale('en')
    )
  ]
};
```

Usage in components:

```typescript
import { Component, inject } from '@angular/core';
import { LocaleService, translate } from '@angular/localize';

@Component({
  selector: 'app-language-switcher',
  template: `
    <div>
      <h2>{{ translate('Current language') }}: {{ currentLocale() }}</h2>
      <button (click)="setLocale('en')">English</button>
      <button (click)="setLocale('es')">Español</button>
      <button (click)="setLocale('fr')">Français</button>
    </div>
  `
})
export class LanguageSwitcherComponent {
  private localeService = inject(LocaleService);
  currentLocale = this.localeService.locale;

  setLocale(locale: string) {
    this.localeService.setLocale(locale);
  }
}
```

### 6. Testing Configuration

#### Enhanced Testing with Signals

```typescript
// counter.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { CounterComponent } from './counter.component';
import { 
  counterState, 
  increment, 
  decrement, 
  reset 
} from '../state/counter.state';

describe('CounterComponent', () => {
  let component: CounterComponent;
  let fixture: ComponentFixture<CounterComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [CounterComponent]
    }).compileComponents();

    // Reset state before each test
    counterState.set({
      count: 0,
      isLoading: false,
      error: null
    });

    fixture = TestBed.createComponent(CounterComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should display initial count', () => {
    const compiled = fixture.nativeElement;
    expect(compiled.querySelector('h2').textContent).toContain('Counter: 0');
  });

  it('should increment count when increment button is clicked', () => {
    const compiled = fixture.nativeElement;
    const incrementButton = compiled.querySelectorAll('button')[1];

    incrementButton.click();
    fixture.detectChanges();

    expect(compiled.querySelector('h2').textContent).toContain('Counter: 1');
  });

  it('should decrement count when decrement button is clicked', () => {
    // First set count to 5
    increment();
    increment();
    increment();
    increment();
    increment();
    fixture.detectChanges();

    const compiled = fixture.nativeElement;
    const decrementButton = compiled.querySelectorAll('button')[0];

    decrementButton.click();
    fixture.detectChanges();

    expect(compiled.querySelector('h2').textContent).toContain('Counter: 4');
  });

  it('should reset count when reset button is clicked', () => {
    // First set count to 5
    increment();
    increment();
    increment();
    increment();
    increment();
    fixture.detectChanges();

    const compiled = fixture.nativeElement;
    const resetButton = compiled.querySelectorAll('button')[2];

    resetButton.click();
    fixture.detectChanges();

    expect(compiled.querySelector('h2').textContent).toContain('Counter: 0');
  });
});
```

## Best Practices for Angular 20 Applications

### 1. Application Structure

Use a feature-based folder structure with signals-based state management:

```
src/
├── app/
│   ├── core/                 # Singleton services, app-level components
│   │   ├── guards/
│   │   ├── interceptors/
│   │   └── services/
│   ├── features/             # Feature modules
│   │   ├── auth/
│   │   │   ├── components/
│   │   │   ├── state/        # Feature-specific state
│   │   │   └── auth.routes.ts
│   │   ├── dashboard/
│   │   └── user-profile/
│   ├── shared/               # Shared components, directives, pipes
│   │   ├── components/
│   │   ├── directives/
│   │   └── pipes/
│   ├── state/                # Global application state
│   ├── app.component.ts
│   ├── app.config.ts
│   └── app.routes.ts
├── assets/
└── environments/
```

### 2. Performance Optimization

#### Lazy Loading with Preloading Strategies

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter, withPreloading, PreloadAllModules } from '@angular/router';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withPreloading(PreloadAllModules)
    )
  ]
};
```

#### Custom Preloading Strategy

```typescript
// core/strategies/selective-preloading.strategy.ts
import { Injectable } from '@angular/core';
import { PreloadingStrategy, Route } from '@angular/router';
import { Observable, of } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class SelectivePreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return route.data?.['preload'] === true ? load() : of(null);
  }
}
```

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter, withPreloading } from '@angular/router';
import { SelectivePreloadingStrategy } from './core/strategies/selective-preloading.strategy';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withPreloading(SelectivePreloadingStrategy)
    )
  ]
};
```

```typescript
// app.routes.ts
export const routes: Routes = [
  // ...
  {
    path: 'dashboard',
    loadChildren: () => import('./features/dashboard/dashboard.routes')
      .then(m => m.DASHBOARD_ROUTES),
    data: { preload: true } // This route will be preloaded
  },
  {
    path: 'admin',
    loadChildren: () => import('./features/admin/admin.routes')
      .then(m => m.ADMIN_ROUTES),
    data: { preload: false } // This route will NOT be preloaded
  }
];
```

#### Optimizing Bundle Size

```typescript
// Use standalone components with granular imports
@Component({
  selector: 'app-feature',
  standalone: true,
  imports: [
    // Import only what you need
    NgIf,
    NgFor,
    RouterLink
  ],
  templateUrl: './feature.component.html'
})
export class FeatureComponent {}
```

### 3. API Communication

#### Using HttpClient with Signals and State Management

```typescript
// users.state.ts
import { state, select, action } from '@angular/core/state';
import { User } from '../models/user.model';

export interface UsersState {
  users: User[];
  selectedUserId: string | null;
  isLoading: boolean;
  error: string | null;
}

export const usersState = state<UsersState>({
  users: [],
  selectedUserId: null,
  isLoading: false,
  error: null
});

// Selectors
export const selectUsers = select(usersState, state => state.users);
export const selectIsLoading = select(usersState, state => state.isLoading);
export const selectError = select(usersState, state => state.error);
export const selectSelectedUserId = select(usersState, state => state.selectedUserId);
export const selectSelectedUser = select(usersState, state => {
  const userId = state.selectedUserId;
  return userId ? state.users.find(user => user.id === userId) : null;
});

// Actions
export const setUsers = action(usersState, 'setUsers', (state, users: User[]) => ({
  ...state,
  users,
  error: null
}));

export const setSelectedUserId = action(usersState, 'setSelectedUserId', (state, id: string | null) => ({
  ...state,
  selectedUserId: id
}));

export const setLoading = action(usersState, 'setLoading', (state, isLoading: boolean) => ({
  ...state,
  isLoading
}));

export const setError = action(usersState, 'setError', (state, error: string | null) => ({
  ...state,
  error,
  isLoading: false
}));

export const addUser = action(usersState, 'addUser', (state, user: User) => ({
  ...state,
  users: [...state.users, user]
}));

export const updateUser = action(usersState, 'updateUser', (state, updatedUser: User) => ({
  ...state,
  users: state.users.map(user => 
    user.id === updatedUser.id ? updatedUser : user
  )
}));

export const deleteUser = action(usersState, 'deleteUser', (state, userId: string) => ({
  ...state,
  users: state.users.filter(user => user.id !== userId),
  selectedUserId: state.selectedUserId === userId ? null : state.selectedUserId
}));
```

```typescript
// users.service.ts
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { catchError, tap } from 'rxjs/operators';
import { throwError } from 'rxjs';
import { User } from '../models/user.model';
import { 
  setUsers, 
  setLoading, 
  setError, 
  addUser, 
  updateUser, 
  deleteUser 
} from './users.state';

@Injectable({
  providedIn: 'root'
})
export class UsersService {
  private http = inject(HttpClient);
  private apiUrl = 'https://api.example.com/users';

  getUsers() {
    setLoading(true);

    return this.http.get<User[]>(this.apiUrl).pipe(
      tap(users => {
        setUsers(users);
        setLoading(false);
      }),
      catchError(error => {
        setError(error.message || 'Failed to load users');
        return throwError(() => error);
      })
    );
  }

  getUserById(id: string) {
    setLoading(true);

    return this.http.get<User>(`${this.apiUrl}/${id}`).pipe(
      tap(user => {
        updateUser(user);
        setLoading(false);
      }),
      catchError(error => {
        setError(error.message || `Failed to load user with ID ${id}`);
        return throwError(() => error);
      })
    );
  }

  createUser(user: Omit<User, 'id'>) {
    setLoading(true);

    return this.http.post<User>(this.apiUrl, user).pipe(
      tap(newUser => {
        addUser(newUser);
        setLoading(false);
      }),
      catchError(error => {
        setError(error.message || 'Failed to create user');
        return throwError(() => error);
      })
    );
  }

  updateUser(id: string, user: Partial<User>) {
    setLoading(true);

    return this.http.patch<User>(`${this.apiUrl}/${id}`, user).pipe(
      tap(updatedUser => {
        updateUser(updatedUser);
        setLoading(false);
      }),
      catchError(error => {
        setError(error.message || `Failed to update user with ID ${id}`);
        return throwError(() => error);
      })
    );
  }

  deleteUser(id: string) {
    setLoading(true);

    return this.http.delete<void>(`${this.apiUrl}/${id}`).pipe(
      tap(() => {
        deleteUser(id);
        setLoading(false);
      }),
      catchError(error => {
        setError(error.message || `Failed to delete user with ID ${id}`);
        return throwError(() => error);
      })
    );
  }
}
```

```typescript
// users-list.component.ts
import { Component, inject, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';
import { UsersService } from '../../services/users.service';
import { 
  selectUsers, 
  selectIsLoading, 
  selectError,
  setSelectedUserId
} from '../../state/users.state';

@Component({
  selector: 'app-users-list',
  standalone: true,
  imports: [CommonModule, RouterModule],
  template: `
    <div class="users-container">
      <h2>Users</h2>

      <button (click)="loadUsers()">Refresh</button>
      <button routerLink="/users/new">Add User</button>

      @if (isLoading()) {
        <div class="loading">Loading users...</div>
      } @else if (error()) {
        <div class="error">
          <p>{{ error() }}</p>
          <button (click)="loadUsers()">Try Again</button>
        </div>
      } @else {
        <ul class="users-list">
          @for (user of users(); track user.id) {
            <li>
              <a [routerLink]="['/users', user.id]" (click)="selectUser(user.id)">
                {{ user.name }}
              </a>
              <span class="email">{{ user.email }}</span>
              <div class="actions">
                <button [routerLink]="['/users', user.id, 'edit']">Edit</button>
                <button (click)="deleteUser(user.id)">Delete</button>
              </div>
            </li>
          } @empty {
            <li class="empty">No users found</li>
          }
        </ul>
      }
    </div>
  `
})
export class UsersListComponent implements OnInit {
  private usersService = inject(UsersService);

  // Inject selectors as signals
  users = inject(selectUsers);
  isLoading = inject(selectIsLoading);
  error = inject(selectError);

  ngOnInit() {
    this.loadUsers();
  }

  loadUsers() {
    this.usersService.getUsers().subscribe();
  }

  selectUser(id: string) {
    setSelectedUserId(id);
  }

  deleteUser(id: string) {
    if (confirm('Are you sure you want to delete this user?')) {
      this.usersService.deleteUser(id).subscribe();
    }
  }
}
```

### 4. Authentication and Authorization

```typescript
// auth.state.ts
import { state, select, action } from '@angular/core/state';

export interface User {
  id: string;
  email: string;
  name: string;
  roles: string[];
}

export interface AuthState {
  user: User | null;
  token: string | null;
  isLoading: boolean;
  error: string | null;
}

export const authState = state<AuthState>({
  user: null,
  token: null,
  isLoading: false,
  error: null
});

// Selectors
export const selectUser = select(authState, state => state.user);
export const selectToken = select(authState, state => state.token);
export const selectIsLoading = select(authState, state => state.isLoading);
export const selectError = select(authState, state => state.error);
export const selectIsAuthenticated = select(authState, state => !!state.token);
export const selectHasRole = (role: string) => 
  select(authState, state => state.user?.roles.includes(role) ?? false);

// Actions
export const setUser = action(authState, 'setUser', (state, user: User | null) => ({
  ...state,
  user
}));

export const setToken = action(authState, 'setToken', (state, token: string | null) => ({
  ...state,
  token
}));

export const setLoading = action(authState, 'setLoading', (state, isLoading: boolean) => ({
  ...state,
  isLoading
}));

export const setError = action(authState, 'setError', (state, error: string | null) => ({
  ...state,
  error,
  isLoading: false
}));

export const login = action(authState, 'login', (state, { user, token }: { user: User, token: string }) => ({
  ...state,
  user,
  token,
  error: null,
  isLoading: false
}));

export const logout = action(authState, 'logout', () => ({
  user: null,
  token: null,
  isLoading: false,
  error: null
}));
```

```typescript
// auth.service.ts
import { Injectable, inject, effect } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Router } from '@angular/router';
import { catchError, tap } from 'rxjs/operators';
import { throwError } from 'rxjs';
import { 
  User, 
  login, 
  logout, 
  setLoading, 
  setError,
  selectToken
} from './auth.state';

interface LoginResponse {
  user: User;
  token: string;
}

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private http = inject(HttpClient);
  private router = inject(Router);
  private apiUrl = 'https://api.example.com/auth';

  // Token signal from state
  private token = inject(selectToken);

  constructor() {
    // Load auth state from localStorage on startup
    this.loadAuthFromStorage();

    // Persist auth state to localStorage when token changes
    effect(() => {
      const currentToken = this.token();
      if (currentToken) {
        localStorage.setItem('auth_token', currentToken);
      } else {
        localStorage.removeItem('auth_token');
      }
    });
  }

  login(email: string, password: string) {
    setLoading(true);
    setError(null);

    return this.http.post<LoginResponse>(`${this.apiUrl}/login`, { email, password }).pipe(
      tap(response => {
        login(response);
        this.router.navigate(['/dashboard']);
      }),
      catchError(error => {
        setError(error.message || 'Login failed');
        return throwError(() => error);
      })
    );
  }

  logout() {
    logout();
    this.router.navigate(['/login']);
  }

  private loadAuthFromStorage() {
    const token = localStorage.getItem('auth_token');

    if (token) {
      // Validate token and get user info
      this.http.get<User>(`${this.apiUrl}/me`, {
        headers: {
          Authorization: `Bearer ${token}`
        }
      }).pipe(
        tap(user => {
          login({ user, token });
        }),
        catchError(() => {
          // Token invalid or expired
          logout();
          return throwError(() => new Error('Session expired'));
        })
      ).subscribe();
    }
  }
}
```

```typescript
// auth.guard.ts
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { selectIsAuthenticated, selectHasRole } from './auth.state';

export const authGuard = () => {
  const isAuthenticated = inject(selectIsAuthenticated);
  const router = inject(Router);

  if (isAuthenticated()) {
    return true;
  }

  return router.parseUrl('/login');
};

export const roleGuard = (requiredRole: string) => {
  const hasRole = inject(selectHasRole(requiredRole));
  const isAuthenticated = inject(selectIsAuthenticated);
  const router = inject(Router);

  if (isAuthenticated() && hasRole()) {
    return true;
  }

  if (isAuthenticated()) {
    return router.parseUrl('/unauthorized');
  }

  return router.parseUrl('/login');
};
```

```typescript
// app.routes.ts
import { Routes } from '@angular/router';
import { authGuard, roleGuard } from './core/guards/auth.guard';

export const routes: Routes = [
  {
    path: 'dashboard',
    loadComponent: () => import('./features/dashboard/dashboard.component')
      .then(m => m.DashboardComponent),
    canActivate: [authGuard]
  },
  {
    path: 'admin',
    loadChildren: () => import('./features/admin/admin.routes')
      .then(m => m.ADMIN_ROUTES),
    canActivate: [() => roleGuard('admin')]
  },
  {
    path: 'login',
    loadComponent: () => import('./features/auth/login/login.component')
      .then(m => m.LoginComponent)
  },
  {
    path: 'unauthorized',
    loadComponent: () => import('./features/auth/unauthorized/unauthorized.component')
      .then(m => m.UnauthorizedComponent)
  }
];
```

## Migration Guide from Angular 19 to Angular 20

### 1. Update Angular CLI

```bash
ng update @angular/cli @angular/core
```

### 2. Adopt the New State Management System

If you're using signals for state management, migrate to the new built-in state management system:

```typescript
// Before (Angular 19)
import { signal, computed } from '@angular/core';

export const count = signal(0);
export const doubled = computed(() => count() * 2);

export function increment() {
  count.update(value => value + 1);
}

// After (Angular 20)
import { state, select, action } from '@angular/core/state';

export interface CounterState {
  count: number;
}

export const counterState = state<CounterState>({
  count: 0
});

export const selectCount = select(counterState, state => state.count);
export const selectDoubled = select(counterState, state => state.count * 2);

export const increment = action(counterState, 'increment', (state) => ({
  ...state,
  count: state.count + 1
}));
```

### 3. Update Hydration Configuration

```typescript
// Before (Angular 19)
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideClientHydration } from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideClientHydration()
  ]
};

// After (Angular 20)
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideClientHydration, withAdvancedHydration, HydrationFeatures } from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideClientHydration(
      withAdvancedHydration({
        features: [HydrationFeatures.ZeroFlicker]
      })
    )
  ]
};
```

### 4. Adopt Web Components Enhancements

```typescript
// Before (Angular 19)
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { createCustomElement } from '@angular/elements';

@Component({
  selector: 'app-rating',
  standalone: true,
  template: `
    <div class="rating">
      @for (star of stars; track $index) {
        <span 
          class="star" 
          [class.filled]="star <= value"
          (click)="setValue(star)">
          ★
        </span>
      }
    </div>
  `
})
export class RatingComponent {
  @Input() value = 0;
  @Output() valueChange = new EventEmitter<number>();

  stars = [1, 2, 3, 4, 5];

  setValue(value: number) {
    this.value = value;
    this.valueChange.emit(value);
  }
}

// Register as custom element manually
const RatingElement = createCustomElement(RatingComponent, { injector });
customElements.define('rating-element', RatingElement);

// After (Angular 20)
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { withWebComponent } from '@angular/elements';

@Component({
  selector: 'app-rating',
  standalone: true,
  template: `
    <div class="rating" part="container">
      @for (star of stars; track $index) {
        <span 
          class="star" 
          part="star"
          [class.filled]="star <= value"
          (click)="setValue(star)">
          ★
        </span>
      }
    </div>
  `,
  providers: [
    withWebComponent({
      name: 'rating-element',
      cssProperties: ['--star-color', '--star-filled-color'],
      cssParts: ['container', 'star'],
      formAssociated: true
    })
  ]
})
export class RatingComponent {
  @Input() value = 0;
  @Output() valueChange = new EventEmitter<number>();

  stars = [1, 2, 3, 4, 5];

  setValue(value: number) {
    this.value = value;
    this.valueChange.emit(value);
  }
}
```

### 5. Update Dependency Injection

```typescript
// Before (Angular 19)
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  // Service implementation
}

// After (Angular 20)
import { injectable } from '@angular/core';

@injectable()
export class UserService {
  // Service implementation

  static scoped() {
    return {
      provide: UserService,
      useFactory: () => new UserService()
    };
  }
}
```

## Examples of Angular 20 Implementation Patterns

### 1. Feature Component with Built-in State Management

```typescript
// state/todo.state.ts
import { state, select, action } from '@angular/core/state';

export interface Todo {
  id: string;
  title: string;
  completed: boolean;
}

export interface TodoState {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
  isLoading: boolean;
  error: string | null;
}

export const todoState = state<TodoState>({
  todos: [],
  filter: 'all',
  isLoading: false,
  error: null
});

// Selectors
export const selectTodos = select(todoState, state => state.todos);
export const selectFilter = select(todoState, state => state.filter);
export const selectIsLoading = select(todoState, state => state.isLoading);
export const selectError = select(todoState, state => state.error);

export const selectFilteredTodos = select(todoState, state => {
  switch (state.filter) {
    case 'active':
      return state.todos.filter(todo => !todo.completed);
    case 'completed':
      return state.todos.filter(todo => todo.completed);
    default:
      return state.todos;
  }
});

export const selectTodoStats = select(todoState, state => {
  const total = state.todos.length;
  const completed = state.todos.filter(todo => todo.completed).length;
  const active = total - completed;
  return { total, completed, active };
});

// Actions
export const setTodos = action(todoState, 'setTodos', (state, todos: Todo[]) => ({
  ...state,
  todos,
  error: null
}));

export const setFilter = action(todoState, 'setFilter', (state, filter: 'all' | 'active' | 'completed') => ({
  ...state,
  filter
}));

export const setLoading = action(todoState, 'setLoading', (state, isLoading: boolean) => ({
  ...state,
  isLoading
}));

export const setError = action(todoState, 'setError', (state, error: string | null) => ({
  ...state,
  error,
  isLoading: false
}));

export const addTodo = action(todoState, 'addTodo', (state, title: string) => {
  const newTodo: Todo = {
    id: Date.now().toString(),
    title,
    completed: false
  };

  return {
    ...state,
    todos: [...state.todos, newTodo]
  };
});

export const toggleTodo = action(todoState, 'toggleTodo', (state, id: string) => ({
  ...state,
  todos: state.todos.map(todo => 
    todo.id === id ? { ...todo, completed: !todo.completed } : todo
  )
}));

export const removeTodo = action(todoState, 'removeTodo', (state, id: string) => ({
  ...state,
  todos: state.todos.filter(todo => todo.id !== id)
}));

export const clearCompleted = action(todoState, 'clearCompleted', (state) => ({
  ...state,
  todos: state.todos.filter(todo => !todo.completed)
}));
```

```typescript
// todo-app.component.ts
import { Component, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import {
  selectFilteredTodos,
  selectFilter,
  selectTodoStats,
  selectIsLoading,
  selectError,
  setFilter,
  addTodo,
  toggleTodo,
  removeTodo,
  clearCompleted,
  setTodos,
  setLoading,
  setError
} from '../state/todo.state';

@Component({
  selector: 'app-todo',
  standalone: true,
  imports: [CommonModule, FormsModule],
  template: `
    <div class="todo-app">
      <h1>Todo App</h1>

      @if (isLoading()) {
        <div class="loading">Loading todos...</div>
      }

      @if (error()) {
        <div class="error">
          <p>{{ error() }}</p>
          <button (click)="loadTodos()">Try Again</button>
        </div>
      }

      <div class="add-todo">
        <input 
          type="text" 
          placeholder="What needs to be done?" 
          [(ngModel)]="newTodoTitle"
          (keyup.enter)="addNewTodo()">
        <button (click)="addNewTodo()">Add</button>
      </div>

      <div class="filters">
        <button 
          [class.active]="filter() === 'all'"
          (click)="setFilter('all')">
          All
        </button>
        <button 
          [class.active]="filter() === 'active'"
          (click)="setFilter('active')">
          Active ({{ stats().active }})
        </button>
        <button 
          [class.active]="filter() === 'completed'"
          (click)="setFilter('completed')">
          Completed ({{ stats().completed }})
        </button>
      </div>

      <ul class="todo-list">
        @for (todo of filteredTodos(); track todo.id) {
          <li [class.completed]="todo.completed">
            <input 
              type="checkbox" 
              [checked]="todo.completed"
              (change)="toggleTodo(todo.id)">
            <span class="title">{{ todo.title }}</span>
            <button class="delete" (click)="removeTodo(todo.id)">×</button>
          </li>
        } @empty {
          <li class="empty">No todos found</li>
        }
      </ul>

      <div class="actions">
        <span>{{ stats().active }} items left</span>
        <button 
          (click)="clearCompleted()"
          [disabled]="stats().completed === 0">
          Clear completed
        </button>
      </div>
    </div>
  `
})
export class TodoAppComponent {
  // Inject selectors as signals
  filteredTodos = inject(selectFilteredTodos);
  filter = inject(selectFilter);
  stats = inject(selectTodoStats);
  isLoading = inject(selectIsLoading);
  error = inject(selectError);

  // Inject actions
  setFilter = inject(setFilter);
  addTodo = inject(addTodo);
  toggleTodo = inject(toggleTodo);
  removeTodo = inject(removeTodo);
  clearCompleted = inject(clearCompleted);

  newTodoTitle = '';

  constructor() {
    this.loadTodos();
  }

  loadTodos() {
    setLoading(true);
    setError(null);

    // Simulate API call
    setTimeout(() => {
      try {
        const savedTodos = localStorage.getItem('todos');
        if (savedTodos) {
          setTodos(JSON.parse(savedTodos));
        } else {
          setTodos([]);
        }
      } catch (err) {
        setError('Failed to load todos');
      } finally {
        setLoading(false);
      }
    }, 500);
  }

  addNewTodo() {
    if (this.newTodoTitle.trim()) {
      addTodo(this.newTodoTitle.trim());
      this.newTodoTitle = '';
      this.saveTodos();
    }
  }

  saveTodos() {
    // Save todos to localStorage whenever they change
    localStorage.setItem('todos', JSON.stringify(selectFilteredTodos()));
  }
}
```

### 2. Micro-Frontend Architecture

```typescript
// Shell Application (app.config.ts)
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideMicroFrontends, MicroFrontendConfig } from '@angular/core/micro-frontends';

import { routes } from './app.routes';

const microFrontends: MicroFrontendConfig[] = [
  {
    name: 'dashboard',
    remoteUrl: 'http://localhost:3001/remoteEntry.js',
    exposedModule: './Dashboard',
    sharedDependencies: ['@angular/core', '@angular/common', '@angular/router']
  },
  {
    name: 'user-profile',
    remoteUrl: 'http://localhost:3002/remoteEntry.js',
    exposedModule: './UserProfile',
    sharedDependencies: ['@angular/core', '@angular/common', '@angular/router']
  },
  {
    name: 'notifications',
    remoteUrl: 'http://localhost:3003/remoteEntry.js',
    exposedModule: './Notifications',
    sharedDependencies: ['@angular/core', '@angular/common']
  }
];

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideMicroFrontends(microFrontends)
  ]
};
```

```typescript
// Shell Application (app.component.ts)
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { MicroFrontendLoader } from '@angular/core/micro-frontends';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, MicroFrontendLoader],
  template: `
    <header class="main-header">
      <h1>My Application</h1>
      <nav>
        <a routerLink="/">Home</a>
        <a routerLink="/dashboard">Dashboard</a>
        <a routerLink="/profile">Profile</a>
      </nav>

      <!-- Notifications micro-frontend loaded directly -->
      <micro-frontend name="notifications"></micro-frontend>
    </header>

    <main>
      <router-outlet></router-outlet>
    </main>

    <footer>
      <p>© 2025 My Application</p>
    </footer>
  `
})
export class AppComponent {}
```

```typescript
// Dashboard Micro-Frontend (remote application)
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-dashboard',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="dashboard">
      <h2>Dashboard</h2>
      <div class="widgets">
        <div class="widget">
          <h3>Sales</h3>
          <p class="value">$12,345</p>
          <p class="trend up">+12% from last month</p>
        </div>
        <div class="widget">
          <h3>Users</h3>
          <p class="value">1,234</p>
          <p class="trend up">+5% from last month</p>
        </div>
        <div class="widget">
          <h3>Conversion</h3>
          <p class="value">3.2%</p>
          <p class="trend down">-0.5% from last month</p>
        </div>
      </div>
    </div>
  `
})
export class DashboardComponent {}

// Export the component for module federation
export default DashboardComponent;
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
    require('@tailwindcss/typography'),
  ],
}
```

Update `styles.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer components {
  .btn-primary {
    @apply bg-primary-600 text-white px-4 py-2 rounded-md hover:bg-primary-700 focus:outline-none focus:ring-2 focus:ring-primary-500 focus:ring-offset-2;
  }

  .form-input {
    @apply mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-primary-500 focus:ring-primary-500;
  }
}
```

### 2. Firebase Integration

```bash
# Install Firebase
npm install firebase @angular/fire
```

Configure in `app.config.ts`:

```typescript
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideFirebaseApp, initializeApp } from '@angular/fire/app';
import { provideAuth, getAuth } from '@angular/fire/auth';
import { provideFirestore, getFirestore } from '@angular/fire/firestore';
import { provideFunctions, getFunctions } from '@angular/fire/functions';
import { provideStorage, getStorage } from '@angular/fire/storage';
import { provideAnalytics, getAnalytics } from '@angular/fire/analytics';

import { routes } from './app.routes';

const firebaseConfig = {
  apiKey: "your-api-key",
  authDomain: "your-auth-domain",
  projectId: "your-project-id",
  storageBucket: "your-storage-bucket",
  messagingSenderId: "your-messaging-sender-id",
  appId: "your-app-id",
  measurementId: "your-measurement-id"
};

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    importProvidersFrom(
      provideFirebaseApp(() => initializeApp(firebaseConfig)),
      provideAuth(() => getAuth()),
      provideFirestore(() => getFirestore()),
      provideFunctions(() => getFunctions()),
      provideStorage(() => getStorage()),
      provideAnalytics(() => getAnalytics())
    )
  ]
};
```

### 3. GraphQL with Apollo Client

```bash
# Install Apollo Client
npm install @apollo/client graphql apollo-angular
```

Configure in `app.config.ts`:

```typescript
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { APOLLO_OPTIONS, ApolloModule } from 'apollo-angular';
import { HttpLink } from 'apollo-angular/http';
import { InMemoryCache, ApolloLink } from '@apollo/client/core';
import { setContext } from '@apollo/client/link/context';

import { routes } from './app.routes';

export function createApollo(httpLink: HttpLink) {
  const http = httpLink.create({ uri: 'https://your-graphql-endpoint' });

  const auth = setContext((_, { headers }) => {
    // Get the authentication token from local storage
    const token = localStorage.getItem('auth_token');

    return {
      headers: {
        ...headers,
        authorization: token ? `Bearer ${token}` : '',
      }
    };
  });

  return {
    link: ApolloLink.from([auth, http]),
    cache: new InMemoryCache(),
    defaultOptions: {
      watchQuery: {
        fetchPolicy: 'cache-and-network',
        errorPolicy: 'all'
      },
      query: {
        fetchPolicy: 'network-only',
        errorPolicy: 'all'
      },
      mutate: {
        errorPolicy: 'all'
      }
    }
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

Angular 20 represents a significant evolution in the Angular framework, building upon the foundations established in Angular 19 with new features and improvements that enhance developer productivity, application performance, and user experience.

The introduction of a built-in state management system based on signals provides a powerful yet simple way to manage application state without third-party libraries. Enhanced hydration capabilities ensure smooth user experiences with zero-flicker page transitions. Native support for micro-frontend architecture enables teams to work independently on different parts of large applications.

The streamlined dependency injection system and improved web components integration make Angular 20 more flexible and interoperable with other technologies. These improvements, combined with the continued focus on performance and developer experience, make Angular 20 a compelling choice for building modern web applications.

By following the guidelines and best practices outlined in this document, you can effectively leverage Angular 20's capabilities to create robust, maintainable, and high-performing applications that meet the demands of today's web ecosystem.

As Angular continues to evolve, staying up-to-date with the latest features and best practices will ensure that your applications remain modern, efficient, and aligned with the framework's recommended patterns.
