#Digital Ocean API v2

Full client-side javascript implementation of the [Digital Ocean v2 (beta) API](https://developers.digitalocean.com/v2/).

##Implementation Basics
A great deal of effort has been put in by the Digital Ocean devs to implement a consistent API, this implementation honors that by providing a very uniform interface, matching the entire API in the same consistent manner.


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
- `error`: If there are any errors, the error argument will be `true`-ish (not `null`, which is the default value), this is typically an object containing an id and a message.
- `result`: If there are no errors, the result argument is populated with either an array (for `list` calls) or an object (for individual item calls)
- `next`: If the API responded with an indication that there are more items in the list, the `next` argument will be a function which can be called (with one argument, a callback function), the `next` argument will be empty if there is no more to fetch for the call.


###CamelCase
Our eyes bleed when using an Object Oriented notation which uses underscores in method names, hence we provide a lowerCamelCased API in our implementation, both in execution and response, but never in response values.


###Limits
####Rate-limit
The API is reasonably rate-limited to 1200 request per hour, if you exceed this, you will receive an error about having reached the limits (I personally believe congratulations would've in order here, as I never ran into these limits developing the library, which required a lot of requests in order to test).

####Item-limit
The Digital Ocean devs have implemented a very sane default of 25 items per page, I have not implemented means to change this. Please open an issue if you need to influence the default (I will be curious for the 'why', so you might add it to the request upfront).


##Token
You first need to create a token for your account, this token is then provided to the library.
I am inclined to provide a proper example which asks for your API token and then stores it in localStorage, of course you could hardcode your token (and push it onto your production website for the world to play with your instances), but I *strongly urge you not to*.

```javascript
//  see of we can obtain the token from localStorage
var token = localStorage.getItem('DOAPIToken');

//  if no token is found, we ask for it
if (!token)
{
	token = prompt('Your DigitalOcean API token please');

	//  ask permission to store the token in localStorage
	if (confirm('Do you want to save this token in localStorage?'))
		localStorage.setItem('DOAPIToken', token);
}

//  and finally pass on the token to the DOv2 instance.
DOv2.token(token);
```




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
	//  please refer to the elaborate example on top of this page on how to safely have the convenience of being able
	//  to refresh without entering the token or (worse!) hardcoding it.
	DOv2.token(token);

	//  your DOv2 calls follow here...
});
```


###Usage
####Actions
Actions is an API call which lists all (limited to 25 per call) actions which have taken place on your account.
```javascript
//  Obtain the first bunch of actions
DOv2.Actions.list(function(error, result, next){
	if (error)
		throw new Error(error);

	//  result is an Array containing one Object per action
	console.log(result);

	if (next)
		console.log('And there is more...');
});
```
Output will be similar to:
```json
[
	{
		id: 12345678,
		status: "completed",
		type: "reboot",
		startedAt: "2014-07-01T08:31:44Z",
		completedAt: "2014-07-01T08:32:06Z",
		resourceId: 1859761,
		resourceType: "droplet",
		region: "nyc1"
	}
	,
	...
]
```
#####id method
Actions can also be retrieved individually by their id's
```javascript
DOv2.Actions.id(12345678, function(error, result){
	if (error)
		throw new Error(error);

	//  result is a single Object
	console.log(result);

	//  no next, there can be only one...
});
```
Output will be similar to:
```json
{
	id: 12345678,
	status: "completed",
	type: "reboot",
	startedAt: "2014-07-01T08:31:44Z",
	completedAt: "2014-07-01T08:32:06Z",
	resourceId: 1234567,
	resourceType: "droplet",
	region: "nyc1"
}
```

####Regions
Regions is a simple API call, listing all the available regions
```javascript
DOv2.Regions.list(function(error, result, next){
	if (error)
		throw new Error(error);

	//  result is an Array containing one Object per region
	console.log(result);

	if (next)
		console.log('And there is more...');
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


####Sizes
Sizes is an API call which lists all (limited to 25 per call) available sizes (for droplets).
```javascript
DOv2.Sizes.list(function(error, result, next){
	if (error)
		throw new Error(error);

	//  result is an Array containing one Object per size
	console.log(size);

	if (next)
		console.log('And there is more...');
});
```
Output will be similar to:
```json
[
	{
		disk: 20,
		memory: 512,
		priceHourly: "0.00744",
		priceMonthly: "5.0",
		regions: [
			"nyc1", "sgp1", ..., "nyc2"
		],
		slug: "512mb",
		transfer: 1000,
		vcpus: 1
	}
	,
	...
]
```
