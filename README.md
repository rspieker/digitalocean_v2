#Digital Ocean API v2

Full client-side javascript implementation of the [Digital Ocean v2 (beta) API](https://developers.digitalocean.com/v2/).

##Token
You first need to create a token for your account, this token is then provided to the library.
```javascript
DOv2.token('<your token here>');
```

##Implementation Basics
A great deal of effort has been put in by Digital Ocean to implement a consistent API, this implementation honors that by providing a very uniform interface, matching the entire API in the same consistent manner.


###Main Interfaces
All of the (current) Digital Ocean API main interfaces are represented by an object of its own, all of these implement the `list` method which is used to obtain the overview of items in that interface.


###Method signature
All methods follow the same simple convention; the last argument is always the callback function. The majority of methods does not require any other arguments. If no callback was provided an error is thrown:
```
//  invoking the droplet.rename method without callback
Error: No callback function provided for rename method
```
If any argument is required and a callback function is provided, the callback will be invoke with the error argument populated with an object indicating what is missing.
```json
//  invoking the droplet.rename method without name
{ id: 'error_name', message: 'Missing argument(s): "name" for rename method' }
```


###Callback signature
Our callback signature follows the same syntax commonly used for nodejs async functions: `function(error, result [, next])`.
- `error`: If there are any errors, the error argument will be 'true-ish' (not `null`, which is the default value), this is typically an object containing an id and a message.
- `result`: If there are no errors, the result argument is populated with either an array (for `list` calls) or an object (for individual item calls)
- `next`: If the API responded with an indication that there are more items in the list, the `next` argument will be a function which can be called (with one argument, a callback function), the `next` argument will be empty if there is no more to fetch for the call.


###CamelCase
Our eyes bleed when using an Object Oriented notation which uses underscores in method names, hence we provide a lowerCamelCased API in our implementation, both in execution and response, but never in response values.



##Implementation
The library tries to stay out of the global scope as much as possible, the only 'polution' is the `DOv2` object, this contains all of the functionality.


###Dependencies
While a stand-alone XMLHTTPRequest implementation could've been written, I chose the Konflux library, it's mine, so why not plug it ;-)

####Token
A token is always required, there are no calls that go without.
```javascript
DOv2.token('<your token here>');
```

####Konflux
You can [get your copy of konflux here](http://build.konfirm.net) (I developed against the 'develop' build). Make sure to have the proper order in your source, konflux first, digitalocean.js after.
In case you're wondering if it sports a 'Document Ready' event:
```javascript
kx.ready(function(){
	DOv2.token('<insert token>');

	DOv2.Actions.list(function(error, result, next){
		if (error)
			throw new Error(error);

		console.log(result);

		if (next)
			console.log('And there is even more');
	});
```


###Actions


###Regions
Regions is a simple API call, listing all the available regions
```javascript
DOv2.Regions.list(function(error, result, next){
	if (error)
		throw new Error(error);

	//  result is an Array containing one Object per region
	result.map(function(region){
		console.log(region);
	});
});
```
Output will be similar to:
```json
[
	{
		available: true,
		features: [
			"virtio", "backups"
		],
		name: "New York 1",
		sizes: [
			"512mb", "1gb", ..., "64gb"
		],
		slug: "nyc1"
	}
	,
	...
]
```
