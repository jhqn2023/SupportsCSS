# SupportsCSS

> Live, in-browser detection of modern CSS support for selectors, features, and at-rules. Applies support-based classes, exposes a results object, and allows custom tests.

Inspired by the legacy of [Modernizr](https://modernizr.com/), this script evaluates a user's browser for cutting-edge modern CSS support beyond the capabilities of `@supports`.

- Classes are added to `<html>` as either `supports-[feature]` or `no-[feature]`, allowing easier progressive enhancement and build strategies
- Checks for selectors like `:has()`, properties like `text-box-trim`, features like relative color syntax, and at-rules like `@layer` - [see full test suite](#test-suite)
- Allows adding custom tests
- Exposes a results object to iterate over detected support, as well as individual results for quick conditional checks in JS

## Why should I use this?

While `@supports` exists to detect support in CSS itself, it notably doesn't (yet) cover at-rules, such as `@container` or `@layer`. Also, `@supports` cannot reliably test for partial implementations. Additionally, the use of classes simplifies creating selectors.

Plus the support classes eliminate the need to guess and test for the right selector combination to use within an `@supports` block.

You might also enjoy the results collection created for easy-access in your JavaScript.

## Installation

> **Important** - When using a CDN, be sure to version-lock since future releases may remove or modify tests as the specs and browser support stabilizes.

**All tests are opt-in** through the `tests` option array, and expect names that match the feature classes as shown in the [test suite](#test-suite).

### Client-side via CDN

Include via a script tag using UNPKG

```html
<script src="https://www.unpkg.com/supports-css@0.1.3"></script>
```

Follow that with a one-time initialization

```html
<script>
  const tests = ['at-container', 'at-container-style', 'at-layer', 'has'];
  window.SupportsCSS && SupportsCSS.init({ tests });
</script>
```

### Client-side module

```html
<script type="module">
  import * as SupportsCSS from "https://cdn.skypack.dev/supports-css@0.1.3";

  const tests = ['at-container', 'at-container-style', 'at-layer', 'has'];
  SupportsCSS.init({ tests });
</script>
```

### Use in Node or a framework

Install

```js
npm install supports-css
```

Import and initialize one-time in a location that will load client-side.

```js
import * as SupportsCSS from 'supports-css';

const tests = ['at-container', 'at-container-style', 'at-layer', 'has'];
SupportsCSS.init({tests});
```

The `init` function does a check for `window` before attempting the tests.

### Options

The following can be passed to the `init()` function:

- `tests` - array of feature class names that indicate which tests to perform, ex. `['nth-of-s', 'scroll-timeline']`
- `supportsPrefix` - pass a string to customize the prefix for supported features, or `false` to remove the prefix
- `unsupportedClasses` - pass `false` to skip adding classes for unsupported features

Example initialization with options:

```js
const tests = ["nth-of-s", "scroll-timeline"];

SupportsCSS.init({ tests, unsupportedClasses: false, supportsPrefix: "css" });
```

## Usage

After install and initialization, `SupportsCSSTests` will be available for global access in client-side scripts. Review [the test suite](#test-suite) for a list of all features tested.

Features classes are added to `<html>` and can be used within your stylesheets to modify selectors. They are also the keys to include in the `tests` array.

- **Supported**: `.supports-[feature]`
- **Unsupported**: `.no-[feature]`

Global names are the keys to access results directly for conditional checks in JavaScript, such as `SupportsCSSTests.ContainerUnits`, which return a boolean.

The test conditions use a combination of CSS API features exposed on the `window` (at-rules and a few others) and the [CSS.supports function](https://developer.mozilla.org/en-US/docs/Web/API/CSS/supports).

### Get all results

An object of all test results is available as `SupportsCSSTests.results`.

<details>
<summary>Example results output</summary>

```js
{
  "at-container": true,
  "at-container-style-properties": true,
  "at-counter-style": true,
  "at-layer": true,
  "at-property": true,
  "at-scope": false,
  "color-function": true,
  "color-mix": true,
  "container-units": true,
  "dynamic-viewport-units": true,
  "has": true,
  "houdini-paint-api": true,
  "individual-transforms": true,
  "logical-properties": true,
  "media-range-syntax": true,
  "nesting": true,
  "nth-of-s": true,
  "overscroll-behavior": true,
  "relative-color-syntax": false,
  "scroll-timeline": false,
  "subgrid": false,
  "text-box-trim": false,
  "trigonometry": true,
  "view-transitions": true
}
```

</details>

<details>
<summary>Generate a list of results and place in DOM</summary>

```js
const resultsList = document.createDocumentFragment();
const list = document.createElement("ol");
resultsList.appendChild(list);
for (const name in SupportsCSSTests.results) {
  const li = document.createElement("li");
  const result = SupportsCSSTests.results[name];
  li.textContent = `${name}: ${result ? "✅" : "❌"}`;
  list.appendChild(li);
}
document.body.appendChild(resultsList);
```

</details>

### JavaScript conditional checks 

Access results directly via the "global name", such as `SupportsCSSTests.AtContainerStyleProperties`. 

<details>
<summary>Example JS conditional</summary>

```js
if (SupportsCSS.AtContainerStyleProperties) {
  // Container style queries with custom properties are supported
}
```

</details>

### Add a custom test 

Custom tests can be added by choosing a name and creating a test condition that will return a boolean. Add as many as you like using `SupportsCSS.addTest()`.

Here's an example to add a test for `accent-color`:

```js
SupportsCSS.addTest("accent-color", CSS.supports("accent-color: red"));
```

Custom tests allow overriding the choices for `supportsPrefix` (3rd argument) and
`unsupportedClasses` (4th argument) made during intialization.

#### Use with caution: `testEnv()`

The `testEnv()` function is also available for testing that requires an isolated environment due to lack of sufficient exposure via the CSS API. An example is described in the section about the [container style queries test](#atcontainerstyleproperties-test).

⚠️ This should really only be used **when absolutely necessary**, meaning there is not another readily-available, reliable method. For most selectors, properties, values, and functions, you can likely devise a test that uses the [CSS.supports function](https://developer.mozilla.org/en-US/docs/Web/API/CSS/supports).

Use of `testEnv()` requires the following arguments, in order:
- styleBlock - a string containing the style block (rules) to use for the test
- el - the DOM element to create to test against, ex. `p`
- prop - the property to assess with `getPropertyValue`
- value - the exact value that should be returned for the prop

Example of using `testEnv()` for a custom test:

```js
const customResult = SupportsCSS.testEnv('p { top: 1px }', "p", "top", "1px");
SupportsCSS.addTest("my-test", customResult);
```

> **Note** that [getPropertyValue](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration/getPropertyValue), the web API used to get the value from the SVG, may return a different value than the original, such as the `rgb()` value that is returned when evalating a color.

## Test Suite

| Feature Class | Global Name | Test Condition |
|---|---|---|
| at-container | AtContainer | `window.CSSContainerRule` |
| at-container-style-properties | AtContainerStyleProperties | * [See explanation](#atcontainerstyleproperties-test) |
| at-counter-style | AtCounterStyle | `window.CSSCounterStyleRule` |
| at-layer | AtLayer | `window.CSSLayerBlockRule` |
| at-property | AtProperty | `window.CSSPropertyRule` |
| at-scope | AtScope | `window.CSSScopeRule` |
| color-function | ColorFunction | `CSS.supports('color: color(srgb 0 0 1)')` |
| color-mix | ColorMix | `CSS.supports('color: color-mix(in lch, white, black)')` |
| container-units | ContainerUnits | `CSS.supports('width: 1cqi')` |
| dynamic-viewport-units | DynamicViewportUnits | `CSS.supports('width: 1dvi')` |
| has | Has | `CSS.supports('selector(:has(+ *))')` <br>(_Possible false positive in Firefox 112_) |
| houdini-paint-api | HoudiniPaintApi | `window.CSS.paintWorklet` |
| individual-transforms | IndividualTransforms | `CSS.supports('transform: scale(1)')` |
| logical-properties | LogicalProperties | `CSS.supports('border-start-start-radius: 1px')` |
| media-range-syntax | MediaRangeSyntax | `window.matchMedia('(width >= 1px)')` |
| nesting | Nesting | `CSS.supports('selector(& a)')` |
| nth-of-s | NthOfS | `CSS.supports('selector(:nth-child(1 of .a))')` |
| overscroll-behavior | OverscrollBehavior | `CSS.supports('overscroll-behavior: none')` |
| relative-color-syntax | RelativeColorSyntax | `CSS.supports('color: rgb(from red r g b / 1%)')` |
| scroll-timeline | ScrollTimeline | `CSS.supports('scroll-timeline-name: a')` |
| subgrid | Subgrid | `CSS.supports('grid-template-rows: subgrid')` |
| text-box-trim | TextBoxTrim | `CSS.supports('(leading-trim: both) or (text-box-trim: both)')` |
| trigonometry | Trigonometry | `CSS.supports('width: calc(1px * cos(1deg))')` |
| view-transitions | ViewTransitions | `window.ViewTransition` |


### How were these features selected?

Features were selected based on:

- `@supports` limitations
- instability of the spec
- freshness to the language
- impact on CSS architecture
- impact on progressive enhancement

### `AtContainerStyleProperties` Test

Support for [container size queries](https://caniuse.com/css-container-queries) was available prior to the availability of [container style queries](https://caniuse.com/css-container-queries-style). And style queries are presently limited to work with only custom property values. Consequently, detecting for `window.CSSContainerRule` does not cover availability of style queries.

To evaluate the availability of container style queries, an SVG is temporarily created and destroyed in order to provide an isolated test environment without side effects. 

The following styles are tested, and if the correct value for `top` is returned, then container style queries are considered supported.

```css
:root { --a: b; } 
@container style(--a: b) { 
  p { top: 1px; } 
}
```

The `testEnv()` function is exported for you to use in your own tests if you encounter another scenario that cannot be accurately assessed through the current CSS API.

## Colophon

Created by [Stephanie Eckles](https://front-end.social/@5t3ph), author of [ModernCSS.dev](https://moderncss.dev), [SmolCSS.dev](https://smolcss.dev), and other [front-end dev resources](https://thinkdobecreate.com).