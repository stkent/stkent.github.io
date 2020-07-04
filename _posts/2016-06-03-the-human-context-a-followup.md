---
title: More On The Nearby APIs
author: Stuart Kent
tags: android, conferences, ios

---

{% include kramdown_definitions.md %}

One of the risks of presenting on nascent APIs is that significant changes are fair game. Luckily, the timing of my Self Conference talk [The Human Context: Exploring Google’s Nearby APIs]({% post_url 2016-05-21-self-conference-2016 %}) a couple weeks back allowed me to quickly update my slides to incorporate _most_ of the changes announced at Google I/O two days earlier. Google Nearby guy [Andrew Bunner](https://twitter.com/andrewbunner){:new_tab} was kind enough to review the recording of my talk and pass along some pointers!

<!--more-->

# Feedback

## Authorization

Google Play Services v9.0.0 introduced an authorization-request flow that is simpler than the “`startResolutionForResult` dance” I used in Calling Card. This simpler flow can be accessed by enabling auto-management of the `GoogleApiClient` used to connect to the Nearby APIs:

{% highlight java %}
googleApiClient = new GoogleApiClient.Builder(this)
    .addApi(Nearby.MESSAGES_API)
    .addConnectionCallbacks(this)
    .enableAutoManage(this, this)
    .build();
{% endhighlight %}

Now there's no need to handle connection errors corresponding to permissions issues manually in `onConnectionFailed`!

However, note this important caveat from the `enableAutoManage` documentation (my emphasis):
    
> This method can only be used if this GoogleApiClient will be the **only** auto-managed client in the containing activity.
    
Since Calling Card also uses an auto-managed `GoogleApiClient` instance to handle authentication (via [Google Sign-In](https://developers.google.com/identity/sign-in/android/){:new_tab}), this simpler Nearby API connection flow was unavailable to me. No big deal - the manual connection error handling is laid out nicely in the [updated documentation](https://developers.google.com/nearby/messages/android/user-consent#manually_connect_to_googleapiclient){:new_tab} and is a lot nicer than the previous authorization flow from Google Play Services v8.4.0 (in which permission was requested later in the process, when a publish or subscribe action was initiated). My implementation matches this more verbose flow pretty closely.

## Publication

Contrary to my original claim, it is possible to publish more than 1 message simultaneously. Doing so is not recommended; instead, use a single consolidated message structure that is capable of carrying all the data you wish to communicate. This keeps subscriber code much cleaner and avoids issues with non-simultaneous receipt of messages from a single publisher.

It might be possible to use multiple published messages to attempt to circumvent the 100KB cap on message size, but I wouldn’t recommend it! Don’t fight the framework.

## Transmission

One of the questions asked at the end of my talk focused on the robustness of token transmission via audio. In particular, I knew that this topic had been addressed during [Andrew’s Google I/O presentation](https://www.youtube.com/watch?v=Acdu2ZdBaZE&t=7m05s){:new_tab} but could not recall the details of that implementation. Andrew linked to a Wikipedia page [describing the technique used](https://en.wikipedia.org/wiki/Direct-sequence_spread_spectrum){:new_tab}. My math background does not include a lot of signal processing, so I’d need to dig in deeper to really understand what’s going on here. Still, it’s nice to have the information for future reference!

# Conclusions

I really appreciate Andrew taking the time to watch my presentation pretty closely. Being able to engage 1-on-1 with developers working on Google APIs is pretty awesome, and often a lot more fruitful than trying to reverse-engineer architectural decisions and intentions in isolation. Plus, the benefits go both ways - in updating my presentation to use the latest and greatest version of the Nearby APIs, I [uncovered and reported a bug](https://twitter.com/andrewbunner/status/734825573105565696){:new_tab} that is now fixed and will be shipped soon.
