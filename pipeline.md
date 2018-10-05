# Pipeline library 

This page walks you through Fly’s `pipeline` library - a library for composing several `fetch` (add link) generators into a single pipeline. 

## Overview 

Pipeline is a `fetch`-like function, which most Edge Applications lean heavily on to accept requests and communicate with a bunch of different backend services. 

The `pipeline()` method allows us to combine multiple fetches into a single function by threading requests through each step in the pipeline ... providing us with middleware type functionality found in most modern web frameworks. 

A “pipeline”, in software terms, is a set of several unique data transformations, linked together into one sequence, where the output of one transformation is the input of the next one. The sequence usually consists of several different functions, executed in parallel, with the output stream of one function being automatically fed in as the input stream of the next.  

Overall, pipelines are made up of one or more “jobs”, where each job has its own set of instructions and its own outcome. These instructions can be tasks, scripts, or even references to external templates. The outcome of one job is used for the next job in the sequence.  

The pipeline structure is outlined below: 

**Pipeline:** 

Job 1  
&npsp;   &npsp;   &npsp;   &npsp;Step 1.1  
&npsp;   &npsp;   &npsp;   &npsp;Step 1.2  
&npsp;   &npsp;   &npsp;   &npsp;Step 1.3  
Output --> Job 2  
Step 2.1  
Step 2.2  
Step 2.3  
Output --> Job 3  
Step 3.1  
Step 3.2  
Step 3.3  
Output --> Job 4... and so on  

## How it works 

Pipelining is a concept commonly used in everyday life. For example, the construction of a car usually requires an assembly line at a car factory. Each specific task in the process of building the car is often done at separate work stations ... such as installing the engine, the wheels, the seats, and the doors. Think of each task/work station as a single step in the pipeline.  

Each work station performs their task, in parallel, each on a different car. Once a task is finished, the car moves on to the next station, where its previous task is _crucial_ to the completion of the next. For example, you can’t install the door handles before installing the doors.  

Suppose that assembling one car requires three steps. One step takes 20 minutes, the next 15, and the last 10. If all three steps were performed at a single station, the factory would make one car every 45 minutes. By using a pipeline with three different stations, the factory can now make the first car in 45 minutes, and then a new one every _20_ minutes. 

## Why is this good? 

As the example above illustrates, pipelining does not decrease overall latency (the time it takes to build one car). It does however increase overall throughput (the rate at which new cars are built after the first one if finished). 

You see, website performance is measured using many different metrics, one of them being **throughput**. Throughput is basically how much information your site can process in a given period of time. It’s a measurement of how much bandwidth is required to handle a load of concurrent users and requests. A higher value throughput means many users can access and fetch data from your website at once, and that’s a good thing. 

The car example is very similar to how it’s done in software development. Let’s say you have an app that optimizes the images on your website. You can use the `pipeline()` method to apply several different transformations to your site, one after another. One step in the pipeline could convert image formats, the next could resize images, and the last could lazy-load images. Each step completes a different task, at the same time, but then constructs the finished product one step at a time, using the output of the last as the input of the next. It would convert images formats, take those newly converted images and resize them, and then take the newly converted/resized images and lazy-load them. And in the end, you would have an app that effectively optimizes your site’s images for your users. Let’s dive a little deeper. 

### Before you begin   

Make sure you have the latest version of Fly installed: `npm install -g @fly/fly`    

### Import the pipeline library  

`import pipeline from "@fly/fetch/pipeline"` 

### Syntax  

This method combines multiple fetches into a single function, allowing for middleware type functionality. 

`pipeline(stages)` 

This method takes 1 **parameter**: `stages` 

***stages***: fetch generator functions that apply additional logic 

Each “stage” should be a function that generates a fetch-like function with additional logic. 

## Example 

Remember that image optimization app we talked about earlier? Let’s take a look at what this would look like using the `pipeline()` function. 

```javascript 
fly.http.respondWith(  
    pipeline(  
        convertFormat,  
        resizeImages,  
        lazyLoad,  
    )  
)  
``` 

As you can see, there are three steps in the pipeline function here, which means that these three functions should be defined elsewhere and invoked here. The first step converts images to a different format, such as WebP, the next step takes those new WebP images and resizes them to properly fit your user’s browser, and the final step takes the WebP images in their new sizes and sets them up to be lazy loaded. An example similar to this one that uses `pipeline` can be found in action, [here](https://fly.io/articles/lighthouse-part-three/).  

#### Up Next: image API (add link)