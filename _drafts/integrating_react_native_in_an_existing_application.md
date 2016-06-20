---
layout: post
title: "Integrating React Native in an existing application"
#date: 2016-06-12 15:37:40 +00:00
categories: react-native
author: Guillermo Orellana
comments: true
tags: react-native,android
---

As I promised by the end of my previous article, ["Writing an Android component for React Native"](https://guillermoorellana.es/react-native/2016/06/12/writing-android-component-for-react-native.html), here is the second approach for React Native and native Android code to live together ~~in peace and harmony~~ and not blow apart.


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


ISSUES:

* 64 bit

```groovy
ndk {
    abiFilters "armeabi-v7a", "x86"
}
```
