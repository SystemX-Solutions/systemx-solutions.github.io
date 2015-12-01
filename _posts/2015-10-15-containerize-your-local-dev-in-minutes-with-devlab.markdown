---
layout: post
title:  "Containerize Your Local Dev in Minutes with DevLab"
date:   2015-10-15 10:00:00
categories: docker devlab nodejs
cover: /assets/images/covers/containers.png
---

>[DevLab](https://github.com/TechnologyAdvice/DevLab) is a utility that allows you to easily containerize your development workflow using Docker. Simply put; it's like having a cleanroom for all of your development processes which contains services (like databases) without needing to setup and maintain these environments manually.

To anyone in web application development it should be obvious that [Docker](https://www.docker.com) is an extremely powerful tool. It's used in CI systems, deployments, and production for scaling applications in ephemeral containers.

However, it's also a powerful tool for developing locally. Almost every application requires resources; databases, other application API's, etc. When working locally you have two options; pollute your environment with these resources or containerize and keep these resources manageable.

## A Simple Use-Case

Let's say you're building an application which needs access to a database. In production this will all be wired up, but locally you need to be able to test the database connection.

I'll be proceeding forward with Node/JavaScript (using ES6 via Babel), but most of this should be easily transferable to other languages and use-cases.

**I've created a [Demo Project](https://github.com/TechnologyAdvice/DevLab-Demo) for reference.**

## Getting Started

Taking a look at the [Demo](https://github.com/TechnologyAdvice/DevLab-Demo) some code is already there in [`/src/index.js`](https://github.com/TechnologyAdvice/DevLab-Demo/blob/master/src/index.js) which is a simple Mongo class for connecting to Mongo and executing a command.

I've set up a corresponding test in [`/test/index.spec.js`](https://github.com/TechnologyAdvice/DevLab-Demo/blob/master/test/index.spec.js), which just creates an instance and runs `stats` against the database.

The [`package.json`](https://github.com/TechnologyAdvice/DevLab-Demo/blob/master/package.json) has a number of `scripts` as well as the dependencies needed.

Basically it has everything in a state where I want to start running tests and building out more logic.

## Setting Up DevLab

First things first: you'll need to have [DevLab](https://github.com/TechnologyAdvice/DevLab) installed which is a simple matter of running a global npm install.

```
npm install devlab -g
```

Now that I have DevLab installed, I need a [`devlab.yml`](https://github.com/TechnologyAdvice/DevLab-Demo/blob/master/devlab.yml) file to control things.

Let's break this down as it's really the crux of what we're looking at in this post:

{% highlight yaml %}
from: node:0.12
services:
  - mongo:3.0:
      name: mongodb
      persist: false
tasks:
  env: env
  clean: rm -rf node_modules
  install: npm install
  test: npm run test
  lint: npm run lint
  build: npm run build
{% endhighlight %}

* I'm telling it with `from` that I want to use Node v.0.12. This container will pull from [DockerHub](https://hub.docker.com/).
* The `services` has one service configured; Mongo v.3.0 which automatically pulls from DockerHub as well. I've given it a name (`mongodb`) and told it to not persist; fresh database/container every time I run.
* I've defined tasks. These should all be pretty self-explanatory.

With `devlab.yml` saved in the root of my project, I'm ready to roll.

## Utilizing DevLab

I have everything configured, and my project is ready for some testing. First, though, I want to get my dependencies installed:

```
devlab install
```

The above will spin up my container (*pulling if necessary), start my services and run the task `install`.

*Note: on first run, this may take a short time as Docker needs to pull the images for your container and service containers. Don't fret - once local, this won't happen again.*

If everything is successful you'll see DevLab spin up your container & service, execute the `install` and complete successfully.

Now that we have everything working we can test our code with the service running to ensure we're all hooked up:

```
lab test
```

*Note the use of `lab` instead of `devlab`. Since you'll be running this often we made an alias which should make things a few less keystrokes.*

When this runs you'll see the following:

![DevLab_Demo](http://zippy.gfycat.com/TheseDefinitiveGarpike.gif)

Again, the container spins up, starts the Mongo service and links it to the primary Node container, runs through the `test` task and completes. Containers get spun-down and stopped, ready for the next task!

## Why DevLab?

At [TechnologyAdvice](http://www.technologyadvice.com) we tried a number of approaches for containerization of local dev environments. Between Makefiles that spun up commands and [Docker-Compose](https://docs.docker.com/compose/) we were tired of our tooling getting in our way, or creating extra tasks for us to manage.

The goal of DevLab is to have a tool with a small footprint, both in application and configuration requirements, which allows for local containerization using Docker.

DevLab was built to create parity between dev and staging/production/etc environments with minimal effort. It also aims to maintain parity with it's support technology, Docker. The nomenclature and config should be familiar to anyone with a basic understanding of LXC.

## It's Only Just Begun

This post covers a very simple (but common) use-case for [DevLab](https://github.com/TechnologyAdvice/DevLab). As with any tool, please take a few minutes and [read the documentation](https://github.com/TechnologyAdvice/DevLab/blob/master/README.md).

DevLab is relatively young, but already in rotation at [TechnologyAdvice](http://www.technologyadvice.com) for all our day-to-day dev tasks. We plan to keep it simple and stable, but are planning new features and functionality to make it even more powerful.

I hope you'll take a few minutes to check it out, and let me know if you have feedback, ideas, or issues.

