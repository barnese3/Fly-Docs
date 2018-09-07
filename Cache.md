# Cache 

This page walks you through how to get started using Fly’s Cache API - an API for accessing a regional, volatile cache. Data stored in `@fly/cache` can have an associated per-key time to live (TTL), and we will evict key data automatically after the elapsed TTL. We will also evict unused data when we need to reclaim space.  

Example use case: cache personalization data geographically close to individual users 

## Before you begin

Make sure you have Fly installed: `npm install -g @fly/fly` 

## Import the cache library 

`import cache from “@fly/cache”` 

# Functions

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