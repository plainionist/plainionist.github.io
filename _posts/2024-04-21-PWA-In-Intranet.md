---
layout: post
title: "Running a PWA in an intranet"
description: How to host and install a PWA in an intranet
tags: [technologies]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

In the [previous article](/Docker-On-PI) I summarized how I used Docker to host
a web application on my Raspberry PI. This successful experiment inspired me to 
start a new one: Let's turn this "regular" web application into a Progressive Web App (PWA),
only hosted and operating in my local network.

It turned out this experiment was a lot trickier than initially thought ...

<!--more-->

After some research I was convinced that enabling PWA for an existing web application should actually be quite simple.
As my web application is based on [Vue.js](https://vuejs.org/) and [Vite](https://vitejs.dev/) I basically followed 
these articles

- https://www.vuemastery.com/blog/getting-started-with-pwas-and-vue3/
- https://vite-pwa-org.netlify.app/guide/
- https://vite-pwa-org.netlify.app/assets-generator/

## Add support for PWA to the Web UI

So I ran

`vue add pwa`

in the root of the Web UI project of the web application which successfully added the service worker.

Then I installed vite support for PWA 

`pnpm i vite-plugin-pwa -D`

and configured the information for the manifest in the vite.config.js

```javascript
import { fileURLToPath, URL } from 'node:url'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { VitePWA } from 'vite-plugin-pwa'

export default defineConfig({
  build: {
    outDir: '../WebApi/dist/public/'
  },
  plugins: [
    vue(),
    VitePWA({
      registerType: 'autoUpdate',
      includeAssets: ['favicon.ico', 'apple-touch-icon-180x180.png', 'logo.png'],
      manifest: {
        name: 'Price Watch',
        short_name: 'Price Watch',
        description: 'Watch list for crypto and stock prices',
        theme_color: '#1e1e1e',
        icons: [
          {
            src: 'pwa-64x64.png',
            sizes: '64x64',
            type: 'image/png'
          },
          {
            src: 'pwa-192x192.png',
            sizes: '192x192',
            type: 'image/png'
          },
          {
            src: 'pwa-512x512.png',
            sizes: '512x512',
            type: 'image/png'
          },
          {
            src: 'maskable-icon-512x512.png',
            sizes: '512x512',
            type: 'image/png',
            purpose: 'maskable'
          }
        ]
      }
    })
  ],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
})
```

Finally I installed vite's assets generator

`@vite-pwa/assets-generator`

and added the following small script to the `package.json` of the web UI

```json
{
  "scripts": {
    "generate-pwa-assets": "pwa-assets-generator --preset minimal-2023 public/logo.png"
  }
}
```

to generate the minimal images and icons from just a single logo

## HTTPS

As a PWA requires a secure connection I added HTTPS support to backend, which would host the Web UI in the production setup.

```javascript
const express = require('express')
const https = require('https')

const configHome = process.env.CONFIG
const key = fs.readFileSync(`${configHome}/selfsigned.key`);
const cert = fs.readFileSync(`${configHome}/selfsigned.crt`);

const app = express()
const server = https.createServer({key: key, cert: cert }, app);

const port = 8001
const httpsPort = 8002

app.use(express.static('public'))

app.listen(port, () => {
  console.log(`Listening at http://localhost:${port}`)
})

server.listen(httpsPort, function () {
  console.log(`Listening at https://localhost:${httpsPort}`)
})
```

## SSL Certificates

So far so easy. The real challenge was to setup the SSL certificates.

I tried using "self signed" certificates as described [in this article](https://github.com/sagardere/set-up-SSL-in-nodejs)
but the browser complains that these certificates are not trusted.

I then learned about [Let's Encrypt](https://pimylifeup.com/raspberry-pi-ssl-lets-encrypt/) but this doesn't seem to work
in a private network.

I finally learned that I need to be come my own "Certificate Authority" to make the browser stop complaining.

Therefore I followed this article: https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/

After having completed these steps i used the "CX file explorer" App on my smartphone (Android) to copy over the `myCA.pem`
and install it on my device

`Security & location > Encryption & credentials > Install a certificate > CA certificate`

## Installation 

The last necessary step was to change the Docker call to export the HTTPS port and also provide access to the SSL certificates
which I copied to `/home/me/price-watch` on my PI

`docker run -d --restart=unless-stopped --name price-watch -p 8001:8001 -p 8002:8002 -v /home/me/price-watch:/app/config -e "CONFIG=/app/config" price-watch:pi`

When I then opened the web application on my smartphone via the HTTPS port my browser offered me to install the application.
The installation went smooth and since then I can use my web application as if it would be a "native app" on my smartphone.
