# Cache API

This page walks you through how to get started using Fly’s Cache API - an API for accessing a regional, volatile cache. 

### Example use cases:  

- cache personalization data geographically close to individual users 
- cache data from an external API  
- cache large files such as images and videos 

## Overview 

Caching is one of the main features of Fly Edge Apps that make them fast. The idea behind caching is that is significantly reduces the amount of work that your app’s server has to do, ultimately speeding your app up for those visiting it. 

Fly was designed to speed up the process of making HTTP requests and delivering data from the server to your user. Alternatively, our Cache API was introduced for the purpose of eliminating those requests in the first place. The goal here is to store data in Fly's global cache, and users will recieve that data from whichever Edge server is geographically closest to them.

Serving cached data and resources is *easily* much faster than traveling to the origin server every single time a user makes a request. The ability to cache and reuse previously fetched resources is a critical aspect of optimizing any app for optimal performance.

## How does it work? 

The major difference between caching with Fly and caching with other CDN’s is - *simplicity*. Setting it up is as easy as writing a few lines of JavaScript.  
Fly's cache is available to you as a key-value store. You simply set a key, assign a value for that key, and (optionally) set a TTL. Data stored in `@fly/cache` can have an associated per-key time to live (TTL), and we will evict key data automatically after the elapsed TTL. We will also evict unused data when we need to reclaim space.

## Before you begin

Make sure you have Fly installed: `npm install -g @fly/fly` 

## Import the cache library 

`import cache from “@fly/cache”` 

# List of functions associated with cache

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

The first thing your app should do is check whether or not you already have the user's request cached. For example, if your user is requesting a video from your app, you’ll want them to first check the cache for that video, rather than loading an entirely fresh video from the server, which could lead to less than ideal loading time.  

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

The key here is the url where the video lives. There’s nothing special about this key, just a unique identifier that we’re choosing to associate with our video for caching and retrieving.

If we find a value at that key, then we’ll send it to the user ... and that’s it! The user will quickly receive the cached version of this video. They’ll be able to click play and almost *instantly* start watching. 

Notice the importance of the `async/await` keywords here. All Fly requests use the modern async/await syntax introduced in ES2017.
 
*Note, if we were searching for cached data in the form of HTML text or similar, we’d want to use the `getString()` function instead.*

If we *don't* find a value at the requested key, then we need to fetch that value from the origin and `set` it in our cache:

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
        await cache.set(key, resp, { ttl: 3600 * 24 }) 
        resp.headers.set("Fly-Cache", "MISS")  
        return new Response(resp) 
    } 
}) 
```

Fly Edge Apps add the `Fly-Cache` header to HTTP Responses. `Fly-Cache: HIT` means that our request was served by the CDN (from the cache), not the origin server. `Fly-Cache: MISS` means that your request was not served by the CDN, but rather served from the origin server. A CDN takes content and distributes that content in lots of places. It's an efficient way to store data on servers around the world and deliver content faster than from a single location. This is one of the features that makes Fly Edge Apps so unique and powerful.

Ultimately, the function above checks the cache for our video, serves the video from the cache if it is found, or fetches the video from the origin and sets it in the cache for 24 hours, speeding it up for future users.

### Updating the Cache

Serving cached data is convenient, quick and easy ... but we certainly don't want to serve the same exact data for all of eternity. At some point, data will change and the cache will need to be updated. One way we achieve this is by setting a TTL (time to live) along with our `set()` method.

Another way to expire a cache value is with `expire()`.

```javascript 
import cache from “@fly/cache”

let key = "www.example.com/videos/header-video"
await cache.expire(key, 259200)
```

This would automatically expire the value in the cache at the given key after 3 days ... allowing for a fresh value to be fetched and cached.

Refer to the list of functions above for more ways to manipulate cache ... such as `del()` for deleting the value at a certain key altogether, `setTags()` for replacing and setting tags at a certain key, and `purgeTag()` for purging all cache entries with the given tag.

#### Next Up: responseCache API (add link)







