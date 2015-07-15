# Spinal
A microservices framework that design for scalability, simple to write and easy to maintenance

[![Build Status](https://travis-ci.org/jitta/spinal.svg?branch=master)](https://travis-ci.org/jitta/spinal)
[![Coverage Status](https://coveralls.io/repos/jitta/spinal/badge.svg)](https://coveralls.io/r/jitta/spinal)

## Installation
```
npm install spinal
```
Want some bleeding edge `npm install jitta/spinal#development`

## Features
- Broker
  - Broker will help all nodes connected
  - Health check between nodes and remove if it fail
  - Load balance method calls from all nodes
  - Hosted queue system
- Nodes
  - Multiple namespace
  - Call/Provide method from same namespace or other namespace (two ways)
  - Cache result from method with specific hash key
  - Provide worker to process jobs

## Overview
- [Start](#start)
- [Call method](#call-method)
- [Provide method](#provide-method)
- [Queue](#queue)
- [Cache](#cache)

## Start
Spinal need broker to handle all call requests between namespace or nodes. Here is how to start a broker
```js
var Broker = require('../').Broker;
var broker = new Broker();
broker.start(function(){
  console.log('Spinal:Broker listening...' + this.port)
});
```
This code will start a broker at default port :7557

## Call method
After we got a broker running here is how to create a node connect to a broker that we just started.
```js
var spinal = new Spinal('spinal://127.0.0.1:7557', {
  namespace: 'english'
});

spinal.provide('hello', function(data, res){
  res.send('Hi ' + data.name);
})

spinal.start()
```
Now start another node to call the method
```js
var spinal = new Spinal('spinal://127.0.0.1:7557', {
  namespace: 'thai'
});

spinal.provide('hello', function(data, res){
  res.send('Sawasdee ' + data.name)
}

spinal.call('english.hello', {name: 'hunt'}, function(err, result){
  console.log(result); // Hi hunt
})

spinal.start()
```
Do not forget to `start()` when you want to call some method. Use `call()` and put `namespace.method_name`

## Provide method
```js
spinal.provide('name', function(data, res){
  // send a result
  res.send('A string');
  res.send(12345);
  res.send(true);
  res.send({a: 1, b: 2});
  // send an error
  res.error('Error message');
  res.error(new Error('Error message'));
  // and support nodejs style callback
  res(null , {a: 1, b:2})
  res(new Error('Something wrong!'))
});
```

## Queue
```js
// create a worker to process a job
// spinalA namespace is `newsletter`
spinalA.worker('send', function(data, res){
  email.send(data.email, function(){
    res.send('ok');
  });
})

// create a job
spinalB.job('newsletter.send', {email: 'some@one.com'})
.priority('high')            // set priority: low, normal, medium, high, critical
.attempts(2)                 // use 2 times to process a job if it fail. try one more time
.ttl(10000)                  // timeout in 10s
.delay(5000)                 // before start delay for 5s
.backoff(10000)              // after fail retry again in 10s
.save(function(err, job_id){ // don't forget to call save()
  console.log('Created ' + job_id);
});
```

## Call method with options

### Timeout
Normally broker will set default timeout at 10 seconds but we can adjust it.
```js
spinal.call('video.conversion',
  {file: 'jitta.mp4'} // first argument need to be set
  {timeout: 500}, // set timeout option here!
  function(err, result){
    // if exceed timeout option will get an error
    err.message === 'timeout error message'
  }
)
```

### Cache
Soon...

## Roadmap
- Core
  - Event broadcast for each namespace (subscribe, emit)
  - Message broadcast to all nodes
  - Optimize performance
- Plugin
  - Cron
  - Dashboard UI
  - Metrics (for tracking response time and number of calls)
