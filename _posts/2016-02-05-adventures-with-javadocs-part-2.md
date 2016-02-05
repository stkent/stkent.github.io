---
layout: post
title: Adventures with Javadocs, part 2
author: Stuart Kent
tags: android, java, javadoc, gradle, open source, library
comments: true

---

This is the follow-up to [Adventures with Javadocs, part 1]({% post_url 2016-01-28-adventures-with-javadocs-part-1 %}). If you haven't already, please go read that - this post will build on the sample project constructed there, and explore the extra configuration needed to properly handle Android framework classes.

<!--more-->

## Introducing Android Framework Classes

Let's add a third test class to our project, again in a separate package:

{% highlight java %}
package com.github.stkent.javadoctests.package3;

import android.os.Bundle;

/**
 * This class depends on an Android-specific type only!
 */
public class TestClassThree {

    private Bundle bundle;

    public TestClassThree(Bundle bundle) { this.bundle = bundle; }

    public Bundle getBundle() { return bundle; }

}
{% endhighlight %}

No other changes have been made yet; in particular, our `docs` task is still configured as follows:

{% highlight groovy %}
task docs(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
}
{% endhighlight %}

Generating Javadocs by executing this task produces all output files described in the last post, as well as these by-now-predictable additions:

<div class="image-container">
	<img src="" />
</div>

However, the command-line output of the `docs` Gradle task includes some new warnings we should understand and resolve:

{% highlight text %}
/Users/stuart/dev/personal/libraries/JavadocTests/library/src/main/java/com/github/stkent/javadoctests/package3/TestClassThree.java:3: error: package android.os does not exist
import android.os.Bundle;
                 ^
/Users/stuart/dev/personal/libraries/JavadocTests/library/src/main/java/com/github/stkent/javadoctests/package3/TestClassThree.java:10: error: cannot find symbol
    private final Bundle bundle;
                  ^
  symbol:   class Bundle
  location: class TestClassThree
/Users/stuart/dev/personal/libraries/JavadocTests/library/src/main/java/com/github/stkent/javadoctests/package3/TestClassThree.java:12: error: cannot find symbol
    public TestClassThree(final Bundle bundle) {
                                ^
  symbol:   class Bundle
  location: class TestClassThree
/Users/stuart/dev/personal/libraries/JavadocTests/library/src/main/java/com/github/stkent/javadoctests/package3/TestClassThree.java:16: error: cannot find symbol
    public Bundle getBundle() {
           ^
  symbol:   class Bundle
  location: class TestClassThree
4 warnings

BUILD SUCCESSFUL
{% endhighlight %}

The code for this portion of the post is available [here](https://github.com/stkent/javadoc-tests/tree/9e9125850ba13b7988ac5105fe826cccd6a2f681).

## Generated Documentation

Here's the summary generated for `TestClassThree`:

<div class="image-container">
	<img src="/assets/images/javadoc-tool-generated-testclassthree.png" />
</div>

The reported warnings did not prevent the generation of appropriate documentation. So, what _was_ their impact?

In the previous post, we observed that constructor parameters were documented in two different formats:

<ol start="1">
  <li>as a hyperlink, without a fully qualified class name, when the parameter type was user-created;</li>
  <li>as plain text, with a fully qualified class name, when the parameter type was auto-imported.</li>
</ol>

The generated documentation for `TestClassThree` introduces a third variation:

<ol start="3">
  <li>as plain text, without a fully qualified class name, when the parameter type is not user-created <em>or</em> auto-imported.</li>
</ol>

As per the last post, the lack of a hyperlink in format 3 is straightforward to understand - since the Android `Bundle` class is not part of the collection of source files for which we are generating documentation, there's no way the `javadoc` tool could know where to link to![^1] To help us understand the difference between text formats 2 and 3, let's return to the `javadoc` documentation and try to understand the "package does not exist" and "symbol not found" errors we received above.

## Class Classifications

The `javadoc` documentation [introduces](http://docs.oracle.com/javase/6/docs/technotes/tools/windows/javadoc.html#terminology) the following terminology to describe the different roles that can be played by Java classes during a documentation-generating run:

> **documented/included classes:** The classes and interfaces for which detailed documentation is generated during a javadoc run.

> **referenced classes:** The classes and interfaces that are explicitly referred to in the definition (implementation) or doc comments of the documented classes and interfaces.

`TestClassOne`, `TestClassTwo` and `TestClassThree` are clearly examples of documented classes. What about `java.lang.String` and `android.os.Bundle`? Both are "explicitly referred to in the definition (implementation) or doc comments of the documented classes and interfaces", so both are referenced classes. However, this additional commentary regarding referenced classes gives us a clue as to why `java.lang.String` and `android.os.Bundle` are handled differently by `javadoc`:

> When the Javadoc tool is run, it should load into memory all of the referenced classes in javadoc's bootclasspath and classpath. [...] The Javadoc tool can derive enough information from the .class files to determine their existence and the fully-qualified names of their members.

The path stored in bootclasspath represents the location of Java's Bootstrap classes (the classes that implement the [Java platform](https://en.wikipedia.org/wiki/Java_(software_platform)#Platform)). This has default value `$JAVA_HOME/jre/lib`, which contains (among other things) compiled class files that collectively form Java's standard library. In particular, the rt.jar JAR contains a compiled String.class file.[^2]

The path stored in classpath represents the location of all referenced classes that are not part of the Java platform. By default, this is an empty path (i.e. no locations are searched to locate additional referenced classes).

## Mystery Understood

We now have enough information to understand the differences between constructor parameter text formats 2 and 3. Recall that we are able to inspect the command-line options passed to the `javadoc` tool for each invocation of the `docs` task by peeking at the javadoc.options file. For the current codebase, this file has the following content:

{% highlight text %}
-d '/Users/stuart/dev/personal/libraries/JavadocTests/library/build/docs/javadoc'
-doctitle 'library API'
-quiet 
-windowtitle 'library API'
'/Users/stuart/dev/personal/libraries/JavadocTests/library/src/main/java/com/github/stkent/javadoctests/package1/TestClassOne.java'
'/Users/stuart/dev/personal/libraries/JavadocTests/library/src/main/java/com/github/stkent/javadoctests/package2/TestClassTwo.java'
'/Users/stuart/dev/personal/libraries/JavadocTests/library/src/main/java/com/github/stkent/javadoctests/package3/TestClassThree.java'
{% endhighlight %}

which indicates that we are utilizing the default bootclasspath and classpath values. Since `java.lang.String` is part of the Bootstrap classes, the `javadoc` tool is able to determine the fully-qualified class name and uses this in the documented method signature for `TestClassOne`:

{% highlight java %}
TestClassOne(java.lang.String string)
{% endhighlight %}

However, `android.os.Bundle` is _not_ part of the (Java) Bootstrap classes, so the `javadoc` tool cannot load the class into memory and determine its fully-qualified name. This leads to the warnings we saw in the output of our `docs` task, and to the text format of the documented method signature for `TestClassThree`:

{% highlight java %}
TestClassThree(Bundle bundle)
{% endhighlight %}

## Mystery Solved

Now that we know why the error occurs, the solution is but a small step away. Our goal should be to modify the classpath used by the `javadoc` tool so that it includes the compiled Android framework classes. We can achieve this by modifying our task configuration as follows:

{% highlight groovy %}
task docs(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath = files(((Object) android.bootClasspath.join(File.pathSeparator)))
}
{% endhighlight %}

This addition uses the `bootClassPath` property that is conveniently exposed by the Android Gradle plugin to locate the appropriate Android classes. Re-running the `docs` task using this configuration results in no warnings, and a documented method signature for `TestClassThree` whose format now matches that of `TestClassOne`:

{% highlight java %}
TestClassThree(android.os.Bundle bundle)
{% endhighlight %}

The code for this portion of the post is available [here](https://github.com/stkent/javadoc-tests/tree/806e5bfcad8000949bfa5158ecc9b0a90f6b377a).

## Next Time

Up next: handling third-party dependencies!

[^1]: Note that it is possible to introduce links to the documentation hosted on d.android.com with a little more configuration work - we'll address this in a later post in the series!
[^2]: You can verify this yourself by unarchiving the `$JAVA_HOME/jre/lib/rt.jar` JAR, and navigating to the `java/lang` subfolder!
