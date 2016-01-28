---
layout: post
title: Adventures with Javadocs, part 1
author: Stuart Kent
tags: android, java, javadoc, gradle, open source, library
comments: true

---

Hello! It's been a while since I wrote a post. This is because I have been busy: first learning Swift and iOS development on a crunchy client project, and more lately working on some open source Android tools. I still have more to write about interpolators - fret not! - but right now it's easier for me to write posts about my primary foci.

Part of publishing high-quality libraries is providing high-quality documentation, which in Android-land means: high-quality Javadocs. There are tons of good resources around that explain proper Javadoc comment format and content, so in this series we'll explore the actual generation of documentation using Gradle/Android Studio. Let's begin!

<!--more-->

## Javadoc Task Type Basics

The Gradle Java plugin provides a [template Javadoc task](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.javadoc.Javadoc.html) with the following description:

> Generates HTML API documentation for Java classes.
> 
> If you create your own Javadoc tasks remember to specify the 'source' property! Without source the Javadoc task will not create any documentation.

For an Android library project, the simplest configuration of this task would therefore be:

{% highlight groovy %}
apply plugin: 'com.android.library'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.2"

    defaultConfig {
        minSdkVersion 16
        targetSdkVersion 23
        versionName "1.0.0"
    }
}

task docTask(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
}
{% endhighlight %}

[This assumes a project with no other dependencies, no flavor/type/variant-specific source files, etc.]

Let's add a super-simple pair of test classes to the project, each in their own package:

{% highlight java %}
package com.github.stkent.javadoctests.package1;

/**
 * This class depends on an automatically-imported type only!
 */
public class TestClassOne {

    private String string;

    public TestClassOne(String string) { this.string = string; }

    public String getString() { return string; }

}
{% endhighlight %}

{% highlight java %}
package com.github.stkent.javadoctests.package2;

import com.github.stkent.javadoctests.package1.TestClassOne;

/**
 * This class depends on a user-created and -imported type only!
 */
public class TestClassTwo {

    private TestClassOne testClassOne;

    public TestClassTwo(TestClassOne testClassOne) { this.testClassOne = testClassOne; }

    public TestClassOne getTestClassOne() { return testClassOne; }

}
{% endhighlight %}

and generate Javadocs by executing our new `docTask` task:

{% highlight bash %}
./gradlew library:clean library:docTask
{% endhighlight %}

which produces the following output files:

<div class="image-container">
	<img src="/assets/images/javadoc-tool-task-output.png" />
</div>

Note in particular that the directory structure of the generated website matches the package structure of the original source files. For example, the class `TestClassOne` is part of the `com.github.stkent.javadoctests.package1` package, and its corresponding documentation page is inside the `com/github/stkent/javadoctests/package1` subdirectory of the generated website.

## Generated Documentation

Before proceeding, I'd like to review a pair of sample pages from this output. This will provide some important context for the next post in this series.

Here's the summary generated for `TestClassOne`:

<div class="image-container">
	<img src="/assets/images/javadoc-tool-generated-testclassone.png" />
</div>

and the corresponding summary generated for `TestClassTwo`:

<div class="image-container">
	<img src="/assets/images/javadoc-tool-generated-testclasstwo.png" />
</div>

The `TestClassTwo` constructor signature includes a hyperlink to the generated documentation page for its lone parameter type (`TestClassOne`). Based on the correspondence between directory structure and package structure we identified earlier, I would postulate that the `javadoc` tool generates this link by parsing the following import statement in `TestClassTwo`:

{% highlight java %}
import com.github.stkent.javadoctests.package1.TestClassOne;
{% endhighlight %}

On the other hand, because the Java `String` type is not part of the source we provided to the `docTask` task, the `javadoc` tool has no way of determining an equivalent hyperlink target to use for the `TestClassOne` constructor parameter type.

## Under The Hood

Gradle's Javadoc task type acts as a wrapper around the command-line `javadoc` tool included with every JDK. To locate yours, run `which javadoc` from the command line (the path to this file should match your `JAVA_HOME` environment variable, if set):

{% highlight bash %}
$ which javadoc
/Library/Java/JavaVirtualMachines/jdk1.8.0_65.jdk/Contents/Home/bin/javadoc
{% endhighlight %}

The temporary javadoc.options file generated by our Gradle task contains options and arguments that are forwarded to this command-line tool. For our example, the javadocs.options file contains these lines:

{% highlight text %}
-d '/Users/stuart/dev/personal/libraries/JavadocTests/library/build/docs/javadoc'
-doctitle 'library API'
-quiet 
-windowtitle 'library API'
'/Users/stuart/dev/personal/libraries/JavadocTests/library/src/main/java/com/github/stkent/javadoctests/package1/TestClassOne.java'
'/Users/stuart/dev/personal/libraries/JavadocTests/library/src/main/java/com/github/stkent/javadoctests/package2/TestClassTwo.java'
{% endhighlight %}

This is pretty minimal configuration - we are setting the output directory, titles to use for the generated website, and providing the collection of source files for which we would like documentation to be generated. To confirm the claim that Gradle's Javadoc task calls `javadoc` with these options and arguments, we can run:

{% highlight bash %}
./gradlew library:clean
{% endhighlight %}

followed by:

{% highlight bash %}
javadoc -d '/Users/stuart/dev/personal/libraries/JavadocTests/library/build/docs/javadoc' -doctitle 'library API' -quiet -windowtitle 'library API' '/Users/stuart/dev/personal/libraries/JavadocTests/library/src/main/java/com/github/stkent/javadoctests/package1/TestClassOne.java' '/Users/stuart/dev/personal/libraries/JavadocTests/library/src/main/java/com/github/stkent/javadoctests/package2/TestClassTwo.java'
{% endhighlight %}

which produces the output files shown below (viewed within Android Studio):

<div class="image-container">
	<img src="/assets/images/javadoc-tool-cli-output.png" />
</div>

This is identical to the output of the Gradle task itself, minus the temporary javadoc.options file! Which is neat, because it means we can leverage all the existing `javadoc` tool documentation to help us overcome some of the challenges we'll be facing in next post, when we integrate Android SDK components and third-party dependencies with our sample project.

The code for this post is available [here](https://github.com/stkent/javadoc-tests/tree/460d0cf6b4f8c4caa481b162261ca413b098b6db).
