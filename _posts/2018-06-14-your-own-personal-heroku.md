---
layout: post
title:  "Your own personal Heroku"
date:   2018-06-14 13:37:00 +01:00
categories: kotlin
author: wiyarmir
comments: true
---

I've just finished doing a talk at the [Milan Kotlin Community Conference](https://milan.kotlincommunityconf.com/) about a recent pet project of mine: [Keynotedex](https://keynotedex.wiyarmir.es/).

Keynotedex ([code here](https://github.com/wiyarmir/keynotedex)) is a project that I created to experiment with Kotlin multiplatform in platforms I am not that confident with, such as web and backend.

One of the most frequently asked questions was "how do you deploy your server code?". I get it, it is all nice and beautiful in the examples, but they never go over the "production" case.

## Finding a home for our backend code

At the end of the day, your Kotlin server code is just a JAR file. You could use [the `war` Gradle plugin](https://docs.gradle.org/current/userguide/war_plugin.html) to generate a standard container you could drop in a server such as [Apache Tomcat](http://tomcat.apache.org/). But that's scary, or at least it seems like overly complicated for a pet project.

[Heroku](https://www.heroku.com/) is a PaaS that lets you deploy apps with git commands. Although more popular within the JavaScript and Ruby communities, they fully support JVM as well. Deploying your code by just doing:

```bash
$ git push herkou master
```

It's priceless. Actually no, there are some limitations on what you can get from Heroku for free. Your service will have some hours of runtime allowed, then capped unless you switch to the paid tier. That's fair enough, but what if you want the flexibility of deploying from git commands, and you already have a spare server from, say, a weird Black Friday shopping rampage? (True story, by the way)

## Enter Dokku and Heroku(ish)

Honouring [the Depeche Mode reference](https://www.youtube.com/watch?v=i2GEOcEcRtY) at the title, somebody already had figured out that question. You can install your own version of Heroku, in your own servers, and use that great workflow.

There are two tools that take part in that: [herokuish](https://github.com/gliderlabs/herokuish) emulates Heroku build and runtime tasks in containers, and [Dokku](https://github.com/dokku/dokku) manages such containers using [Docker](https://www.docker.com/). And it's lightweight. They claim to be "The smallest PaaS implementation you've ever seen".

## That's a lot of DevOps jargon, show me how to do it

Nobody is going to be able to teach you how to get Dokku up better [than their own docs](http://dokku.viewdocs.io/dokku/) (and those won't go out of date as quick as this article).

# Preparing our code to run on Heroku(ish)

Even if I did go for Dokku, everything here applies to a Heroku deployment as well. I will be basing my examples on code written using [Ktor](http://ktor.io). Other frameworks for some of the techniques may work, but your mileage may vary.

Remember what I mentioned at the beginning, our code is bundled into just a JAR file. In theory, we could just run `java -jar server.jar`. However, doing that will point out one thing: regular JAR files don't really carry their dependencies over.

## Stuffing all your deps inside a JAR

In order for our dependencies to be in the classpath at runtime, we need to carry them over and create what is known as a fat JAR or uber JAR. You will understand the naming when you check the size of that new JAR. There's a Gradle plugin that will do it for you, called [`shadow`](https://github.com/johnrengelman/shadow). 

Applying it to our backend module will create new tasks that will take the outcome of our `jar` task and fill it up with the classes from our dependencies. If you run `./gradlew server:shadowJar` you will find in your `build/libs` a new JAR file with the suffix `-all`. That's the one! 

There are lots of things you can customise in this process. If you are up for it, the best way is to create a task of type `ShadowJar`, the same way you would create one of type `Jar`.

As an example, this is the task I created to have a release style shadow JAR, with the purpose of adding some extra sources not present in the regular JAR.

```groovy
task releaseJar(type: ShadowJar) {
    manifest {
        attributes "Main-Class": "keynotedex.backend.ServerKt"
    }
    classifier = 'release'
    from([sourceSets.release.output, sourceSets.main.output])
    configurations = [project.configurations.compile]
}
```

Since had a new task created, I had to define where my main class is. This way, the JAR file can be run from the terminal. This is not necessary if you are just using the output of the `shadowJar` task, it will pick whatever [the `application` plugin](https://docs.gradle.org/current/userguide/application_plugin.html) says.

Note that the property `classifier` is what will be appended to your JAR file name, so in this case a file with the name `backend-0.0.1-release.jar` will be generated. Now, take a moment to test that, when running the JAR file, your server spins up.

## IKEA instructions, but for your PaaS

Now we are able to build a shadow JAR that contains our dependencies, but is our service able to do so? Unless we want to commit our build output (please don't), we will have to tell our service how to reach that goal state.

According to [the official Heroku docs](https://devcenter.heroku.com/articles/deploying-gradle-apps-on-heroku#overview) (and Herokuish follows them as expected), when it detects a Gradle project, Heroku will attempt invoke one command:

```bash
$ ./gradlew stage
```

We can take advantage of that and use that hook to trigger the whole shadow JAR process. You do not need to write a fully fledged shiny new task, you can just make it depend on your backend tasks like this:

```groovy
task stage() {
    group "distribution"
    dependsOn(':backend:shadowJar')
}
```

Remember, this task must be defined inside the root Gradle project for it to work.

## How do you turn this on

Now that the service knows how to build it, we have to tell it how to run it. In our case, we create a file at the root of our repository called `Procfile`, which contains all the services we define and how to run them. Mine looks like this, but remember my JAR classifier is `release`:

```
web: java -Dserver.port=$PORT $JAVA_OPTS -jar backend/build/libs/*-release.jar
```

`$PORT` and `$JAVA_OPTS` will be defined by the service at run time, so make sure to pass those on and use them in your code, otherwise your backend may not be able to reach the internet.

## Ship it! :shipit:

Write the aforementioned command, play your favourite tune, sit back, relax.

```bash
$ git push dokku master
```

Here is a little eye candy showing how it looks like to deploy Keynotedex:

<script src="https://asciinema.org/a/M5Xlz2XsY7mkHfURPfQx2UQXZ.js" id="asciicast-M5Xlz2XsY7mkHfURPfQx2UQXZ" async></script>

## Bonus stage: easy HTTPS with Letsencrypt

Until now, both Heroku and Herokuish environments are able to do what I describe, but this one is exclusive of Dokku.

HTTPS is very important, I am not even going to discuss it here. And [Let's Encrypt](https://letsencrypt.org/) has enabled many people to obtain what used to be an expensive luxury: SSL certificates.

Fear not, you can use [dokku-letsencrypt](https://github.com/dokku/dokku-letsencrypt): a dokku plugin for automagically obtaining SSL certificates for your domain, and enabling HTTPS. And it is as easy as deploying a new version. This is how I renew my certificate in Keynotedex:

```bash
$ dokku letsencrypt keynotedex
```

# Final thoughts

The Heroku style of deployment has made publishing a backend service as easy as deploying an APK into an Android phone. This is great because makes backend less scary and confusing.

In addition, Dokku allows you to have full control over the system, and use it in environments where you would not be allowed to put things into a public cloud.

This should not be thought as exclusive for human-triggered deployments, as this simplification will surely make systems for deploy-on-merge way simpler. 

Thanks to [Jose A. Corbacho](https://twitter.com/corbyjerez) for taking the time to review this article.