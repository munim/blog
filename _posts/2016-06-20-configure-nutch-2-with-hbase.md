---
layout: post
title: "Configure Nutch 2.x with HBase"
excerpt_separator: <!--more-->
description : "Detailed tutorial to setup Nutch 2.x from source and HBase"
author: munim
tags:
    - search-engine
---

Setup Nutch 2.x is quite tricky in terms of Nutch 1.x and the main feature of 2.x is that it uses `gora` backend. One of the implementation of `gora` is HBase and thus I'll use HBase to configure Nutch 2.x.

Every version is Nutch 2.x is tied with a version of HBase, so it's very important to use the mentioned version of HBase.<!--more-->

`gora-hbase` has a `rev=0.6.1` that comes with Nutch 2.3.1 which is tied with HBase 0.98.8 Hadoop2 and you can download from this [link](http://archive.apache.org/dist/hbase/hbase-0.98.8/hbase-0.98.8-hadoop2-bin.tar.gz).

So, let's get down to business.

## Install HBase

HBase is fairly easy to setup and need to follow couple of steps.

Before you proceed, check if you've Java 1.6/1.7 installed and `JAVA_HOME` is setup correctly.

First download [HBase 0.98.8 Hadoop2](http://archive.apache.org/dist/hbase/hbase-0.98.8/hbase-0.98.8-hadoop2-bin.tar.gz)

I'd generally extract to `/opt/` and create a symbolic-link to `/opt/hbase` for my ease of use, but that's your call. For this article, I'll use `/opt/hbase`.

If you're using Ubuntu/Debian, make sure you've added `localhost` to your `/etc/hosts` as below

	127.0.0.1 localhost
	127.0.0.1 ubuntu.ubuntu-domain ubuntu

Now, open `/opt/hbase/conf/hbase-env.sh` and update the `JAVA_HOME`.

	export JAVA_HOME=/usr/java/default

Now, open `opt/hbase/conf/hbase-site.xml` and put the following configuration:

```xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>file:///opt/hbase-db</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/path/to/trynutch/zookeeper</value>
  </property>
</configuration>
```

`hbase.rootdir` should be a non-existent directory in your FS where HBase will create itself otherwise Hbase will try to migrate.

That should be it.

Fire up HBase using the following command:

	# /opt/hbase/bin/hbase shell

## Install Nutch 2.x

[Download Nutch 2.x](http://www.apache.org/dyn/closer.cgi/nutch/) source code, as the time of writing this article Nutch 2.3.1 is the latest and hence I'll use it.

Like HBase, I extracted the archive to `/opt/nutch`.

You'll now need to compile it from source using Ant. Once Ant install, just issue the following command under `/opt/nutch`

	# ant runtime

This is going to take a long time because all the dependency required will be downloaded from Maven repository etc. Just be patient and wait.

Once the source is built, head to `/opt/nutch/runtime/local/conf` and modify `hbase-site.xml` and put the same configuration we've put above in HBase configuration.

Make the following changes in `nutch-default.xml`

```xml
<property>
  <name>file.content.limit</name>
  <value>524288</value>
  <description>The length limit for downloaded content using the file
   protocol, in bytes. If this value is nonnegative (>=0), content longer
   than it will be truncated; otherwise, no truncation at all. Do not
   confuse this setting with the http.content.limit setting.
  </description>
</property>
<property>
  <name>http.content.limit</name>
  <value>524288</value>
  <description>The length limit for downloaded content using the http
  protocol, in bytes. If this value is nonnegative (>=0), content longer
  than it will be truncated; otherwise, no truncation at all. Do not
  confuse this setting with the file.content.limit setting.
  </description>
</property>
<property>
  <name>db.max.outlinks.per.page</name>
  <value>500</value>
  <description>The maximum number of outlinks that we'll process for a page.
  If this value is nonnegative (>=0), at most db.max.outlinks.per.page outlinks
  will be processed for a page; otherwise, all outlinks will be processed.
  </description>
</property>
```

Now open `nutch-site.xml` and put the following in the `configuration` block:

```xml
<property>
	<name>http.agent.name</name>
	<value>crawler</value>
</property>
<property>
	<name>storage.data.store.class</name>
	<value>org.apache.gora.hbase.store.HBaseStore</value>
	<description>Default class for storing data</description>
</property>
```

Now, we need to restrict Nutch not to crawl the entire web and to do that, we'll edit `regex-urlfilter.txt` modify the following at the end

	+. # Remove this line
	+https?://([a-z0-9]*\.)*example.com # Add this line

This makes sure that Nutch only crawls `example.com` and all its subdomains only.

We'll need provide a list of URLs that Nutch will going to be crawling into. Create a directory `urls` under `/opt/nutch/runtime/local` and put your domain names in a file.

	#Filename: /opt/nutch/runtime/local/urls/allowed.txt
	https://example.com
	https://foo.example.com

That's it. Now, we'll use `/opt/nutch/runtime/local/bin/crawl` command to crawl `example.com` using the below parameters:

	# bin/crawl urls first-crawl 3

- `bin/crawl` is the shell script that has sequential execution of nutch life-cycle.
- `urls` is the directory name that contains list of all domains that nutch will crawl.
- `first-crawl` is an unqiue crawl key.
- `3` is the number of rounds crawl is going to perform, in other case the depth of the site you're going to crawl.

That's all about it. Please comment below if you've any questions.