---
layout: post
title: "Avoiding the clean boilerplate: Welcome Project Lombok"
date: 2016-10-08 18:00:40 +01:00
categories: android
author: Guillermo Orellana
comments: true
tags: android,lombok,clean
---

(Intro)

# Project Lombok

[Project Lombok](https://projectlombok.org/) is magic made code. Well, it's actually an annotation processor that generates code, but instead of creating derivated clases we have to guess (I am looking at you, Dagger2), it *injects* the generated code in our class. Told you, Magic!

# Project Lombok and Android

Project Lombok needs little to no configuration, but there are some special cases we need to consider when using it in Android. In order to customise the behaviour of Lombok, we can put a file named `lombok.config` in one of the roots of our project. I usually have it at the root of the module or modules that actually make use of Lombok, since we might choose not to use it in all of them.

# java.beans.ConstructorProperties

Android lacks of the beans package (thankfully), so when you try to use one of the constructor family annotations, you will get a compile time error similar to this one:

```
```

In order to disable such annotations from generating, just add this line to your `lombok.config`:

```
lombok.anyConstructor.suppressConstructorProperties = true

```

# javax.annotation.Generated

```
lombok.addGeneratedAnnotation = false
```

# Comparison

# Alternatives

# Gains
