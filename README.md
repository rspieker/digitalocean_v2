#Digital Ocean API v2

Full client-side javascript implementation of the [Digital Ocean v2 (beta) API](https://developers.digitalocean.com/v2/).

##Token
You first need to create a token for your account, this token is then provided to library.
```
DOv2.token('<your token here>');
```

##Regions
Regions is a simple API call, listing all the available regions
```
DOv2.Regions.list(function(error, result, next){
	if (error)
		throw new Error(error);

	//  result is an Array containing one Object per region
	result.map(function(region){
		console.log(region);
	});
});
```
