---
layout: post
title: A rough plan
date: 2015-10-31 16:00:10 +00:00
---

Here's a plan for the proving of the continuous development and deployment process using the MixRadio open-source tools. Note that `mr-this` and `mr-that` refer to services, built on clojure and lein, that can be deployed to support various parts of the process. See the [MixRadio github](https://github.com/mixradio) for more details. I'm sure this plan will evolve substantially as I go along but, at least, here is my _back of a fag packet_ starting point:

 - create a (rather noddy) helloworld service using the `mr-clojure` template (a lein template for creating a ready to deploy and run clojure web-service).
 - create a new AWS account into which I'll setup several useful tools and services and then deploy and use my service. I'll have to be careful to stick within the rules of the _free tier_!
 - Create a mechanism for storing versioned releases of the service (a yum repo to store the service's RPMs).
 - Create a mechanism for generating AMIs from releases of the service (this uses the `mr-baker` mechanism).
 - But first I need to create an instance of `mr-baker` in the AWS account.
 - I need a way to understand what services I have in my AWS environment (`mr-lister` does this).
 - I need a way to configure the AWS environment (`mr-pedantic` controls this).
 - I need a way to ....well there are many more things.... perhaps I'll just write a summary blog entry at the end.

So, for the first things: I need to create my noddy test web application. I'll describe this in the next blog entry.
