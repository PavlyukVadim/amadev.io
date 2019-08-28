---
path: /posts/promises-in-javascript
date: '2019-08-26'
title: ⏰ Promises in JavaScript
category: async-js
metaTitle: Promises in JavaScript
metaDescription: Promises in JavaScript
metaKeywords: 'javascript, js, js core, promise, closure, array, number, string, bool'
hidden: true
---

## Table of content:

* [Promises overview](#)

### Promises overview

Promise is't object that represrnts a future value and contains inner state (pending, fulfilled, rejected).

How to create promise? Just use ```revealing constructor```: 

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('foo')
  }, 1000)
})
```

#### Promise.then

Promises have a special method ```then(onFulfilled, onRejected)``` for adding handlers:

```js
promise.then((data) => { console.log(data) }) // 'foo'
```

Note: You can apply chaining to ```then``` methods, and it then method retuns a new promise the next method will be activated with a real value when the promise resolves.

Let's create a function that produces a promises:
```js
const asyncFn = (params) => new Promise((resolve) => {
  setTimeout(() => {
    resolve(params)
  }, Math.random() * 100)
})
```

So the calls of asyncFn you can chained like that:

```js
asyncFn(1).
  then((data) => {
    console.log('data', data)
    return asyncFn(2)
  }).
  then((data) => {
    console.log('data', data)
  })

// data 1
// data 2
```

Also, you can get the last promise from the chain:

```js
const promise = asyncFn(1).
  then((data) => {
    console.log('data', data)
    return asyncFn(2)
  })

promise
  .then((data) => {
    console.log('data', data)
  })

// data 1
// data 2
```

When you returns from ```then``` method not an immediate value, but another promise, that promise will be ```asynchronously``` unwrap:

```js
const p3 = new Promise((resolve, reject) => resolve('B'))

const p1 = new Promise((resolve, reject) => resolve(p3))

const p2 = new Promise((resolve, reject) => resolve('A'))

p1.then((data) => console.log(data))
p2.then((data) => console.log(data))

// A B
```

Note: all non-function types in ```then``` method a silent ignored:

```js
const promise = asyncFn(1).
  then((data) => {
    console.log('data', data)
    return asyncFn(2)
  }).
  then('foo').
  then({})

promise
  .then((data) => {
    console.log('data', data)
  })

// data 1
// data 2
```

#### Promise.catch

To handle error you can use or onRejected function inside method ```then``` or use ```catch``` method:

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => { reject(new Error('foo')) }, 1000)
})

promise
  .then((data) => { console.log(data) })
  .catch((err) => { console.log(err) }) // Error: foo
```

### Promises imutation

The very important Promise feature, that when it resolves(rejects), it becomes an immutable value that can be observed many times.

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => { resolve('foo') }, 1000)
  setTimeout(() => { reject() }, 2000)
})

promise
  .then((data) => { console.log(data) }) // foo
  .catch((err) => { console.log(err) }) // ignored
```

Also, when you add multiple handles on promise, you get the same results:

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => { resolve(Math.random()) }, 1000)
  setTimeout(() => { reject() }, 2000)
})

promise
  .then((data) => { console.log(data) }) // 0.41118968976944426
  .catch((err) => { console.log(err) }) // ignored

promise
  .then((data) => { console.log(data) }) // 0.41118968976944426
  .catch((err) => { console.log(err) }) // ignored
```

### Immediately resolved Promise

There are shortcats that immediately sets Promise state: ```Promise.resolve(..)``` and ```Promise.reject(..)```.

These two promises are equivalent:

```js
const p1 = new Promise((resolve, reject) => { resolve('foo') })
const p2 = Promise.resolve('foo')
```

Note: immediately fulfilled Promise cannot be observed synchronously, when you call then(..) on a Promise, even if that Promise was already resolved, the callback (```then(..)```) will be called asynchronously.

And even more important feature of ```Promise.resolve(..)``` that it can normalize ```thenable``` object into real Promise.

### Built-in Promise patterns

#### Promise.all

```Promise.all``` returns a Promise with results two or more parallel/concurrent tasks:

* takes array of Promise instances (or thenables/immediate values, that will be passed through ```Promise.resolve(..)```)

* returned promise will be fulfilled when all inner tasks are fulfilled

* returned promise will be rejected if one of those tasks is rejected

* with passed ```[]``` imideatly resolved with ```undefined```


#### Promise.race

```Promise.race``` returns a first fulfilled Promise:

* takes array of Promise instances (or thenables/immediate values, that will be passed through ```Promise.resolve(..)```)

* returned promise will be fulfilled when one of those tasks is fulfilled

* returned promise will be rejected if one of those tasks is rejected

* Note: with passed ```[]``` infinity pending

### Promise underhood

Promises don't deny a callbacks, they just propose a new approach of using them inside a abstraction with a state that becomes an immutable value when result is ready, and calls callbacks that declared via inner method ```then```.
Let's try to implement such abstraction:

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class Thenable {
  constructor(executor) {
    this.status = PENDING
    this.result = null
    this.next = null

    if (typeof executor === 'function') {
      executor(this.resolve.bind(this))
    } else {
      this.status = FULFILLED
      this.result = executor
    }
  }

  then(fn) {
    this.fn = fn
    const next = new Thenable()
    this.next = next
    return next
  }

  resolve(value) {
    this.status = FULFILLED
    const fn = this.fn
    if (!fn) return
    const result = fn(value)
    // next.then(value => {
    if (this.next) {
      this.next.resovle(result)
    }
  }
}

const asyncFn = (params) => new Thenable((resolve) => {
  setTimeout(() => {
    resolve(params)
  }, Math.random() * 100)
})

asyncFn('foo')
  .then((value) => { console.log('data1', value); return 'bar' })
  .then((value) => console.log('data2', value))

```

### Cancable Promise



### Decorator of Promises chaining

It can be useful to create decorator that emulates chaining by fucntions calls:

Let's modify our asyncFn by adding output before resolving for inspection of function resolves oreder:

```js
const asyncFn = (params) => new Promise((resolve) => {
  setTimeout(() => {
    console.log('params', params)
    resolve(params)
  }, Math.random() * 100)
})
```

When we call it, it would executes in random order:

```js
asyncFn(1), asyncFn(2), asyncFn(3), asyncFn(4), asyncFn(5)

// params 5
// params 3
// params 2
// params 1
// params 4
```

But what if we need to call them one by one?
Whit emulation of chaining:

```js
asyncFn(1)
  .then(() => asyncFn(2))
  .then(() => asyncFn(3))
  .then(() => asyncFn(4))
  .then(() => asyncFn(5))

// params 1
// params 2
// params 3
// params 4
// params 5
```

You can use the follows decorator:

```js
function chainingDecorator(f) {
  let prev = Promise.resolve()
  return function(...params) {
    prev = prev.then(() => f(...params))
    return prev
  }
}

const chainedFn = chainingDecorator(asyncFn)

chainedFn(1), chainedFn(2), chainedFn(3), chainedFn(4), chainedFn(5)

// params 1
// params 2
// params 3
// params 4
// params 5
```

Advanced sections:

Jobs queue:

<!-- http://www.ecma-international.org/ecma-262/6.0/index.html#sec-promise-jobs -->

```js
setTimeout(() => {
  console.log('c')
}, 0)

const immediatePromise = Promise.resolve('d')
const immediatePromise2 = Promise.resolve('e')

immediatePromise
  .then(() => { console.log('d11'); return immediatePromise2 })
  .then(() => { console.log('d12') })

immediatePromise
  .then(() => { console.log('d21'); return 'd2' })
  .then((data) => { console.log('d22') })

const b = () => { console.log('b') }
const a = () => { console.log('a'); b() }

a()

// a
// b
// d11
// d21
// d22
// d12
// c
```