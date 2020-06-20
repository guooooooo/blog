---
title: Nodejs之events模块
date: 2019-11-08
tags:
 - Node
categories:
 - Node
---

## Nodejs之events模块

Nodejs的核心模块`events`模块是典型的**发布/订阅**实现

### 基本用法

```js
const util = require('util')
const EventEmitter = require('events')

// class MyEmitter extends EventEmitter {}
function MyEmitter () {}
util.inherits(MyEmitter, EventEmitter)

const myEmitter = new MyEmitter()

myEmitter.on('newListener', (type) => {
  if (type === 'event') {
    console.log('yes')
  }
})

myEmitter.on('event', () => {
  console.log('event');
})

myEmitter.once('once', () => {
  console.log('once');
})

myEmitter.emit('event')
```

### 常用api实现

```js

function EventEmitter () {
  this._event = Object.create(null)
}

EventEmitter.prototype.on = function (eventName, callback) {
  // 原型继承时没有继承父类属性
  if (!this._event) {
    this._event = Object.create(null)
  }
  // 监听绑定的事件 不是newListener就触发newListener的回调
  if (eventName !== 'newListener') {
    this.emit('newListener', eventName)
  }

  if (this._event[eventName]) {
    this._event[eventName].push(callback)
  } else {
    this._event[eventName] = [callback]
  }
}

EventEmitter.prototype.once = function (eventName, callback) {
  let one = (...args) => {
    callback(...args)
    // 触发一次之后删除自己
    this.off(eventName, one)
  }
  one.l = callback
  this.on(eventName, one)
}

EventEmitter.prototype.emit = function (eventName, ...args) {
  this._event[eventName] && this._event[eventName].forEach(fn => fn(...args))
}

EventEmitter.prototype.off = function (eventName, callback) {
  if (this._event[eventName]) {
    this._event[eventName] = this._event[eventName].filter(fn => fn !== callback && fn.l !== callback)
  }
}

module.exports = EventEmitter
```