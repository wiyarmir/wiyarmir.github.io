---
layout: post
title: "How to convince your PM into refactoring tests"
date: 2016-08-16 12:00:40 +01:00
author: Guillermo Orellana
comments: true
tags: testing,android
redirect_from: /testing/2016/08/16/how-to-convince-your-PM-into-refactoring-tests
---

I know, the two words your (Product Owner\|Product Manager\|Project Manager) hates the most, **tests** and **refactoring** together! Those don't burn points in the chart! Am I trying to make them have a heart attack? Well, not really. It all depends on how you pitch the idea to them.

# The case

For instance, if we take a test file with this header:

![](/assets/article_images/2016-08-16-how-to-convince-your-PM-into-refactoring-tests/androidtest.png)

It is recognisable that this is an Android Instrumentation Test. Nothing wrong with them, but I believe RxJava has nothing to do with the Android framework itself, so why is this an instrumentation test? I can think about many reasons. Legacy structure, old build tools, inertia... But we can do better now.

What about this other header:

![](/assets/article_images/2016-08-16-how-to-convince-your-PM-into-refactoring-tests/junittest.png)

You can quickly spot this is the same class, but is now running as a pure JUnit test. Why even bother doing it?

## Speed

One of our current main issues in the Badoo Android team is our build speed (see more on that in [our AMA follow-up blog post](https://techblog.badoo.com/blog/2016/08/31/top-3-badoo-android-ama/)). Android instrumentation tests definitely do not help with it.

![Before](/assets/article_images/2016-08-16-how-to-convince-your-PM-into-refactoring-tests/androidtestbuildtime.png)

The exact same test cases (8 to be exact) run in 22% of the time when done as a pure JUnit test. Most of this drop comes from the required instrumentation steps such as dexing, installing both the application and the instrumentation APK and bootstrapping them. This causes a significant overhead. It is true that this overhead does not grow too much with the number of test files and cases, but think about how many times you run just a handful test cases to ensure you don't break something while modifying it.

![After](/assets/article_images/2016-08-16-how-to-convince-your-PM-into-refactoring-tests/junittestbuildtime.png)

I agree that 25 seconds is still far away from perfect, but in our huge Badoo project that number is bliss! 


## Temptation free

Your unit tests must not interact with the Android framework. If you can't mock it, then you have a design problem. But this same PM that you are trying to convince needs this new feature super quick, and oh the tests are failing, and it would be *so* easy to just pass some `Context` around... Nope, not today.

## It's not that difficult

Providing it is already a decoupled from Android framework test, it only takes you moving it from `src/androidTest` to `src/test` and adding a few `@RunWith`, `@Test`, `@Before` and `@BeforeClass` annotations. How long would it take you, half a minute? Even if it took a couple of minutes, the second time you ran this test you would already have a benefit out of it.

# Let's do the math

Until now, it's all benefits for the dev team, but maybe your (PO\|PM\|PM) won't see that just yet. Let's add some numbers. Say you have 200 instrumentation tests. Out of those, 100 are easily converted to pure JUnit tests. Say they take 2 minutes to run, and let's be conservative and say you save 50% of the time, that's 1 minute.

What value adds 1 minute? If the tests are ran 16 times a day (say once every half an hour in an ideal 8 hour work day), that's 16 minutes per developer per day. 10 devs in your team, 2 week long sprints (thus 10 working days), it adds up to 1600 minutes or 26.6 hours. That's more than 3 of those *developer days* that (PO\|PM\|PM)s love so much to talk about. How about that!

# Wrap up

There is no lost cause if there is data backing it up. Numbers don't lie - although they can be presented in very suggestive ways...

Of course, your mileage may vary, and in this example numbers are totally made up, but is not an unreasonable reasoning. next time you want to justify something to management, just tell them what they want to hear (backed up with actual data 😉)
