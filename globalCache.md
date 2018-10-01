# global cache API 

The Fly global cache API, allows eventually consistent modifications to all caches in all regions. The concept of eventual consistency in this case can be described as “when you update cached data within your app, *eventually* all Edge servers will receive that data and make necessary updates”.

## Overview 

Caching is one of the main features of Fly Edge Apps that make them fast and powerful. The idea behind caching at the Edge is to significantly reduce the amount of work that your app’s server has to do when a user requests it, ultimately speeding your app up for those visiting it.

Fly's cache is regional, which is ideal for Edge Applications since most global caches vary tremendously between regions. But there can be shared content scattered around the world, and it's useful to be able to delete it all in one go. 

The `cache.global` API lets developers distribute cache notifications around the world. It turns the Fly cache into something like a distributed, eventually consistent global cache.  

## How it works  

The process of Fly’s global cache API is simplified below: 

1. You make changes to your cache using one of the `global` API methods (`del` or `purgeTag`) 
2. You deploy to Fly servers, `fly deploy`, to apply changes 
3. Updates are distributed to all of Fly servers across the world 
4. *Eventually* all servers will receive the changes and update accordingly 

So why eventually and not immediately? Two major reasons: **availability** and **scalability**.  

Eventual consistency relies on having a distributed system (rather than a centralized one), hence the concept of globally distributed Edge servers. With strict consistency (immediate updates), you can only scale so far before things start to slow down. And scalability is important to support the load created by high volumes of traffic. Typically, high traffic volume requires numerous low latency servers handling requests. Eventually consistent servers implement read/write operations at a *much* lower latency than the latter ... ultimately meaning that numerous low latency servers working together to deliver consistent data eventually is *much* faster and more scalable than strict consistency, which can significantly improve the performance of your app.  

To sum up, if high scale and very low latency are both critical components for you and your app, eventual consistency is key. Eventual consistency trades “instant” updating for higher availability and faster client-side performance. These trade-offs allow us to provide the best possible experience for our users.

## Before you begin  

Make sure you have the latest version of Fly installed: `npm install -g @fly/fly`  

## Import the cache library  

`import cache from “@fly/cache”`  

# List of functions associated with `cache.global`  

## del 

Notifies all caches to delete data at the specified key. 

`await cache.global.del("key-to-delete")` 

This function takes 1 parameter: 

- **key**: string --> the key to delete 

And returns a Promise (boolean) - A promise that resolves as soon as the del notification is sent. Since regional caches are eventually consistent, this may return before every cache is updated. 

## purgeTag 

Notifies all regional caches to purge keys with the specified tag. 

`await cache.global.purgeTag("key-to-purge")` 

This function takes 1 parameter: 

- **tag**: string --> the tag to purge 

And returns a Promise (boolean) - A promise that resolves as soon as the purge notification is sent. Since regional caches are eventually consistent, this may return before every cache is updated. 

## How to implement global cache into your Fly Edge App 

Use `cache.global.del(key)` to *eventually* delete all named keys in all regions. For example: 

```javascript 
import cache from “@fly/cache”

fly.http.respondWith(async function (req) { 
    const url = new URL(req.url) 
    if (url.pathname === "/delete") { 
        await cache.set("my-key:delete", `${Date.now()}`) 
        await cache.global.del("my-key:delete") 
    } 
    return new Response("hello there") 
}) 
``` 

Or use `cache.global.purgeTag(tag)` to *eventually* purge all cached items with a given tag. In practice, these notifications only take a few hundred milliseconds to propagate, but when the internet is being obstinate it can take longer. 

#### Up Next: Fetch API (add link)