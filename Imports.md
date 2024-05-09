#d3 #js #imports #link
>Accessible d3 gallery [here](https://d3-graph-gallery.com/)
>Accessible color scheme [here]([Color Schemes / D3 | Observable (observablehq.com)](https://observablehq.com/@d3/color-schemes))
>
### Histogram
``` Javascript
import {Histogram} from "@d3/histogram"
```
![[Pasted image 20240507100335.png]]
#### Modify histogram
```Javascript
Histogram(values, {width, height:200, color: "steelblue"})
``` 
![[Pasted image 20240507100842.png]]
> It is now shorter on the y-axis and blue
#### Inject dynamic data
```Javascript
Histogram(randoms, {width, height:200, color:"steelblue", domain:[-10, 10]})
```
![[Pasted image 20240507101402.png]]
> Now displays dynamically. *Domain* [-10, 10] is important to fit the data with the change of distribution

> 
