---
layout: post
title: "Integrating React Native in an existing application"
#date: 2016-06-12 15:37:40 +00:00
categories: react-native
author: Guillermo Orellana
comments: true
tags: react-native,android
---

As I promised by the end of my previous article, ["Writing an Android component for React Native"](https://guillermoorellana.es/react-native/2016/06/12/writing-android-component-for-react-native.html), here is the second approach for React Native and native Android code to live together ~~in peace and harmony~~ and not blow apart. Since this is a follow-up to the previous article, I will jump straight into matter. If you need a brief introduction on React Native and how it's organised on Android side, please visit the forementioned link.

# Getting it to work

After following the docs sample, you might end with an activity similar to this. Et Voilá! Well... Not really. I found a few rough edges not mentioned in the docs.

```java
public class ReactActivity extends AppCompatActivity implements DefaultHardwareBackBtnHandler {

    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mReactRootView = new ReactRootView(this);
        mReactInstanceManager = ReactInstanceManager.builder()
                .setApplication(getApplication())
                .setBundleAssetName("index.android.bundle")
                .setJSMainModuleName("index.android")
                .addPackage(new MainReactPackage())
                .setUseDeveloperSupport(BuildConfig.DEBUG)
                .setInitialLifecycleState(LifecycleState.RESUMED)
                .build();
        mReactRootView.startReactApplication(mReactInstanceManager, "ReactSample", null);

        setContentView(mReactRootView);
    }

    @Override
    protected void onPause() {
        super.onPause();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onPause();
        }
    }

    @Override
    protected void onResume() {
        super.onResume();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onResume(this, this);
        }
    }

    @Override
    public void onBackPressed() {
        if (mReactInstanceManager != null) {
            mReactInstanceManager.onBackPressed();
        } else {
            super.onBackPressed();
        }
    }

    @Override
    public void invokeDefaultOnBackPressed() {
        super.onBackPressed();
    }
}
```
## java.lang.UnsatisfiedLinkError: could find DSO to load: libreactnativejni.so

Uh Oh! This sounds scary! What happened here? Well, I decided to test it on a Samsung Galaxy S7 (running Marshmallow, that also will be important in a second or two)... Which happens to be a 64 bit phone. And React Native [does not provide a 64bit version of its binary](https://github.com/facebook/react-native/issues/2814). Usually Android would do a fallback, but this does not happen if you have a 64bit dependency in theory, but in practice I found it happening in a blank modern project.

![Uh-oh](linking-error.png)

### Solution

To avoid this, we will have to use some deprecated API in the gradle plugin.

```groovy
ndk {
    abiFilters "armeabi-v7a", "x86"
}
```

Did I mention it's a *deprecated* API?

![Ugh](deprecated.png)

## BadTokenException: Unable to add window  -- permission denied for this window type

One very cool thing about developing in React Native is that you get to see the exceptions overlaid on top of your app if you are in debug mode. 

One very cool thing about Android Marshmallow is its new permission system, which allows you to take the power back and have a granular control on what you allow your app to do or not.

One very **not cool** thing about our empty app is that we do not request permission to draw over apps, but React Native will attempt to use it anyway, resulting on this beautiful stacktrace.

![Unexpected exception](badtokenexception.png)

### Solution

```java
if (Build.VERSION.SDK_INT >= >= Build.VERSION_CODES.M) {
    // Get permission to show redbox in dev builds.
    if (!Settings.canDrawOverlays(this)) {
        Intent serviceIntent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION);
        startActivity(serviceIntent);
    }
}
```
