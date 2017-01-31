![Build Status](https://travis-ci.org/RationaleEmotions/just-ask.svg?branch=master)

# just-ask

As the name suggests, this library is eventually going to end up becoming a simple prototype that can be enhanced to 
represent an "On-demand Grid" (which explains the reason behind the name **just-ask** ).
 
## Why an On-Demand Grid ?

A static Grid (i.e., a hub and a fixed number of nodes) is a good start for setting up a remote execution infrastructure. 
But once the usage of the hub starts going up, problems start creeping up. 
Nodes go stale and start causing false failures for the tests and require constant maintenance (restart).  

Some of it can be solved by embedding a ["self healing"](https://rationaleemotions.wordpress.com/2013/01/28/building-a-self-maintaining-grid-environment/) mechanism into the hub, 
but when it comes to scaling the Grid infrastructure this also does not help a lot.

**just-ask** is an on-demand grid,  wherein there are no fixed nodes attached to the grid. 
As and when tests make hit the hub, a node is spun off, the test is routed to the node and after usage the node is 
cleaned up. The on-demand node can be a docker container that hosts a selenium node.

 ## Pre-requisites
 
 **just-ask** requires : 
 * **JDK 8**.
 * A Selenium Grid of version **3.0.1** or higher.


## How to use

In order to consume **just-ask** for your straight forward on-demand grid needs, following instructions need to be 
followed:
* First download the uber jar from [here](http://repo1.maven.org/maven2/com/rationaleemotions/just-ask/). Make sure 
you download the uber jar i.e., the jar name that ends with `jar-with-dependencies`.
* Download the latest Selenium standalone jar from [here](http://www.seleniumhq.org/download/).
* Now create a configuration JSON file as shown below :

```json
{
  "dockerHost": "192.168.43.130",
  "dockerPort": "2375",
  "localhost" : "0.0.0.0",
  "dockerImagePort" : "4444",
  "maxSession" : 5,
  "mapping" : [
    {
      "browser" : "chrome",
      "target" : "selenium/standalone-chrome:3.0.1"
    }, {
      "browser" : "firefox",
      "target" : "selenium/standalone-firefox:3.0.1"
    }
  ]
}

```
* Start the On-demand Grid using the below command (Here the JVM argument `-Dconfig.file` is used to specify the 
location of the JSON configuration file that we created above.)

```
java -Dconfig.file=config.json -cp selenium-server-standalone-3.0.1.jar:just-ask-<VERSION>-jar-with-dependencies.jar org.openqa.grid.selenium.GridLauncherV3 
-role hub -servlets com.rationaleemotions.servlets.EnrollServlet
```

* Now wire in the Ghost Proxy into this on-demand hub by loading the URL : 
**http://localhost:4444/grid/admin/EnrollServlet**

* That's about it. The On-demand Grid is now ready for use.

## Understanding the JSON configuration file.
The meaning of each of the attributes of the JSON file is as below :

* `dockerHost` - Represents the Host name of the Docker Daemon.
* `dockerPort` - Represents the port on which the Docker Daemon is listening to.
* `localhost` - Represents the hostname of the machine on which the Docker Daemon is running on (Its safe to leave 
its value as `0.0.0.0` )
* `dockerImagePort` - Represents the port number that is exposed inside the docker container.
* `maxSession` - Represents the maximum number of concurrent sessions that can be supported by the On-demand Hub 
after which new test session requests will be queued.
* `mapping` - Represents a set of key-value pairs wherein `browser` represents the `browser flavor` and `target` 
represents the name of the docker image that is capable of supporting the respective `browser`


## How to customize and use.

In-case you would like to wire in your own Custom Server implementation, following is how it can be done :

**just-ask** is a [Maven](https://maven.apache.org/guides/getting-started/) artifact. In order to 
consume it, you merely need to add the following as a dependency in your pom file.

```xml
<dependency>
    <groupId>com.rationaleemotions</groupId>
    <artifactId>just-ask</artifactId>
    <version>1.0.0</version>
</dependency>
```

Now that you have added the above as a maven dependency, you build your own implementation of the Server, by 
implementing the interface `com.rationaleemotions.server.ISeleniumServer`.

After that, you can wire in your implementation via the JVM argument : `-Dserver.impl=com.foo.bar.FooBar` (here `com
.foo.bar.FooBar` is an implementation of `com.rationaleemotions.server.ISeleniumServer`) through the start-up command

```
java -Dserver.impl=com.foo.bar.FooBar -cp selenium-server-standalone-3.0.1.jar:just-ask-<VERSION>-jar-with-dependencies.jar org.openqa.grid.selenium
.GridLauncherV3 -role hub -servlets com.rationaleemotions.servlets.EnrollServlet
```

## Building the code on your own

In order to get started with using this library here are the set of instructions that can be followed :
 
 * Build the code using `mvn clean package`
 * Drop the built jar (you will find two jars, so please make sure you pick up the uber jar which would have its name
  around something like this `just-ask-<VERSION>-jar-with-dependencies.jar` ) in the directory that contains the 
  selenium server standalone.
 * Start the selenium hub using the command `java -cp selenium-server-standalone-3.0.1.jar:just-ask-<VERSION>-jar-with-dependencies.jar org.openqa.grid.selenium.GridLauncherV3 -role hub -servlets com.rationaleemotions.servlets.EnrollServlet`
 * Explicitly register the *ghost node* by loading the URL `http://localhost:4444/grid/admin/EnrollServlet` in a browser.
 
 Now the **On-demand Grid** is ready for use.
