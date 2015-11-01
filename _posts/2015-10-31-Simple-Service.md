---
layout: post
title: Building a simple service with mr-clojure
date: 2015-10-31 17:00:00 +00:00
---

I'm going to assume that you know about clojure, the programming language, and leiningen, the _de-facto_ build tool. If not
[Clojure for the brave and true](http://www.braveclojure.com) is a good place to start.

### Creating a new service

MixRadio's [mr-clojure](https://github.com/mixradio/mr-clojure) is a lein template that can be used to generate a clojure-based web 
application. It creates an application that can be run locally and that has additional features that allows it to be 
packaged up and deployed into, and run, in a live environment. There are many things that need to go around it to make this work, but running it locally is a good starting point.

So, to start with, assuming you have a working clojure/lein2 environment, run:

```
$ lein new mr-clojure helloworld
```

This creates a `helloworld` directory containing the new application. Note that `lein new` collects the most recent template from where it is stored in [clojars.org](http://clojars.org), the public repository of clojure assets. Change to the new directory and straightaway you can run the service:

```
$ ./acceptance wait
```

You'll see some logging messages as the service starts. Once it has started you can make a request using curl:

```
$ curl http://localhost:8080/healthcheck
```

which gives the json response:

```
{
  "name": "helloworld",
  "version": "1.0.0-SNAPSHOT",
  "success": true,
  "dependencies": []
}
```

Note that the service comes with three testing options:

 1. `lein midje :filter unit` will run unit tests.
 1. `./acceptance` is intended for testing the service in isolation. A jetty server running the service is launched and the service can by called over http. Any http calls that the service makes should be caught by a http-level mocking approach (e.g. [rest-cljer](https://github.com/whostolebenfrog/rest-cljer)).
 1. `./integration` is intended for testing the service with it communicating with services on which it depends.

The service created form the `mr-clojure` template does nothing except return a response to HTTP GETs on the `/healthcheck` and `/ping` resources. Simple tests for these resources are included.

### Committing to git

Having created the new service, now is a good time to commit it to a source code management system. I'll assume you know about `github` and just say use the usual approach to create a new github repository called `helloworld` and commit the new service to it.

### Writing some new code

Just for completeness - after all, this service does nothing - write some code that actually does something useful. I know it's rather noddy but I just created a simple `/hello` resource and added a function to the `src/helloworld/web.clj` file:

```clojure
(defn- hello
  [{:keys [query-params]}]
  (let [name (get query-params "name")]
    {:status 200
     :body (json/generate-string {:message (if (empty? name)
                                             "What is your name?"
                                             (str "Hello " name))})}))
```

and, in the routes section:

```clojure
(defroutes routes
  ...

  (GET "/hello" req
       (hello req))

  ...)
```

To see this working, save the file, run `./acceptance wait` or `./integration wait` and curl the new resource:

```
$ curl http://localhost:8080/hello?name=Bob
```

to get the response:

```
{
  "message" : "Hello Bob"
}
```

Prize for clojure coder of the year please ;-)

To see the full code for the service look go to [github.com/mdaley/helloworld](https://github.com/mdaley/helloworld).

### Continuous integration and release

The next step will be to set up a continuous integration system and to generate releases of the application that can
be deployed onto a server (or, as I'll describe later, be _baked_ into an AMI). I'll describe this in the next blog entry. 




