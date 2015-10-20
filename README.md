#Digital Ocean API v2

Full client-side javascript implementation of the [Digital Ocean v2 (beta) API](https://developers.digitalocean.com/v2/).

##Implementation Basics
A great deal of effort has been put in by the Digital Ocean devs to implement a consistent API, this implementation honors that by providing a very uniform interface, matching the entire API in the same consistent manner.


###Main Interfaces
All of the (current) Digital Ocean API main interfaces are represented by an object of its own, all of these implement the `list` method which is used to obtain the overview of items in that interface.


###Method signature
All methods follow the same simple convention; the last argument is always the callback function. The majority of methods does not require any other arguments. If no callback was provided an error is thrown:
```javascript
//  invoking the droplet.rename method without callback
Error: No callback function provided for rename method
```
If any argument is required and a callback function is provided, the callback will be invoke with the error argument populated with an object indicating what is missing.
```javascript
//  invoking the droplet.rename method without name
{ id: 'error_name', message: 'Missing argument(s): "name" for rename method' }
```


###Callback signature
Our callback signature follows the same syntax commonly used for nodejs async functions: `function(error, result [, next])`.
- `error`: If there are any errors, the error argument will be `true`-ish (not `null`, which is the default value), this is typically an object containing an id and a message.
- `result`: If there are no errors, the result argument is populated with either an array (for `list` calls) or an object (for individual item calls)
- `next`: If the API responded with an indication that there are more items in the list, the `next` argument will be a function which can be called (with one argument, a callback function), the `next` argument will be empty if there is no more to fetch for the call.


###CamelCase
Our eyes bleed when using an Object Oriented notation which has underscores in method names, hence we provide a lowerCamelCased API in our implementation, both in execution and response keys, but never in response values.


###Limits
####Rate-limit
The API is reasonably rate-limited to 1200 request per hour, if you exceed this, you will receive an error about having reached the limits (I personally believe congratulations would've in order here, as I never ran into these limits developing the library, which required a lot of requests in order to test).

####Item-limit
The Digital Ocean devs have implemented a very sane default of 25 items per page, I have not implemented means to change this. Please open an issue if you need to influence the default (I will be curious for the 'why', so you might add it to the request upfront).


##Token
You first need to [create a token](https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-api-v2) for your account, this token is then provided to the library.
I am inclined to provide a proper example which asks for your API token and then stores it in localStorage, of course you could hardcode your token (and push it onto your production website for the world to play with your instances), but I *strongly urge you not to* (if you insist on doing this, please make sure to give me the link too).

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
####Getting an overview
All API endpoints have a `list` method, which obtains an overview (paginated if needed) of the items in the endpoint.
Lists would be your basic starting point in order to find out more detailed information such as ID's and names.
- [x] `Actions.list` - Obtain the actions of your account ([DO Reference: List all Actions](https://developers.digitalocean.com/v2/#list-all-actions))
- [x] `Domains.list` - Obtain the domains in your account ([DO Reference: List all Domains](https://developers.digitalocean.com/documentation/v2/#list-all-domains))
- [x] `Droplets.list` - Obtain the droplets in your account ([DO Reference: List all Droplets](https://developers.digitalocean.com/v2/#list-all-droplets))
- [x] `Images.list` - Obtain the available images, both your own images as the ones provided by Digital Ocean ([DO Reference: List all Images](https://developers.digitalocean.com/v2/#list-all-images))
- [ ] `Keys.list` - *TODO*
- [x] `Regions.list` - Obtain all available regions where you can create new Droplets ([DO Reference: List all Regions](https://developers.digitalocean.com/v2/#list-all-regions))
- [x] `Sizes.list` - Obtain all available sizes for Droplets ([DO Reference: List all Sizes](https://developers.digitalocean.com/v2/#list-all-sizes))

#####List Example
```javascript
DOv2.Droplets.list(function(error, result, next){
	if (!error)
		console.log(result);  //  Contains an array of Droplet items

	if (next)
	{
		//  next is a function which expects a callback function and will retrieve the next items for this call
		//  in this case: Droplets
		console.log('and there are more..');
	}
});
```


####API
- [x] Actions
	- [x] `list`
	- [x] `id`
- [x] Domains
	- [x] `list`
	- [ ] `fetch` *TODO*
	- [ ] `create` *TODO*
	- [ ] `destroy` *TODO*
- [ ] DomainRecords *TODO*
- [x] Droplets
	- [x] `list`
	- [x] `id`
	- [ ] `create` *TODO*
	- [x] Droplet Methods
		- [x] `kernels` - obtain a list of all available kernels for this droplet
		- [x] `snapshots` - obtain a list of all available snapshots of this droplet
		- [x] `backups` - obtain a list of all available backups of this droplet
		- [x] `actions` - obtain a list of all actions for this droplet
		- [x] `destroy` - destroy the droplet
		- [x] `reboot` - reboot the droplet
		- [x] `powerCycle` - turn the droplet off and on again (like successively calling `powerOff` and then `powerOn`)
		- [x] `shutdown` - shut down a droplet
		- [x] `powerOn` - power on the droplet
		- [x] `powerOff` - turn the droplet off
		- [x] `passwordReset` - request a new password (this will be sent by e-mail, not become available in the API)
		- [x] `resize` - resize the droplet to another size *(requires the new size as first argument)*
		- [x] `rebuild` - rebuild the droplet from an image *(requires an image id for your own images or a 'slug' (name) for DO's images as first argument)*
		- [x] `rename` - rename the droplet *(requires a string name as first argument)*
		- [x] `enableIPv6` - enable IPv6 on the droplet _(will only work if IPv6 is available in the region where the droplet resides)_
		- [x] `disableBackups` - disable backups of the droplet
		- [x] `enablePrivateNetworking` - enable private networking for the droplet
- [x] Images
- [ ] Keys *TODO*
- [x] Regions
- [x] Size
