---
title: promise
date: 2019-10-23
tags:
 - JavaScript
categories:
 - JavaScript
---

## promise的基本实现

### promise基础版

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class Promise {
  constructor(executor) {
    this.status = PENDING
    this.value = undefined
    this.reason = undefined

    this.onResolvedCallbacks = []
    this.onRejectedCallbacks = []

    let resolve = value => {
      if (this.status === PENDING) {
        this.status = FULFILLED
        this.value = value
        this.onResolvedCallbacks.forEach(fn => fn())
      }
    }
    let reject = reason => {
      if (this.status === PENDING) {
        this.status = REJECTED
        this.reason = reason
        this.onRejectedCallbacks.forEach(fn => fn())
      }
    }

    try {
      executor(resolve, reject)
    } catch (error) {
      reject(error)
    }
  }

  then(onFulfilled, onRejected) {
    if (this.status === FULFILLED) {
      onFulfilled(this.value)
    }
    if (this.status === REJECTED) {
      onRejected(this.reason)
    }
    if (this.status === PENDING) {
      onResolvedCallbacks.push(() => {
        onFulfilled(this.value)
      })
      onRejectedCallbacks.push(() => {
        onRejected(this.reason)
      })
    }
  }
}
```

### 实现then的链式调用

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class Promise {
  constructor(executor) {
    this.status = PENDING
    this.value = undefined
    this.reason = undefined

    this.onResolvedCallbacks = []
    this.onRejectedCallbacks = []

    let resolve = value => {
      if (this.status === PENDING) {
        this.status = FULFILLED
        this.value = value
        this.onResolvedCallbacks.forEach(fn => fn())
      }
    }
    let reject = reason => {
      if (this.status === PENDING) {
        this.status = REJECTED
        this.reason = reason
        this.onRejectedCallbacks.forEach(fn => fn())
      }
    }

    try {
      executor(resolve, reject)
    } catch (error) {
      reject(error)
    }
  }

  then(onFulfilled, onRejected) {
    // then方法返回一个新的promise
    let promise2 = new Promise((resolve, reject) => {
      // 上一个promise的执行结果决定promise2是成功还是失败
      if (this.status === FULFILLED) {
        try {
          let x = onFulfilled(this.value)
          resolve(x)
        } catch (e) {
          reject(e)
        }
      }
      if (this.status === REJECTED) {
        try {
          let x = onRejected(this.reason)
          resolve(x)
        } catch (e) {
          reject(e)
        }
      }
      if (this.status === PENDING) {
        onResolvedCallbacks.push(() => {
          try {
            let x = onFulfilled(this.value)
            resolve(x)
          } catch (e) {
            reject(e)
          }
        })
        onRejectedCallbacks.push(() => {
          try {
            let x = onRejected(this.reason)
            resolve(x)
          } catch (e) {
            reject(e)
          }
        })
      }
    })
    return promise2
  }
}
```

### 考虑then的返回值

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

const resolvePromise = (promise2, x, resolve, reject) => {
  if (promise2 === x) {
    reject(new TypeError(`Chaining cycle detected for promise #<Promise>`))
  }
  // 判断x是不是一个普通值 
  if ((typeof x === 'object' && x !== null) || typeof x === 'function') {
    let called // 默认没有调用 成功 和 失败 (成功或者失败只能调一次)
    // 判断x是不是promise 取then
    try {
      let then = x.then
      if (typeof then === 'function') {
        // 则认为x为promise 然后采用promise的结果
        then.call(x, y => {
          if (called) return
          called = true
          // y有可能还是一个promise 递归解析
          resolvePromise(promise2, y, resolve, reject)
        }, r => {
          if (called) return
          called = true
          reject(r)
        })
      } else {
        resolve(x)
      }
    } catch (e) {
      if (called) return
      called = true
      reject(e)
    }
  } else {
    resolve(x)
  }
}

class Promise {
  constructor(executor) {
    this.status = PENDING
    this.value = undefined
    this.reason = undefined

    this.onResolvedCallbacks = []
    this.onRejectedCallbacks = []

    let resolve = value => {
      // 如果一个promise resolve了一个新的promise 会等到这个内部的promise执行完成
      if (value instanceof Promise) {
        return value.then(resolve, reject)
      }
      if (this.status === PENDING) {
        this.status = FULFILLED
        this.value = value
        this.onResolvedCallbacks.forEach(fn => fn())
      }
    }
    let reject = reason => {
      if (this.status === PENDING) {
        this.status = REJECTED
        this.reason = reason
        this.onRejectedCallbacks.forEach(fn => fn())
      }
    }

    try {
      executor(resolve, reject)
    } catch (error) {
      reject(error)
    }
  }

  then(onFulfilled, onRejected) {
    // 可选参数
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : val => val
    onRejected = typeof onRejected === 'function' ? onRejected : err => {throw err}
    // then方法返回一个新的promise
    let promise2 = new Promise((resolve, reject) => {
      // 上一个promise的执行结果决定promise2是成功还是失败
      if (this.status === FULFILLED) {
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value)
            resolvePromise(promise2, x, resolve, reject)
          } catch (e) {
            reject(e)
          }
        })
      }
      if (this.status === REJECTED) {
        setTimeout(() => {
          try {
            let x = onRejected(this.reason)
            resolvePromise(promise2, x, resolve, reject)
          } catch (e) {
            reject(e)
          }
        })
      }
      if (this.status === PENDING) {
        onResolvedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onFulfilled(this.value)
              resolvePromise(promise2, x, resolve, reject)
            } catch (e) {
              reject(e)
            }
          })
        })
        onRejectedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onRejected(this.reason)
              resolvePromise(promise2, x, resolve, reject)
            } catch (e) {
              reject(e)
            }
          })
        })
      }
    })
    return promise2
  }
}
```

### 补充完整方法

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

const resolvePromise = (promise2, x, resolve, reject) => {
  if (promise2 === x) {
    reject(new TypeError(`Chaining cycle detected for promise #<Promise>`))
  }
  // 判断x是不是一个普通值 
  if ((typeof x === 'object' && x !== null) || typeof x === 'function') {
    let called // 默认没有调用 成功 和 失败 (成功或者失败只能调一次)
    // 判断x是不是promise 取then
    try {
      let then = x.then
      if (typeof then === 'function') {
        // 则认为x为promise 然后采用promise的结果
        then.call(x, y => {
          if (called) return
          called = true
          // y有可能还是一个promise 递归解析
          resolvePromise(promise2, y, resolve, reject)
        }, r => {
          if (called) return
          called = true
          reject(r)
        })
      } else {
        resolve(x)
      }
    } catch (e) {
      if (called) return
      called = true
      reject(e)
    }
  } else {
    resolve(x)
  }
}

const isPromise = value => {
  if ((typeof value === 'object' && value !== null) || typeof value === 'function') {
    return typeof value.then === 'function'
  }
  return false
}

class Promise {
  constructor(executor) {
    this.status = PENDING
    this.value = undefined
    this.reason = undefined

    this.onResolvedCallbacks = []
    this.onRejectedCallbacks = []

    let resolve = value => {
      // 如果一个promise resolve了一个新的promise 会等到这个内部的promise执行完成
      if (value instanceof Promise) {
        return value.then(resolve, reject)
      }
      if (this.status === PENDING) {
        this.status = FULFILLED
        this.value = value
        this.onResolvedCallbacks.forEach(fn => fn())
      }
    }
    let reject = reason => {
      if (this.status === PENDING) {
        this.status = REJECTED
        this.reason = reason
        this.onRejectedCallbacks.forEach(fn => fn())
      }
    }

    try {
      executor(resolve, reject)
    } catch (error) {
      reject(error)
    }
  }

  then(onFulfilled, onRejected) {
    // 可选参数
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : val => val
    onRejected = typeof onRejected === 'function' ? onRejected : err => {throw err}
    // then方法返回一个新的promise
    let promise2 = new Promise((resolve, reject) => {
      // 上一个promise的执行结果决定promise2是成功还是失败
      if (this.status === FULFILLED) {
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value)
            resolvePromise(promise2, x, resolve, reject)
          } catch (e) {
            reject(e)
          }
        })
      }
      if (this.status === REJECTED) {
        setTimeout(() => {
          try {
            let x = onRejected(this.reason)
            resolvePromise(promise2, x, resolve, reject)
          } catch (e) {
            reject(e)
          }
        })
      }
      if (this.status === PENDING) {
        onResolvedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onFulfilled(this.value)
              resolvePromise(promise2, x, resolve, reject)
            } catch (e) {
              reject(e)
            }
          })
        })
        onRejectedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onRejected(this.reason)
              resolvePromise(promise2, x, resolve, reject)
            } catch (e) {
              reject(e)
            }
          })
        })
      }
    })
    return promise2
  }

  catch(errCallback) {
    return this.then(null, errCallback)
  }

  // 不管前面的 Promise 是fulfilled还是rejected，都会执行回调函数callback
  // finally方法总是会返回原来的值
  finally(callback) {
    let P = this.constructor
    return this.then(
      value => P.resolve(callback()).then(() => value),
      reason => P.resolve(callback()).then(() => {throw reason})
    )
  }

  // 按顺序执行
  static all(promises) {
    return new Promise((resolve, reject) => {
      let arr = [] // 存放最终结果
      let i = 0
      let processData = (index, data) => {
        arr[index] = data
        if (++i === promises.length) {
          resolve(arr)          
        }
      }
      for (let i = 0; i < promises.length; i++) {
        let current = promises[i];
        if (isPromise(current)) {
          current.then(data => {
            processData(i, data)
          }, (e) => reject(e))
        } else {
          processData(i, current)
        }
      }
    })
  }

  static race(promises) {
    return new Promise((resolve, reject) => {
      for (let i = 0; i < promises.length; i++) {
        let current = promises[i];
        if (!isPromise(current)) {
          current = Promise.resolve(current)
        }
        current.then(data => resolve(data), (e) => reject(e))
      }
    })
  }

  static resolve(value) {
    return new Promise((resolve, reject) => {
      resolve(value)
    })
  }

  static reject(reason) {
    return new Promise((resolve, reject) => {
      reject(reason)
    })
  }
}
```

