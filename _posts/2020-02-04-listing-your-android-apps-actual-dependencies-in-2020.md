---
title: Listing Your Android App’s (Actual) Dependencies In 2020
author: Stuart Kent
tags: android

---

{% include kramdown_definitions.md %}

Gradle lets us list all the dependencies of our projects using the [`dependencies` task](https://docs.gradle.org/6.1.1/userguide/viewing_debugging_dependencies.html#sec:listing_dependencies){:new_tab}. Here’s a small snippet of sample output:

{% highlight text %}
+--- org.jetbrains.kotlin:kotlin-android-extensions-runtime:1.3.61
|    \--- org.jetbrains.kotlin:kotlin-stdlib:1.3.61
|         +--- org.jetbrains.kotlin:kotlin-stdlib-common:1.3.61
|         \--- org.jetbrains:annotations:13.0
+--- org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.61
|    \--- org.jetbrains.kotlin:kotlin-stdlib:1.3.61 (*)
+--- androidx.appcompat:appcompat:1.1.0
|    +--- androidx.annotation:annotation:1.1.0
|    +--- androidx.core:core:1.1.0
|    |    +--- androidx.annotation:annotation:1.1.0
|    |    +--- androidx.lifecycle:lifecycle-runtime:2.0.0 -> 2.1.0
.    .
.    .
.    .
{% endhighlight %}

This output shows both the direct and [indirect (aka transitive)](https://en.wikipedia.org/wiki/Transitive_dependency){:new_tab} dependencies of our project. Indirect dependencies are indicated by indentation, so in the example above:

- `androidx.appcompat:appcompat:1.1.0` is a **direct** dependency of our project;
- `androidx.core:core:1.1.0` is a direct dependency of `androidx.appcompat:appcompat:1.1.0`, and is therefore an **indirect** dependency of our project;
- `androidx.lifecycle:lifecycle-runtime` is a direct dependency of `androidx.core:core:1.1.0` which is a direct dependency of `androidx.appcompat:appcompat:1.1.0`, and is therefore a (doubly) **indirect** dependency of our project; etc.

Running `./gradlew :app:dependencies` on an Android project outputs [**many**](https://github.com/gradle/gradle/issues/11648){:new_tab} separate lists of dependencies; one for each Gradle [configuration](https://docs.gradle.org/6.1.1/userguide/dependency_management_for_java_projects.html#sec:configurations_java_tutorial){:new_tab}. While the term may sound unfamiliar, you’ve definitely interacted with configurations before; the `implementation` keyword you use when declaring dependencies of your app refers to a configuration with the same name.

We can reduce the verbosity of the `dependencies` task by passing the name of a single configuration using the `--configuration` parameter:

{% highlight bash %}
./gradlew :app:dependencies --configuration someConfiguration
{% endhighlight %}

The rest of this post explains which Gradle configurations we should use with the `dependencies` task as Android developers in 2020. Hint: not `implementation` :)

<!--more-->

# Historical Context

<a id="configuration_purpose_list"></a> Prior to Gradle 3.4, configurations could be used for one or more of [several distinct purposes](https://docs.gradle.org/6.1.1/userguide/dependency_management_for_java_projects.html#sec:configurations_java_tutorial){:new_tab}:

1. holding a list of direct dependencies declared via the `dependencies` block of our `build.gradle` files;
2. resolving and operating on a complete dependency tree (effectively a [classpath](https://en.wikipedia.org/wiki/Classpath_(Java)){:new_tab}) based on a list of direct dependencies **and** a target usage (compilation, runtime, etc);
3. declaring project artifacts for other projects to consume.

In these versions of Gradle, the [typical advice for Android developers](https://stackoverflow.com/a/44496539/2911458){:new_tab} was to run the `dependencies` task using the `compile` configuration:

{% highlight bash %}
./gradlew :app:dependencies --configuration compile
{% endhighlight %}

This did exactly what you probably guessed; it output the full dependency tree based on the dependencies you’d declared to be part of the `compile` configuration in your `build.gradle` file’s `dependencies` block. However, the target usage being considered was not immediately apparent.

# Modern Gradle

In Gradle 3.4, new configurations were restricted to each being used for [exactly one](https://docs.gradle.org/6.1.1/userguide/dependency_management.html#sec:resolvable-consumable-configs){:new_tab} of the three purposes described [above](#configuration_purpose_list). This improved separation of concerns allowed some previously-impossible performance optimizations to be implemented.

As part of this change, the `compile` configuration was deprecated and replaced by the `api` and `implementation` configurations. Running `./gradlew :app:dependencies --configuration compile` now results in the output below:

{% highlight text %}
compile - Compile dependencies for 'main' sources (deprecated: use 'implementation' instead).
No dependencies
{% endhighlight %}

You might guess that the proper way to inspect Android app dependencies using modern Gradle versions is to run

{% highlight bash %}
./gradlew :app:dependencies --configuration implementation
{% endhighlight %}

If you try this, you’ll see output similar to:

{% highlight text %}
implementation - Implementation only dependencies for 'main' sources. (n)
+--- org.jetbrains.kotlin:kotlin-android-extensions-runtime:1.3.61 (n)
+--- org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.61 (n)
+--- androidx.appcompat:appcompat:1.1.0 (n)
+--- androidx.core:core-ktx:1.1.0 (n)
+--- com.google.android.material:material:1.0.0 (n)
+--- maps-sdk-3.0.0-beta (n)
+--- com.google.android.gms:play-services-location:17.0.0 (n)
\--- com.google.android.gms:play-services-gcm:17.0.0 (n)

(n) - Not resolved (configuration is not meant to be resolved)
{% endhighlight %}

This output only includes our direct dependencies and does not include any indirect dependencies, reducing its usefulness. The clue as to why is in the note at the bottom:

{% highlight text %}
(configuration is not meant to be resolved)
{% endhighlight %}

This is signalling to us that the `implementation` configuration is **only** for holding a list of direct dependencies and (unlike the deprecated `compile` configuration) cannot also be used for resolving a complete dependency tree.

The [upgrade notes for Gradle 6](https://docs.gradle.org/6.1.1/userguide/upgrading_version_5.html#dependencies_should_no_longer_be_declared_using_the_compile_and_runtime_configurations){:new_tab} indicate that `compileClasspath` and `runtimeClasspath` are the configurations we should use for dependency resolution in modern Gradle projects:

> The `implementation`, `api`, `compileOnly` and `runtimeOnly` configurations should be used to declare dependencies and the `compileClasspath` and `runtimeClasspath` configurations to resolve dependencies.

The relationships between these configurations are shown in the diagram below[^1]. Configurations that can be used to resolve a complete dependency tree are marked (R) and colored blue.

<div class="image-container">
  <img src="/assets/images/listing-your-android-apps-actual-dependencies-in-2020-java-library-plugin-configurations.png" />
</div>

In Android projects, there is one `compile` and one `runtime` classpath configuration for each build variant. These configurations are named `{variant}CompileClasspath` and `{variant}RuntimeClasspath`. In a project with default build variants, we can therefore inspect our resolved dependency tree using one of the following commands:

{% highlight bash %}
./gradlew :app:dependencies --configuration debugCompileClasspath
./gradlew :app:dependencies --configuration releaseCompileClasspath
./gradlew :app:dependencies --configuration debugRuntimeClasspath
./gradlew :app:dependencies --configuration releaseRuntimeClasspath
{% endhighlight %}

If you use custom build variants, replace `debug` or `release` with the name of the variant you are interested in inspecting. If you declare `compileOnly` or `runtimeOnly` dependencies, pay more attention to whether you wish to inspect resolved compilation (`{variant}CompileClasspath`) or runtime (`{variant}RuntimeClasspath`) dependencies and select your configuration accordingly.

# Dependency Insights

`{variant}CompileClasspath` and `{variant}RuntimeClasspath` are also appropriate configurations to pass as parameters to the `dependencyInsight` Gradle task. This task provides more details on a specific dependency and how it was resolved, and requires us to pass a configuration. An example invocation and output snippet is shown below:

{% highlight bash %}
./gradlew app:dependencyInsight \
        --dependency androidx.appcompat:appcompat: \
        --configuration releaseCompileClasspath
{% endhighlight %}

{% highlight text %}
androidx.appcompat:appcompat:1.1.0
   ...
   Selection reasons:
      - By constraint : releaseRuntimeClasspath uses version 1.1.0
{% endhighlight %}

[^1]: [Gradle documentation description of Java library plugin configurations](https://docs.gradle.org/6.1.1/userguide/java_library_plugin.html#sec:java_library_configurations_graph){:new_tab}.
