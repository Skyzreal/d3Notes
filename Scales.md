#js #d3 #visual 
> Most fundamental data graphics tool. A D3 scale is a function. 
```Javascript
fruits = [
  {name: "🍊", count: 21},
  {name: "🍇", count: 13},
  {name: "🍏", count: 8},
  {name: "🍌", count: 5},
  {name: "🍐", count: 3},
  {name: "🍋", count: 2},
  {name: "🍎", count: 1},
  {name: "🍉", count: 1}
]
```
![[Pasted image 20240507110655.png]]
> We can then retrieve all the fruits by either their name (nominal) or their count (quantitative) 
```Javascript
fruits.map(d => d.count) // quantitative
fruits.map(d => d.name) // nominal
```
> *Quantitative* counts the amount of fruits ![[Pasted image 20240507111039.png]]
> *Nominal* refers to the name![[Pasted image 20240507111053.png]]
#### Here is a bar chart example
> Name dimension is mapped to the y-position, while the count dimension is mapped to the x-position

```Javascript
x = d3.scaleLinear()
	.domain([0, d3.max(fruits, d => d.count)])
	.range([margin.left, width - margin.right])
	.interpolate(d3.interpolateRound)
// x = f(n)
y = d3.scaleBand()
	.domain(fruits.map(d=> d.name))
	.range([margin.top, height - margin.bottom])
	.padding(0.1)
	.round(true)
// y = f(i)
```
![[Pasted image 20240507113926.png]]
>This uses *method chaining* to work, but there is a longhand equivalent for those that prefer:
``` Javascript
{
  const x = d3.scaleLinear();
  x.domain([0, d3.max(fruits, d => d.count)]);
  x.range([margin.left, width - margin.right]);
  x.interpolate(d3.interpolateRound);
  return x;
}
```
>Using D3 scale returns a visual value (x or y position), that corresponds to the abstract value (*count* for example)
```Javascript
y("🍇") //46
x(21) //640
x(0) //30
```
>1. The y coordinate for the name grapes
>2. The x coordinate where count = 21
>3. The x coordinate where count = 0
>Note: By convention, most D3 charts apply margins. So x where the count is 0 does not return 0, it returns the size of the left margin![[Pasted image 20240507134507.png]]
>So, if we want the real width of the bar where count = 21, we must do:
```Javascript
x(21) - x(0) //610
```
>To know the height of each bar, we can do:
```Javascript
y.bandwidth() //20
```
### Embedded expressions
>Hypertext literal (htl): convenient way to render HTML or SVG markup
>Dynamic expression, such as the width of the bar, can be embedded in literals as ${...}
```Javascript
htl.html`<svg viewBox="0 0 ${width} 33" style="max-width: ${width}px; font: 10px sans-serif; display: block;">
  <rect fill="steelblue" x="${x(0)}" width="${x(count) - x(0)}" height="33"></rect>
  <text fill="white" text-anchor="end" x="${x(count)}" dx="-6" dy="21">${count}</text>
</svg>`
```
![[Pasted image 20240507140329.png]]
>Another example, this time using arrays of DOM elements
```Javascript
htl.html`<svg viewBox="0 ${margin.top} ${width} ${height - margin.top}" style="max-width: ${width}px; font: 10px sans-serif;">
  <g fill="steelblue">
    ${fruits.map(d => htl.svg`<rect y="${y(d.name)}" x="${x(0)}" width="${x(d.count) - x(0)}" height="${y.bandwidth()}"></rect>`)}
  </g>
  <g fill="white" text-anchor="end" transform="translate(-6,${y.bandwidth() / 2})">
    ${fruits.map(d => htl.svg`<text y="${y(d.name)}" x="${x(d.count)}" dy="0.35em">${d.count}</text>`)}
  </g>
</svg>`
```
![[Pasted image 20240507140827.png]]
>Another example, but with axes to improve the readability of the charts
```Javascript
htl.html`<svg viewBox="0 0 ${width} ${height}" style="max-width: ${width}px; font: 10px sans-serif;">
  <g fill="steelblue">
    ${fruits.map(d => htl.svg`<rect y="${y(d.name)}" x="${x(0)}" width="${x(d.count) - x(0)}" height="${y.bandwidth()}"></rect>`)}
  </g>
  <g fill="white" text-anchor="end" transform="translate(-6,${y.bandwidth() / 2})">
    ${fruits.map(d => htl.svg`<text y="${y(d.name)}" x="${x(d.count)}" dy="0.35em">${d.count}</text>`)}
  </g>
  ${d3.select(htl.svg`<g transform="translate(0,${margin.top})">`)
    .call(d3.axisTop(x))
    .call(g => g.select(".domain").remove())
    .node()}
  ${d3.select(htl.svg`<g transform="translate(${margin.left},0)">`)
    .call(d3.axisLeft(y))
    .call(g => g.select(".domain").remove())
    .node()}
</svg>`
```
![[Pasted image 20240507141721.png]]
>D3 axes require selections (which are used in the example above)
>Add the axis, we first create a G element. Then, we select that element using d3.select(), then we invoke selection.call to render the axis into the G element, and again to remove the domain path. 
>Last, we retrieve the G element by calling selection.node, and embed it in the outer literal
Scales can also be used for <span style="font-size: 14px; font-weight: bold; background: linear-gradient(to right, red, orange, yellow, green, blue, indigo, violet); -webkit-background-clip: text; color: transparent;">c<span style="animation: rainbow 2s linear infinite;">o</span>l<span style="animation: rainbow 2s linear infinite 0.2s;">o</span>r</span> (see all color schemes at [[Imports]])
```Javascript
color = d3.scaleSequential()
    .domain([0, d3.max(fruits, d => d.count)])
    .interpolator(d3.interpolateBlues)
```

>The code above defines a sequential scale. 
>Interpolator takes a value between 0 and 1 and returns the corresponding visual value (which is usually a [color scheme]([[Imports]]))
Examples:
```Javascript
color(0) //rgb(247, 251, 255)
color(21) //rgb(8, 48, 107)
```
Now we can add <span style="font-size: 14px; font-weight: bold; background: linear-gradient(to right, red, orange, yellow, green, blue, indigo, violet); -webkit-background-clip: text; color: transparent;">c<span style="animation: rainbow 2s linear infinite;">o</span>l<span style="animation: rainbow 2s linear infinite 0.2s;">o</span>r</span> to our bar chart
```Javascript
htl.html`<svg viewBox="0 ${margin.top} ${width} ${height - margin.top}" style="max-width: ${width}px; font: 10px sans-serif;">
  <g>
    ${fruits.map(d => htl.svg`<rect fill="${color(d.count)}" y="${y(d.name)}" x="${x(0)}" width="${x(d.count) - x(0)}" height="${y.bandwidth()}"></rect>`)}
  </g>
  <g text-anchor="end" transform="translate(-6,${y.bandwidth() / 2})">
    ${fruits.map(d => htl.svg`<text fill="${d3.lab(color(d.count)).l < 60 ? "white" : "black"}" y="${y(d.name)}" x="${x(d.count)}" dy="0.35em">${d.count}</text>`)}
  </g>
  ${d3.select(htl.svg`<g transform="translate(${margin.left},0)">`)
    .call(d3.axisLeft(y))
    .call(g => g.select(".domain").remove())
    .node()}
</svg>`
```
![[Pasted image 20240507142806.png]]


