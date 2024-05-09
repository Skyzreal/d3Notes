#js #d3 #visual #shapes 
### Path
SVG Path Commands
<table style="border-collapse: collapse; width: 100%; margin-bottom: 20px;"> <thead> <tr> <th style="border: 1px solid #ddd; background-color: #f2f2f2; padding: 8px; text-align: left;">Command</th> <th style="border: 1px solid #ddd; background-color: #f2f2f2; padding: 8px; text-align: left;">Description</th> </tr> </thead> <tbody> <tr> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Mx,y</td> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Move to the specified point [x, y]</td> </tr> <tr> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Lx,y</td> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Draw a line to the specified point [x, y]</td> </tr> <tr> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">hx</td> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Draw a horizontal line of length x</td> </tr> <tr> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">vy</td> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Draw a vertical line of length y</td> </tr> <tr> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">z</td> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Close the current subpath</td> </tr> </tbody> </table>
More information [here](https://www.w3.org/TR/SVG/paths.html#TheDProperty) for more in-depth
### Charts
Now, let's imagine we want to see several years of Apple stock price as a linechart
First, we read the data from the file ([[Access Data]]):
```Javascript 
data = FileAttachment("appl=boiler.csv").csv({typed:true})
```
![[Pasted image 20240508073347.png]]
>Snippet of the array
Then, we define two scales, y and x:
```Javascript
//define x
x = d3.scaleUtc()
    .domain(d3.extent(data, d => d.date))
    .range([margin.left, width - margin.right])
//define y
y = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.upper)])
    .range([height - margin.bottom, margin.top])
```
Now, we can move to our first point. So we have to start with *Mx, y* to move to the first point, followed repeatedly by *Lx, y* to draw a line to each subsequent point
``` Javascript
{
  let path = `M${x(data[0].date)},${y(data[0].close)}`;
  for (let i = 1; i < data.length; ++i) {
    path += `L${x(data[i].date)},${y(data[i].close)}`;
  }
  return path;
}
//Snippet of the return: "M40,177.39579502267992L40.20851769911505" (In reality, is much larger)
```
>It first initializes a variable path with the starting point, using our previous y and x scales
>Then, it iterates through the remaining data points in the data array and appends a line (*L*) to the path string, which specifies the coordinate of the next point (Note: Again, it uses the x and y scales to convert the date and close values of each data point)
>After constructing the path string after looping through everything, it returns the path
Here's a simple to understand the result statement (useful for debugging):
```
"M10,20L30,40L50,60"
```

<table style="border-collapse: collapse; width: 100%; margin-bottom: 20px;"> <thead> <tr> <th style="border: 1px solid #ddd; background-color: #f2f2f2; padding: 8px; text-align: left;">Path String</th> <th style="border: 1px solid #ddd; background-color: #f2f2f2; padding: 8px; text-align: left;">Description</th> </tr> </thead> <tbody> <tr> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">M10,20</td> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Move the cursor to the point (10, 20).</td> </tr> <tr> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">L30,40</td> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Draw a line to (30, 40).</td> </tr> <tr> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">L50,60</td> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Draw another line to (50, 60).</td> </tr> </tbody> </table>

>Then now, we can use d3.line to return a default line generator. 
>We can configure the line with functions to return the x and y position for a given data point *d*
```Javascript
line = d3.line()
    .x(d => x(d.date))
    .y(d => y(d.close))
```
>Passing the line generator the data returns the corresponding SVG path data string 
```Javascript
line(data)
```
>Finally, creating the SVG
```Javascript
htl.html`<svg viewBox="0 0 ${width} ${height}">
  <path d="${line(data)}" fill="none" stroke="steelblue" stroke-width="1.5" stroke-miterlimit="1"></path>
  ${d3.select(htl.svg`<g>`).call(xAxis).node()}
  ${d3.select(htl.svg`<g>`).call(yAxis).node()}
</svg>`
```
![[Pasted image 20240508081449.png]]
>Note that here, to avoid misleading spikes, the [miter limit]([stroke-miterlimit - SVG: Scalable Vector Graphics | MDN (mozilla.org)](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/stroke-miterlimit)) has been set to 100%
For an area chart, there's a similar d3.arena:
```Javascript
area = d3.area()
    .x(d => x(d.date))
    .y0(y(0))
    .y1(d => y(d.close))
```
>Then we can set the SVG:
```Javascript
htl.html`<svg viewBox="0 0 ${width} ${height}">
  <path fill="steelblue" d="${area(data)}"></path>
  ${d3.select(htl.svg`<g>`).call(xAxis).node()}
  ${d3.select(htl.svg`<g>`).call(yAxis).node()}
</svg>`
```
![[Pasted image 20240508081511.png]]
>If, instead, we need an area with a variable baseline like a [stacked area chart]([Stacked Area Chart / D3 | Observable (observablehq.com)](https://observablehq.com/@d3/stacked-area-chart)) or [streamgraph]([Stacked Area Chart, Streamgraph / D3 | Observable (observablehq.com)](https://observablehq.com/@d3/streamgraph)) pass *area.y0* a function instead
```Javascript
areaBand = d3.area()
    .x(d => x(d.date))
    .y0(d => y(d.lower))
    .y1(d => y(d.upper))
```
Here's the SVG for it:
``` Javascript
htl.html`<svg viewBox="0 0 ${width} ${height}">
  <path d="${areaBand(data)}" fill="steelblue"></path>
  ${d3.select(htl.svg`<g>`).call(xAxis).node()}
  ${d3.select(htl.svg`<g>`).call(yAxis).node()}
</svg>`
```
![[Pasted image 20240508081528.png]]
>To complete the display of Bollinger bands, we can show the central moving average and daily closing price
>So we need to use multiple path for multiple colors
```Javascript
lineMiddle = d3.line()
    .x(d => x(d.date))
    .y(d => y(d.middle))
```
>And the SVG:
```Javascript
htl.html`<svg viewBox="0 0 ${width} ${height}">
  <path d="${areaBand(data)}" fill="#ddd"></path>
  <g fill="none" stroke-width="1.5" stroke-miterlimit="1">
    <path d="${lineMiddle(data)}" stroke="#00f"></path>
    <path d="${line(data)}" stroke="#000"></path>
  </g>
  ${d3.select(htl.svg`<g>`).call(xAxis).node()}
  ${d3.select(htl.svg`<g>`).call(yAxis).node()}
</svg>`
```
![[Pasted image 20240508081801.png]]
>Note: Lines and areas are designed to work together, you can derive the line corresponding to an area's baseline or topline by calling area.lineY0 or area.lineY1. It's useful for decorating an area with a top or bottom stroke
``` Javascript
htl.html`<svg viewBox="0 0 ${width} ${height}">
  <path d="${areaBand(data)}" fill="#ddd"></path>
  <g fill="none" stroke-width="1.5" stroke-miterlimit="1">
    <path d="${areaBand.lineY0()(data)}" stroke="#00f"></path>
    <path d="${areaBand.lineY1()(data)}" stroke="#f00"></path>
  </g>
  ${d3.select(htl.svg`<g>`).call(xAxis).node()}
  ${d3.select(htl.svg`<g>`).call(yAxis).node()}
</svg>`
```
![[Pasted image 20240508081931.png]]
### Arc charts
>[Pie charts](https://observablehq.com/@d3/pie-chart), [donut charts](https://observablehq.com/@d3/donut-chart) and [sunbursts](https://observablehq.com/@d3/sunburst)
>Similar to how lines are areas are configured by x and y, an arc is configured by *innerRadius*, *outerRadius* and *endAngle* (angle with radian). 
```Javascript
arc = d3.arc()
    .innerRadius(210)
    .outerRadius(310)
    .startAngle(([startAngle, endAngle]) => startAngle)
    .endAngle(([startAngle, endAngle]) => endAngle)
```
>The arc generator above is configured to accept an array(startAngle, endAngle)
```Javascript
arc([Math.PI / 2, Math.PI]) // from 90° to 180°
```
>It will return something that looks like this:
>"M310,0A310,310,0,0,1,0,310L0,210A210,210,0,0,0,210,0Z"
<table style="border-collapse: collapse; width: 100%; margin-bottom: 20px;"> <thead> <tr> <th style="border: 1px solid #ddd; background-color: #f2f2f2; padding: 8px; text-align: left;">Parameter</th> <th style="border: 1px solid #ddd; background-color: #f2f2f2; padding: 8px; text-align: left;">Description</th> </tr> </thead> <tbody> <tr> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">M310,0</td> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Move to the point (310,0)</td> </tr> <tr> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">A310,310,0,0,1,0,310</td> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Draw an elliptical arc from the current point to (0,310) with radii (310,310)</td> </tr> <tr> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">L0,210</td> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Draw a line to the point (0,210)</td> </tr> <tr> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">A210,210,0,0,0,210,0</td> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Draw another elliptical arc to close the path</td> </tr> <tr> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Z</td> <td style="border: 1px solid #ddd; padding: 8px; text-align: left;">Close the path</td> </tr> </tbody> </table>
And now for the SVG:
```Javascript
htl.html`<svg viewBox="-320 -320 640 640" style="max-width: 640px;">
  ${Array.from({length: n}, (_, i) => htl.svg`<path stroke="black" fill="${d3.interpolateRainbow(i / n)}" d="${arc([i / n * 2 * Math.PI, (i + 1) / n * 2 * Math.PI])}"></path>`)}
</svg>`
```
![[Pasted image 20240508095711.png]]
>Now, if we recall the [fruits]([[Scales]]) dataset, we can make it so each arc is proportional to its value
```Javascript
pieArcData = d3.pie()
    .value(d => d.count)
  (fruits)
```
>To produce a donut chart (with some padding and corner radius)
```Javascript
arcPie = d3.arc()
    .innerRadius(210)
    .outerRadius(310)
    .padRadius(300)
    .padAngle(2 / 300)
    .cornerRadius(8)
```
>And the SVG
```Javascript
htl.html`<svg viewBox="-320 -320 640 640" style="max-width: 640px;" text-anchor="middle" font-family="sans-serif">
  ${pieArcData.map(d => htl.svg`
    <path fill="steelblue" d="${arcPie(d)}"></path>
    <text fill="white" transform="translate(${arcPie.centroid(d).join(",")})">
      <tspan x="0" font-size="24">${d.data.name}</tspan>
      <tspan x="0" font-size="12" dy="1.3em">${d.value.toLocaleString("en")}</tspan>
    </text>
  `)}
</svg>`
```
![[Pasted image 20240508102714.png]]
