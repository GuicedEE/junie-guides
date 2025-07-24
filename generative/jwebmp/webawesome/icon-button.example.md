# Icon Button Examples

## Basic Usage

```html
<wa-icon-button name="star" label="Favorite"></wa-icon-button>
```

## With Different Icon Variants

```html
<!-- Regular variant -->
<wa-icon-button name="heart" variant="regular" label="Like"></wa-icon-button>

<!-- Solid variant -->
<wa-icon-button name="heart" variant="solid" label="Like"></wa-icon-button>

<!-- Brand icons -->
<wa-icon-button name="github" family="brands" label="GitHub"></wa-icon-button>
<wa-icon-button name="twitter" family="brands" label="Twitter"></wa-icon-button>
```

## Disabled State

```html
<wa-icon-button name="trash" label="Delete" disabled></wa-icon-button>
```

## As Links

```html
<wa-icon-button 
  name="external-link" 
  label="Visit Website" 
  href="https://example.com" 
  target="_blank">
</wa-icon-button>

<wa-icon-button 
  name="download" 
  label="Download File" 
  href="/assets/document.pdf" 
  download="document.pdf">
</wa-icon-button>
```

## Using Custom SVG Source

```html
<wa-icon-button src="/assets/icons/custom-icon.svg" label="Custom Action"></wa-icon-button>
```

## Styling Icon Buttons

```html
<!-- Using inline styles -->
<wa-icon-button 
  name="bell" 
  label="Notifications" 
  style="font-size: 1.5rem; color: #4a90e2;">
</wa-icon-button>

<!-- Using CSS classes -->
<style>
  .large-icon-button {
    font-size: 2rem;
  }
  .warning-icon-button {
    color: #f39c12;
    --background-color-hover: rgba(243, 156, 18, 0.1);
    --background-color-active: rgba(243, 156, 18, 0.2);
  }
</style>

<wa-icon-button 
  name="exclamation-triangle" 
  label="Warning" 
  class="large-icon-button warning-icon-button">
</wa-icon-button>

<!-- Using direct styling inputs -->
<wa-icon-button 
  name="heart" 
  label="Favorite" 
  [color]="'#ff0000'" 
  [fontSize]="'24px'" 
  [backgroundColor]="'#f8f8f8'">
</wa-icon-button>

<!-- Dynamic direct styling -->
<wa-icon-button 
  name="star" 
  label="Star" 
  [color]="isActive ? '#f39c12' : '#cccccc'" 
  [fontSize]="isLarge ? '2rem' : '1rem'">
</wa-icon-button>
```

## Using with Angular Bindings

```html
<wa-icon-button 
  [name]="iconName" 
  [family]="iconFamily" 
  [variant]="iconVariant"
  [label]="iconLabel"
  [disabled]="isDisabled"
  (focusEvent)="onFocus($event)"
  (blurEvent)="onBlur($event)">
</wa-icon-button>

<!-- Conditionally showing different icons -->
<wa-icon-button 
  [name]="isActive ? 'toggle-on' : 'toggle-off'" 
  [variant]="isActive ? 'solid' : 'regular'"
  [label]="isActive ? 'Deactivate' : 'Activate'"
  (click)="toggleActive()">
</wa-icon-button>

<!-- Using direct styling with Angular bindings -->
<wa-icon-button 
  [name]="iconName" 
  [label]="iconLabel" 
  [color]="iconColor" 
  [backgroundColor]="iconBackgroundColor" 
  [fontSize]="iconFontSize">
</wa-icon-button>
```

## Common Use Cases

```html
<!-- In a toolbar -->
<div class="wa-cluster wa-gap-s">
  <wa-icon-button name="bold" label="Bold"></wa-icon-button>
  <wa-icon-button name="italic" label="Italic"></wa-icon-button>
  <wa-icon-button name="underline" label="Underline"></wa-icon-button>
  <wa-icon-button name="align-left" label="Align Left"></wa-icon-button>
  <wa-icon-button name="align-center" label="Align Center"></wa-icon-button>
  <wa-icon-button name="align-right" label="Align Right"></wa-icon-button>
</div>

<!-- In a card header -->
<wa-card [withHeader]="true">
  <div slot="header" class="wa-split">
    <span>Card Title</span>
    <div class="wa-cluster wa-gap-xs">
      <wa-icon-button name="edit" label="Edit"></wa-icon-button>
      <wa-icon-button name="trash" label="Delete"></wa-icon-button>
    </div>
  </div>
  Card content goes here.
</wa-card>

<!-- With tooltip -->
<wa-icon-button id="info-button" name="circle-info" label="More Information"></wa-icon-button>
<wa-tooltip for="info-button">Click for more information</wa-tooltip>
```
