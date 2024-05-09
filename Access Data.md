#d3 #js #csv 
### Retrieve data from CSV
```Javascript
FileAttachment("temperature.csv")
```
 ![[Pasted image 20240507102039.png]]
> Returns a handle (blob, buffer), that allows to choose the desired representation
#### Return content as a string
```Javascript
FileAttachment("temperature.csv").text()
```
`date, temperature
`2011-10-01,62.7
`2011-10-02,59.9 
`2011-10-03,59.1`
>Displays the file content as a string
#### Return content as objects
> More useful than previous methods
```Javascript
FileAttachment("temperature.csv"). csv()
```
![[Pasted image 20240507102535.png]]
>Must be careful, objects properties will be by default strings
>In this example, it would be more beneficial for date to be a date object and temperature to be a float
#### Simple solution
>To render the object properties as something other than string, must use typed
``` Javascript
FileAttachment("temperature.csv"). csv({typed:true})
```
![[Pasted image 20240507102759.png]]
>This way, the properties are not strings anymore
#### Explicit conversion
>If we are not sure that our data is compatible with d3.autoType (typed:true), we should parse the data explicitly
```Javascript
data = {
	const text = await FileAttachment("temperature.csv").text();
	const parseDate = d3.utcParse("%Y-%m-%d");
	return d3.csvParse(text, ({date, temperature}) => ({
	date: parseDate(date),
	temparature:+temperature
	}))
}
```
> See [[Variables and block]] if don't know how to make a block of code
> Now, the data is conveniently represented and we can get to work
>
