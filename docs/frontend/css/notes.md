## clearfix

### Method 1
To ensure that mainContainer has at least the height of the content in leftContent when both leftContent and rightContent are floating elements, you can use the clearfix technique. This technique is used to clear the float and make the container wrap around its floated children.
```css
#mainContainer {
    overflow: auto; /* or overflow: hidden; */
}

#leftContent {
	float: left;	/* your other styles for leftContent */
}

#rightContent {
    float: left;	/* your other styles for rightContent */
}
```
the overflow: auto; property on mainContainer triggers a new block formatting context, causing it to contain its floated children. This prevents the container from collapsing when all of its children are floated.

### Method 2
Alternatively, you can use a **clearfix** class in your HTML:
```html
<div id="mainContainer" class="clearfix">
    <div id="leftContent">
        <!-- content of leftContent -->
    </div>
    <div id="rightContent">
        <!-- content of rightContent -->
    </div>
</div>
```

and css:
```css
.clearfix::after {
    content: "";
    display: table;
    clear: both;
}

```