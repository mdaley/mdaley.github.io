---
layout: post
title: Jenkins Continuous Build and Testing
date: 2015-11-04 16:00:00 +00:00
---

Next I need a way of building and testing the service that I've created on demand, every time code is committed to the github
repository. After that, I'll want to be able to create a RPM that I can deploy. This will use jenkins to build the RPM, upload it to a yum repository and then update the version of the code stored in git.

I'll do this on a spare local workstation. OK, as I haven't got one of those, I'll use a ubuntu VM, but the same approach applies
in both cases.

### Installing and setting up jenkins

Create a local VM of Ubuntu 14.04. There are instructions all over the internet for this. Being on a Mac, I used 'parallels'.
The only thing I changed was to set up the network to use Bridged Mode on the Default Adapter, so that the VM is accessible from
the mac host on an IP address.

Next install a Java JDK. I chose Java 8 and obtained it from here:

[http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

Here are some suitable instructions:

[https://www.digitalocean.com/community/tutorials/how-to-manually-install-oracle-java-on-a-debian-or-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-manually-install-oracle-java-on-a-debian-or-ubuntu-vps)

You end up with java installed in something like `/opt/jdk/jdk1.8.0_65`.

I won't go into the details of install jenkins. Just go to the following link and follow the instructions:

[https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu)

After completing installation, you'll be able to open jenkins' UI on `http://{vm-ip-address}:8080`.

(On the VM, in a command window, type `ifconfig` to find out the ip address of the VM)

![empty jenkins]({{ site.url }}/images/empty-jenkins.png)

Now, I want create a continuous integration job for my project that runs whenever code is checking in.

First I need to make sure that jenkins is set up to be able to access the git repository. It's worth pointing out at this point
that you would probably want a private repository rather than a free public one; but the principles are the same.

On the ubuntu server, log on as an admin user and then:

```
$ sudo su jenkins
```

to become the jenkins user.

Try to clone the helloworld repository using the SSH URL:

```
$ git clone git@github.com:mdaley/helloworld.git
```

This won't work as the jenkins user needs to have access SSH keys registered in github. Follow the instructions here to install
SSH keys for the jenkins user:

[https://help.github.com/articles/generating-ssh-keys/](https://help.github.com/articles/generating-ssh-keys/)

Now `git clone` will work.

The jenkins user will also have to have the clojure _leiningen_ build system installed so that clojure/lein based projects can be built. Whilst still logged in as the jenkins user install lein - see instructions at [http://leiningen.org/#install](http://leiningen.org/#install). Make sure that `lein` is on the path of the jenkins user. Also, take note of the path of lein jar installed in `/var/lib/jenkins/.lein/self-installs` as you'll need this later. 

Several plugins need to be installed in jenkins. In this case, through _Manage Jenkins->Manage Plugins_ install:

 - GIT plugin - allows integration with git
 - leiningen plugin - this allows the clojure _leiningen_ build environment to be used.
 - Port allocator plugin - allows random ports to be used to avoid several jobs running at once having collisions because several services are attempting to be run up on the same port.
 - AnsiColor plugin - allows jenkins logs to be coloured for easier readability.

and you can turn off things you don't care about:

 - CVS plugin
 - Subversion plugin

Next, add the JDK installed earlier at _Manage Jenkins->Configure System->JDKs_. Just add a name for the JDK and its directory location.

Finally, add the full path of the lein jar at _Manage Jenkins->Leiningen Builder_.
 
Save the jenkins settings and restart the server (or just jenkins itself).

### Creating the continuous integration job on jenkins

From _New Item_ in jenkins create a new _Freestyle Project_ called `helloworld-ci`.

Set the following:

 - Tick _Discard old builds_, setting the max number of builds to keep to a suitable number; I chose 5 but it's up to you (don't want to run out of space though).
 - Set Source code management to _Git_.
 - Enter the repository URL, in my case it is `git@github.com:mdaley/helloworld.git`.
 - Tick _Poll SCM_ and enter a value of `H/2 * * * *` which means poll every two minutes.
 - Tick _Assign unique TCP ports to avoid collisions_ and add two _Plain TCP Port_ values named `SERVICE_PORT` and `RESTDRIVER_PORT`. These are, respectively, the port on which the service will run and the port on which service mocks run during acceptance testing (the tests don't make use of this yet in this noddy service, but in a real service it is exceptionally useful. See [https://github.com/whostolebenfrog/rest-cljer](https://github.com/whostolebenfrog/rest-cljer) for details).
 - Tick _Color ANSI Console Output_ - this just makes build logs look nicer.
 
 Then finally, to make testing actually occur, add a _Build Step_ of type _execute shell_ and enter `./all`. This will run the `all` script which runs unit tests, the integration tests and the acceptance tests.
 
Save the job and then run it. Click on the running job (the red line) and you'll see the build in progress: the code will be downloaded from the git repository, the dependencies will be downloaded, the service will be run up and tests will run. Once all sets
of tests have completed successfully you'll see a SUCCESS message. The build will be marked as succeeded (with a blue circle).

To demonstrate continuous build and testing in action, make a small change to the code (perhaps change the response from the _/hello_ resource) so that the acceptance tests will fail and then commit it to git. After a few minutes You'll see the helloworld-ci job run and fail. Now, fix the broken service so that acceptance tests pass again and commit the change to git. After a few minutes the continuous integration job will run and pass.
 
OK, that's enough for now; I'll do the building of an RPM in the next post.

![empty jenkins]({{ site.url }}/images/jenkins-ci-success.png)
