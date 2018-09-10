# Cache API

This page walks you through how to get started using Fly’s Cache API - an API for accessing a regional, volatile cache. 

### Example use cases:  

- cache personalization data geographically close to individual users 
- cache data from an external API  
- cache large images and videos 

## Overview 

Caching is one of the main features of Fly Edge Apps that make them fast. The idea behind caching is to reduce the amount of work that your app’s server has to do. Fly was designed to speed up the process of making HTTP requests and delivering data from the server to your user. Alternatively, our Cache API was introduced for the purpose of eliminating those requests in the first place. 

## How does it work? 

The major difference between caching with Fly and caching with other CDN’s is - *simplicity*. Setting it up is as easy as writing a few lines of JavaScript.  

Fly's cache is available to you as a key-value store. You simply set a key, assign a value for that key, and (optionally) set a TTL. Data stored in `@fly/cache` can have an associated per-key time to live (TTL), and we will evict key data automatically after the elapsed TTL. We will also evict unused data when we need to reclaim space.

## Before you begin

Make sure you have Fly installed: `npm install -g @fly/fly` 

## Import the cache library 

`import cache from “@fly/cache”` 

# List of functions associated with cache

## set 

Set a value at the specified key, with an optional ttl. 

`await cache.set(“test-key”, “test-value”, ttl)` 

This function takes 3 parameters: 

- **key**: string  
The key to add or overwrite 

- **value**: string or ArrayBuffer  
Data to store at the specified key, up to 2MB 

- **ttl** (optional): number  
Time to live (a number in seconds) 

And returns a Promise (boolean) - true if the set was successful. 

## get

Get an ArrayBuffer value (or null) from the cache. 

`await cache.get(“test-key”)` 

This function takes 1 parameter:  

- **key**: string  
The key to get

And returns a Promise (ArrayBuffer) - raw bytes stored for provided key or null if empty. 

## getString 

Get a string value (or null) from the cache. 

`await cache.getString(“test-key”)` 

This function takes 1 parameter:  

- **key**: string  
The key to get 

And returns a Promise (any) - data stored at the key, or null if none exists. 

## expire 

Add or overwrite a key's time to live (ttl). 

`await cache.expire(“test-key”, ttl)` 

This function takes 2 parameters: 

- **key**: string  
The key to modify 

- **ttl**: number  
Expiration time remaining in seconds 

And returns a Promise (boolean) - true if ttl was successfully updated. 

## del 

Deletes the value (if any) at the specified key. 

`await cache.del(“test-key”)` 

This function takes 1 parameter: 

- **key**: string  
Key to delete 

And returns a Promise (boolean) - true if delete was successful.

## setTags 

Replace tags for a given cache key. 

`await cache.setTags(“key”, [“tag1”, "tag2", "tag3"])` 

This function takes 2 parameters: 

- **key**: string  
The key to modify 

- **tags**: string[]  
Tags to apply to key 

And returns a Promise (boolean) true if tags were successfully updated. 

## purgeTag 

Purges all cache entries with the given tag. 

`await cache.purgeTag(“tag”)` 

This function takes 1 parameter: 

- **tag**: string  
Tag to purge 

And returns a Promise (string[]). 

## How to implement caching into your app 

The first thing your app should do is check whether or not you already have the user's request cached. For example, if your user is requesting a video from your app, you’ll want them to first check the cache for that video, rather than loading an entirely fresh video from the server, at the risk of high latency.  

You’ll do this by checking the cache for the **key** associated with that video, and calling the `get()` function (because we’re looking for a binary value).  

Your key will be a string equal to whatever unique identifier you want to assign to the value (the video). 

```javascript 
import cache from “@fly/cache”
 
fly.http.respondWith(async function(){ 
    let key = "www.example.com/videos/header-video"
    let resp = await cache.get( key ) 

    if( resp ) {  
        resp.headers.set("Fly-Cache", "HIT")  
        return new Response(resp) 
    } else { 
        console.log(“Sorry, nothing is in the cache at that key”) 
    } 
}) 
```  

The key here is the url where the video lives. There’s nothing special about this key, just a unique identifier that we’re choosing to associate with our video caching and retrieving.

If we find a value at that key, then we’ll send it to the user ... and that’s it! The user will quickly receive the cached version of this video. They’ll be able to click play and almost *instantly* be able to start watching. 

Notice the importance of the async/await keywords here. All Fly requests use the modern async/await syntax introduced in ES2017.
 
*Note, if we were searching for cached data in the form of HTML text, we’d want to use the `getString()` function.*

If we don't find a value at the requested key, we need to get it and set it in the cache. So, our function would look something like this instead: 

```javascript 
import cache from “@fly/cache”

fly.http.respondWith(async function(){ 
    let key = "www.example.com/videos/header-video"
    let resp = await cache.get( key )

    if( resp ) {  
        resp.headers.set("Fly-Cache", "HIT")  
        return new Response(resp) 
    } else { 
        resp = await fetch(key) 
        await cache.set(key, resp, { ttl: 3600 }) 
        resp.headers.set("Fly-Cache", "MISS")  
        return new Response(resp) 
    } 
}) 
```




