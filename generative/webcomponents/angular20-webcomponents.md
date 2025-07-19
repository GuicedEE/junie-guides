# Angular 20 Web Components Guide

## Introduction

This guide provides comprehensive information on how to create components that can function as both Angular components and Web Components (Custom Elements) using Angular 20. With Angular 20's enhanced Web Components integration, developers can build reusable UI elements that work seamlessly across different frameworks or in vanilla HTML applications.

Web Components are a set of standardized web platform APIs that allow you to create custom, reusable, and encapsulated HTML elements. Angular 20 provides first-class support for both consuming and producing Web Components, enabling you to leverage Angular's powerful features while creating framework-agnostic components.

## Understanding Angular Components vs. Web Components

### Angular Components

Angular components are the building blocks of Angular applications:

- **Framework-specific**: Designed to work within the Angular ecosystem
- **Rich feature set**: Two-way data binding, dependency injection, change detection
- **Angular lifecycle**: Uses Angular's component lifecycle hooks
- **Template syntax**: Uses Angular's template syntax with directives and pipes
- **Scoped styles**: CSS encapsulation through Angular's view encapsulation

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-button',
  standalone: true,
  template: `
    <button [class]="variant" (click)="onClick()">
      {{ label }}
    </button>
  `,
  styles: [`
    button {
      padding: 8px 16px;
      border-radius: 4px;
      cursor: pointer;
    }
    .primary {
      background-color: #3498db;
      color: white;
    }
    .secondary {
      background-color: #f1f1f1;
      color: #333;
    }
  `]
})
export class ButtonComponent {
  @Input() label = 'Button';
  @Input() variant: 'primary' | 'secondary' = 'primary';
  @Output() clicked = new EventEmitter<void>();

  onClick() {
    this.clicked.emit();
  }
}
```

### Web Components (Custom Elements)

Web Components are a set of standardized browser APIs:

- **Framework-agnostic**: Work in any JavaScript framework or vanilla HTML
- **Shadow DOM**: Provides style and DOM encapsulation
- **Custom Elements**: Defines new HTML elements with custom behavior
- **HTML Templates**: Reusable HTML structures
- **Standard lifecycle**: Uses the Custom Elements lifecycle callbacks

```javascript
class CustomButton extends HTMLElement {
  static get observedAttributes() {
    return ['label', 'variant'];
  }

  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.render();
  }

  get label() {
    return this.getAttribute('label') || 'Button';
  }

  get variant() {
    return this.getAttribute('variant') || 'primary';
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue !== newValue) {
      this.render();
    }
  }

  connectedCallback() {
    this.shadowRoot.querySelector('button').addEventListener('click', this.onClick.bind(this));
  }

  disconnectedCallback() {
    this.shadowRoot.querySelector('button').removeEventListener('click', this.onClick.bind(this));
  }

  onClick() {
    this.dispatchEvent(new CustomEvent('clicked'));
  }

  render() {
    this.shadowRoot.innerHTML = `
      <style>
        button {
          padding: 8px 16px;
          border-radius: 4px;
          cursor: pointer;
        }
        .primary {
          background-color: #3498db;
          color: white;
        }
        .secondary {
          background-color: #f1f1f1;
          color: #333;
        }
      </style>
      <button class="${this.variant}">${this.label}</button>
    `;
  }
}

customElements.define('custom-button', CustomButton);
```

## Creating Dual-Purpose Components with Angular 20

Angular 20 introduces enhanced capabilities for creating components that can function both as Angular components and as Web Components. This is achieved through the new `withWebComponent` provider.

### Basic Approach

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
  // Enable Web Component capabilities
  providers: [
    withWebComponent({
      // Define custom element name
      name: 'rating-element',
      // Expose CSS custom properties
      cssProperties: ['--star-color', '--star-size', '--star-filled-color'],
      // Expose CSS shadow parts
      cssParts: ['container', 'star'],
      // Enable form association
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

### Usage in Different Contexts

#### As an Angular Component

```typescript
import { Component } from '@angular/core';
import { RatingComponent } from './rating.component';

@Component({
  selector: 'app-product-review',
  standalone: true,
  imports: [RatingComponent],
  template: `
    <div class="review-form">
      <h3>Rate this product</h3>
      <app-rating [value]="rating" (valueChange)="updateRating($event)"></app-rating>
      <p>Your rating: {{ rating }}/5</p>
    </div>
  `
})
export class ProductReviewComponent {
  rating = 0;

  updateRating(value: number) {
    this.rating = value;
    console.log(`Product rated: ${value}/5`);
  }
}
```

#### As a Web Component

```html
<!DOCTYPE html>
<html>
<head>
  <title>Product Review</title>
  <script src="elements.js"></script>
  <style>
    rating-element {
      --star-color: #ddd;
      --star-filled-color: #ff9900;
      --star-size: 32px;
    }

    rating-element::part(star) {
      transition: transform 0.2s;
    }

    rating-element::part(star):hover {
      transform: scale(1.2);
    }
  </style>
</head>
<body>
  <div class="review-form">
    <h3>Rate this product</h3>
    <rating-element value="0"></rating-element>
    <p id="rating-display">Your rating: 0/5</p>
  </div>

  <script>
    const ratingElement = document.querySelector('rating-element');
    const ratingDisplay = document.getElementById('rating-display');

    ratingElement.addEventListener('valueChange', (event) => {
      const rating = event.detail;
      ratingDisplay.textContent = `Your rating: ${rating}/5`;
      console.log(`Product rated: ${rating}/5`);
    });
  </script>
</body>
</html>
```

## Building and Registering Web Components

### Step 1: Create Angular Components

First, create your Angular components as you normally would, but with the `withWebComponent` provider:

```typescript
// src/app/components/toggle-switch.component.ts
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { withWebComponent } from '@angular/elements';

@Component({
  selector: 'app-toggle-switch',
  standalone: true,
  template: `
    <label class="switch" part="container">
      <input 
        type="checkbox" 
        [checked]="checked" 
        (change)="onChange($event)"
        part="input">
      <span class="slider" part="slider"></span>
    </label>
  `,
  styles: [`
    :host {
      display: inline-block;
    }
    .switch {
      position: relative;
      display: inline-block;
      width: 60px;
      height: 34px;
    }
    .switch input {
      opacity: 0;
      width: 0;
      height: 0;
    }
    .slider {
      position: absolute;
      cursor: pointer;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background-color: var(--background-color, #ccc);
      transition: .4s;
      border-radius: 34px;
    }
    .slider:before {
      position: absolute;
      content: "";
      height: 26px;
      width: 26px;
      left: 4px;
      bottom: 4px;
      background-color: white;
      transition: .4s;
      border-radius: 50%;
    }
    input:checked + .slider {
      background-color: var(--active-color, #2196F3);
    }
    input:checked + .slider:before {
      transform: translateX(26px);
    }
  `],
  providers: [
    withWebComponent({
      name: 'toggle-switch',
      cssProperties: ['--background-color', '--active-color'],
      cssParts: ['container', 'slider', 'input'],
      formAssociated: true
    })
  ]
})
export class ToggleSwitchComponent {
  @Input() checked = false;
  @Output() checkedChange = new EventEmitter<boolean>();

  onChange(event: Event) {
    const target = event.target as HTMLInputElement;
    this.checked = target.checked;
    this.checkedChange.emit(this.checked);
  }
}
```

### Step 2: Register Components in a Module

Create a module to register all your web components:

```typescript
// src/app/elements/elements.module.ts
import { NgModule, Injector } from '@angular/core';
import { createCustomElement } from '@angular/elements';
import { ToggleSwitchComponent } from '../components/toggle-switch.component';
import { RatingComponent } from '../components/rating.component';

@NgModule({
  imports: [
    ToggleSwitchComponent,
    RatingComponent
  ]
})
export class ElementsModule {
  constructor(private injector: Injector) {}

  ngDoBootstrap() {
    // The custom elements will be automatically defined by the withWebComponent provider
    // No need to manually call createCustomElement and define
  }
}
```

### Step 3: Bootstrap the Elements Module

Create a separate entry point for your web components:

```typescript
// src/elements-main.ts
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { ElementsModule } from './app/elements/elements.module';

platformBrowserDynamic()
  .bootstrapModule(ElementsModule)
  .catch(err => console.error(err));
```

### Step 4: Configure Build Process

Update your Angular configuration to build the elements bundle:

```
// angular.json
{
  "projects": {
    "your-project": {
      "architect": {
        "build": {
          "configurations": {
            "elements": {
              "main": "src/elements-main.ts",
              "outputHashing": "none",
              "optimization": true,
              "outputPath": "dist/elements"
            }
          }
        }
      }
    }
  }
}
```

### Step 5: Build and Use

Build your elements bundle:

```bash
ng build --configuration=elements
```

Use the generated bundle in any HTML page:

```html
<!DOCTYPE html>
<html>
<head>
  <script src="elements.js"></script>
</head>
<body>
  <toggle-switch checked></toggle-switch>
  <rating-element value="3"></rating-element>
</body>
</html>
```

## Advanced Features and Best Practices

### 1. Input Property Handling

When creating dual-purpose components, ensure proper handling of input properties:

```typescript
@Component({
  // ...
})
export class DualPurposeComponent {
  private _complexData: any;

  // For primitive values, use standard @Input()
  @Input() label: string;

  // For complex objects, implement custom getter/setter
  @Input()
  get complexData(): any {
    return this._complexData;
  }
  set complexData(value: any) {
    this._complexData = value;
    // When used as a web component, the value might come as a string
    if (typeof value === 'string') {
      try {
        this._complexData = JSON.parse(value);
      } catch (e) {
        console.error('Invalid JSON in complexData attribute');
      }
    }
  }
}
```

### 2. Event Handling

Ensure events work in both Angular and Web Component contexts:

```typescript
@Component({
  // ...
})
export class DualPurposeComponent {
  // Standard Angular output
  @Output() valueChange = new EventEmitter<any>();

  emitValue(value: any) {
    // This works in both contexts
    this.valueChange.emit(value);
  }
}
```

### 3. Styling Considerations

Design your component styles to work well in both contexts:

```typescript
@Component({
  // ...
  styles: [`
    :host {
      display: block;
      /* Use CSS custom properties for theming */
      --primary-color: #3498db;
      --text-color: #333;
    }

    .container {
      color: var(--text-color);
      background-color: var(--primary-color);
    }

    /* Expose parts for external styling */
    .button {
      /* This element can be styled using ::part(button) */
    }
  `],
  providers: [
    withWebComponent({
      // ...
      cssProperties: ['--primary-color', '--text-color'],
      cssParts: ['button']
    })
  ]
})
export class DualPurposeComponent {
  // ...
}
```

### 4. Form Integration

Make your components work with Angular forms and native forms:

```typescript
@Component({
  // ...
  providers: [
    withWebComponent({
      // ...
      formAssociated: true
    })
  ]
})
export class FormInputComponent implements ControlValueAccessor {
  @Input() value: string;
  @Output() valueChange = new EventEmitter<string>();

  // Implement ControlValueAccessor for Angular forms
  onChange: any = () => {};
  onTouched: any = () => {};

  writeValue(value: string): void {
    this.value = value;
  }

  registerOnChange(fn: any): void {
    this.onChange = fn;
  }

  registerOnTouched(fn: any): void {
    this.onTouched = fn;
  }

  updateValue(value: string) {
    this.value = value;
    this.valueChange.emit(value);
    this.onChange(value);
    this.onTouched();
  }
}
```

### 5. Lazy Loading Web Components

For performance optimization, load web components only when needed:

```typescript
// src/app/lazy-elements.service.ts
import { Injectable, Injector } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class LazyElementsService {
  private loadedElements = new Set<string>();

  constructor(private injector: Injector) {}

  async loadElement(elementName: string): Promise<void> {
    if (this.loadedElements.has(elementName)) {
      return;
    }

    switch (elementName) {
      case 'rating-element':
        await import('./components/rating.component');
        break;
      case 'toggle-switch':
        await import('./components/toggle-switch.component');
        break;
      // Add more elements as needed
    }

    this.loadedElements.add(elementName);
  }
}
```

## Testing Web Components

### Unit Testing

Test your components in both Angular and Web Component contexts:

```typescript
// Testing as an Angular component
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { RatingComponent } from './rating.component';

describe('RatingComponent (as Angular component)', () => {
  let component: RatingComponent;
  let fixture: ComponentFixture<RatingComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [RatingComponent]
    }).compileComponents();

    fixture = TestBed.createComponent(RatingComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should emit valueChange when a star is clicked', () => {
    spyOn(component.valueChange, 'emit');
    component.setValue(3);
    expect(component.valueChange.emit).toHaveBeenCalledWith(3);
  });
});

// Testing as a Web Component
describe('RatingComponent (as Web Component)', () => {
  let element: HTMLElement;

  beforeEach(async () => {
    // Ensure the custom element is defined
    await import('./rating.component');

    // Create the element
    element = document.createElement('rating-element');
    document.body.appendChild(element);
  });

  afterEach(() => {
    document.body.removeChild(element);
  });

  it('should create the custom element', () => {
    expect(element).toBeTruthy();
    expect(element.tagName.toLowerCase()).toBe('rating-element');
  });

  it('should respond to attribute changes', () => {
    element.setAttribute('value', '3');

    // Allow for async rendering
    setTimeout(() => {
      const filledStars = element.shadowRoot.querySelectorAll('.star.filled');
      expect(filledStars.length).toBe(3);
    });
  });

  it('should dispatch events', (done) => {
    element.addEventListener('valueChange', (event: any) => {
      expect(event.detail).toBe(4);
      done();
    });

    const fourthStar = element.shadowRoot.querySelectorAll('.star')[3];
    fourthStar.click();
  });
});
```

### Integration Testing

Test how your web components interact with other parts of your application:

```typescript
import { TestBed } from '@angular/core/testing';
import { AppComponent } from './app.component';

describe('AppComponent with Web Components', () => {
  beforeEach(async () => {
    // Import and register web components
    await import('./components/rating.component');
    await import('./components/toggle-switch.component');

    await TestBed.configureTestingModule({
      imports: [AppComponent]
    }).compileComponents();
  });

  it('should use web components correctly', () => {
    const fixture = TestBed.createComponent(AppComponent);
    fixture.detectChanges();

    const compiled = fixture.nativeElement as HTMLElement;
    const ratingElement = compiled.querySelector('rating-element');
    const toggleElement = compiled.querySelector('toggle-switch');

    expect(ratingElement).toBeTruthy();
    expect(toggleElement).toBeTruthy();

    // Test interactions
    toggleElement.setAttribute('checked', '');
    // Verify expected behavior
  });
});
```

## Deployment Strategies

### 1. Standalone Web Components

Deploy your web components as standalone bundles that can be used in any application:

```bash
# Build the elements bundle
ng build --configuration=elements

# The output will be in dist/elements
# You can then serve or distribute these files
```

### 2. Integrated in Angular Applications

Use your components directly in Angular applications:

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { RatingComponent } from './components/rating.component';
import { ToggleSwitchComponent } from './components/toggle-switch.component';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    RatingComponent,
    ToggleSwitchComponent
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

### 3. Micro Frontend Architecture

Use web components as part of a micro frontend architecture:

```html
<!-- shell-app/index.html -->
<!DOCTYPE html>
<html>
<head>
  <title>Shell Application</title>
  <!-- Load shared components -->
  <script src="https://cdn.example.com/shared-components.js"></script>
</head>
<body>
  <header>
    <custom-header></custom-header>
  </header>

  <main>
    <!-- Load micro frontend 1 -->
    <script type="module" src="https://team1.example.com/app1.js"></script>
    <micro-app-1></micro-app-1>

    <!-- Load micro frontend 2 -->
    <script type="module" src="https://team2.example.com/app2.js"></script>
    <micro-app-2></micro-app-2>
  </main>

  <footer>
    <custom-footer></custom-footer>
  </footer>
</body>
</html>
```

## Conclusion

Angular 20's enhanced Web Components integration provides a powerful way to create components that can be used both within Angular applications and as standalone web components. By following the patterns and practices outlined in this guide, you can build reusable UI elements that work across different frameworks and technologies.

The ability to create dual-purpose components opens up new possibilities for code sharing, design system implementation, and micro frontend architectures. With Angular 20's `withWebComponent` provider, you can leverage Angular's powerful features while creating framework-agnostic components that can be used anywhere.

As web component standards continue to evolve and browser support improves, this approach will become increasingly valuable for building modular, maintainable, and interoperable web applications.
