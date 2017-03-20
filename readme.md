# CSS Coding Guidelines

I use [PostcSS](https://github.com/postcss/postcss) with [CSSNext](https://github.com/MoOx/postcss-cssnext) which includes [Autoprefixer](https://github.com/postcss/autoprefixer) and few more additional plugins for enhanced development experience. From provided functionality, I use only variables, functions and imports. Nothing more.


## Configuration file

See file structure at the end of the guide.

The configuration stylesheets `app.css` is UTF-8 encoded and imports [normalize.css](https://github.com/necolas/normalize.css). All base settings go into this file.

The base font size is set to `62.5%` which results in `10px` on browsers where the base font size is not changed. Box-sizing is set to `border-box` and if needed is overridden under specific selectors only.

I use this as a base for `app.css`:
```css
/* app.css */
@charset "utf-8";
@import "normalize.css/normalize.css";

:root {
  /* here goes project-specific variables */
}

html {
  font-size: 62.5%;
  box-sizing: border-box;
}

body {
  margin: 0;
}

*, *:before, *:after {
  box-sizing: inherit;
}

img {
  vertical-align: top;
}

a {
  color: inherit;
}

a img {
  border: 0;
}
```
I typically set default color and font family along with other project-specific settings in this file.

`app.css` is imported in `main.css` only. Read further for more explanation on this.

## Main entry file

In the main entry file I first import `app.css` and then remaining CSS files lastly ending file with importing utility classes which is needed to override some rules because of order how selectors are located in output file.
```css
/* main.css */
@import "app.css";
@import "components/*.css";
/* etc... */
@import "utilities/*.css";
```

This file is not intended to have any style rules. All rules are kept in other files.
All imports are done in this file only. For imports, I use [Easy Import](https://github.com/TrySound/postcss-easy-import) which inlines all import files and allows me to skip manual import of `app.css` in every single file.


## Variables names

The variables naming convention is designed to scale well and be easy to decode.

Pattern: `<property>-<value>[--<componentName>][-<modifier>]`

The parts divided by a dash are written in camel case. Dashes are used only to separate logical parts of the pattern.

The syntax gives the understanding of where and under what conditions we are setting a value of certain property. This is an example of how I define a color variable:
```css
/* app.css */
:root {
  --color-green-500: rgba(0, 255, 0, 1);
}
```
With this I define that the variable targets a _color_ property and will hold a value of _green color_ with modification of _500_.

Here is how I target variable for specific usage:
```css
/* components/button.css */
:root {
  --color-normal--button-green: var(--color-green-400, rgb(0, 230, 0));
  --color-hover--button-green: var(--color-green-300, rgb(0, 240, 0));
  --color-disabled--button-green: color(var(--color-green-400) alpha(-50%));
}
```
I'm defining that variables target _color_ propery under certaing _conditions_ (normal, hover or disabled in the given example) of _button's component_ with modification of _green_.

Global variables go into `app.css` while components specific variables live within respective component's file. From the examples above you can see that more general `--color-green` is defined in `app.css` where button's specific variables live in `components/button.css`.

There may be a situation where I have to share some values across components. In this case, I define a general variable in `app.css` and use where appropriate. Color variables are a good example. Another example would be:
```css
/* app.css */
:root {
  --height-small--header: 6em;
}

/* components/alert.css */
:root {
  --top-small--alert: var(--height-small--header);
}

/* components/navigation.css */
:root {
  --height-small--navigation: var(--height-small--header);
  --height-small--navigationList: calc(100vh - var(--height-small--header));
}
```

## Colors

When styling components I'm using colors only defined in `app.css`. I use only `rgb` and `rgba`.
```css
/* app.css */
:root {
  --color-green: rgb(0, 255, 0);
  --color-red: rgba(255, 0, 0, 1);
}
```


## Media Queries

I predefine all media queries in `app.css` and use only these throughout style files. Media rules go at the bottom of files which follows mobile-first approach.

I use mobile-first approach when defining media queries.
```css
/* app.css */
@custom-media --large-screen-up screen and (min-width: 1280px);
@custom-media --medium-screen-up screen and (min-width: 769px);
@custom-media --small-screen-only screen and (max-width: 768px);
```

I try to avoid usage of words like "mobile" or "desktop" as these keywords do not represent the actual situation with screens. Therefore I'm targeting various screen sizes and pixel densities when needed to fit design needs.

Good usage is:
```css
/* components/header.css */
.header {
  height: 6em;
}

@media (--large-screen-up) {
  .header {
    height: 9.5em;
  }
}
```

Bad usage would be:
```css
/* components/header.css */
.header {
  height: 6em;
  
  @media (--large-screen-up) {
    height: 9.5em;
  }
}
```


## Units

I use `em` exclusively for setting the font size, margins, paddings, etc. It is allowed to use `rem` when resetting inherited font size or scaling the whole component. `em` gives a better pixel rounding in various screen sizes and pixel densities.

For size and offset attributes, I use either `em` or viewport units (`vw` and `vh`).

Using `em` as units give the possibility to scale components as a whole. By simply setting `font-size` I can scale all elements which inherit from components root.

Working with `em` units involve some calculations. Instead of explicitly setting `10px` I now have to set `1em` and so on. This is achieved by setting root element's font-size to `62.5%` which is `10px`, otherwise, math would be more complicates as I should have divided by 16, not 10.


## Selectors

The selectors naming convention is designed to make stylesheets scalable by avoiding high specificity. With applied rules, I avoid many shortcomings of CSS (e.g. lack of style encapsulation, selectors order, high specificity, etc.) and better communicate the relationships between class names.

### Javascript

Pattern: `js-<targetName>`

I never select elements by their component's class names directly as changing them would break my JS code. Instead, I use `js-` prefixed class names.

I always include a verb in the class name as typically I'm performing some action with JS code. The `.js-click-openHeaderMenu` is less likely to conflict while more general `.js-menu` could easily get you in trouble.

I never style elements by JS-targeted selectors as this utility class is not intended for that.

### Utility classes

Utility classes are low-level traits. Utility classes can be applied and combined straight on elements and used along with component class names.

Utility classes are designed to be used where repetition and modifiers could be avoided. For example `.l-floatRight` or `.t-verticalCenter`, etc. Latter, for example, is used in situations where I don't want to implement a component's modification with vertically centered text.

#### Layout utility classes

Pattern: `l-<className>`

Layout utility classes are used for positioning,

#### Typography utility classes

Pattern: `t-<className>`

Typography utility classes are used for styling texts. These classes are widely used in a content of various components.

Usage example of utility classes:
```html
<div class="userCard">
  <div class="userCard-displayName t-colorRed">...</div>
  <div class="userCard-bio l-pullLeft">...</div>
  <div class="userCard-occupation l-pullRight t-fontSizeSmall">...</div>
</div>
```

How do I know when to use utility classes? If I don't know all possible modifications of a component, I start with utility classes. When usage starts to repeat on certain component then probably it is a time to add a modifier to the component.


### Components

Pattern: `<componentName>[-descendantName]`

Both `componentName` and `descendantName` are written in camel case.

A component name:
```css
/* components/user_card.css */
.userCard { ... }
```

A component's descendant name targets a descendant node of a component. It is responsible for adding a style to the descendant only.
```css
/* components/user_card.css */
.userCard-displayName { ... }
```

#### Modifiers

Modifiers are an adjoining class name written in camel case and never are used in standalone. Modifiers are applied to component's base only.

Pattern: `mod-<modifierName>`

Usage:
```css
/* components/user_card.css */
.userCard.mod-ceo { ... }
```

If a modifier is intended to style a descendant of a component then I introduce a nested selector:
```css
/* components/user_card.css */
.userCard.mod-ceo .userCard-displayName { ... }
```
This gives a higher specificity and will easily override initial styles.

### State

A state is an adjoining class name written in camel case and never is used in standalone. A state is applied to component's base only.

Pattern: `is-<stateOfComponent>`

Usage:
```css
/* components/dropdown.css */
.dropdown.is-open { ... }
```

If a state is intended to style a descendant of a component then I introduce a nested selector:
```css
/* components/dropdown.css */
.dropdown.is-open .dropdown-itemsList { ... }
```
This gives a higher specificity and will easily override initial styles.

## Usage of components

Typically a project will have a structure where one component contains another component. Certain properties should be styled in various situations to make a fit of these components. For example, a `.button` may be included in a `.userCard`. As these components are isolated and don't know about each other and don't fit together right away, I have to make them fit together. I never set a new class on an included component, but instead, I use a descendant component to properly position it and add a modifier class on the included component to style it properly.

**Good** implementation:
```html
<div class="userCard">
  <div class="userCard-editButton">
    <button class="button mod-gray">Edit</button>
  </div>
</div>
```

**Bad** implementation would be:
```html
<div class="userCard">
  <button class="button userCard-editButton">Edit</button>
</div>
```


## Formatting

I use `.editorconfig` with these settings:
```
[*.css]
indent_style = space
indent_size = 2
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
```

When writing multiple selectors I place each on their own line. After each rule block I place an empty line and I use double quotes where needed:
```css
/* components/button.css */
.button:hover,
.button:focus {
  ...
}

.button.is-active,
.button:active {
  ...
}

/* components/code.css */
.code {
  font-family: "Courier New", Courier, monospace;
}

.code.mod-css {
  background-image: url("/assets/bacgrounds/code-css.png");
  ...
}
```

## File structure

This is a typical file structure if React is not involved. With React I would keep CSS files next to React components.

```
css
├── atoms
│   ├── button.css
│   └── form_input.css
├── molecules
│   └── navigation.css
├── organisms
│   └── header.css
├── pages
│   └── landing_page.css
├── utilities
│   ├── layout.css
│   └── typography.css
├── app.css
└── main.css
```