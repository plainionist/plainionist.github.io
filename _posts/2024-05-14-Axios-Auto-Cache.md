---
layout: post
title: "How to automatically cache response objects?"
description: How to automatically cache the response objects of your Web Apis using Axios?
tags: [technologies]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

- caching response objects from web api can significantly improve perofmrance of web apps
- if readonly then vuex or pinia are over engineered
- wouldnt it be great simple automatic caching?

<!--more-->

axios supports adapters

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

such an adapter is a simple function which takes axios config object and returns response object

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

requires two small helper funcitons to create the cache key

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

and to parse text into json

```javascript
function parseJson (data) {
  try {
    return JSON.parse(data)
  } catch (error) {
    return data
  }
}
```

there are NPM packages with cache impl
a simple one based on sessionStorage could look like this

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
