#d3 #js #computation 
### Some summary statistics
``` JAVASCRIPT
[d3.min(values), d3.median(values), d3.max(values)]
```
#### Compute extents of data
```Javascript
d3.extent(data, d => d.date)
```
![[Pasted image 20240507104352.png]]
> Extent is used to return the minimum and maximum value in an array
> <span style="color:red">If the array is empty, it will return undefined as the output</span>
### Retrieve associated values
⚠️ Useful for debugging or deriving new [[Scales#Scales]]
```Javascript
x.domain() //Array(2) [0, 21]
x.range() //Array(2)[30, 640]
```
> Domain is the complete set of values (in the case of the example, 0 to 21)
> Range 
> Range defines how the scale maps input values to visual representations
