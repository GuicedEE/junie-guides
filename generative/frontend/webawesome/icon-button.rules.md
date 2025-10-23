📌 This directive assumes compliance with general [Web Awesome Angular Rules](../../../RULES.md).

# Icon Button Rules (`<wa-icon-button>`) for Angular Wrapper

This document defines the integration rules and guidelines for creating the Angular wrapper around the Web Awesome `<wa-icon-button>` web component.

---

## Component Purpose

Icon buttons are compact buttons that only contain an icon, and are used for triggering actions in dense UIs like toolbars.

---

## Selector

Use the component selector:

```ts
selector: 'wa-icon-button'
```

---

## Inputs

All inputs should support string binding by default. Boolean and numeric values should be explicitly coerced.

| Input             | Type                                         | Notes                                          |
| ----------------- | -------------------------------------------- | ---------------------------------------------- |
| `name`            | `string`                                     | Required. Icon name.                           |
| `family`          | `string`                                     | Optional. Icon family.                         |
| `variant`         | `string`                                     | Optional. Icon variant (e.g., solid, regular). |
| `label`           | `string`                                     | Required for accessibility.                    |
| `library`         | `string`                                     | Optional. For custom icon sets.                |
| `src`             | `string`                                     | Optional. External SVG. Use with caution.      |
| `href`            | `string`                                     | Optional. Converts button into anchor.         |
| `target`          | `'_blank' \| '_self' \| '_parent' \| '_top'` | Optional. Used with `href`.                    |
| `download`        | `string`                                     | Optional. Suggests a filename for download.    |
| `disabled`        | `boolean`                                    | Coerced. Disables the button.                  |
| `color`           | `string`                                     | Optional. Sets the text color.                 |
| `backgroundColor` | `string`                                     | Optional. Sets the background color.           |
| `fontSize`        | `string`                                     | Optional. Sets the font size.                  |

---

## Outputs (Events)

The wrapper should re-emit the following native events using `@Output()`:

| Event        | Description                          |
| ------------ | ------------------------------------ |
| `focusEvent` | Emitted when the button gains focus. |
| `blurEvent`  | Emitted when the button loses focus. |

> ℹ️ Note: Use `focusEvent` and `blurEvent` instead of `focus` and `blur` to avoid name collision with DOM methods and improve clarity.

---

## Methods

Expose the following methods via `@ViewChild()` for programmatic control:

```ts
click(): void;
focus(options?: FocusOptions): void; // method
blur(): void;
```

---

## Accessibility

Always require a `label` input to ensure proper screen reader support.

---

## Styling

Icon buttons support custom CSS via Shadow Parts:

| Part   | Description               |
| ------ | ------------------------- |
| `base` | The outer button wrapper. |

Custom properties for color theming:

* `--background-color-hover`
* `--background-color-active`
* `--text-color-hover`
* `--text-color-active`

---

## Behavior Notes

* Font size can be inherited via CSS (e.g., `style="font-size: 1.5rem"`).
* When `href` is defined, render as `<a>`; otherwise, render as `<button>`.
* Tooltip can be linked using `wa-tooltip`'s `for` attribute referencing the `id` of the button.

---

## Related Components

* [Icon Rules](../icon/icon.rules.md): Used to render the icon inside the button.

---

## Example Usage

```html
<wa-icon-button
  name="gear"
  variant="solid"
  label="Settings"
  color="#222"
  backgroundColor="#eee"
  fontSize="1.75rem"
  (focusEvent)="onFocus()"
  (blurEvent)="onBlur()"
></wa-icon-button>
```

---

## Auto-generated Files

This rule set should trigger generation of:

* `icon-button.component.ts`
* `icon-button.component.spec.ts`
* `icon-button.rules.md` (this file)
* `icon-button.examples.md`
* update to `index.ts` export barrel
