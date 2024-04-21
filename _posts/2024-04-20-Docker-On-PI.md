---
layout: post
title: "My first Docker container"
description: How to build a docker container hosting a Node.Js based web application.
tags: [technologies]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

![Docker]({{ site.url }}/assets/docker.png "Docker")

Even though Docker is popular already for quite a while, I never had an opportunity to give it a try.
Until recently when I bought a Raspberry PI to host some web applications. 

Of course, hosting these web applications directly on the PI would have been simpler and more efficient
but then I still wouldn't have any hands on experience with Docker.

Here is the summary of my first Docker experiment.

<!--more-->

## The Web Application

The web application consists of a [Node.js](https://nodejs.org/en) backend and 
a [Vue.js](https://vuejs.org/) based Web UI, built with [Vite](https://vitejs.dev/).

![Project setup]({{ site.url }}/assets/price-watch-setup.png "Project Setup")

The backend and the frontend are separate projects because I plan to rewrite the backend in Rust soon.

To make the backend serving the Web UI in production, the Web UI is configured to build directly
into the "public" folder of the web server.

```javascript
import { fileURLToPath, URL } from 'node:url'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  build: {
    outDir: '../WebApi/dist/public/'
  },
  plugins: [
    vue(),
  ],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
})
```

## Dockerfile

My first working Dockerfile produced a 270 MB image which was an unexpected high overhead for a small Node.js based web application.
I did some research, learned about minimal Linux distributions for Raspberry PI and about two stage Docker files and finally came 
up with the Dockerfile below which produces a 65 MB image. 

```dockerfile
ARG NODE_VERSION=18.15.0
FROM --platform=linux/arm64/v8 node:${NODE_VERSION}-alpine as build-stage

ENV NODE_ENV development

WORKDIR /usr/src/app

COPY src/price-watch/WebUI/package.json ./WebUI/
RUN npm install --prefix ./WebUI/

COPY src/price-watch/WebApi/package.json ./WebApi/
RUN npm install --prefix ./WebApi/

COPY src/price-watch/WebApi ./WebApi/
COPY src/price-watch/WebUI ./WebUI/

WORKDIR /usr/src/app/WebUI/
RUN npm run build 

WORKDIR /usr/src/app/WebApi/
RUN npm run build


FROM --platform=linux/arm64/v8 alpine as production-stage

RUN apk add --update nodejs

RUN addgroup -S node && adduser -S node -G node
USER node

COPY --from=build-stage /usr/src/app/WebApi/dist /app/
WORKDIR /app/

EXPOSE 8001

CMD node server.js
```

## Build & Deploy

To build a Docker image I have set up a dedicated builder

`docker buildx create --name pi-builder --use`

and used it to build the image

`docker buildx build --platform linux/arm64 -t price-watch:pi . --load`

I have saved the image locally

`docker save price-watch:pi --output price-watch.tar`

and copied it over to my PI. There I loaded into docker

`docker load --input price-watch.tar`

and successfully started my first Docker container

`docker run -d --restart=unless-stopped --name price-watch -p 8001:8001 price-watch:pi`

