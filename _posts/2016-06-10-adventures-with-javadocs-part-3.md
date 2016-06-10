---
layout: post
title: Adventures with Javadocs, part 3
author: Stuart Kent
tags: android, java, javadoc, gradle, open source, library
comments: true

---

This is the follow-up to Adventures with Javadocs parts [1]({% post_url 2016-01-28-adventures-with-javadocs-part-1 %}) and [2]({% post_url 2016-02-05-adventures-with-javadocs-part-2 %}). If you haven't already, please go read those - this post continues to build on the sample project constructed there, and explores the extra configuration needed to properly handle classes supplied by third party dependencies.

<!--more-->

# Introducing Classes From Third Party Dependencies

Let's add Google's [Gson](https://github.com/google/gson) as a third party dependency of our library:

{% highlight groovy %}
dependencies {
    compile 'com.google.code.gson:gson:2.6.2'
}
{% endhighlight %}

We also introduce a fourth test class to our project, again in a separate package, that depends only on Gson classes:

{% highlight java %}
package com.github.stkent.javadoctests.package4;

import com.google.gson.Gson;

/**
 * This class depends on a third party type only!
 */
public class TestClassFour {

    private final Gson gson;

    public TestClassFour(final Gson gson) { this.gson = gson; }

    public Gson getGson() { return gson; }

}
{% endhighlight %}

<!--more-->

Our `docs` task is still configured as follows:

{% highlight groovy %}
task docs(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath = files(((Object) android.bootClasspath.join(File.pathSeparator)))
}
{% endhighlight %}

and running it produces all output from before plus the expected additions for our new package/class:

<div class="image-container">
	<img src="/assets/images/javadoc-tool-task-extra-output-package4.png" />
</div>

However, similarly to when we first included Android classes in our library project, the command-line output of the `docs` task includes new warnings associated with our new class:

{% highlight text %}
/Users/stkent/dev/personal/libraries/javadoc-tests/library/src/main/java/com/github/stkent/javadoctests/package4/TestClassFour.java:3: error: package com.google.gson does not exist
import com.google.gson.Gson;
                      ^
/Users/stkent/dev/personal/libraries/javadoc-tests/library/src/main/java/com/github/stkent/javadoctests/package4/TestClassFour.java:10: error: cannot find symbol
    private final Gson gson;
                  ^
  symbol:   class Gson
  location: class TestClassFour
/Users/stkent/dev/personal/libraries/javadoc-tests/library/src/main/java/com/github/stkent/javadoctests/package4/TestClassFour.java:12: error: cannot find symbol
    public TestClassFour(final Gson gson) {
                               ^
  symbol:   class Gson
  location: class TestClassFour
/Users/stkent/dev/personal/libraries/javadoc-tests/library/src/main/java/com/github/stkent/javadoctests/package4/TestClassFour.java:16: error: cannot find symbol
    public Gson getGson() {
           ^
  symbol:   class Gson
  location: class TestClassFour
4 warnings

BUILD SUCCESSFUL
{% endhighlight %}

This should not be at all surprising. Based on our investigations in the last post, we know that `com.google.gson.Gson` is a newly-added _referenced class_ whose definition is currently _not_ included in the classpath we are supplying to the `javadoc` tool. Let's fix that.

The code for this portion of the post is available [here](https://github.com/stkent/javadoc-tests/tree/528fd7f).

# Adding Third Party Classes To The Classpath

Since third party dependencies are, by definition, not bundled with our Android SDK install, we will need to do a little bit more work to locate their compiled class files. The simplest way to achieve this is to leverage the `libraryVariants` method exposed by the Android library project plugin. This allows us to redefine our `docs` task as follows:

{% highlight groovy %}
android.libraryVariants.all { variant ->
    if (variant.name == 'release') {
        task docs(type: Javadoc) {
            source = variant.javaCompile.source
            classpath = files(((Object) android.bootClasspath.join(File.pathSeparator)))
            classpath += files(variant.javaCompile.classpath.files)
        }
    }
}
{% endhighlight %}

There are a few new things going on here, so let's break this code down a bit.

- The `android.libraryVariants.all` method accepts a closure that will be applied to each variant of our library. (If you need a refresher on what a variant is, see [the Android Gradle Plugin documentation](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Build-Type-Product-Flavor-Build-Variant).);

- We define our docs task based on the variant with name 'release' only. This is fine for our simple test library, since we aren't leveraging product flavors at all. For more complex library project structures, it may be desirable to define a separate Javadoc-generating task per-variant[^1];

- We have updated how we locate source files. Rather than explicitly linking to a single source set (`android.sourceSets.main.java.srcDirs`), we now reference the possibly-aggregated source set `variant.javaCompile.source`. This makes no difference in our simplified case, but again may be required if you are leveraging product flavors;

- We have added the files located at `variant.javaCompile.classpath.files` to the `javadoc` task's classpath. These files include the compiled versions of all our dependencies, which is exactly what we were looking for! Super-convenient.

To confirm I'm not lying about that last point, here's the content of the javadoc.options file generated by executing this redefined `docs` task:

{% highlight text %}
-classpath '/Users/stkent/Library/Android/sdk/platforms/android-23/android.jar:/Users/stkent/.gradle/caches/modules-2/files-2.1/com.google.code.gson/gson/2.6.2/f1bc476cc167b18e66c297df599b2377131a8947/gson-2.6.2.jar'
-d '/Users/stkent/dev/personal/libraries/javadoc-tests/library/build/docs/javadoc'
-doctitle 'library API'
-quiet 
-windowtitle 'library API'
'/Users/stkent/dev/personal/libraries/javadoc-tests/library/src/main/java/com/github/stkent/javadoctests/package1/TestClassOne.java'
'/Users/stkent/dev/personal/libraries/javadoc-tests/library/src/main/java/com/github/stkent/javadoctests/package2/TestClassTwo.java'
'/Users/stkent/dev/personal/libraries/javadoc-tests/library/src/main/java/com/github/stkent/javadoctests/package3/TestClassThree.java'
'/Users/stkent/dev/personal/libraries/javadoc-tests/library/src/main/java/com/github/stkent/javadoctests/package4/TestClassFour.java'
{% endhighlight %}

That long first line confirms that the `javadoc` tool will search for referenced classes within the Android platform and Gson JARs!

# Results

Let's check our actual Javadoc output and verify that it includes the full package names for classes in third-party dependencies:

<div class="image-container">
	<img src="/assets/images/javadoc-tool-generated-testclassfour.png" />
</div>

Perfect :-)

The code for this portion of the post is available [here](https://github.com/stkent/javadoc-tests/tree/2c8a42c).

# Generalizing

I mentioned in a couple places that the `docs` task we defined would not necessarily be suitable for a library that utilizes product flavors. For completeness, here's how we could declare a separate Javadoc-generating task for each library variant:

{% highlight groovy %}
android.libraryVariants.all { variant ->
    task("${variant.name}Docs", type: Javadoc) {
        source = variant.javaCompile.source
        classpath = files(((Object) android.bootClasspath.join(File.pathSeparator)))
        classpath += files(variant.javaCompile.classpath.files)
    }
}
{% endhighlight %}

# Next Time

Linking to referenced classes, and more!