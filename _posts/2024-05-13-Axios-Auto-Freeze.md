---
layout: post
title: "Auto freeze response objects with Axios interceptor"
description: How to freeze response objects from the Web API automatically using an Axios interceptor?
tags: [technologies]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

I didn't investigated it on my own, but some research indicated that Vue.Js performs better when the objects
bound to the UI controls are frozen. Of course, calling `Object.freeze` manually in every Vue.Js component
is cumbersome and error-prone so here is how I automatically freeze all response objects I receive
from my Web API using Axios.

<!--more-->

```javascript
import axios from 'axios'

function create ({ baseUrl, autoFreeze = true }) {
  const API = axios.create({
    withCredentials: true,
    baseURL: baseUrl
  })

  API.interceptors.response.use(response => {
    if (autoFreeze) {
      return {
        status: response.status,
        data: deepFreeze(response.data)
      }
    } else {
      return response
    }
  })

  return API
}
```

In order to make sure that, every object bound to the UI is frozen, 
I use a simple custom function to freeze the response object recursively.

```javascript
function deepFreeze (o) {
  if (o === undefined) {
    return o
  }

  Object.freeze(o)

Object.getOwnPropertyNames(o).forEach(prop => {
    if (o[prop] !== null && typeof o[prop] === 'object' && !Object.isFrozen(o[prop])) {
      deepFreeze(o[prop])
    }
  })

  return o
}
```

