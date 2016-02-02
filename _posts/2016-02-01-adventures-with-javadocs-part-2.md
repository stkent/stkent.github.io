---
layout: post
title: Adventures with Javadocs, part 2
author: Stuart Kent
tags: android, java, javadoc, gradle, open source, library
comments: true

---

This is the follow-up to [Adventures with Javadocs, part 1](). If you haven't already, please go read that - this post will build on the sample project constructed there. 

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
task docTask(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
}
{% endhighlight %}

Generating Javadocs by executing this task produces all output files described in the last post, as well as these by-now-predictable additions:

<div class="image-container">
	<img src="" />
</div>

However, the command-line output of the `docTask` Gradle task includes some new warnings we should understand and resolve:

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
  <li>as plain text, without a fully qualified class name, when the parameter type is not user-created _or_ auto-imported.</li>
</ol>
