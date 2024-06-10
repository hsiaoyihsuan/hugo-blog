---
title: "How to Dockerize NestJS"
date: 2024-05-14
categories:
  - Docker
  - NodeJS
tags:
  - NodeJS
  - NestJS
  - Back-end
  - Docker
---
This article provide a straightforward guide to dockerizing NestJS application for both develpoment and production environments.

---
## Dockerfile for Development
First, let's create a `Dockerfile.dev` for the development environment within our NestJS repository:
```Dockerfile
# Dockerfile.dev
FROM node:20-alpine

WORKDIR /usr/src/app

EXPOSE 3000

CMD ["npm", "run", "start:dev"]

COPY package*.json ./

RUN npm install

COPY . .
```
In this Dockerfile, we specify the Node.js version along with the Alpine Linux distribution to minimize the image size. Note the order of commands such as `EXPOSE` and `CMD` to optimize Docker's caching mechanism. Since code modifications are frequent during development, we place the `COPY` command towards the end of the Dockerfile to reduce rebuild time.

Now, let's build the image from `Dockerfile.dev`:
```bash
$ docker build -t nest-dev -f Dockerfile.dev .
```
After building, let's check the image size:
```bash
$ docker images
REPOSITORY                  TAG       IMAGE ID       CREATED              SIZE
nest-dev                    latest    7105d40d77e3   About a minute ago   327MB
```
Building this Docker image with the base image `node:20-alpine` yields a size of 327MB, which is quite efficient. However, if we were to use `node:20` instead of `node:20-alpine` as the base image, the image size would increase significantly, up to 1.29GB.

## Dockerfile for Production
Even though 327MB is relatively small, we can further optimize the image size for the production environment by removing unnecessary files and packages during the build process. Here's the `Dockerfile.prod` for production:
```Dockerfile
FROM node:20-alpine AS builder

COPY . .

RUN npm install && npm run build && npm install --prod

FROM node:20-alpine

WORKDIR /usr/src/app

COPY --from=builder /node_modules ./node_modules

COPY --from=builder /dist ./dist

COPY /.env ./.env

EXPOSE 3000

CMD ["node", "./dist/main"]
```
In this Dockerfile, we utilize **Multi-stage builds** to first compile the TypeScript code into JavaScript and then install only the production dependencies, thereby significantly reducing the image size.

Let's build the production image and compare the size difference with the development version:
```bash
$ docker build -t nest-prod -f Dockerfile.prod .

$ docker images
REPOSITORY                  TAG       IMAGE ID       CREATED              SIZE
nest-dev                    latest    7105d40d77e3   About a minute ago   327MB
nest-prod                   latest    04f3894b4cf6   About a minute ago   141MB
```
As you can see, the production image size is half of the development version, resulting in reduced tranportation time and storage requirements.
