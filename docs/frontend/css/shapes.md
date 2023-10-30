## Resource
- [Julia Miocene's Youtube channel](https://www.youtube.com/watch?v=LKwbGLv1Re4&ab_channel=JuliaMiocene)
- Artist: [Mark Rise](https://markrise.art/)

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
The background property is a shorthand property for the followings

- background-color
- background-image
- background-position
- background-size
- background-repeat
- background-origin
- background-clip
- background-attachment

```css
background: bg-color bg-image position/bg-size bg-repeat bg-origin bg-clip bg-attachment initial|inherit;
```

!!! note "Note: forward slash("/")"
	If one of the properties in the shorthand declaration is the `bg-size` property, you must use a / (slash) to separate it from the bg-position property: `bg-position / bg-size`

CSS allows you to add multiple background. The different backgrounds are separated by commas (`,`), and the images are stacked on top of each other, where the first image is closest to the viewer. For example:
```css
#example1 {
  background-image: url(img_flwr.gif), url(paper.gif);
  background-position: right bottom, left top;
  background-repeat: no-repeat, repeat;
}
/*This can be compressed into 1 sentence:*/

#example1 {
  background: 
  	url(img_flwr.gif) right bottom no-repeat, 
  	url(paper.gif) left top repeat;
}
```

### mask-image
to create a mask layer to place over an element to partially or fully hide portions of the element. The mask layer can be one of the following formats:
- a PNG image
- an SVG image
- a CSS gradient
- an SVG <mask> element

!!! warning "Attention"
	Most browsers only have partial support for CSS masking. You will need to use the -webkit- prefix in addition to the standard property in most browsers.


More example of **masking technics**: https://www.w3schools.com/css/css3_masking.asp 

### box-shadow
besides using this property for shadowing, we can also use it to duplicate existing element.

`box-shadow: none|h-offset v-offset blur spread color |inset|initial|inherit;`

- `spread`:	Optional. The spread radius. A positive value increases the size of the shadow, a negative value decreases the size of the shadow
- `inset`: Optional. Changes the shadow from an outer shadow (outset) to an inner shadow	

To define multiple `box-shadow`, use `,` to separate

```css
box-shadow: 
	inset 	/* inner shadow */
	 0 		/* horizontal offset */
	 -13px	/* vertical offset */
	 0 		/* blur radius */
	 -10px	/* spread - to resize */
	 var(--dark-skin);	
```

### aspect-ratio
to define the ratio between width and height of an element.

## Useful CSS functions
You can find a full list of CSS functions [here](https://www.w3schools.com/cssref/css_functions.php).

### linear-gradient()
`background-image: linear-gradient(direction, color-stop1, color-stop2, ...);`

`direction` value could be:
- `to right`
- `to bottom right`
- `180deg`

Example:
```css
background: linear-gradient(
  140deg,            /* Gradient angle */
  var(--skin) 50%,   /* Color at 50% position */
  transparent calc(50% + 1px) /* Transparent color at 50% + 1px position */
);
```

### calc()
to perform a calculation to be used as the property value.
!!! warning "space"
	In the the expression of `calc(expression)`, you need to add space before and after the mathematic operation. For instance: `calc(50% + 1px)`

### max() 
### min()