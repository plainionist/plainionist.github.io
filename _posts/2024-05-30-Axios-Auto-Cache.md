---
layout: post
title: "How to automatically cache response objects?"
description: How to automatically cache the response objects of your Web Apis using Axios?
tags: [technologies]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

One of my web applications has quite some pages which show quite a bit of data.

In order to achieve smooth navigation between the pages, without bothering the user with progress indicators
when the data of a page is loaded from the Web API every time the user navigates to a particular page,
the data needs to be cached in the browser.

As most of the data is readonly, frameworks like [Vuex](https://vuex.vuejs.org/) or similar state patterns seem to be "too heavy" in this case.

Here is my simple approach to automatically cache the response objects from the Web APIs which provides
great performance of the Web application without introducing much complexity.

<!--more-->

The web application uses [axios](https://github.com/axios/axios) to call the Web APIs.

Axios supports adapters which allows custom handling of requests.

```javascript
import axios from 'axios'
import createCacheAdapter from './api-cacheAdapter.js'

function create ({ baseUrl, autoCache = true }) {
  let API = null

  if (autoCache) {
    API = axios.create({
      withCredentials: true,
      baseURL: baseUrl,
      headers: { 'Cache-Control': 'no-cache' },
      adapter: createCacheAdapter()
    })
  } else {
    API = axios.create({
      withCredentials: true,
      baseURL: baseUrl
    })
  }

  return API
}
```

Such an adapter is a simple function which takes an axios config object and returns a response object.

```javascript
function createCacheAdapter () {
  const adapter = getAdapter('xhr')
  const cache = new Cache()

  return async config => {
    if (config.method !== 'get' || config.forceUpdate) {
      console.log('no-api-cache')
      return adapter(config)
    }

    const cacheKey = createCacheKey(config.url, config.params, config.paramsSerializer)

    const cachedData = cache.get(cacheKey)

    if (!cachedData) {
      try {
        config.responseType = 'text'
        const response = await adapter(config)

        cache.set(cacheKey, response.data)

        return {
          ...response,
          data: parseJson(response.data)
        }
      } catch (reason) {
        cache.remove(cacheKey)
        throw reason
      }
    }

    return Promise.resolve({
      data: parseJson(cachedData),
      status: 200,
      config
    })
  }
}
```

The simple cache adapter above requires two small helper functions to create the cache key

```javascript
function createCacheKey (url, params, paramsSerializer) {
  const builtURL = axios.getUri({ url, params, paramsSerializer })

  const [urlPath, queryString] = builtURL.split('?')

  if (queryString) {
    const paramsPair = queryString.split('&')
    return `${urlPath}?${paramsPair.sort().join('&')}`
  }

  return builtURL
}
```

and to parse text into a Json object

```javascript
function parseJson (data) {
  try {
    return JSON.parse(data)
  } catch (error) {
    return data
  }
}
```

The actual cache implementation is independent from the adapter.
Any of the freely available NPM packages should be suitable.

A simple cache implementation based on sessionStorage could look like this:

```javascript
class Cache {
  constructor (limit) {
    this.limit = limit || 25
    this.cacheMap = new Map()

    const keys = sessionStorage.getItem('keys')
    this.keys = keys ? JSON.parse(keys) : []
  }

  #saveKeys () {
    sessionStorage.setItem('keys', JSON.stringify(this.keys))
  }

  get (key) {
    if (!this.cacheMap.has(key)) {
      const value = sessionStorage.getItem(key)
      if (value) {
        this.cacheMap.set(key, parseJson(value))
      } else {
        return null
      }
    }

    const item = this.cacheMap.get(key)

    this.#updateKeyOrder(key)

    return item
  }

  #removeOldestItem () {
    const oldestKey = this.keys.shift()
    sessionStorage.removeItem(oldestKey)
    this.cacheMap.delete(oldestKey)
  }

  set (key, value) {
    if (this.cacheMap.size >= this.limit && !this.cacheMap.has(key)) {
      // Cache is full, remove the least recently used item
      this.#removeOldestItem()
    }

    while (true) {
      try {
        sessionStorage.setItem(key, value)
        break
      } catch (e) {
        console.log(`WARNING: sessionStorage full! Shrinking and retry (${this.cacheMap.size}) ...`)

        if (this.cacheMap.size > 0) {
          this.#removeOldestItem()
        } else {
          console.log('Item too big - cached in application only!')
          break
        }
      }
    }

    this.cacheMap.set(key, value)
    this.#updateKeyOrder(key)
  }

  #updateKeyOrder (key) {
    // Remove the key if it already exists to reorder it
    const index = this.keys.indexOf(key)
    if (index > -1) {
      this.keys.splice(index, 1)
    }

    this.keys.push(key) // add key as the most recently used
    this.#saveKeys()
  }

  remove (key) {
    const index = this.keys.indexOf(key)
    if (index > -1) {
      this.keys.splice(index, 1)
      this.#saveKeys()
    }

    sessionStorage.removeItem(key)
    this.cacheMap.delete(key)
  }

  clear () {
    for (const key of this.keys) {
      sessionStorage.removeItem(key)
    }

    this.cacheMap.clear()
    this.keys = []

    this.#saveKeys()
  }
}
```

As with every caching strategy, the challenge might be to decide when to invalidate the cache.
In my case the cache only invalidates when the session of the user ends.
