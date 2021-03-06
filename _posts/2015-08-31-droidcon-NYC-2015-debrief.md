---
title: Droidcon NYC 2015 Debrief
author: Stuart Kent
tags: android, conference, slides, videos

---

{% include kramdown_definitions.md %}

It was great to see so many prominent members of the Android community attending and presenting at Droidcon NYC 2015. Let’s review some of my favorite talks! If you only have time to watch 6 of the ~65 talks recorded, I’d suggest starting here. (Note the diversity of the topics represented in this list - a tribute to the all-encompassing scope of Droidcon NYC.)

<!--more-->

## Simple HTTP with Retrofit 2 - [Jake Wharton](https://twitter.com/JakeWharton){:new_tab} ([Slides](https://speakerdeck.com/jakewharton/simple-http-with-retrofit-2-droidcon-nyc-2015){:new_tab} / [Video](https://www.youtube.com/watch?v=KIAoQbAu3eA){:new_tab})

Though this talk didn’t contain many surprises if you’ve been following the [proposed Retrofit 2 spec](https://github.com/square/retrofit/issues/297){:new_tab} and [recent progress](https://github.com/square/retrofit/pull/845){:new_tab}, Jake presented an exceptionally clear discussion of both the limitations of Retrofit 1, and their solutions as implemented in Retrofit 2. At a high level, the most significant changes may be summarized as follows:

- a new `Call` type is introduced to represent a _single_ network request and response. Each instance encapsulates both raw and processed (e.g. deserialized) request and response information. A `Call` object may be used to execute a network request either synchronously _or_ asynchronously - no more need to include duplicate entries in our annotated services to achieve this.

- OkHttp is now the only supported HTTP client. This permits the removal of several public classes that existed solely to support alternative HTTP client implementations, as well as adding the ability to safely expose OkHttp types in the public API of Retrofit (e.g. to allow direct inspection or customization of the request/response);

- deserialization can now be attempted by multiple prioritized type converters. Semantically-related network requests that happen to return data in different formats (e.g. JSON vs XML) can now be grouped in a single annotated service;

- request cancelation is now supported.

As icing on the cake, Jake [released Retrofit 2.0.0-beta1](https://github.com/square/retrofit/blob/master/CHANGELOG.md){:new_tab} right after his talk. The API is supposed to be close to final, and the library fairly stable. Have at it!

<div class="image-container">
	<img src="/assets/images/droidcon-nyc-2015-debrief-jake-wharton.jpg" width="50%" />
</div>

## Gradle: From User to Addict - [Jake Ouellette](https://twitter.com/jakeout){:new_tab} ([Slides]() / [Video](https://www.youtube.com/watch?v=-C7TtnPJ7ms){:new_tab})

This talk was thrilling - fast-paced and packed with great information. If you are looking to become more experienced with Gradle, I’d recommend this talk as a follow-up to Dan Lew’s [excellent introductory video](https://www.youtube.com/watch?v=fHhf1xG0pIA){:new_tab}. Dan focused more on basic Groovy syntax and Gradle task sequencing, and Jake extends these ideas with detailed discussions of Groovy’s unusual scope resolution and Gradle task sequencing nuances. Fair warning: you’ll definitely need to pause and rewind in a couple spots to give yourself time to properly parse what Jake says.

## Data Binding Techniques - [Jacob Tabak](https://twitter.com/JacobTabak){:new_tab} ([Slides](https://speakerdeck.com/jacobtabak/data-binding-techniques-at-droidcon-nyc-2015){:new_tab} / [Video](https://www.youtube.com/watch?v=WdUbXWztKNY){:new_tab})

Most of the usage information presented here was familiar from Google I/O videos. However, Jacob provided some interesting context around data binding in general, and Google’s implementation of data binding in particular:

- originally slated for release as part of Android Lollipop, the ballooning scope of that OS release meant that data binding support was temporarily shelved last year;

- data binding implementations have historically suffered from performance issues. However, in some cases, the Android implementation provided is more performant than equivalent hand-written code. For example, manually calling `findViewById` 5 times in your `Activity`’s `onCreate` method triggers five separate traversals of the view hierarchy; with data binding, all 5 view references can be initialized in a single pass.

Despite vaguely knowing about almost all of the presented material already, I left this talk feeling super-motivated to begin exploring data binding immediately. The current API is pretty much finalized, so it’s safe to begin using; however, note that IDE support remains very limited. If you’re still skeptical, consider the awesome implications of this slide alone:

<div class="image-container">
	<img src="/assets/images/droidcon-nyc-2015-debrief-binding-adapter.png" width="75%" />
</div>

## Using Styles and Themes Without Going Crazy - [Dan Lew](https://twitter.com/danlew42){:new_tab} ([Slides](https://speakerdeck.com/dlew/using-styles-and-themes-without-going-crazy-1){:new_tab} / [Video](https://www.youtube.com/watch?v=Jr8hJdVGHAk){:new_tab})

The beginning of this talk seemed somewhat familiar, and some rapid Googling turned up Dan’s [late-2014 article](http://blog.danlew.net/2014/11/19/styles-on-android/){:new_tab} on the same topic. I read that article when it was published, and remember being excited that someone had finally formalized my half-formed thoughts regarding semantically-identical vs visually-identical view styling.

This talk expands upon that original post, and includes

- more details on how to discover available attributes and create custom attributes;

- tips for determining which attributes are influencing which on-screen elements (using a debug theme, or hierarchy viewer’s "dump theme" button).

<div class="image-container">
	<img src="/assets/images/droidcon-nyc-2015-debrief-just-deduce-it.jpg" width="50%" />
</div>

## Why, Hello There, Camera 2 API - [Huyen Dao](https://twitter.com/queencodemonkey){:new_tab} ([Slides](https://speakerdeck.com/randomlytyping/android-camera-2-api){:new_tab} / [Video](https://www.youtube.com/watch?v=BLScKJkOpy8){:new_tab})

The new Camera 2 API is a radical departure from the original Camera API. Initially, the magnitude of this change can seem surprisingly. The new API is a little more powerful (still shot burst mode is easier to implement, for example), but it’s far from obvious that this increased flexibility necessitated a total API overhaul. Huyen’s talk focused on the conceptual shift in low-level camera architecture introduced in version 3 of Android’s camera Hardware Abstraction Layer - in particular, the change in how requests for camera data are represented - and explained how we can interpret the changes in the high-level Camera 2 API in terms of these low-level details.

A great example of a talk that promotes understanding, rather than mere knowledge, of an API; the why, not just the what.

## Android is the World Phone - [Corey Leigh Latislaw](https://twitter.com/corey_latislaw){:new_tab} ([Slides](https://speakerdeck.com/colabug/android-is-the-world-phone){:new_tab} / [Video](https://www.youtube.com/watch?v=sYUHdPXR8MM){:new_tab})

Corey described the challenges developers face when building apps for customers in emerging and developing markets. Unlike most of the talks I’ve seen on this topic, Corey’s managed to address both philosophical and practical issues. Especially thought-provoking were the literacy statistics for these populations - could you build an app that is intuitive enough to be used by someone who cannot read?
