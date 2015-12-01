---
layout: post
title:  "Simple, Containerized CI/CD with Devlab + CircleCI"
date:   2015-10-26 10:00:00
categories: docker devlab ci cd
cover: /assets/images/covers/container_ships.jpg
---

Parity is important; not only in ensuring all of your environments are going to always run the same, but it can be a time-saver and prevent duplication of build/deploy code as well.

At [TechnologyAdvice](http://www.technologyadvice.com) we embraced containerization via [DevLab](https://github.com/TechnologyAdvice/DevLab). If you're not familiar with [DevLab](https://github.com/TechnologyAdvice/DevLab) check out [this blog post](http://blog.fluidbyte.net/containerize-your-local-dev-in-minutes-with-devlab/).

In short; DevLab allows you to build a single control file that then handles the Docker setup, run, and tear-down. It was built as a Docker-Compose alternative; to pick up some pieces that were missing or just not solid.

## Parity on CI with CircleCI

In our quest for the perfect stack we tried a number of CI tools, but the one we landed on due to its Docker support and numerous other features was [CircleCI](https://circleci.com/).

We've got parity on our local dev systems, now we wanted to be able to use that single config file with our services, setup, and tasks defined to easily run our CI as well as our deployment. Turns out this wasn't hard at all.

## The Core Components

Obviously having DevLab setup is a given, but from there we needed a `circle.yml` file for CircleCI to do its thing.

{% highlight yaml %}
machine:
  services:
    - docker
  node:
    version: 0.12

dependencies:
  override:
    - npm install -g devlab
    - lab install

test:
  override:
    - lab ci
{% endhighlight %}

In our `devlab.yml` we added a `ci` task which runs the multiple tasks we want to be hit during build:

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
  ci: .lint .test .build
{% endhighlight %}

*Note: the `ci` task uses [task links](https://github.com/TechnologyAdvice/DevLab#running-multiple-tasks) with the preceeding `.` to reference other tasks*

Because CircleCI doesn't support `--rm` in the docker command we had to make one more simple modification, in the project settings we created an environment variable to disable `--rm` on Devlab:

{% highlight yaml %}
DEVLAB_NO_RM = true
{% endhighlight %}

That's it. Everything is all set to run on CircleCI. Our entire build ran in about 2 minutes. This can be sped up by [caching docker layers](https://circleci.com/docs/docker#caching-docker-layers).

**If you'd like to see the full setup check out the [DevLab Demo Repo](https://github.com/TechnologyAdvice/DevLab-Demo).**

## No Redefining Resources!

What's great about this (as mentioned earlier) is we have parity. We didn't need to setup anything and our base container and all our linked service containers are setup in the `devlab.yml`.

All we had to do was configure CircleCI with our existing setup that ran locally and add one line to our DevLab config to tell it what to run.

## Get Shipping

From here it's an easy leap to shipping/deploys. CircleCI has some great [docs](https://circleci.com/docs), but as an easy example, we wanted our builds to push to [DockerHub](http://hub.docker.com). We added the obligatory `Dockerfile` to our repo then added this to our `circle.yml`:

{% highlight yaml %}
deployment:
  staging:
    branch:
      - staging
    commands:
      - docker login -e $DOCKER_EMAIL -u $DOCKER_USER -p $DOCKER_PASS
      - docker build -t $DOCKER_REPO:${CIRCLE_SHA1} .
      - docker push $DOCKER_REPO
{% endhighlight %}

After setting up our environment variables, any push to staging now gets deployed to DockerHub.

In a matter of about 10 minutes we had our CI/CD setup and running with complete parity to our local dev environments.