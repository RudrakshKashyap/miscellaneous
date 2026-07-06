## 1. The Core Difference: `box-sizing`

By default, the raw web uses a box-sizing strategy called `content-box`. When you add padding or borders to an element, the browser **adds** those pixels to the overall width and height, causing layout bloat.

Tailwind's Preflight injects a global reset that forces every element into `border-box` calculation:

```css
/* Injected automatically when Preflight is ENABLED */
*,
::before,
::after {
  box-sizing: border-box;
  border-width: 0;
  border-style: solid;
  border-color: theme("borderColor.DEFAULT", currentColor);
}
```

### The Math Shift:

If you define a card component with a width of `300px`, padding of `20px`, and a border of `2px`:

| Preflight Enabled (`border-box`)                                                                                          | Preflight Disabled (`content-box`)                                                                                        |
| ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **Total Rendered Width: 300px**                                                                                           | **Total Rendered Width: 344px**                                                                                           |
| The browser squeezes the inner content zone down to `256px` so the padding and borders fit **inside** your defined width. | The browser sets the content zone to `300px` and adds the padding and borders **on top** of it ($300 + 20 + 20 + 2 + 2$). |

---

## 2. Default Margin and Padding Stripping

Beyond the box-sizing math, Preflight aggressively flattens the default margins and paddings out of native HTML elements to ensure cross-browser consistency:

- **When Enabled:** Headings (`<h1>`, `<h2>`), paragraphs (`<p>`), lists (`<ul>`), and figures have their margins completely stripped to `0`.
- **When Disabled:** The browser restores its native user-agent stylesheet defaults. Your layout will suddenly inherit unexpected vertical margins and list padding out of nowhere, shifting adjacent elements around the screen.

---

## 3. Borders and the Box Model

Another major detail in Preflight is that it sets `border-width: 0` globally while initializing `border-style: solid`.

- **When Enabled:** If you add a color utility like `border-blue-500`, **nothing will show up** unless you explicitly add a width class like `border`. The border occupies `0px` of space in the box model layout until specified.
- **When Disabled:** If you use a native fallback element or custom CSS that relies on browser defaults, the browser may apply a default `1px` medium inset/outset border, which immediately alters the layout geometry calculation.

---

## Summary Checklist

If you turn **Preflight off** (`preflight: false` in your Tailwind configuration), your box model calculations revert to legacy browser mechanics:

- Elements will calculate layout dimensions using `content-box` instead of `border-box` unless you manually apply `box-sizing: border-box`.

- UI elements will expand wider/taller than their explicitly configured utility styles (`w-64`, `h-32`) when padding or borders are present.
- Native browser user-agent margins will return to typography tags, changing the visual space elements occupy.
