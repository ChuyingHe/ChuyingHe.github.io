## Resource
- [Julia Miocene's Youtube channel](https://www.youtube.com/watch?v=LKwbGLv1Re4&ab_channel=JuliaMiocene)

## Useful CSS attributes
Here we collect some important CSS attributes for drawing in CSS: 
### content
`content: normal|none|counter|attr|string|open-quote|close-quote|no-open-quote|no-close-quote|url|initial|inherit;` is usually used with pseudo-selector `::after` or `::before`. For instance:
```css
.item::before {
	content: "- ";
}
```

### translate
`translate: x-axis y-axis z-axis|initial|inherit;` change position of an element:

### border-radius
`border-radius: 1-4 length|% / 1-4 length|%|initial|inherit;` defines the radius of the element's corners.

```css
border-radius: 2em 1em 4em / 0.5em 3em;
```
!!! note "Note: forward slash("/")"
	The forward slash ("/) is used to separate the horizontal (x-axis) and vertical (y-axis) radii:
	`[horizontal radius] / [vertical radius]`

An equivalent way to write them separately:
```css
border-top-left-radius: 2em 0.5em;
border-top-right-radius: 1em 3em;
border-bottom-right-radius: 4em 0.5em;
border-bottom-left-radius: 1em 3em;
```

### background
The background property is a shorthand property for:

- background-color
- background-image
- background-position
- background-size
- background-repeat
- background-origin
- background-clip
- background-attachment

`background: bg-color bg-image position/bg-size bg-repeat bg-origin bg-clip bg-attachment initial|inherit;`

!!! note "Note: forward slash("/")"
	If one of the properties in the shorthand declaration is the bg-size property, you must use a / (slash) to separate it from the bg-position property

## Useful CSS functions
You can find a full list of CSS functions [here](https://www.w3schools.com/cssref/css_functions.php).
`background-image: linear-gradient(direction, color-stop1, color-stop2, ...);`

`direction` value could be:
- `to right`
- `to bottom right`
- `180deg`