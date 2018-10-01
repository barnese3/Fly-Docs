# Fetch API

This page walks you through an impressive JavaScript feature that helps make Fly Edge Apps customizable and easy to work with – the Fetch API, an API that starts the process of fetching a resource from the network and returns a promise that resolves to the Response object representing the response to your request.

## Overview

Fly Edge Apps are written in JavaScript and deployed all over the world. And since they're JavaScript, you can make them do almost anything. They're especially good at pulling content from just about anywhere and then customizing that content on-the-fly … enter the significance of “fetching”.
 
With Edge Apps, you can use the same exact `fetch()` method available in standard JavaScript, including the modern async/await syntax. This smooth method replaced that cringeworthy XMLHttpRequest (XHR) we used back in the day to make requests, interact with servers, and retrieve data from URL’s.

The Fetch API provides a super clean way to do all the wonderful things that you could do with XHR, plus more. It creates an entirely new way of making server requests ... filled with promises and a more powerful, flexible set of features.

`fetch()` allows us to make network requests similar to the dreaded XMLHttpRequest. Only, instead of using callbacks, the Fetch API uses Promises, which enables simpler and cleaner usability without having to remember the complexity of the XMLHttpRequest API.

## How it works 

`fetch()` is a global method that starts the process of fetching a resource from the network, and returns a promise that resolves to the Response object representing the response to your request. The `fetch()` method is available in pretty much any situation where you’d want to fetch resources in your Edge App.

Usually when there’s a permissions issue or something similar, `fetch()` will return a Promise in the form of a `TimeoutError` to indicate that a network error was encountered or that a value received is not of the expected type. A successful `fetch()` will include a `Response.ok` property value of `true`.

`fetch(req, init)`

The fetch method takes 2 **parameters**: `req` and `init`

***req***: RequestInfo
This parameter defines the resource that you wish to fetch. It can either be a direct URL string or a Request object for the resource you're fetching.

***init***: FlyRequestInit (optional)
This parameter represents an options object containing any custom settings that you want to apply to the request. This parameter is optional and the possible options are:

`body`: any body that you want to add to your request. This can be a Blob, BufferSource, FormData, string or null.

`cache`:  the cache mode you want to use for the request, which will control how the request will interact with the browser's HTTP cache. This should be a RequestCache value. See [Request.cache](https://developer.mozilla.org/en-US/docs/Web/API/Request/cache) for available values.

`credentials`: the request credentials you want to use for the request: `omit`, `same-origin`, or `include`.

`headers`: any headers you want to add to your request, contained within a `Headers` object or an object literal with `ByteString` values.

`integrity`: contains the subresource integrity value of the request. This will be in the form of a string.

`keepalive`: this option will be a boolean value which can be used to allow the request to outlive the page.

`method`: this option represents the Request method in the form of a string. For example, “get” or “post”.

`mode`: the mode you want to use for the request. For example, `cors`, `no-cors`, or `same-origin`.

`redirect`: this option indicates which redirect mode to use. For example, `follow` will automatically follow redirects, `error` will abort with an error if a redirect occurs, and `manual` will handle redirects manually. The default is usually `follow`.

`referrer`: this option is a string specifying `no-referrer`, `client`, or a URL. The default is `client`.

`referrerPolicy`: this option specifies the value of the referrer HTTP header: `no-referrer`, `no-referrer-when-downgrade`, `origin`, `origin-when-cross-origin` or `unsafe-url`.

`signal`: this option will be an `AbortSignal` object instance which allows you to communicate with a fetch request and abort it if desired via an `AbortController`.

`timeout`: this option will be a number indicating the number of seconds the client will wait for a response from the server until a TimeoutError is raised.
 
The `fetch()` methods takes these parameters, issues an HTTP/HTTPS request and returns a **Promise** that resolves to a **Response** object.

## How to use 

With this method, it is possible to fetch data from an API, URL, website, database, local file, server, etc... just like you normally would with JavaScript. 

If you want to fetch data from a local file, first create a .fly.yml file in the root of your app and define the files here: 

```javascript
# .fly.yml
files:
    - image.png
```

Then, fetch the file from anywhere else within your app:

```javascript
async function imageFetch() {
    const image = await fetch("file://image.png")
    image.headers.set("content-type", "image/png")
    return image
}
```

Or, fetch data from an API and store it in Fly’s global cache like this:

```javascript
async function apiFetch() {
    const url = 'https://randomuser.me/api/?results=20'
    const data = await fetch(url)

    fly.cache.set("api-data-2018", data, 86400) // cache for 24 hours
    return data
}
```

The awesome thing about fetching with Fly is that we've designed it to be lightning fast and, when combined with `fly.cache` (add link), server load is reduced tremendously. In a nutshell, `fetch/Response/Request` are incredibly well-designed APIs ... and they can be used to build an API for almost any kind of HTTP service.

#### Next Up: proxy (add link)