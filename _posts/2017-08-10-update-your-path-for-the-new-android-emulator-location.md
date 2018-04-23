---
title: Update Your Path For The New Android Emulator Location
author: Stuart Kent
tags: android

---

Since March 2017 (v25.3.0), the Android Emulator has been [released separately from the rest of the Android SDK tools](https://developer.android.com/studio/releases/emulator.html#25-3). As part of implementing this change, the `emulator` binary was ‘promoted’ from`${ANDROID_SDK_ROOT}/tools/`[^1]  to its own top-level directory, `${ANDROID_SDK_ROOT}/emulator/`. This relocation can cause some issues. I'll show you how to avoid them 🙂.

<!--more-->

At the time of the change, it was indicated that [old versions of Android Studio should be unaffected](https://developer.android.com/studio/releases/sdk-tools.html). However, I recently needed to launch the emulator from the command line to test customized Android system images, and in doing so discovered some rough edges:

{% highlight none %}
$ ${ANDROID_SDK_ROOT}/tools/emulator -avd my-custom-avd
PANIC: Missing emulator engine program for 'x86' CPU.
{% endhighlight %}

If I run this same command using the relocated `emulator`, the AVD launches successfully:

{% highlight none %}
$ ${ANDROID_SDK_ROOT}/emulator/emulator -avd my-custom-avd
HAXM is working and emulator runs in fast virt mode
{% endhighlight %}

This is despite the fact that

{% highlight none %}
$ ${ANDROID_SDK_ROOT}/tools/emulator -version
{% endhighlight %}

and

{% highlight none %}
$ ${ANDROID_SDK_ROOT}/emulator/emulator -version
{% endhighlight %}

both report the same version information:

{% highlight none %}
Android emulator version 26.1.2.0 (build_id 4077558) (CL:500db745bd44dbc6000413b5e8969d83216ff7cd)
{% endhighlight %}

I'm guessing the error is due to a discrepency in the emulator-related files found in each location:

{% highlight none %}
$ ls ${ANDROID_SDK_ROOT}/tools/ | grep "emulator"
emulator
emulator-check
{% endhighlight %}

{% highlight none %}
$ ls ${ANDROID_SDK_ROOT}/emulator/ | grep "emulator"
emulator
emulator-check
emulator64-arm
emulator64-crash-service
emulator64-mips
emulator64-x86
{% endhighlight %}

I'm not sure _why_ this discrepancy exists 🤷‍♂️.

If, like me, all you care about is having easy command-line access to the newest `emulator` binary, I recommend updating your `$PATH` to include

{% highlight none %}
${ANDROID_SDK_ROOT}/emulator
{% endhighlight %}

Make sure this appears before[^2] any existing reference to

{% highlight none %}
${ANDROID_SDK_ROOT}/tools
{% endhighlight %}

in your `$PATH` so that the correct `emulator` binary is prioritized:

{% highlight none %}
$ which emulator
/Users/stkent/Library/Android/sdk/emulator/emulator
{% endhighlight %}

Happy emulating!

[^1]: In writing this post I also discovered for the first time that `${ANDROID_HOME}` has been deprecated in favor of `${ANDROID_SDK_ROOT}`! Read more in the [Android Studio User Guide section on Environment Variables](https://developer.android.com/studio/command-line/variables.html#envar).

[^2]: Run `echo $PATH` to check the full contents of your `$PATH` variable. Searched paths are separated by a colon and searched first-to-last.
