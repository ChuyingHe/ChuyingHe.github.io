# Web Design for Web Developers

## Rules
### Typography
1. Use font size 15-25 pixels for main content
2. Use (really) big font size for headlines - no limitations
	- either big size or heavy weight, dont do both
3. Use line spacing betwwen 120-150%
5. 45-90 characters per line
6. Use general "good" fonts & only use ONE typeface e.g.:
	- sans-serif:
		- more neutral, clean, simple, modern
		- example: Open Sans, Lato, Raleway, Monsterratm, PT Sans - all available on Google Fonts
	- serif: 
		- traditional purposes, Storytelling, long reading
		- examples: Cardo, Merriweather, PT Serif

### Color
1. Use only one "base color" other than gray, black and white, starting with [flatcolor](https://flatuicolors.com), after choose one Base-Color, you can choose some variants, and make it to a Color-Palette:

<img src="./color_palette.png">

2. If you REALLY need to take more than one color, use tool to combine the best color palette

3. Use color to draw attention

4. Never use black in your design

5. Choose colors wisely
	- color: passion
	- orange: creativity, confidence, draw attention but not too much as Red
	- yello: pleasent
	- green: health, money, nature
	- blue: truth, most beloved
	- purple: wealth, loyalty
	- pink: romance
	- brown: confidence, comfort


### Image

1. Put text directly on the image: need high contrast between the Image and Text
2. Overlay the image: image+half-transparent-color-bg
3. Put text in Box: box should be still opaque, bg of the box can be all possible color
4. Blur the image: the whole pic, or just one spot of blur
5. The floor fade: use one corner of image which has very high contrast against the text

```css
# Overlay the image. Example: http://jsfiddle.net/drpak8vy/1/
.darken {
	background-image: linear-gradient(rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0.5)), url(YOUR IMAGE HERE);
}

# Put text in a box. Example: http://jsfiddle.net/qg83m36p/
.text-box {
	background-color: rgba(0, 0, 0, 0.5);
	color: #fff;
	display: inline;
	padding: 10px;
}

# Floor fade. Example: http://jsfiddle.net/gRzPF/409/
.floor-fade {
	background: linear-gradient(to bottom, rgba(0, 0, 0, 0), rgba(0, 0, 0, 0.6) ), url(YOUR IMAGE HERE);
}
```

### Icons
1. to list Features/Steps
2. for Actions and Links:
	- should be clear
	- still use labels besides the icon
3. Icons should NOT take a center stage
4. Use Icon Fonts:
	- Static Image will get blur when you scale 300x
	- Vector icon font will never get blur


### Spacing & Layout
1. Use but NOT exaggrate whitespace between:
	- elements
	- groups of elements
	- sections
2. Define hierarchy:
	- Decide where you want your audience to look first
	- establish a flow that corresponds to your content's msg
	- use whitespace to build that flow

### Inspiration
1. Collect designs you like
2. Why do they look good?
3. what do these sites have in common?
4. How were they built in HTML & CSS? (use google chrome)

First make you website alike your favorite web.

### Conversion rate
1. offer free gift first (e-book)
2. strong call to action btn: appear as often as possible
3. Grab user attention: e.g. pop-up box for email-subscription
4. tell your user the benefit: "Try it for free"
5. dont ask more info that you dont need: only email address (and name?)
6. use social references: customer comments
7. use urgency: "limited time remaining" or "over 320 bought", "only 1 left in stock"


Where to find inperation? [codingheroes](https://codingheroes.io/resources/)
