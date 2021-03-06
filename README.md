## Http GSC Plugin
This plugin allows you to do http requests within your gsc scripts.

Usage:
```
callback(content) // callback function receiving the request content as string
{
    iprintln("out:" + content); // print the result
}

...

httpget("127.0.0.1/test.php", ::callback);
```

httpget interprets the received contents as a string which can be read from the callback

#### Compiling:
This plugin is written in D. You can get the needed compiler [here](https://dlang.org/download.html) and the dub build system [here](https://code.dlang.org/download). To compile the plugin run `dub --arch=x86 --build=release` in the folder containing `package.json`.


# Examples
## Simply JSON HTTP GET Request

```
init()
{
	for(;;)
	{
		httpgetjson("http://freegeoip.net/json/8.8.8.8", ::callback); // get country for 8.8.8.8 every 5 seconds
		wait 5; 
	}	
}

callback(handle)
{
	if(handle == 0)
	{
		//error parsing json. Do something about this error case
		return;
	}
	res = jsongetstring(handle, "country_name");
	jsonreleaseobject(handle); // release the plugin internal json data
	iprintln(res); // prints country name
}                                                                 
```

## Get deeply nested data in a JSON HTTP GET Request

test.php:
```
{                                                                                         
    "success": {                                                                          
        "total": 1                                                                        
    },                                                                                    
    "contents": {                                                                         
        "quotes": [                                                                       
            {                                                                             
                "quote": "If you cannot do great things, do small things in a great way.",
                "length": "62",                                                           
                "author": "Napoleon Hill",                                                
                "tags": [                                                                 
                    "inspire",                                                            
                    "small-things"                                                        
                ],                                                                        
                "category": "inspire",                                                    
                "date": "2016-09-05",                                                     
                "title": "Inspiring Quote of the day",                                    
                "background": "https://theysaidso.com/img/bgs/man_on_the_mountain.jpg",   
                "id": "VO7q_Ezldx9gvzlO4jujJgeF"                                          
            }                                                                             
        ]                                                                                 
    }                                                                                     
}    
```

_somescript.gsc:
```

init()
{
	for(;;)
	{
		httpgetjson("http://127.0.0.1/test.php", ::callback);
		wait 5;
	}

	level thread onPlayerConnect();	
}

callback(handle)
{
	if(handle == 0)
	{
		//error parsing json. Do something about this error case
		return;
	}
	jsongetint(handle, "contents.quotes.size"); //returns count of array elements. In this case it is 1.
	jsonsetstring(handle, "contents.quotes.0.magic", "val"); // sets magic to "val"
	jsongetstring(handle, "contents.quotes.0.magic"); // returns "val"
	jsongetstring(handle, "contents.quotes.0.quote"); // returns "If you cannot do great things, do small things in a great way."
	jsonreleaseobject(handle);
}
```

## JSON HTTP POST Request

Authentication example

GSC:
```
onPlayerConnect()
{
	for(;;)
	{
		level waittill ( "connecting", player );
		
		sid = player getsteamid64();
		pid = player getplayerid64();
		name = player.name;
		httppostjson("http://test.local/playerjoins", 
					 "{'steamid64':" + sid + ", 'playerid64':" + pid +", 'name':'" + name + "'}", // JSON encoded post data
					 ::playerjoinsCallback,
					 player); // if a entity is provided "self" is set in the callback 
	
		player thread onPlayerSpawned();
	}
}

playerjoinsCallback(handle)
{
	self iprintln("joined"); // self is set to the connected player
	jsonreleaseobject(handle);
}
```


