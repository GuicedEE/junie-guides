# WaBadgeComponent

An Angular wrapper for the Web Awesome `<wa-badge>` web component that provides declarative usage, input binding, and integration with Angular templates.

## Features

- Binds attributes: `variant`, `appearance`, `pill`, `pulse`
- Enables Angular-style class and style bindings
- Allows slot projection for content
- Enables font sizing via `[style.fontSize]`
- Supports custom styling via CSS variables

## Installation

The component is part of the Angular Awesome library. No additional installation steps are required if you're already using the library.

## Usage

Import the component in your module or standalone component:

```typescript
import { WaBadgeComponent } from 'angular-awesome';

@Component({
  // ...
  standalone: true,
  imports: [WaBadgeComponent]
})
export class YourComponent { }
```

Use it in your templates:

```html
<wa-badge variant="brand">Brand</wa-badge>
```

## API Reference

### Inputs

| Input           | Type                                                        | Default    | Description                                |
|-----------------|-------------------------------------------------------------|------------|--------------------------------------------|
| variant         | 'brand' \| 'neutral' \| 'success' \| 'warning' \| 'danger' \| 'inherit' | 'inherit'  | The color variant of the badge             |
| appearance      | 'accent' \| 'filled' \| 'outlined'                          | 'accent'   | The appearance style of the badge          |
| pill            | boolean \| null                                             | undefined  | Whether the badge has rounded corners      |
| pulse           | boolean \| null                                             | undefined  | Whether the badge has a pulsing animation  |
| fontSize        | string                                                      | undefined  | Optional font-size override                |
| backgroundColor | string                                                      | undefined  | Custom background color (--background-color) |
| borderColor     | string                                                      | undefined  | Custom border color (--border-color)       |
| textColor       | string                                                      | undefined  | Custom text color (--text-color)           |

### Content Projection

The component supports content projection through the default slot:

```html
<wa-badge variant="success">
  <!-- Content goes here -->
  Success
</wa-badge>
```

## Examples

See [badge.example.md](./badge.example.md) for detailed usage examples.

## Implementation Details

The component wraps the Web Awesome `<wa-badge>` web component and provides Angular-specific features:

- Uses `customElements.whenDefined()` to ensure the web component is defined before interacting with it
- Maps attributes to the underlying web component
- Provides type safety for inputs
- Enables Angular-style binding for attributes and styles
- Uses ViewEncapsulation.None to ensure proper styling

## Requirements Fulfilled

This implementation satisfies all requirements specified in the [badge.rules.md](./badge.rules.md) file:

- ✅ Represents `<wa-badge>` as an Angular declarative component
- ✅ Binds attributes: `variant`, `appearance`, `pill`, `pulse`
- ✅ Enables Angular-style class and style bindings
- ✅ Allows slot projection for content
- ✅ Enables font sizing via `[style.fontSize]`
- ✅ Supports custom styling via CSS variables
