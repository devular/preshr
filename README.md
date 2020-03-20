# Preshr

> Yes we're irresponsibly building a Node framework

## Why?

I've been writing lots of Node code recently and here's a laundry list of things I would like:

- Designed to be deployed on bare metal servers, PaaS, and Serverless from the get go
- Bearing that in mind, four robust abstractions for the features I write the most:
- Request Handlers
- Procedures - no side effects - Synchronous
- Side Effects - deliberate API - Asynchronous
- Response management - Rendering, returning JSON, something else?
- Thinking about how to get some sort of in-memory and/or redis backed queue system - with multiple workers - working reliably
- Why not try building something?

## Why Preshr?

It's gonna be precious, and fresher... no

**R**equest **H**andlers, **P**rocedures, **S**ide **E**ffects, and **R**esponses - anagrammed into _Preshr_

## Roadmap

- Likely use Fastify for Request Handlers, and Responses in early versions
- What's a cool testing library - do people still use Mocha?
- Probably write it in TypeScript
- Look at [Bull](https://github.com/OptimalBits/bull) for the queueing system
- Need to think about unambiguous routing
- Utilise throng for clustering out of the box
- ES6 transpilation? Entry points for queue processors?
-

## API

Not sure yet, but I've got the idea of

```js
import
  preshr,
  procedure,
  sideEffect,
  sideEffectOnQueue
} from "@preshr/core";

preshr.get("/", opts, (request, respond) => {
  const [fileSize] = procedure(() =>
    fs.appendFileSync("log.txt", JSON.stringify(request.json()))
  );

  sideEffect(() => email.send("test", "foo@bar.com"));

  // Add job to queue with options (job type, repeatable)
  sideEffectOnQueue('image-processing', options, () =>
    imageAndText.render("background-image.jpg", "Hello World")
  );

  // Straight forward response with object syntax
  respond({
    payload: {
      status: "Completed",
      fileSize
    },
    contentType: 'json'
    statusCode: 200
  })

});

preshr.start({
  port: 3000,
  // Preshr will support clustering out of the box
  clustering: {
    workers: 4,       // Number of workers (defaults to cpu count)
    lifetime: 10000,  // ms to keep cluster alive (Infinity)
    grace: 4000,
  },
  // Initialise queues
  queues: [
    {
      name: 'image-creation',
      // Processors are sandboxed *processes*
      // that cannot crash the worker and utilise
      // multiple cores
      processor: '/path/to/my/processor.js',
      // Global job completed on any worker for this queue
      completed: (jobId) => console.log(`${jobId} completed`)
      options: {
        // Rate limiting queues
        limiter: {
          max: 1000,
          duration: 5000
        }
      }
    },
    {
      name: 'video-comppression',
      // It's be great if these queue processors
      // could be greated in a folder and then
      // webpacked as entry points out, so we can
      // write ES6 everywhere, and also a centralised config
      processor: '/path/to/video/processor.js'
    }
  ]
});
```
