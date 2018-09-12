# responseCache API 

This page walks you through how to get started using Fly’s responseCache API - an API for efficiently caching Response objects in the regional Fly cache, whereas a “Response object” is simply the response returned by the server after the client makes an HTTP request. 

Example use case: 

- request images and place the response body/metadata in the cache 

## Overview 

The Hypertext Transfer Protocol (HTTP) allows for communication between clients and servers. For example, a client (browser) submits an HTTP request to the server; then the server returns a response *back* to the client. The response contains status information about the request (response headers, status, tags, etc..) as well as the content of the request (response body). All of this information can be cached for the purpose of faster retrieval in the future. 

The performance of web sites and applications can be *significantly* improved by reusing previously fetched resources. This is because the goal of caching is to reduce latency and network traffic, therefore lessening the time needed to display a particular resource. By properly making use of HTTP caching, apps *automatically* become more responsive. Thus, we created this API - for you to efficiently cache and retrieve response objects. 

## How it works 

Response caching is a technique that stores a copy of a Response object and serves it back when requested. Fly’s `responseCache` is available to you as a key-value store. You simply set a key and assign a value to that key, as well as an optional TTL (time to live). When a client makes an HTTP request for a particular resource, the cache should first be checked to see if it has the requested resource in its store. If it does, it intercepts the request and returns its cached copy, rather than re-downloading and re-creating a totally new response from the originating server.  

By placing response objects in Fly’s responseCache and deploying your app to Fly’s servers, user’s will receive your content, when requested, from whichever Fly server is closest to them. [Click here](https://fly.io/docs/#datacenter-locations) for a list of Fly’s current data centers. 

This method achieves several goals. It mainly eases the load of the origin server where your app lives so that it doesn’t need to serve all clients all by itself. It also improves performance by bringing data closer to the client ... which means when a resource is requested, it will take less time to transmit the resource back. For most apps, a feature like this is a *major* component in achieving high performance.  

## Before you begin 

Make sure you have Fly installed: `npm install -g @fly/fly`  

## Import the responseCache library  

`import { responseCache } from “@fly/cache”` 

# List of functions associated with `responseCache` 

## get 

Get a Response object from the cache. 

`await responseCache.get(“test-key”)`  

This function takes 1 parameter:   

- **key**: string 
cache key to get 

And returns a Promise (Response & object) - The response associated with the key, or null if empty. 

## getMeta 

Gets Request metatadata from the cache. 

`await responseCache.getMeta(“test-key”)` 

This function takes 1 parameter:   

- **key**: string
cache key to get metadata for 

And returns a Promise in the form of the Response’s metadata with properties: 
- **at**: number 
- **headers**: object 
- **status**: number 
- **tags**: string[] 
- **ttl**: number 

## set 

Stores a Response object in the Fly cache. 

`await responseCache.set(“test-key”, resp, ttl)`  

This function takes 3 parameters:   

- **key**: string   
cache key to set 

- **resp**: response 
the response to cache 

- **options** (optional) 
	- **ttl** (time to live): a number in seconds 
	- **tags**: string[] 
	- **skipCacheHeaders**: string[] 
	- **onlyIfEmpty**: boolean 

And returns a Promise (boolean) - true if the set was successful. 

## setMeta 

Sets metadata for a Response object in the Fly cache. 

`await responseCache.setMeta(“test-key”, meta, ttl)`  

This function takes 3 parameters:   

- **key**: string   
cache key to set

- **meta**: metadata 
metadata for the Response 
	- **at**: number 
	- **headers**: object 
	- **status**: number 
	- **tags**: string[] 
	- **ttl**: number 

- **options** (optional) 
	- **ttl** (time to live): a number in seconds 
	- **tags**: string[] 
	- **onlyIfEmpty**: boolean 

And returns a Promise (boolean) - true if the set was successful. 

## setTags 

Replace tags for a cached Response. 

`await responseCache.setTags(“test-key”, [“tag1”, “tag2”, “tag3”])`  

This function takes 2 parameters:   

- **key**: string   
the key to modify 

- **tags**: string[] 
tags to apply to key 

And returns a Promise (boolean) - true if tags were successfully updated. 

## touch 

Resets the "age" of the cached Response object 

`await responseCache.touch(“test-key”)`  

This function takes 1 parameter:   

- **key**: string   
Response to "touch" 

And returns a Promise (boolean) - true if the reset was successful. 

## del

Deletes a Response object from the cache. 

`await responseCache.del(“test-key”)`  

This function takes 1 parameter:   

- **key**: string   
key to delete 

And returns a Promise (boolean) - true if the delete was successful. 

## expire 

Sets a new expiration time for a Response object.
 
This function takes 2 parameters:   

- **key**: string   
Key to set an expiration for 

- **ttl**: number 
time to live (a number in seconds) 

And returns a Promise (boolean) - true if the expire was successful. 

## How to implement responseCache into your app 

Checking the cache, fetching requests, and writing Response objects to the cache is a very popular feature in Fly Edge Apps. Use the methods above for caching/retrieving HTTP response objects within your own Edge App. 
 
```javascript 
import { responseCache } from “@fly/cache” 
const url = "http://example.com" 
let resp = await responseCache.get(url)  
if(!resp){ 
    resp = await fetch(url) 
    await responseCache.set(url, resp, { ttl: 3600 }) // cache for an hour 
}
```

Additionally, when the server returns a response, it also emits a collection of HTTP headers, describing its content-type, length, and more. This information is cached along with the body of the response. This information includes Response.headers (the Headers object associated with the response), Response.status (the status code of the response), Response.url (the URL of the response) and more. 

Caching was created to improve app performance. The goal of response object caching is to eliminate the need to constantly send and receive requests and responses. Therefore, this method of response caching ultimately reduces the number of network round-trips and network bandwidth requirements. 

#### Up Next: global cache API
 

 

 