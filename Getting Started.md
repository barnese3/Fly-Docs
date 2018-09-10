# Getting Started 

In this guidebook, you'll learn how to get up and running with Fly Edge Apps and how to use all of the wonderful features/libraries that make Edge development a breeze. 

You'll learn how to build *and* deploy an application in a full development environment using modern JavaScript and our lightning fast Edge libraries where your content gets optimized *as* it’s delivered to your users.  

Let's get started!  

### Step 1: Installation  

Install Fly globally: `npm install -g @fly/fly`  
Or inside your project as a devDependency: `npm install --save-dev @fly/fly`  

*Note: Make sure you have node.js and npm installed on your computer.* 

### Step 2: Create an index.js file  

Write some code to your index.js file: `fly.http.respondWith(()=> new Response("My First Fly Edge App"))`  

*Note: Your app must contain an index.js file.*  

### Step 3: Start the fly server  

Start up the fly server using the command: `fly server`  
See your app by visiting http://localhost:3000 in your browser.  

TADA! You should see "My First Fly Edge App" when you visit your browser. So, what's even happening here you might ask?   

## How it works

* Your JavaScript files are being bundled with webpack, a technique used to trim down your application’s load time. With webpack, it’s possible to bundle multiple assets, files, libraries and even images into a single file so your user can load your entire app in a *single* 'get' request, as opposed to multiple ‘get’ round-trips to the server.
* You have the ability to customize your configuration (e.g. context, entry, output, etc), loaders, and other specific information relating to your build by creating a webpack.fly.config.js file (which will be loaded for you).
* Fly uses npm packages compatible with the v8 JavaScript engine, so you don't have access to node.js-specific concepts or packages. 

At this point you're seeing, "My First Fly Edge App", when you visit your browser. But that’s not all that fun, right? Let's dive a little deeper, unlock the magic of Fly, and learn how we can optimize our app's content for speed and efficiency. 

*First, it's important to note that Fly specific APIs and functionality modules are available via the `fly` global variable or by `import`-ing the library directly into your app.* 

For example:

```javascript 
fly.cache.set(“key”, “value”, ttl) 
```

AND... 

```javascript 
import cache from “@fly/cache”
cache.set(“key”, “value”, ttl) 
``` 
...are both using the Cache API efficiently. We’ll learn more about Fly’s Cache API (add link) in upcoming pages... 

**Below are a couple of basic example functions that use some of Fly Edge App’s most popular features/libraries. These should help you get your app off the ground and start exploring!** 

### Fetch content from anywhere, optimize/customize it and then display it in your app 

Create a `.fly.yml` file to specify properties such as your application's name, configuration settings, and files. By default, Fly will read your `.fly.yml` file in your current working directory.   

By specifying a files property in your `.fly.yml` file, it's possible to use `fetch` to load files without having to bundle them *directly* in your JavaScript. The `files` property here will be an array of files, relative to your `.fly.yml` to include in deployment. These files can be accessed via `fetch("file://path/to/file")`. Here’s what a `.fly.yml` file should look like: 

```javascript
# .fly.yml 
app: my-online-store 
config: 
    foo: bar 
files: 
    - example-file.txt
```

The `app` property is your fly.io app name. This can be omitted, but it’s useful for deployment purposes. The `config` property will be arbitrary settings for your app, which are accessible in your code via the global variable `app.config`. 

To access any files under the `files` property, simply use the `fetch` method: 

```javascript
// index.js 
fly.http.respondWith(async function(){ 
    const response = await fetch("file://example-file.txt") 
    response.headers.set("content-type", "text/html") 
    return response 
})
``` 

The `example-file.txt` is now available to customize and display, however you’d like.  

## Resize and Optimize Images

Use Fly's Image API to enable responsive images and deliver the *exact* images you need for each user without compromising load time. 

```javascript
import { Image } from “@fly/image” 
const response = await fetch("http://example.com/images/balloon.jpg") 
const original = new Image(await resp.arrayBuffer()) 
let image = original.resize(500).withoutEnlargement() 
const accept = req.headers.get("accept") || "" 
if (accept.includes("webp")) { 
   image = image.webp() 
   resp.headers.set("content-type", "image/webp")
} 
image = await image.toImage() 
return new Response(image.data, resp)
```

This function fetches an image from an external source (you could also fetch a local image by storing that image in your app and retrieving it via `fetch("file://balloon.jpg")`). Either way, fetching with Fly is *very* fast. This function then creates a new Fly image instance and stores the image’s binary data for speedy inputs and outputs. It then resizes the output image to 500px (so long as the input image is larger). Next, it converts the image to the `webp` format for image compression and fast loading time. Then, it applies all of our changes and creates a new image instance. Finally, it creates an HTTP Response object and displays the image. 

For a fully functional image API example, see the [bundled example app](https://github.com/superfly/fly/blob/master/apps/watermark-image/index.js).

## Deploy your Edge App

When you're ready to ship, deploy to our global hosting infrastructure with a *single* command, `fly deploy`

### 1. Login  

If you don't have a fly.io account, create one - https://fly.io/app/sign-up. From the command line, enter `fly login`. You will be prompted to enter your email address and password (whatever you used to sign up with).  

### 2. Create your app  

Create your app from the command line - `fly apps create [name]`. Name is optional here. If you do not specify a name, one will be created for you.  

### 3. Set your app property  

After you create your fly app, you will see something like... *App blue-flower-1972 created! Add it to your .fly.yml like: `app: blue-flower-1972`.* Navigate to your .fly.yml file and set your app property to your new app name.  

### 4. Deploy!  

Enter `fly deploy` from the command line. You should see:  

```  
Deploying blue-flower-1972  
Generating Webpack config...  
Compiling app w/ options: { watch: false, uglify: true }  
Compiled app bundle (hash: 86d857757fd0972e217b08a28455f2d160f2bdba)  
Bundle size: 0.008871078491210938MB  
Bundle compressed size: 0.0021800994873046875MB  
Deploying v1 globally, should be updated in a few seconds.  

```  

Check out your app by visiting ***app-name.edgeapp.net***. 

### How deployment works 

* Your code was bundled using the module bundler, webpack, which helps you improve performance by giving you control over how assets are built and fetched by the user. 

* Your code was also uglified to save space by automatically removing white spaces, changing big variable names to small ones and compressing files, ultimately reducing the size of the file and helping with performance.  

* Your code, source map and files are added to a simple tarball (a computer file format that combines and compresses multiple files). It's then gzipped to decrease bandwidth and uploaded to the fly.io API using your `FLY_TOKEN`.

* We create an immutable "release" to ship your app. Changing anything (by using `fly deploy` or `fly secrets set`) will trigger a new release which will be deployed automatically. 

* Your code is distributed instantly(-ish) across our global fleet of servers.

Congrats! You now have a simple Fly app that fetches files and optimizes images. 

## What else can I do with my Fly Edge App? 

The power of Fly also allows you to add unlimited hostnames with a simple POST using our hostname API, put your Glitch project on a custom domain, build a global application load balancer, test locally using fly test, deliver http requests with unbelievable low latency using fly.cache, connect to any Heroku app, improve your Google Lighthouse Scores and more!

Next up: Cache API (add link) 

 