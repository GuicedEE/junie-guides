# Angular 20 Web Components in Micro Frontend Architecture

## Introduction

This guide provides comprehensive information on how to integrate three powerful modern web development approaches: Angular 20, Web Components, and Micro Frontend Architecture. By combining these technologies, developers can create modular, maintainable, and scalable applications that leverage the strengths of each approach.

Angular 20's enhanced support for Web Components and Micro Frontends makes it an ideal framework for building complex, distributed frontend applications. This guide will help you understand how these technologies work together and provide practical implementation strategies.

## Understanding the Three Technologies

### Angular 20

Angular 20 is a platform and framework for building single-page client applications using HTML and TypeScript. Key features relevant to this integration include:

- **Native Micro-Frontend Support**: Built-in tooling for module federation and micro-frontend architecture
- **Enhanced Web Components Integration**: First-class support for creating and consuming Web Components
- **Reactive State Management**: Built-in state management solution based on signals
- **Improved Hydration**: Zero-flicker hydration with intelligent state reconciliation

### Web Components

Web Components are a set of standardized web platform APIs that allow you to create custom, reusable, and encapsulated HTML elements:

- **Framework-Agnostic**: Work in any JavaScript framework or vanilla HTML
- **Shadow DOM**: Provides style and DOM encapsulation
- **Custom Elements**: Defines new HTML elements with custom behavior
- **HTML Templates**: Reusable HTML structures

### Micro Frontend Architecture

Micro Frontend Architecture extends the concepts of microservices to frontend development:

- **Independent Teams**: End-to-end ownership of features by autonomous teams
- **Technology Agnosticism**: Freedom for teams to choose their technology stack
- **Isolation and Resilience**: Issues in one micro frontend should not affect others
- **Independent Deployment**: Teams can deploy their work without coordinating with other teams

## Theoretical Foundation: Why Combine These Technologies?

### Complementary Strengths

1. **Angular 20** provides a robust framework with powerful features for building complex applications
2. **Web Components** offer framework-agnostic reusability and encapsulation
3. **Micro Frontends** enable team autonomy and independent deployment

### Benefits of Integration

1. **Scalable Team Organization**:
   - Different teams can work on different micro frontends
   - Teams can use Angular 20 for complex features while exposing functionality as Web Components

2. **Technical Flexibility**:
   - Angular 20 components can be exposed as Web Components for use in any framework
   - Legacy applications can be gradually modernized by incorporating Angular 20 micro frontends

3. **Enhanced Maintainability**:
   - Clear boundaries between micro frontends reduce complexity
   - Web Components provide a standardized interface between different parts of the application

4. **Future-Proofing**:
   - Web Components are a W3C standard with long-term browser support
   - Micro frontends can be individually upgraded or replaced without affecting the entire application

## Architecture Patterns

### 1. Shell Application with Angular 20 Micro Frontends

In this pattern, a shell application (container) loads and orchestrates multiple Angular 20 micro frontends:

```
┌─────────────────────────────────────────────────────────┐
│                   Shell Application                      │
│                                                         │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────┐ │
│  │  Micro Frontend │  │  Micro Frontend │  │   Micro  │ │
│  │      Team A     │  │      Team B     │  │ Frontend │ │
│  │    (Angular 20) │  │    (Angular 20) │  │  Team C  │ │
│  └─────────────────┘  └─────────────────┘  └──────────┘ │
└─────────────────────────────────────────────────────────┘
```

### 2. Web Components as Integration Points

Each micro frontend exposes its functionality as Web Components, allowing for framework-agnostic integration:

```
┌─────────────────────────────────────────────────────────┐
│                   Shell Application                      │
│                                                         │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────┐ │
│  │ <team-a-widget> │  │ <team-b-widget> │  │ <team-c- │ │
│  │                 │  │                 │  │  widget>  │ │
│  └─────────────────┘  └─────────────────┘  └──────────┘ │
└─────────────────────────────────────────────────────────┘
     ▲                    ▲                    ▲
     │                    │                    │
┌────┴────┐         ┌────┴────┐         ┌─────┴────┐
│ Angular │         │ Angular │         │  React   │
│ Team A  │         │ Team B  │         │  Team C  │
└─────────┘         └─────────┘         └──────────┘
```

### 3. Domain-Driven Decomposition

Align micro frontends with business domains, following DDD principles:

```
┌─────────────────────────────────────────────────────────┐
│                   Shell Application                      │
│                                                         │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────┐ │
│  │                 │  │                 │  │          │ │
│  │  Product Domain │  │  Order Domain   │  │  User    │ │
│  │                 │  │                 │  │  Domain  │ │
│  └─────────────────┘  └─────────────────┘  └──────────┘ │
└─────────────────────────────────────────────────────────┘
```

## Implementation Guide

### 1. Setting Up the Shell Application

First, create an Angular 20 application that will serve as the shell:

```bash
# Install the latest Angular CLI
npm install -g @angular/cli

# Create a new Angular 20 project
ng new micro-frontend-shell --standalone

# Navigate to the project directory
cd micro-frontend-shell
```

Configure the shell application to load micro frontends:

```typescript
// src/app/app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { routes } from './app.routes';
import { provideWebComponentLoader } from '@angular/elements';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    // Add support for loading Web Components
    provideWebComponentLoader()
  ]
};
```

### 2. Creating Micro Frontends as Angular 20 Applications

For each micro frontend, create a separate Angular 20 application:

```bash
# Create a new Angular 20 project for the micro frontend
ng new product-micro-frontend --standalone

# Navigate to the project directory
cd product-micro-frontend
```

Configure the micro frontend to be exposed as a Web Component:

```typescript
// src/app/product-widget/product-widget.component.ts
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { withWebComponent } from '@angular/elements';

@Component({
  selector: 'app-product-widget',
  standalone: true,
  template: `
    <div class="product-widget" part="container">
      <h2>{{ title }}</h2>
      <div class="product-list">
        @for (product of products; track product.id) {
          <div class="product-item" part="product-item">
            <h3>{{ product.name }}</h3>
            <p>{{ product.description }}</p>
            <button (click)="selectProduct(product)">View Details</button>
          </div>
        }
      </div>
    </div>
  `,
  styles: [`
    :host {
      display: block;
      font-family: Arial, sans-serif;
    }
    .product-widget {
      border: 1px solid #ddd;
      border-radius: 4px;
      padding: 16px;
    }
    .product-item {
      margin-bottom: 16px;
      padding: 12px;
      border: 1px solid #eee;
      border-radius: 4px;
    }
    button {
      background-color: var(--primary-color, #3498db);
      color: white;
      border: none;
      padding: 8px 12px;
      border-radius: 4px;
      cursor: pointer;
    }
  `],
  providers: [
    withWebComponent({
      name: 'product-widget',
      cssProperties: ['--primary-color', '--text-color'],
      cssParts: ['container', 'product-item'],
      formAssociated: false
    })
  ]
})
export class ProductWidgetComponent {
  @Input() title = 'Products';
  @Input() products: any[] = [];
  @Output() productSelected = new EventEmitter<any>();

  selectProduct(product: any) {
    this.productSelected.emit(product);
  }
}
```

### 3. Building and Exposing Micro Frontends as Web Components

Configure the build process for the micro frontend:

```json
{
  "projects": {
    "product-micro-frontend": {
      "architect": {
        "build": {
          "configurations": {
            "production": {
              "outputHashing": "all",
              "optimization": true
            },
            "webcomponent": {
              "outputHashing": "none",
              "optimization": true,
              "outputPath": "dist/product-widget",
              "main": "src/main.webcomponent.ts"
            }
          }
        }
      }
    }
  }
}
```

Create a separate entry point for the Web Component:

```typescript
// src/main.webcomponent.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { ProductWidgetComponent } from './app/product-widget/product-widget.component';

bootstrapApplication(ProductWidgetComponent)
  .catch(err => console.error(err));
```

Build the Web Component:

```bash
ng build --configuration=webcomponent
```

### 4. Integrating Micro Frontends in the Shell Application

Load and use the micro frontends in the shell application:

```typescript
// src/app/app.component.ts
import { Component, inject, OnInit } from '@angular/core';
import { WebComponentLoader } from '@angular/elements';

@Component({
  selector: 'app-root',
  standalone: true,
  template: `
    <div class="shell-container">
      <header>
        <h1>Micro Frontend Demo</h1>
        <nav>
          <button (click)="activateTab('products')">Products</button>
          <button (click)="activateTab('orders')">Orders</button>
          <button (click)="activateTab('users')">Users</button>
        </nav>
      </header>

      <main>
        @if (activeTab === 'products') {
          <product-widget 
            title="Featured Products" 
            [products]="products"
            (productSelected)="handleProductSelected($event)">
          </product-widget>
        }

        @if (activeTab === 'orders') {
          <order-widget 
            [userId]="currentUserId">
          </order-widget>
        }

        @if (activeTab === 'users') {
          <user-profile-widget 
            [userId]="currentUserId"
            (userUpdated)="handleUserUpdated($event)">
          </user-profile-widget>
        }
      </main>
    </div>
  `,
  styles: [`
    .shell-container {
      max-width: 1200px;
      margin: 0 auto;
      padding: 20px;
    }
    header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
      padding-bottom: 20px;
      border-bottom: 1px solid #eee;
    }
    nav button {
      margin-left: 10px;
      padding: 8px 16px;
      background-color: #f5f5f5;
      border: 1px solid #ddd;
      border-radius: 4px;
      cursor: pointer;
    }
    nav button:hover {
      background-color: #e5e5e5;
    }
    main {
      min-height: 500px;
    }
  `]
})
export class AppComponent implements OnInit {
  private webComponentLoader = inject(WebComponentLoader);

  activeTab = 'products';
  currentUserId = 'user123';
  products = [
    { id: 1, name: 'Product 1', description: 'Description for product 1' },
    { id: 2, name: 'Product 2', description: 'Description for product 2' },
    { id: 3, name: 'Product 3', description: 'Description for product 3' }
  ];

  async ngOnInit() {
    // Load the Web Components
    await this.webComponentLoader.loadWebComponent(
      'product-widget', 
      'https://cdn.example.com/product-widget/main.js'
    );

    await this.webComponentLoader.loadWebComponent(
      'order-widget', 
      'https://cdn.example.com/order-widget/main.js'
    );

    await this.webComponentLoader.loadWebComponent(
      'user-profile-widget', 
      'https://cdn.example.com/user-profile-widget/main.js'
    );
  }

  activateTab(tab: string) {
    this.activeTab = tab;
  }

  handleProductSelected(product: any) {
    console.log('Product selected:', product);
    // Handle product selection
  }

  handleUserUpdated(user: any) {
    console.log('User updated:', user);
    this.currentUserId = user.id;
    // Handle user update
  }
}
```

### 5. Implementing Cross-Micro Frontend Communication

Create a shared event bus for communication between micro frontends:

```typescript
// src/app/shared/event-bus.service.ts
import { Injectable } from '@angular/core';
import { Subject, Observable } from 'rxjs';
import { filter, map } from 'rxjs/operators';

export interface EventBusEvent {
  type: string;
  payload?: any;
}

@Injectable({
  providedIn: 'root'
})
export class EventBusService {
  private eventSubject = new Subject<EventBusEvent>();

  publish(event: EventBusEvent): void {
    this.eventSubject.next(event);
  }

  on<T>(eventType: string): Observable<T> {
    return this.eventSubject.pipe(
      filter(event => event.type === eventType),
      map(event => event.payload)
    );
  }
}
```

Use the event bus in micro frontends:

```typescript
// In the product micro frontend
import { inject } from '@angular/core';
import { EventBusService } from '../shared/event-bus.service';

export class ProductWidgetComponent {
  private eventBus = inject(EventBusService);

  selectProduct(product: any) {
    this.productSelected.emit(product);

    // Also publish to the event bus for other micro frontends
    this.eventBus.publish({
      type: 'PRODUCT_SELECTED',
      payload: product
    });
  }
}

// In the order micro frontend
import { inject, OnInit, OnDestroy } from '@angular/core';
import { EventBusService } from '../shared/event-bus.service';
import { Subscription } from 'rxjs';

export class OrderWidgetComponent implements OnInit, OnDestroy {
  private eventBus = inject(EventBusService);
  private subscription: Subscription;

  ngOnInit() {
    // Listen for product selection events
    this.subscription = this.eventBus.on<any>('PRODUCT_SELECTED')
      .subscribe(product => {
        console.log('Order widget received product selection:', product);
        // Handle the product selection
      });
  }

  ngOnDestroy() {
    this.subscription.unsubscribe();
  }
}
```

## Domain-Driven Design Integration

Apply DDD principles to your micro frontend architecture:

### 1. Bounded Contexts as Micro Frontends

Each micro frontend represents a bounded context with:

- Its own domain model
- Clear integration points with other contexts
- Well-defined boundaries that reflect business capabilities

```typescript
// product-domain/models/product.model.ts
export interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  category: string;
  // Product-specific properties
}

// order-domain/models/order.model.ts
export interface Order {
  id: string;
  customerId: string;
  items: OrderItem[];
  status: OrderStatus;
  // Order-specific properties
}

export interface OrderItem {
  productId: string;
  productName: string; // Duplicated from Product for independence
  quantity: number;
  price: number; // Price at time of order
}

export enum OrderStatus {
  PENDING = 'PENDING',
  PROCESSING = 'PROCESSING',
  SHIPPED = 'SHIPPED',
  DELIVERED = 'DELIVERED',
  CANCELLED = 'CANCELLED'
}
```

### 2. Context Mapping with Anti-Corruption Layers

Implement anti-corruption layers to translate between different domain models:

```typescript
// order-domain/services/product-acl.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { map } from 'rxjs/operators';
import { OrderItem } from '../models/order.model';

// This service translates between the Product domain model and the Order domain model
@Injectable({
  providedIn: 'root'
})
export class ProductAntiCorruptionLayerService {
  constructor(private http: HttpClient) {}

  // Fetch product from Product domain and translate to Order domain
  getProductForOrder(productId: string) {
    return this.http.get<any>(`/api/products/${productId}`)
      .pipe(
        map(productDto => this.translateToOrderItem(productDto))
      );
  }

  private translateToOrderItem(productDto: any): OrderItem {
    return {
      productId: productDto.id,
      productName: productDto.name,
      quantity: 1, // Default quantity
      price: productDto.price
    };
  }
}
```

### 3. Domain Events for Communication

Use domain events for communication between bounded contexts:

```typescript
// shared/domain-events/product-events.ts
export interface ProductAddedToCartEvent {
  type: 'PRODUCT_ADDED_TO_CART';
  payload: {
    productId: string;
    quantity: number;
    userId: string;
  };
}

// shared/domain-events/order-events.ts
export interface OrderPlacedEvent {
  type: 'ORDER_PLACED';
  payload: {
    orderId: string;
    customerId: string;
    totalAmount: number;
  };
}

// product-domain/components/product-detail.component.ts
import { EventBusService } from '../../shared/event-bus.service';

export class ProductDetailComponent {
  constructor(private eventBus: EventBusService) {}

  addToCart(product: any, quantity: number) {
    // Local logic

    // Publish domain event
    this.eventBus.publish({
      type: 'PRODUCT_ADDED_TO_CART',
      payload: {
        productId: product.id,
        quantity,
        userId: this.currentUserId
      }
    });
  }
}
```

## Best Practices and Considerations

### 1. Design System Integration

Create a shared design system as Web Components:

```typescript
// design-system/components/ds-button.component.ts
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { withWebComponent } from '@angular/elements';

@Component({
  selector: 'ds-button',
  standalone: true,
  template: `
    <button 
      [class]="variant" 
      [disabled]="disabled"
      (click)="handleClick()">
      <ng-content></ng-content>
    </button>
  `,
  styles: [`
    :host {
      display: inline-block;
    }
    button {
      padding: 8px 16px;
      border-radius: 4px;
      font-family: var(--font-family, 'Arial, sans-serif');
      font-size: var(--font-size, 14px);
      cursor: pointer;
      border: none;
    }
    button.primary {
      background-color: var(--primary-color, #3498db);
      color: white;
    }
    button.secondary {
      background-color: var(--secondary-color, #f1f1f1);
      color: #333;
    }
    button:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }
  `],
  providers: [
    withWebComponent({
      name: 'ds-button',
      cssProperties: [
        '--primary-color', 
        '--secondary-color', 
        '--font-family',
        '--font-size'
      ],
      formAssociated: false
    })
  ]
})
export class DesignSystemButtonComponent {
  @Input() variant: 'primary' | 'secondary' = 'primary';
  @Input() disabled = false;
  @Output() clicked = new EventEmitter<void>();

  handleClick() {
    this.clicked.emit();
  }
}
```

### 2. Performance Optimization

Implement lazy loading for micro frontends:

```typescript
// src/app/app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: 'products',
    loadComponent: () => import('./micro-frontends/product/product-container.component')
      .then(m => m.ProductContainerComponent)
  },
  {
    path: 'orders',
    loadComponent: () => import('./micro-frontends/order/order-container.component')
      .then(m => m.OrderContainerComponent)
  },
  {
    path: 'users',
    loadComponent: () => import('./micro-frontends/user/user-container.component')
      .then(m => m.UserContainerComponent)
  }
];
```

### 3. Versioning Strategy

Implement a versioning strategy for micro frontends:

```typescript
// src/app/app.component.ts
import { WebComponentLoader } from '@angular/elements';

export class AppComponent implements OnInit {
  private webComponentLoader = inject(WebComponentLoader);

  async ngOnInit() {
    // Load specific versions of Web Components
    await this.webComponentLoader.loadWebComponent(
      'product-widget', 
      'https://cdn.example.com/product-widget/v1.2.3/main.js'
    );

    await this.webComponentLoader.loadWebComponent(
      'order-widget', 
      'https://cdn.example.com/order-widget/v2.0.1/main.js'
    );
  }
}
```

### 4. Authentication and Authorization

Implement shared authentication across micro frontends:

```typescript
// src/app/shared/auth.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

export interface User {
  id: string;
  name: string;
  email: string;
  roles: string[];
}

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private currentUserSubject = new BehaviorSubject<User | null>(null);

  get currentUser$(): Observable<User | null> {
    return this.currentUserSubject.asObservable();
  }

  get currentUser(): User | null {
    return this.currentUserSubject.value;
  }

  login(credentials: { username: string; password: string }) {
    // Authentication logic
    // ...

    // Update current user
    this.currentUserSubject.next({
      id: 'user123',
      name: 'John Doe',
      email: 'john@example.com',
      roles: ['user']
    });
  }

  logout() {
    this.currentUserSubject.next(null);
  }

  hasRole(role: string): boolean {
    return this.currentUser?.roles.includes(role) || false;
  }
}
```

## Example Application: E-Commerce Platform

Here's a complete example of an e-commerce platform using Angular 20, Web Components, and Micro Frontend Architecture:

### Shell Application

```typescript
// src/app/app.component.ts
import { Component, inject } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { WebComponentLoader } from '@angular/elements';
import { AuthService } from './shared/auth.service';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  template: `
    <div class="app-container">
      <header>
        <div class="logo">E-Commerce Platform</div>
        <nav>
          <a routerLink="/products">Products</a>
          <a routerLink="/cart">Cart</a>
          <a routerLink="/orders">Orders</a>
          <a routerLink="/profile">Profile</a>
        </nav>
        <div class="auth-controls">
          @if (authService.currentUser) {
            <span>Welcome, {{ authService.currentUser.name }}</span>
            <button (click)="authService.logout()">Logout</button>
          } @else {
            <button (click)="showLoginForm()">Login</button>
          }
        </div>
      </header>

      <main>
        <router-outlet></router-outlet>
      </main>

      <footer>
        <p>&copy; 2025 E-Commerce Platform</p>
      </footer>
    </div>
  `,
  styles: [/* styles here */]
})
export class AppComponent {
  authService = inject(AuthService);
  webComponentLoader = inject(WebComponentLoader);

  async ngOnInit() {
    // Load shared Web Components
    await this.webComponentLoader.loadWebComponent(
      'ds-button',
      'https://cdn.example.com/design-system/v1.0.0/ds-button.js'
    );

    await this.webComponentLoader.loadWebComponent(
      'ds-input',
      'https://cdn.example.com/design-system/v1.0.0/ds-input.js'
    );
  }

  showLoginForm() {
    // Show login form logic
  }
}
```

### Product Catalog Micro Frontend

```typescript
// product-catalog/product-list.component.ts
import { Component, inject } from '@angular/core';
import { withWebComponent } from '@angular/elements';
import { ProductService } from './services/product.service';
import { EventBusService } from '../shared/event-bus.service';

@Component({
  selector: 'app-product-list',
  standalone: true,
  template: `
    <div class="product-list-container">
      <h2>{{ title }}</h2>

      <div class="filters">
        <ds-input 
          placeholder="Search products"
          (valueChange)="searchProducts($event)">
        </ds-input>

        <select (change)="filterByCategory($event)">
          <option value="">All Categories</option>
          @for (category of categories; track category) {
            <option [value]="category">{{ category }}</option>
          }
        </select>
      </div>

      <div class="products-grid">
        @for (product of filteredProducts; track product.id) {
          <div class="product-card">
            <img [src]="product.imageUrl" [alt]="product.name">
            <h3>{{ product.name }}</h3>
            <p class="price">{{ '$' + product.price.toFixed(2) }}</p>
            <p class="description">{{ product.description }}</p>
            <ds-button 
              variant="primary"
              (clicked)="addToCart(product)">
              Add to Cart
            </ds-button>
          </div>
        }
      </div>
    </div>
  `,
  styles: [/* styles here */],
  providers: [
    withWebComponent({
      name: 'product-list',
      cssProperties: ['--card-background', '--card-border-radius'],
      cssParts: ['product-card', 'filters']
    })
  ]
})
export class ProductListComponent {
  private productService = inject(ProductService);
  private eventBus = inject(EventBusService);

  title = 'Product Catalog';
  products: any[] = [];
  filteredProducts: any[] = [];
  categories: string[] = [];

  ngOnInit() {
    this.loadProducts();
  }

  async loadProducts() {
    this.products = await this.productService.getProducts();
    this.filteredProducts = [...this.products];
    this.categories = [...new Set(this.products.map(p => p.category))];
  }

  searchProducts(term: string) {
    if (!term) {
      this.filteredProducts = [...this.products];
      return;
    }

    term = term.toLowerCase();
    this.filteredProducts = this.products.filter(product => 
      product.name.toLowerCase().includes(term) || 
      product.description.toLowerCase().includes(term)
    );
  }

  filterByCategory(event: Event) {
    const category = (event.target as HTMLSelectElement).value;

    if (!category) {
      this.filteredProducts = [...this.products];
      return;
    }

    this.filteredProducts = this.products.filter(product => 
      product.category === category
    );
  }

  addToCart(product: any) {
    // Publish domain event
    this.eventBus.publish({
      type: 'PRODUCT_ADDED_TO_CART',
      payload: {
        productId: product.id,
        name: product.name,
        price: product.price,
        quantity: 1
      }
    });
  }
}
```

### Shopping Cart Micro Frontend

```typescript
// shopping-cart/cart.component.ts
import { Component, inject, OnInit, OnDestroy } from '@angular/core';
import { withWebComponent } from '@angular/elements';
import { Subscription } from 'rxjs';
import { CartService } from './services/cart.service';
import { EventBusService } from '../shared/event-bus.service';

@Component({
  selector: 'app-cart',
  standalone: true,
  template: `
    <div class="cart-container">
      <h2>Shopping Cart</h2>

      @if (items.length === 0) {
        <p class="empty-cart">Your cart is empty</p>
      } @else {
        <div class="cart-items">
          @for (item of items; track item.productId) {
            <div class="cart-item">
              <div class="item-details">
                <h3>{{ item.name }}</h3>
                <p class="price">{{ '$' + item.price.toFixed(2) }}</p>
              </div>

              <div class="quantity-controls">
                <button (click)="decreaseQuantity(item)">-</button>
                <span>{{ item.quantity }}</span>
                <button (click)="increaseQuantity(item)">+</button>
              </div>

              <div class="item-total">
                {{ '$' + (item.price * item.quantity).toFixed(2) }}
              </div>

              <button class="remove-button" (click)="removeItem(item)">
                Remove
              </button>
            </div>
          }
        </div>

        <div class="cart-summary">
          <div class="subtotal">
            <span>Subtotal:</span>
            <span>{{ '$' + calculateSubtotal().toFixed(2) }}</span>
          </div>

          <div class="tax">
            <span>Tax:</span>
            <span>{{ '$' + calculateTax().toFixed(2) }}</span>
          </div>

          <div class="total">
            <span>Total:</span>
            <span>{{ '$' + calculateTotal().toFixed(2) }}</span>
          </div>

          <ds-button 
            variant="primary"
            (clicked)="checkout()">
            Proceed to Checkout
          </ds-button>
        </div>
      }
    </div>
  `,
  styles: [/* styles here */],
  providers: [
    withWebComponent({
      name: 'shopping-cart',
      cssProperties: ['--primary-color', '--secondary-color'],
      cssParts: ['cart-item', 'cart-summary']
    })
  ]
})
export class CartComponent implements OnInit, OnDestroy {
  private cartService = inject(CartService);
  private eventBus = inject(EventBusService);
  private subscription: Subscription;

  items: any[] = [];

  ngOnInit() {
    this.loadCartItems();

    // Listen for product added to cart events
    this.subscription = this.eventBus.on('PRODUCT_ADDED_TO_CART')
      .subscribe(product => {
        this.addToCart(product);
      });
  }

  ngOnDestroy() {
    this.subscription.unsubscribe();
  }

  async loadCartItems() {
    this.items = await this.cartService.getCartItems();
  }

  addToCart(product: any) {
    const existingItem = this.items.find(item => item.productId === product.productId);

    if (existingItem) {
      existingItem.quantity += product.quantity;
    } else {
      this.items.push(product);
    }

    this.cartService.updateCart(this.items);
  }

  increaseQuantity(item: any) {
    item.quantity += 1;
    this.cartService.updateCart(this.items);
  }

  decreaseQuantity(item: any) {
    if (item.quantity > 1) {
      item.quantity -= 1;
      this.cartService.updateCart(this.items);
    }
  }

  removeItem(item: any) {
    this.items = this.items.filter(i => i.productId !== item.productId);
    this.cartService.updateCart(this.items);
  }

  calculateSubtotal(): number {
    return this.items.reduce((total, item) => total + (item.price * item.quantity), 0);
  }

  calculateTax(): number {
    return this.calculateSubtotal() * 0.08; // 8% tax
  }

  calculateTotal(): number {
    return this.calculateSubtotal() + this.calculateTax();
  }

  checkout() {
    // Publish checkout event
    this.eventBus.publish({
      type: 'CHECKOUT_INITIATED',
      payload: {
        items: this.items,
        subtotal: this.calculateSubtotal(),
        tax: this.calculateTax(),
        total: this.calculateTotal()
      }
    });

    // Navigate to checkout
    window.location.href = '/checkout';
  }
}
```

## Testing Strategies

### 1. Unit Testing Web Components

```typescript
// product-list.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { ProductListComponent } from './product-list.component';
import { ProductService } from './services/product.service';
import { EventBusService } from '../shared/event-bus.service';

describe('ProductListComponent', () => {
  let component: ProductListComponent;
  let fixture: ComponentFixture<ProductListComponent>;
  let productServiceSpy: jasmine.SpyObj<ProductService>;
  let eventBusSpy: jasmine.SpyObj<EventBusService>;

  beforeEach(async () => {
    const productServiceMock = jasmine.createSpyObj('ProductService', ['getProducts']);
    const eventBusMock = jasmine.createSpyObj('EventBusService', ['publish']);

    await TestBed.configureTestingModule({
      imports: [ProductListComponent],
      providers: [
        { provide: ProductService, useValue: productServiceMock },
        { provide: EventBusService, useValue: eventBusMock }
      ]
    }).compileComponents();

    productServiceSpy = TestBed.inject(ProductService) as jasmine.SpyObj<ProductService>;
    eventBusSpy = TestBed.inject(EventBusService) as jasmine.SpyObj<EventBusService>;

    productServiceSpy.getProducts.and.returnValue(Promise.resolve([
      { id: '1', name: 'Product 1', price: 10, category: 'Category A', description: 'Description 1' },
      { id: '2', name: 'Product 2', price: 20, category: 'Category B', description: 'Description 2' }
    ]));

    fixture = TestBed.createComponent(ProductListComponent);
    component = fixture.componentInstance;
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should load products on init', async () => {
    await component.ngOnInit();
    expect(component.products.length).toBe(2);
    expect(component.filteredProducts.length).toBe(2);
    expect(component.categories).toEqual(['Category A', 'Category B']);
  });

  it('should filter products by search term', async () => {
    await component.ngOnInit();
    component.searchProducts('Product 1');
    expect(component.filteredProducts.length).toBe(1);
    expect(component.filteredProducts[0].name).toBe('Product 1');
  });

  it('should publish event when adding to cart', async () => {
    await component.ngOnInit();
    const product = component.products[0];
    component.addToCart(product);

    expect(eventBusSpy.publish).toHaveBeenCalledWith({
      type: 'PRODUCT_ADDED_TO_CART',
      payload: {
        productId: product.id,
        name: product.name,
        price: product.price,
        quantity: 1
      }
    });
  });
});
```

### 2. Integration Testing Micro Frontends

```typescript
// integration.spec.ts
import { TestBed } from '@angular/core/testing';
import { AppComponent } from './app.component';
import { EventBusService } from './shared/event-bus.service';
import { CartService } from './shopping-cart/services/cart.service';

describe('Micro Frontend Integration', () => {
  let eventBus: EventBusService;
  let cartService: jasmine.SpyObj<CartService>;

  beforeEach(async () => {
    const cartServiceMock = jasmine.createSpyObj('CartService', ['updateCart']);

    await TestBed.configureTestingModule({
      imports: [AppComponent],
      providers: [
        EventBusService,
        { provide: CartService, useValue: cartServiceMock }
      ]
    }).compileComponents();

    eventBus = TestBed.inject(EventBusService);
    cartService = TestBed.inject(CartService) as jasmine.SpyObj<CartService>;
  });

  it('should update cart when product added event is published', () => {
    // Simulate product added to cart event
    eventBus.publish({
      type: 'PRODUCT_ADDED_TO_CART',
      payload: {
        productId: '1',
        name: 'Test Product',
        price: 10,
        quantity: 1
      }
    });

    // Verify cart service was called
    expect(cartService.updateCart).toHaveBeenCalled();
  });
});
```

### 3. End-to-End Testing

```typescript
// e2e/app.e2e-spec.ts
import { browser, element, by } from 'protractor';

describe('E-Commerce App', () => {
  beforeEach(async () => {
    await browser.get('/');
  });

  it('should display product list', async () => {
    await browser.get('/products');
    const productCards = element.all(by.css('.product-card'));
    expect(await productCards.count()).toBeGreaterThan(0);
  });

  it('should add product to cart', async () => {
    await browser.get('/products');

    // Click add to cart button on first product
    const addToCartButton = element.all(by.css('.product-card ds-button')).first();
    await addToCartButton.click();

    // Navigate to cart
    await browser.get('/cart');

    // Verify product was added
    const cartItems = element.all(by.css('.cart-item'));
    expect(await cartItems.count()).toBe(1);
  });
});
```

## Deployment Strategies

### 1. Independent Deployment

Deploy each micro frontend independently:

```bash
# Deploy product catalog micro frontend
cd product-catalog
ng build --configuration=production
aws s3 sync dist/product-catalog s3://your-bucket/product-catalog/v1.0.0/

# Deploy shopping cart micro frontend
cd shopping-cart
ng build --configuration=production
aws s3 sync dist/shopping-cart s3://your-bucket/shopping-cart/v1.0.0/

# Deploy shell application
cd shell
ng build --configuration=production
aws s3 sync dist/shell s3://your-bucket/shell/
```

### 2. Continuous Integration/Continuous Deployment

Set up CI/CD pipelines for each micro frontend:

```yaml
# .github/workflows/product-catalog.yml
name: Product Catalog CI/CD

on:
  push:
    branches: [ main ]
    paths:
      - 'product-catalog/**'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Setup Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '18'

    - name: Install dependencies
      run: |
        cd product-catalog
        npm ci

    - name: Run tests
      run: |
        cd product-catalog
        npm test -- --no-watch --no-progress --browsers=ChromeHeadless

    - name: Build
      run: |
        cd product-catalog
        npm run build -- --configuration=production

    - name: Deploy
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Upload to S3
      run: |
        aws s3 sync product-catalog/dist/product-catalog s3://your-bucket/product-catalog/v1.0.0/
```

### 3. Versioning and Rollback

Implement a version management system:

```typescript
// src/app/version-manager.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

interface VersionConfig {
  productCatalog: string;
  shoppingCart: string;
  checkout: string;
  userProfile: string;
}

@Injectable({
  providedIn: 'root'
})
export class VersionManagerService {
  private config: VersionConfig;

  constructor(private http: HttpClient) {}

  async loadVersionConfig(): Promise<VersionConfig> {
    if (!this.config) {
      this.config = await this.http.get<VersionConfig>('/assets/versions.json').toPromise();
    }
    return this.config;
  }

  async getMicroFrontendUrl(name: string): Promise<string> {
    const config = await this.loadVersionConfig();
    const version = config[name as keyof VersionConfig];
    return `https://cdn.example.com/${name}/${version}/main.js`;
  }
}
```

## Troubleshooting Common Issues

### 1. Communication Between Micro Frontends

**Problem**: Micro frontends cannot communicate with each other.

**Solution**: Ensure the event bus is properly shared:

```typescript
// Make sure the EventBusService is provided at the root level
// src/app/app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { EventBusService } from './shared/event-bus.service';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    // Provide EventBusService at the root level
    { provide: EventBusService, useClass: EventBusService }
  ]
};
```

### 2. Styling Conflicts

**Problem**: Styles from one micro frontend affect another.

**Solution**: Use Shadow DOM encapsulation and CSS custom properties:

```typescript
@Component({
  // ...
  encapsulation: ViewEncapsulation.ShadowDom,
  styles: [`
    :host {
      /* Scoped to this component only */
      --component-color: var(--global-color, #3498db);
    }
  `]
})
```

### 3. Loading Performance

**Problem**: Loading multiple micro frontends causes performance issues.

**Solution**: Implement lazy loading and caching:

```typescript
// src/app/web-component-cache.service.ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class WebComponentCacheService {
  private loadedComponents = new Set<string>();

  isLoaded(name: string): boolean {
    return this.loadedComponents.has(name);
  }

  markAsLoaded(name: string): void {
    this.loadedComponents.add(name);
  }

  async loadScript(url: string): Promise<void> {
    return new Promise((resolve, reject) => {
      const script = document.createElement('script');
      script.src = url;
      script.type = 'module';
      script.onload = () => resolve();
      script.onerror = (error) => reject(error);
      document.head.appendChild(script);
    });
  }
}
```

## Conclusion

Combining Angular 20, Web Components, and Micro Frontend Architecture creates a powerful approach to building modern web applications. This integration allows teams to work independently while creating a cohesive user experience, leveraging the strengths of each technology:

- **Angular 20** provides a robust framework with powerful features
- **Web Components** offer framework-agnostic reusability and encapsulation
- **Micro Frontends** enable team autonomy and independent deployment

By following the patterns and practices outlined in this guide, you can build scalable, maintainable, and future-proof applications that can evolve with your business needs. The combination of these technologies is particularly well-suited for large-scale applications with multiple teams working on different features.

As these technologies continue to evolve, this approach will become increasingly valuable for building modular, maintainable, and interoperable web applications.
