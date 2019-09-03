---
layout: post
title: "RxJava - FlatMap tricks"
date: 2016-09-27 09:00:40 +01:00
author: Guillermo Orellana
comments: true
tags: rxjava,java
redirect_from: /rxjava/2016/09/27/rx-java-flatmap-tricks
---

Quick one: Have you ever had to convert one layer model to the next one, and iterate through all the child elements to convert them as well? Not saying there is something wrong with that, but with a bit of RxJava magic we can make it look so much cooler.

Say you have your *VeryCleanArchitecture™* data source

```java
@NonNull
@Override
public Single<List<InmojiGroup>> get() {
    final List<InmojiCampaignCategory> stuff = StuffSDK.getStuff();
    final List<InmojiGroup> networkStuff = new ArrayList<>();
    for (InmojiCampaignCategory stuffCategory : stuff) {
        List<InmojiItem> items = new ArrayList<>();
        for (StuffItem stuffCategoryItem : stuffCategory.getStuffItems()) {
            items.add(NetworkStuffItem.from(stuffCategoryItem));
        }
        networkStuff.add(new NetworkStuffGroup(stuffCategory.getTitle(), 
                                                  items));
    }
    return Single.just(networkStuff);
}
```

Nice and easy. But it is not *reactive enough*. With a bit of salt and pepper, we can make it look *VeryCleanMuchReactiveArchitecture™* worthy. However, notice that we need to access to some property of the original parent to compose our `NetworkStuffGroup`. A simple `flatMap` won't do it, since we would only have access to the child items. 

But what about doing like this:


```java
@NonNull
@Override
public Single<List<NetworkStuffGroup>> get() {
    return Observable.from(StuffSDK.getStuff())
        .flatMap(
            stuffCategory ->
                Observable.from(stuffCategory.getStuffItems())
                    .map(NetworkStuffItem::from)
                    .toList(),
            (stuffCategory, networkStuffItems) 
                -> new NetworkStuffGroup(
                    stuffCategory.getTitle(), 
                    networkStuffItems)
        )
        .toList()
        .toSingle();
}
```

What the heck was that? Calling `flatMap` with two arguments? Well, it is not straightforward, but [one of the `flatMap` overloads](http://reactivex.io/RxJava/javadoc/rx/Observable.html#flatMap%28rx.functions.Func1,%20rx.functions.Func2%29) has a resultSelector as a second parameter. 

We can use as `resultSelector` a function that takes the item before the flatMap and the grouped collection after the flatmap. This way we can easily create our network model group, without any confusing usage of `Pair<T,U>`. 

And that's all folks! May your code be reactive and your `Observable` never randomly unsubscribe. 
