#js #d3 #visual #animation 
>Animation commands attention
>Animation is not a single graphic, but a *sequence* of graphics over time. 
>This sequence is usually represented as a cell (or function) that returns the graphic for a given time *t*. For simplicity, normalized time is used, where *t=0* is the start of the animation and *t=1* is the end
```Javascript
reveal = path => path.transition()
    .duration(5000)
    .ease(d3.easeLinear)
    .attrTween("stroke-dasharray", function() {
      const length = this.getTotalLength();
      return d3.interpolate(`0,${length}`, `${length},${length}`);
    })
```
SVG:
```Javascript
htl.html`<svg viewBox="0 0 ${width} ${height}">
  <path d="${line(data)}" fill="none" stroke="steelblue" stroke-width="1.5" stroke-miterlimit="1" stroke-dasharray="${lineLength * t},${lineLength}"></path>
  ${d3.select(htl.svg`<g>`).call(xAxis).node()}
  ${d3.select(htl.svg`<g>`).call(yAxis).node()}
</svg>`
lineLength = htl.svg`<path d="${line(data)}">`.getTotalLength()
//lineLength = 1501.7098388671875
```
![[Pasted image 20240508111015.png]]
>This animation is available on the [Observable website]([Learn D3: Animation / D3 | Observable (observablehq.com)](https://observablehq.com/@d3/learn-d3-animation))

To assist with animation, D3 provides interpolators. The most generic accepts numbers, colors, strings-of-numbers, arrays and objects
```Javascript
strokeDasharray = d3.interpolate(`0,${lineLength}`, `${lineLength},${lineLength}`)
strokeDasharray(t) //"0,1501.7098388671875"
```
>You can either specify the interpolator (using [transition.attrTween](https://d3js.org/d3-transition/modifying#transition_attrTween)) or let D3 choose (using [transition.attr](https://d3js.org/d3-transition/modifying#transition_attr) or [transition.style](https://d3js.org/d3-transition/modifying#transition_style)). Being explicit allows more advance interpolation methods such as [zooming](https://observablehq.com/@d3/d3-interpolatezoom), [gamma-corrected RGB blending](https://d3js.org/d3-interpolate/color#interpolateColor_gamma), or even [shape blending](https://observablehq.com/@mbostock/hello-flubber).
```Javascript
ramp(d3.interpolateRgb("steelblue", "orange"))
```
![[Pasted image 20240508120345.png]]
```Javascript
ramp(d3.interpolateRgb.gamma(2.2)("steelblue", "orange"))
```
![[Pasted image 20240508120409.png]]
### Generators
Another great tool to control animation by Observable: [_generators_](https://observablehq.com/@observablehq/introduction-to-generators) 
When a generator cell yields a value, its execution is suspended until the next animation frame, up to sixty times per second
```Javascript
{
  replay2; // reference the button so this cell re-runs on click
  for (let i = 0, n = 300; i < n; ++i) {
    yield i;
  }
}
{
  replay2; // reference the button so this cell re-runs on click

  const path = htl.svg`<path d="${line(data)}" fill="none" stroke="steelblue" stroke-width="1.5" stroke-miterlimit="1">`;

  const chart = htl.html`<svg viewBox="0 0 ${width} ${height}">
    ${path}
    ${d3.select(htl.svg`<g>`).call(xAxis).node()}
    ${d3.select(htl.svg`<g>`).call(yAxis).node()}
  </svg>`;

  for (let i = 0, n = 300; i < n; ++i) {
    const t = (i + 1) / n;
    path.setAttribute("stroke-dasharray", `${t * lineLength},${lineLength}`);
    yield chart;
  }
}
```
![[Pasted image 20240508130702.png]]
>Small animation where the graph starts at the beginning and then progressively gets to the end. 

If your animation is simple and doesn't require smooth transitions, you can write the graphic once and leave it. Observable's dataflow makes it easy to adjust the graphic once and leave it. 
However, if your graphics is more complex and needs smooth transitions of efficient updates for better performance, then you should use transitions or generators.
You have the option to combine different approaches. For example, you can create a static graphic initially. However, you can also add functionality to transition to a different state when needed. This transition can be triggered by calling a method like `chart.update`, which adjusts the graphic based on a new set of data or parameters. In the provided code, this method is invoked when the selected value of a radio input changes. The graphic's structure remains consistent with earlier examples, even though the code uses `d3-selection` instead of HTML template literals. Here's an example:
```Javascript
chart = {
  const svg = d3.create("svg")
      .attr("width", width)
      .attr("height", height)
      .attr("viewBox", [0, 0, width, height])
      .attr("style", "max-width: 100%; height: auto; height: intrinsic;");

  const zx = x.copy(); // x, but with a new domain.

  const line = d3.line()
      .x(d => zx(d.date))
      .y(d => y(d.close));

  const path = svg.append("path")
      .attr("fill", "none")
      .attr("stroke", "steelblue")
      .attr("stroke-width", 1.5)
      .attr("stroke-miterlimit", 1)
      .attr("d", line(data));

  const gx = svg.append("g")
      .call(xAxis, zx);

  const gy = svg.append("g")
      .call(yAxis, y);

  return Object.assign(svg.node(), {
    update(domain) {
      const t = svg.transition().duration(750);
      zx.domain(domain);
      gx.transition(t).call(xAxis, zx);
      path.transition(t).attr("d", line(data));
    }
  });
}
```
![[Pasted image 20240508133110.png]]
![[Pasted image 20240508132937.png]]
>This example demonstrates another handy feature of D3 axes: by switching to [transition.call](https://d3js.org/d3-transition/control-flow#transition_call) instead of selection.call, the change in the x-axis is now animated rather than instantaneous and synchronized with the transitioning path.