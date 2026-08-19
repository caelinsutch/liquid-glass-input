# liquid-glass-input

A glass text input built with [`@ybouane/liquidglass`](https://liquid-glass.ybouane.com/) — realistic WebGL glass refraction applied to a real, focusable `<input>`.

| File | What it is |
| --- | --- |
| `index.html` | The minimal pattern. One glass search field over a photo. |
| `controls.html` | The same field plus a live control panel for every glass property, an FPS meter, and a **Copy JSON** button. |

## Run it

The demos must be served over HTTP. `file://` fails, because `html-to-image` and the cross-origin background image both need real HTTP headers.

```bash
python3 -m http.server 8899
# then open http://localhost:8899/ or http://localhost:8899/controls.html
```

## How the input works

The library injects its shader `<canvas>` as the glass element's **first child**, with `pointer-events:none` and `z-index:-1`. Real DOM children therefore stay visible, clickable, and focusable above the shader output.

You cannot make the `<input>` itself a glass element, because a glass element must host that child canvas. Wrap it instead:

```html
<div id="root">                      <!-- position: relative -->
  <img id="bg" crossorigin="anonymous" src="...">   <!-- background inside the root -->
  <label class="glass-field">        <!-- the glass element, a direct child of the root -->
    <input type="search" placeholder="Search anything">
  </label>
</div>
```

```js
import { LiquidGlass } from 'https://cdn.jsdelivr.net/npm/@ybouane/liquidglass/dist/index.js';
await LiquidGlass.init({ root, glassElements: [field] });
```

## Gotchas found while building this

1. **The glass element needs its own stacking context.** Set `isolation: isolate`. Without it, the `z-index:-1` canvas escapes to the root stacking context and paints *behind* the background image, so the glass looks like it never rendered.
2. **Use `outline` for the focus ring, not `box-shadow`.** A negative z-index child paints above the parent background layer, so a `box-shadow` hides under the canvas. `outline` paints last.
3. **Keystrokes are free.** Typing changes `value`, not the DOM, so the subtree `MutationObserver` never fires and no re-capture runs.
4. **Do not set `button: true` on a field.** Press mode flattens the bevel and fights the focus state.
5. **One instance per form.** Every instance opens its own WebGL context, and browsers cap them near 16. Pass all fields to a single `init()` call.
6. **Use `font-size: 16px` minimum**, or iOS zooms on focus.
7. **The background image needs `crossorigin="anonymous"`**, or the canvas becomes tainted and texture upload fails.
8. **`bevelMode`, `button`, and `floating` are read at `init()`.** Change them with `destroy()` then `init()`. Numeric options update live through `data-config`.
9. **Canvases are re-measured on window resize only.** After a programmatic size change, dispatch a synthetic `resize` event.
10. **Ship a `backdrop-filter` fallback** for no-WebGL and for reduced-transparency users.

## Keep the panel outside the root

The control panel in `controls.html` is a sibling of `#root`. Inside the root, the shader would capture and refract it, and every slider move would trigger a DOM re-capture.
