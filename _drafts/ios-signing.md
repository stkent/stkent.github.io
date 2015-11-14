---
layout: post
title: Base Post
author: Stuart Kent
tags: tag1, tag2

---

This is the first part of the post (= the summary). It's formatted from markdown into html.

<!--more-->

Questions

- What are all the components we care about?
- What are the purposes of each component?
- When and where is each component created?
- When and where is each component managed?
- Why are certs needed above and beyond a public/private key pair? (See understanding crypto p345 for an explanation of this).

Apple appears to be the trust anchor here (~ the CA). See https://www.apple.com/certificateauthority/, https://en.wikipedia.org/wiki/Trust_anchor

Also good: https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingCertificates/MaintainingCertificates.html#//apple_ref/doc/uid/TP40012582-CH31-SW41