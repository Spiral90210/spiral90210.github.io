---
layout: post
title: Local Caching Maven Proxy
---

So, here's the drill. Your company has mandated a specific maven config to use that routes *everything* (including central) via their (often flaky, and often sonotype nexus - not related) servers. Which makes sense for a build machine for sure, but less so when you're trying to iterate quickly in development. Combine this with mavens tendency to re-check absolutely everything, and throw in working remotely over a VPN that was fitted by a 'lowest estimate' outfit, and building can be a bit of a pain in the arse. So, what do you do? You install a local caching maven proxy!

The theory is simple - when you need a dependency/plugin, you talk to your proxy instead of the actual place, and if it's not there, it will fetch it for you. Subsequent requests are served up locally, so they're super quick. You may wonder why this is necessary, given that maven will cache to ~/.m2/repository/, but for things like snapshow dependencies, and just sometimes for the hell of it, it will just try and call out anyway. A caching proxy can give you more control over this behaviour e.g. cache snapshots for X amount of time.

There are a few different options for this, but I chose [Apache Archiva](https://archiva.apache.org/index.cgi). To get it working properly you need to

* Install it
* Configure it
* Make it start on boot
* Add the repositories to proxy
* Configure ~/.m2/settings.xml to use it

## Installing it
It's as simple as downloading & extracting it. I used version 2.2.3 [Here](http://apache.mirrors.nublue.co.uk/archiva/2.2.3/binaries/apache-archiva-2.2.3-bin.zip), and put it in /opt/apache-archiva-2.2.3/

## Configure it
All I wanted to do here was change the default port from 8080 to somthing less likely to create a conflict. I chose 8071. You edit this in the config file here: ```/opt/apache-archiva-2.2.3/conf/jetty.xml```. Look for a line like this, and change the port to whatever you want: ```<Set name="port"><SystemProperty name="jetty.port" default="8071"/></Set>```

## Make it start on boot
I'm on OSX Sierra, so that's what these instructions are for. 

* Create a file at {% highlight bash %}sudo touch /Library/LaunchDaemons/org.apache.archiva.plist{% endhighlight %}
* Fill it with the xml below - **change the username** ;)
* chown it like {% highlight bash %}sudo chown root:wheel /Library/LaunchDaemons/org.apache.archiva.plist{% endhighlight %}
* Load it {% highlight bash %}sudo launchctl load -w /Library/LaunchDaemons/org.apache.archiva.plist{% endhighlight %}
* Start it {% highlight bash %}sudo launchctl start org.apache.archiva.plist{% endhighlight %}

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
"http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>org.apache.archiva</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/apache-archiva-2.2.3/bin/archiva</string>
        <string>console</string>
    </array>
    <key>Disabled</key>
    <false/>
    <key>RunAtLoad</key>
    <true/>
    <key>UserName</key>
    <string>YOUR_OWN_USERNAME</string>
    <key>StandardOutPath</key>
    <string>/opt/apache-archiva-2.2.3/logs/launchd.log</string>
</dict>
</plist>
{% endhighlight %}

## Add the repositories
First thing to do is login to the interface [http://localhost:8071/](http://localhost:8071/) and set the admin password (you use lastpass, right?).
The default setup has 2 repositories - internal and snapshots. I was quite happy with that. You need to add the repositories configured in your own settings.xml file - Under the administration section on the left nav, select 'Repositories', then select 'Remote Repositories Management'. Fire away and add each of your configured ones in here.

## Configure your ~/.m2/settings.xml to use it
**TAKE A BACKUP BEFORE YOU MODIFY IT!** 

You know, just in case it all goes tits up. You can now modify your settings file to point to your new repo. I actually left all my current settings in a profile that was deactivated, and added the ones for the local proxy under a new one - you can see exerpts below:
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <pluginGroups></pluginGroups>
  <proxies></proxies>
  <servers></servers>
  <mirrors></mirrors>
  <profiles><profile>
      <id>local-proxy</id>
      <repositories>
        <repository>
          <id>local-proxy</id>
          <name>Local Internal repo</name>
          <url>http://localhost:8071/repository/internal/</url>
          <layout>default</layout>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
        </repository>
       
        <repository>
          <id>local-proxy-snapshots</id>
          <name>Local Internal repo snapshots</name>
          <url>http://localhost:8071/repository/snapshots/</url>
          <layout>default</layout>
          <snapshots>
            <enabled>true</enabled>
          </snapshots>
        </repository>
      </repositories>
   
      <pluginRepositories>
        <pluginRepository>
          <id>local-proxy</id>
          <name>Local Internal repo</name>
          <url>http://localhost:8071/repository/internal/</url>
          <layout>default</layout>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
        </pluginRepository>
   
        <pluginRepository>
          <id>local-proxy-snapshots</id>
          <name>Local Internal repo snapshots</name>
          <url>http://localhost:8071/repository/snapshots/</url>
          <layout>default</layout>
          <snapshots>
            <enabled>true</enabled>
          </snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
  <activeProfiles>
    <activeProfile>local-proxy</activeProfile>
  </activeProfiles>
</settings>
{% endhighlight %}

And that's you! You're all set up to use a local maven cache :) If you want to have a look at a repository and see whats in there, you can navigate to [http://localhost:8071/repository/internal](), as well as search through it through the admin ui.