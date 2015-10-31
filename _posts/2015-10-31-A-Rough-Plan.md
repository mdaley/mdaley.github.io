---
layout: post
title: A rough plan
date: 2015-10-31 16:00:10 +00:00
---

Here's a rough plan for the creation of my continuous deployment and development process:

 - create a helloworld service using the mr-clojure template (mr-clojure is a lein template for creating a ready to deploy and run clojure web-service).
 - create a new AWS account into which I'll setup several useful tools and services and then deploy and use my service. I'll have to be careful to stick within the rules of the _free tier_.
 - Create a mechanism for storing versioned releases of the service (a yum repo to store the service's RPMs).
 - Create a mechanism for generating AMIs from releases of the service (this uses the mr-baker mechanism).
 - But first I need to create an instance of mr-baker in the AWS account.
 - I need a way to understand what services I have in my AWS environment (mr-lister does this).
 - I need a way to configure the AWS environment (mr-pedantic controls this).
 - I need a way to ....well there are many more things.... and I'll update this list as I go, so it keeps a record of the overall approach.

So, for the first things: I want to get mr-baker working in my environment. I'll describe this in the next blog entry.
