# Getting Started: Data Integration

What you'll build
-----------------

This guide walks you through creating a simple application that retrieves data from Twitter, manipulates the data, then writes it to a file.

What you'll need
----------------

- About 15 minutes
 - A favorite text editor or IDE
 - [JDK 6][jdk] or later
 - [Maven 3.0][mvn] or later

[jdk]: http://www.oracle.com/technetwork/java/javase/downloads/index.html
[mvn]: http://maven.apache.org/download.cgi

How to complete this guide
--------------------------

Like all Spring's [Getting Started guides](/getting-started), you can start from scratch and complete each step, or you can bypass basic setup steps that are already familiar to you. Either way, you end up with working code.

To **start from scratch**, move on to [Set up the project](#scratch).

To **skip the basics**, do the following:

 - [Download][zip] and unzip the source repository for this guide, or clone it using [git](/understanding/git):
`git clone https://github.com/springframework-meta/{@project-name}.git`
 - cd into `{@project-name}/initial`
 - Jump ahead to [Create a resource representation class](#initial).

**When you're finished**, you can check your results against the code in `{@project-name}/complete`.

<a name="scratch"></a>
Set up the project
------------------
First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Maven](https://maven.apache.org) and [Gradle](http://gradle.org) is included here. If you're not familiar with either, refer to [Getting Started with Maven](../gs-maven/README.md) or [Getting Started with Gradle](../gs-gradle/README.md).

### Create the directory structure

In a project directory of your choosing, create the following subdirectory structure; for example, with `mkdir -p src/main/java/hello` on *nix systems:

    └── src
        └── main
            └── java
                └── hello

### Create a Maven POM

`pom.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-integration-initial</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.bootstrap</groupId>
        <artifactId>spring-bootstrap-starters</artifactId>
        <version>0.5.0.BUILD-SNAPSHOT</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.bootstrap</groupId>
            <artifactId>spring-bootstrap-web-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.integration</groupId>
            <artifactId>spring-integration-core</artifactId>
            <version>2.2.4.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.integration</groupId>
            <artifactId>spring-integration-twitter</artifactId>
            <version>2.2.4.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.integration</groupId>
            <artifactId>spring-integration-file</artifactId>
            <version>2.2.4.RELEASE</version>
        </dependency>
    </dependencies>

    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>http://repo.springsource.org/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>http://repo.springsource.org/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

</project>
```

TODO: mention that we're using Spring Bootstrap's [_starter POMs_](../gs-bootstrap-starter) here.

> Note to experienced Maven users who don't use an external parent project: You can take it out later, it's just there to reduce the amount of code you have to write to get started.


<a name="initial"></a>
Define an Integration Plan
--------------------------

For this guide's sample application, you're going to define a Spring Integration plan that reads tweets from Twitter, transforms them into an easily readable `String`, and appends that `String` to the end of a file.

To define an integration plan, you simply create a Spring XML configuration using a handful of elements from Spring Integration's XML namespaces. Specifically, for the desired integration plan, you'll need to work with elements from 3 of Spring Integration's namespaces: core, twitter, and file.

The following XML configuration file defines the integration plan:

`src/main/resources/hello/integration.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:int="http://www.springframework.org/schema/integration"
    xmlns:twitter="http://www.springframework.org/schema/integration/twitter"
    xmlns:file="http://www.springframework.org/schema/integration/file"
    xsi:schemaLocation="http://www.springframework.org/schema/integration http://www.springframework.org/schema/integration/spring-integration.xsd
		http://www.springframework.org/schema/integration/file http://www.springframework.org/schema/integration/file/spring-integration-file-2.2.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/integration/twitter http://www.springframework.org/schema/integration/twitter/spring-integration-twitter-2.2.xsd">

    <twitter:search-inbound-channel-adapter id="tweets" 
            query="#HelloWorld" 
            twitter-template="twitterTemplate">
        <int:poller fixed-rate="5000"/>
    </twitter:search-inbound-channel-adapter>

    <int:transformer 
            input-channel="tweets" 
            expression="payload.fromUser + '  :  ' + payload.text + @newline" 
            output-channel="files"/>

    <file:outbound-channel-adapter id="files"
            mode="APPEND"
            charset="UTF-8"
            directory="/tmp/si"
            filename-generator-expression="'HelloWorld'"/>

</beans>
```

As you can see, there are three integration elements in play here:

 * `<twitter:search-inbound-channel-adapter>` - An inbound adapter that searches Twitter for tweets with "#HelloWorld" in the text. It is injected with a `TwitterTemplate` from [Spring Social][SpringSocial] to perform the actual search. As configured here, it polls every 5 seconds. Any matching tweets are placed into a channel named "tweets" (corresponding with the adapter's ID).
 * `<int:transformer>` - Transformed tweets in the "tweets" channel, extracting the tweet's author (`payload.fromUser`) and text (`payload.text`) and concatenating them into a readable `String`. The `String` is then written through the output channel named "files".
 * `<file:outbound-channel-adapter>` - An outbound adapter that writes content from its channel (here named "files") to a file. Specifically, as configured here, it will append anything in the "files" channel to a file at `/tmp/si/HelloWorld`.

This simple flow is illustrated like this:

![A flow plan that reads tweets from Twitter, transforms them to a String, and appends them to a file.](images/tweetToFile.png)

Notice that the integration plan references a couple of beans that aren't defined in `integration.xml`. Specifically, the "twitterTemplate" bean that is injected into the search inbound adapter and the "newline" bean referenced in the transformer are not defined here. Those beans will be declared separately in JavaConfig as part of the main class of the application.

Make the application executable
-------------------------------

Although it is common to configure a Spring Integration plan within a larger application, perhaps even a web applicaion, there's no reason that it can't be defined in a simpler standalone application. That's what you'll do next, creating a main class that kicks off the integration plan and also declares a handful of beans to support the integration plan. You'll also build the application into a standalone executable JAR file.

### Create a main class
`src/main/java/hello/Application.java`
```java
package hello;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportResource;
import org.springframework.core.env.Environment;
import org.springframework.social.oauth2.OAuth2Operations;
import org.springframework.social.oauth2.OAuth2Template;
import org.springframework.social.twitter.api.Twitter;
import org.springframework.social.twitter.api.impl.TwitterTemplate;

@Configuration
@ImportResource("/hello/integration.xml")
public class Application {

    public static void main(String[] args) {
        new AnnotationConfigApplicationContext(Application.class);
    }
    
    @Bean
    public String newline() {
        return System.getProperty("line.separator");
    }
    
    @Bean
    public Twitter twitterTemplate(OAuth2Operations oauth2) {
        return new TwitterTemplate(oauth2.authenticateClient().getAccessToken());
    }
    
    @Bean
    public OAuth2Operations oauth2Template(Environment env) {
        return new OAuth2Template(env.getProperty("clientId"), env.getProperty("clientSecret"), "", "https://api.twitter.com/oauth2/token");
    }
    
}
```

As you can see, this class provies a `main()` method that loads the Spring application context. It's also annotated as a `@Configuration` class, indicating that it will contain some bean definitions.

Specifically, there are three beans created in this class:

 * The `newline()` method creates a simple `String` bean containing the underlying system's newline character(s). This is used in the integration plan to place a newline at the end of the transformed tweet `String`.
 * The `twitterTemplate()` method defines a `TwitterTemplate` bean that is injected into the `<twitter:search-inbound-channel-adapter>`.
 * The `oauth2Template()` method defines a Spring Social `OAuth2Template` bean used to obtain a client access token when creating the `TwitterTemplate` bean.

Note that the `oauth2Template()` method references the `Environment` to get "clientId" and "clientSecret" properties. Those properties are ultimately client credentials you are given when you [register your application with Twitter][register-twitter-app]. By fetching them from the `Environment`, it prevents you from having to hardcode them in this configuration class. You'll need them when you [run the application](#run), though.

Finally, notice that `Application` is configured with `@ImportResource` to import the integration plan defined in `/hello/integration.xml`. 

### Build an executable JAR

Now that your `Application` class is ready, you simply instruct the build system to create a single, executable jar containing everything. This makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

Add the following configuration to your existing Maven POM:

`pom.xml`
```xml
    <properties>
        <start-class>hello.Application</start-class>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

The `start-class` property tells Maven to create a `META-INF/MANIFEST.MF` file with a `Main-Class: hello.Application` entry. This entry enables you to run the jar with `java -jar`.

The [Maven Shade plugin][maven-shade-plugin] extracts classes from all jars on the classpath and builds a single "über-jar", which makes it more convenient to execute and transport your service.

Now run the following to produce a single executable JAR file containing all necessary dependency classes and resources:

    mvn package

[maven-shade-plugin]: https://maven.apache.org/plugins/maven-shade-plugin

<a name="run"></a>
Running the Application
-----------------------

Now you can run the application from the jar:
```
$ java -DclientId={YOUR CLIENT ID} -DclientSecret={YOUR CLIENT SECRET} -jar target/gs-integration-complete-0.1.0.jar

... app starts up ...
```

Note that you'll need to be sure to specify your application's client ID and secret in place of the placeholders shown here.

Once the application starts up, it will connect to Twitter and start fetching tweets that match the search criteria of "#HelloWorld". It will process those tweets through the integration plan you defined, ultimately appending the tweet's author and text to a file at `/tmp/si/HelloWorld`.

After the application has been running for awhile, you should be able to view the file at `/tmp/si/HelloWorld` to see the data from a handful of tweets. On a UNIX-based operating system, you also choose to tail the file to see the results as they are written:

```sh
$ tail -f /tmp/si/HelloWorld
```

You should see something like this (the actual tweets may differ):

```sh
BrittLieTjauw  :  Now that I'm all caught up on the bachelorette I can leave my room #helloworld
mishra_ravish  :  Finally, integrated #eclim. #Android #HelloWorld
NordstrmPetite  :  Pink and fluffy #chihuahua #hahalol #boo #helloworld http://t.co/lelHhFN3gq
GRhoderick  :  Ok Saint Louis, show me what you got. #HelloWorld
```

Summary
-------
Congratulations! You have just developed a simple application that uses Spring Integration to fetch tweets from Twitter, process them, and write them to a file. There's a lot more that Spring Integration can do

[zip]: https://github.com/springframework-meta/gs-accessing-facebook/archive/master.zip
[u-war]: /understanding/war
[u-tomcat]: /understanding/tomcat
[u-application-context]: /understanding/application-context
[`SpringApplication`]: http://static.springsource.org/spring-bootstrap/docs/0.5.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/bootstrap/SpringApplication.html
[`@Component`]: http://static.springsource.org/spring/docs/current/javadoc-api/org/springframework/stereotype/Component.html
[`@EnableAutoConfiguration`]: http://static.springsource.org/spring-bootstrap/docs/0.5.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/bootstrap/context/annotation/SpringApplication.html
[`DispatcherServlet`]: http://static.springsource.org/spring/docs/current/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html
[SpringSocial]: http://www.springsource.org/spring-social
[register-twitter-app]: /gs-register-twitter-app/README.md
