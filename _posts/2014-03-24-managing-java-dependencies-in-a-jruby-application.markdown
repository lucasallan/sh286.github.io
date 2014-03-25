---
layout: post
title:  "Managing java dependencies in a JRuby application"
date:   2014-03-24 16:49:00
---

I recently started writing an application using JRuby and Rails 4, and I wanted an easy and good way ruby-styled tool to automatically manage the java dependencies.


### LockJar

[LockJar][[lockjar] is a Rubygem that provides Java dependencies management to you JRuby application in the same style that Bundler does with Ruby dependencies.
Another good reason to use LockJar is that it can be integrated with Bundler.

Add it to your Gemfile and then create the file LockJar. This is how my LockJar looks:

```
repository 'http://repo1.maven.org/'
local_repo 'lib/java'

jar 'com.netflix.astyanax:astyanax-core:1.56.48'
jar 'com.netflix.astyanax:astyanax-thrift:1.56.48'
jar 'com.netflix.astyanax:astyanax-cassandra:1.56.48'
```

In the first line I have the maven repository I want to use.
Second line is where the LockJar will save the jar files. In my case, I'm saving inside my rails application directory so Rails can load all these files for me. If nothing is specified, then the LockJar will store the jar files in its default location Maven.

The following lines are the dependencies. They should be listed in the maven style: GroupID/Org : ArtifactID/Name : version

#### Integration with Bundler

You just need to add the follow line to your Gemfile:

```
require 'lock_jar/bundler'
```

And everytime you run bundle update/install it will also check your java dependencies and download them if necessary.


[astyanax]: https://github.com/Netflix/astyanax
[lockjar]: https://github.com/mguymon/lock_jar
