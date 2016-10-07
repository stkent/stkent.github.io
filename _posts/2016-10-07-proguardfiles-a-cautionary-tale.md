---
layout: post
title: proguardFiles&#58; A Cautionary Tale
author: Stuart Kent
tags: android
comments: true

---

This week, an assumption I made about the Android Gradle plugin method `proguardFiles` nearly resulted in a minor security slip. Let's all learn from my mistake!

<!--more-->

# Background

The client codebase I'm currently working on has three build types: `debug`, `beta`, and `release`. The `beta` build type is a minor variation of the `debug` build type, so it's configured using the Android Gradle plugin's [`initWith`](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Build-Types) method as shown below:

{% highlight java %}
buildTypes {

  debug {
    // ...
  }

  beta {
    initWith(buildTypes.debug)
    // ...
  }

  release {
    // ...
  }

}
{% endhighlight %}

My tasks today included enabling [ProGuard](https://developer.android.com/studio/build/shrink-code.html) for every single build type. Here's the code I initially wrote to accomplish this:

{% highlight groovy %}
buildTypes {

  debug {
    // ...

    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro', 'proguard-debug.pro'
  }

  beta {
    initWith(buildTypes.debug)
    // ...

    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
  }

  release {
    // ...

    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
  }

}
{% endhighlight %}

The common ProGuard rules defined in the `proguard-android.txt` and `proguard-rules.pro` files are applied to all three build types. The extra ProGuard rules defined in the `proguard-debug.pro` file are applied to the `debug` build type only.

Right?

# Wrong!

The `proguard-debug.pro` file contains exactly one line:

{% highlight text %}
-dontobfuscate
{% endhighlight %}

[Obfuscation](https://en.wikipedia.org/wiki/Obfuscation_(software)) is a useful (but certainly not impenetrable) defense against [reverse engineering](https://en.wikipedia.org/wiki/Reverse_engineering) of a compiled application. However, I have found in the past that it interferes with Android Studio's debugger, so I like to disable it for the non-production build variants I actively develop with.

As described above, my intention was to disable obfuscation for the `debug` build type _only_, leaving obfuscation enabled for the `beta` and `release` build types. To test that this was working as expected, I assembled a `beta` build and inspected the APK contents using [ClassyShark](https://github.com/google/android-classyshark). Here's what our `Parcelable` utility class looked like in ClassyShark:

<div class="image-container">
	<img src="/assets/images/proguardfiles-a-cautionary-tale-no-obfuscation.png" width="100%" />
</div>

Those method names are _definitely_ not obfuscated.

# Huh?

Confused, I jumped to the definition of the `proguardFiles` method commonly used to apply ProGuard configuration to a build type (remember that in Android Studio you can do this using the [“Go To Declaration” shortcut](https://www.jetbrains.com/help/idea/2016.2/navigating-to-declaration-or-type-declaration-of-a-symbol.html)):

{% highlight java %}
public BuildType proguardFiles(Object... files) {
  Object[] var2 = files;
  int var3 = files.length;

  for(int var4 = 0; var4 < var3; ++var4) {
    Object file = var2[var4];
    this.proguardFile(file);
  }

  return this;
}
{% endhighlight %}

Inside the `for` loop, each configuration file is passed to the `proguardFile` method:

{% highlight java %}
public BuildType proguardFile(Object proguardFile) {
  this.getProguardFiles().add(this.project.file(proguardFile));
  return this;
}
{% endhighlight %}

Ah-ha!

The `proguardFiles` method **adds** to an internal list of configuration files rather than specifying a **new** list of configuration files. In my opinion this is [not obvious from the method name](https://en.wikipedia.org/wiki/Principle_of_least_astonishment). At least the [documented behavior is clear](https://google.github.io/android-gradle-dsl/2.2/com.android.build.gradle.internal.dsl.BuildType.html#com.android.build.gradle.internal.dsl.BuildType:proguardFiles(java.lang.Object[])), though confusingly the name `proguardFiles` may also be used to refer to a [getter](https://google.github.io/android-gradle-dsl/2.2/com.android.build.gradle.internal.dsl.BuildType.html#com.android.build.gradle.internal.dsl.BuildType:proguardFiles) defined on the same type!

Armed with this knowledge, we can now walk through exactly what happens (with respect to ProGuard rules) when the `beta` build type is configured:

1. `initWith(buildTypes.debug)` is called, which results in three ProGuard configuration files (`proguard-android.txt`, `proguard-rules.pro`, and `proguard-debug.pro`) being added to the `beta` build type's internal list;

2. `proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'` is called, which results in duplicates of the `proguard-android.txt` and `proguard-rules.pro` being added to the `beta` build type's internal list.

The result: `proguard-debug.pro` is unintentionally included in the `beta` build type's internal list of configuration files, and the `-dontobfuscate` ProGuard rule is therefore applied when packaging our application. This is consistent with what we found in the decompiled APK.

# setProguardFiles to the rescue

The most obvious solution to this problem might be to avoid initializing the `beta` build type using the `debug` build type. However, this would have lead to a lot more duplication in our build.gradle file. Luckily there's a better way.

The `BuildType` class exposes a [`setProguardFiles` method](https://google.github.io/android-gradle-dsl/2.2/com.android.build.gradle.internal.dsl.BuildType.html#com.android.build.gradle.internal.dsl.BuildType:setProguardFiles(java.lang.Iterable)), defined as follows:

{% highlight java %}
public BuildType setProguardFiles(Iterable<?> proguardFileIterable) {
  this.getProguardFiles().clear();
  this.proguardFiles(Iterables.toArray(proguardFileIterable, Object.class));
  return this;
}
{% endhighlight %}

This is exactly how I originally assumed the `proguardFiles` method worked! Any existing configuration files are explicitly **replaced** by those contained in the argument passed to `setProguardFiles`. So, here's a fixed version of our build.gradle file:

{% highlight groovy %}
buildTypes {

  debug {
    // ...

    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro', 'proguard-debug.pro'
  }

  beta {
    initWith(buildTypes.debug)
    // ...

    // New!
    setProguardFiles([getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'])
  }

  release {
    // ...

    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
  }

}
{% endhighlight %}

To check that obfuscation was now properly enabled for `beta` builds, I assembled a new APK and navigated to our `Parcelable` utility class using ClassyShark:

<div class="image-container">
	<img src="/assets/images/proguardfiles-a-cautionary-tale-obfuscation.png" width="100%" />
</div>

Much better!

# Takeaways

- The Android Gradle plugin source code is accessible - utilize it;

- Proactively seek ways to validate your assumptions, no matter how obviously-correct they may seem/feel;

- Double-triple-check changes that potentially impact application security.