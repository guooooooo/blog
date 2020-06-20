---
title: 解析query字符串
date: 2019-12-26
tags:
 - 数据结构与算法
categories:
 - 数据结构与算法
---

```js
function parse(str) {
  return str.split('&').reduce((o, kv) => {
    const [key, value] = kv.split('=')
    // 值为空不处理  handle case2
    if (!value) {
      return o
    }
    deep_set(o, key, value)
    return o
  }, {})
}

function deep_set(o, key, value) {
  // 递归处理对象赋值 handle case3
  const path = key.split(/[\[\]]/g).filter(x => x)
  let i = 0
  for (; i < path.length-1; i++) {
    if (o[path[i]] === undefined) {
      // 变量是数字的话则按数组处理 handle case4
      if (path[i+1].match(/^\d+$/)) {
        o[path[i]] = []
      } else {
        o[path[i]] = {}
      }
    }
    o = o[path[i]]
  }
  // 对相应编码过的特殊字符进行解码 handle case5
  o[path[i]] = decodeURIComponent(value)
}

// test case 1
console.log(parse('a=1&b=2&f=hello'))  // { a: '1', b: '2', f: 'hello' }
// test case 2
console.log(parse('a&b=&f=hello'))  // { f: 'hello' }
// test case 3
console.log(parse('a[name][short]=fox&a[com]=ali&b=yy'))  //  { a: { name: { short: 'fox' }, com: 'ali' }, b: 'yy' }
// test case 4
console.log(parse('a[0]=fox&a[1]=ali'))  //  { a: [ 'fox', 'ali' ] }
// test case 5
console.log(parse('color=blue%20sky'))  //  { color: 'blue sky' }
```