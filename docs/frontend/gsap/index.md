[TOC]


# Installation

1. `cdnjs` url
2. `npm install gsap`
3. download local copy in format of `zip`

# What can GSAP move?

- SVG
- Canvas
- WebGL
- JS Object


# Basic Grammar

```js
// 1.param: html element selector: 
// 		- could be class-name-selector, id-selector and s.o.
//		- select list of elements: `.logo, .icon`
// 2.param: configuratoin for the animation: is a JSON object
gsap.to(".logo", {duration: 2, x: 300})
```

In the background, the `gsap` tool changes the CSS style of the html element `.logo` using `transform: translate(x, y)`

## Configuration
!!! note "2rd param: configuration"
	1. `backgroundColor` instead of `background-color`: Because `-` is interpreted as "minus" in JS.
	2. `"20%"` instead of `20%`: quotes ensure the string format, because `%` is an operator in JS.
	3. border attribute also give in string format like this: `border: "5px solid white"`
	4. `ease` controls the movement style of the animation. See [doc](https://greensock.com/docs/v3/Eases)

Callback functions can also be used as part of the configuration, the available callbacks are:

- `onComplete`: invoked when the animation has completed.
- `onStart`: invoked when the animation begins
- `onUpdate`: invoked every time the animation updates (on every frame while the animation is active).
- `onRepeat`: invoked each time the animation repeats.
- `onReverseComplete`: invoked when the animation has reached its beginning again when reversed.

Except the `gsap.to()`, there are various other [GSAP Attributes & Methods](https://greensock.com/docs/v3/GSAP).


## Utility functions
In the configuration, for the numeric values like `y`, you can either give it a number, or a **randomization function** which generates a random number for you, there are 2 different methods:
```js
{
	duration: 1,

	// Method 1: Math.random() generates a number from 0 to <1
	y: () => Math.random()*400 - 200,

	// Method 2: 
	x: "random(-200, 200)"
}
```

!!! note "\"random(x,y)\""
	this **function as string** is one of the **utility functions** provided by GSAP. More to read [here](https://greensock.com/docs/v3/GSAP/gsap.utils)

## Tween
We call the result of a `gsap.to()` a `tween`. All the GSAP Methods returns an instance of `tween`:
```js
let tween = gsap.to(".logo", {duration: 2, x: 300})
```

The [tween](https://greensock.com/docs/v3/GSAP/Tween) has attributes and methods to control the animation. For instance:
```js
//now we can control it!
tween.pause();
tween.seek(2);
tween.progress(0.5);
tween.play();
```

## Timeline
We can give the animations a sequence using `timeline`:
```js
var tl = gsap.timeline();

// start at exactly 1 second into the timeline (absolute)
tl.to(".green", { x: 600, duration: 2 }, 1);

// insert at the start of the previous animation
tl.to(".purple", { x: 600, duration: 1 }, "<");

// insert 1 second after the end of the timeline (a gap)
tl.to(".orange", { x: 600, duration: 1 }, "+=1");
```

The third params is the `position`, this is used to decide the sequence of the tweens. Timeline itself also provides some special properties, for example:
```js
let tl = gsap.timeline({repeat: -1, repeatDelay: 1, yoyo: true})
```

More possible possibilities checkout [here](https://greensock.com/docs/v3/GSAP/Timeline)

# Plugins
Besides the GSAP Basic functions in the gsap package, there are also plenty of plugins that offered only in `cdn` format. For instance [ScrollTrigger](https://greensock.com/scrolltrigger/) is super cool and free of charge! Check [here](https://greensock.com/docs/) for more information. 