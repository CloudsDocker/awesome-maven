---
layout: page
title: Maven-Notes
tags:
- java
- maven
---
# Maven philosophy
- â€œIt is important to note that in the pom.xml file you specify the what and not the how. The pom.xml file can also serve as a documentation tool, conveying your project dependencies and their versions.â€

# Concepts
## Life cycles
Maven defines three Lifecycles – default, clean and site and each Lifecycle consists of predefined Phases. 

The clean argument which we pass to mvn command is phase which is in lifecycle named clean.

Maven Lifecycle Phases clean lifecycle
The clean lifecycle contains three phases pre-clean, clean and post-clean .
When we invoke command mvn clean, Maven loads Clean Lifecycle and executes the phases pre-clean and clean.
We can invoke any of these three phases and all preceding phases up to and including the invoked phase are executed sequentially.


Lifecycle phases can’t do anything by themselves. For example, phase clean by itself doesn’t have ability or functionality to delete the build directory. It delegate the task to a plugin named maven-clean-plugin. So, lifecycles phases are just some predefined steps which Maven invokes sequentially one after another. As we can see, phases are similar to the steps in a job which are executed one after another.

# Concepts summary
Lifecycles, Lifecycle Phases, Plugins and Plugin Goals are the core of Maven. and we summarize the concepts learned so far:
Maven comes with three lifecycles – default, clean and site.
each lifecycle is made up of lifecycle phases and in all, there are 28 phases – default 21, clean 3 and site 4.
when a lifecycle phase is invoked using mvn command, all preceding phases are executed sequentially one after another.
lifecycle phases by themselves doesn’t have any capabilities to accomplish some task and they rely on plugins to carryout the task.
depending on project and packaging type, Maven binds various plugin goals to lifecycle phases and goals carryout the task entrusted to them.


### default lifecycle
Default Lifecycle
The most important of the three lifecycles is the Default Lifecycle . Maven uses default lifecycle to build, test and distribute the project. Default lifecycle contains 21 phases.

Project may contain resources such as properties, XML configuration files etc., and phases process-resources and process-test-resources copy and process such resources files. and in a later chapter, we explain the resource management in detail.
The phases compile and test-compile complies the source Java files and test files respectively.
The phases package, install, and deploy are used to distribute the project. As we have already seen, the package phase creates JAR file of resources and compiled classes for distribution. The phase install, installs the artifacts of the project i.e jar and pom.xml to the local repository at $HOME/.m2 so that other projects can use them as dependencies. The phase deploy installs the artifacts of the project to a remote repository (probably on Internet) so that a wider group of projects can use it as dependency. We will cover these phases in a later chapter.


## goal
If we see the usage description of mvn command, apart from the options it accepts only two things – goal or phase.
Maven Lifecycle Phases - mvn usage description

mvn [options] [<goal(s)>] [<phase(s)>]

For example, we can directly compile the source with the following command.
$ cd simple-app
$ mvn compiler:compile
To run a goal with mvn, use the format <plugin prefix>:<goal>. In the above example, compiler:compile, the compiler is plugin prefix of maven-compiler-plugin and compile is the goal name. We can get the prefix of all plugins from Maven Plugin Directory.

--
When we invoke a goal directly, Maven executes just that goal, whereas when we invoke a lifecycle phase all the phases up to that phase are executed. We can see this in action with following example.
$ cd simple-app
$ mvn clean
$ mvn surefire:test

---
In very few situations we invoke plugin goals directly and more often than not, lifecycle phases are preferred.

Lifecycle Phases and Plugin Goals
When a lifecycle phase is run, depending on project type and packaging type, Maven binds plugin goals to lifecycle phases.
When we run mvn package in a Java Project.

---
To process-resources phase, Maven binds resources goal of maven-resources-plugin and to test phase, it binds test goal of maven-surefire-plugin and so on.
What’s happens at package phase is bit interesting. In a Java Project, Maven binds jar goal of maven-jar-plugin. However, when we run the same command in a webapp project, up to test phase Maven binds same goals, but to the package phase Maven binds war goal of maven-war-plugin the war:war instead of jar:jar.

### To see what goals bined to lifecycle phase
```bash
mvn help:describe -Dcmd=install
```
[INFO] 'install' is a phase corresponding to this plugin:
org.apache.maven.plugins:maven-install-plugin:2.4:install

It is a part of the lifecycle for the POM packaging 'pom'. This lifecycle includes the following phases:
* validate: Not defined
* initialize: Not defined
* generate-sources: Not defined
* process-sources: Not defined
* generate-resources: Not defined
* process-resources: Not defined
* compile: Not defined
* process-classes: Not defined
* generate-test-sources: Not defined
* process-test-sources: Not defined
* generate-test-resources: Not defined
* process-test-resources: Not defined
* test-compile: Not defined
* process-test-classes: Not defined
* test: Not defined
* prepare-package: Not defined
* package: Not defined
* pre-integration-test: Not defined
* integration-test: Not defined
* post-integration-test: Not defined
* verify: Not defined
* install: org.apache.maven.plugins:maven-install-plugin:2.4:install
* deploy: org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy



## Plugin
Maven is – at its heart – a plugin execution framework; all work is done by plugins.

Plugins are broadly grouped as build plugins and reporting plugins. 

Plugins are artifacts that provide goals to Maven. Furthermore, a plugin may have one or more goals wherein each goal represents a capability of that plugin. For example, the Compiler plugin has two goals: compile and testCompile. The former compiles the source code of your main code, while the latter compiles the source code of your test code.

Plugin Goals
Maven plugin is a collection of one or more goals which do some task or job. It is also known as Mojo – Maven Plain Old Java Object.


Similarly, Maven uses maven-compiler-plugin to compile the source and test files and it provides three goals – compiler:compile, compiler:testCompile and compiler:help.


Suffice it to say for now that a plugin is a collection of goals with a general common purpose. For example the jboss-maven-plugin, whose purpose is "deal with various jboss items".

To configure plugins, we use project build element in pom.xml. The next listing shows the top level elements used to configure a plugin.
pom.xml
```xml
<project>
...
  <build>
    ...
    <plugins>
      <plugin>
        <groupId>...</groupId>
        <artifactId>...</artifactId>
        <version>...</version>
        <configuration>...</configuration>        
        <executions>
           <execution>...</executions>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

The elements are
- build : defines the project build.
- plugins : parent element for one or more <plugin> elements.
- plugin : the plugin to configure.
- groupId, artifactId, version : coordinates of the plugin to configure.
- configuration : holds the parameters and properties to be passed to the plugin.
- executions : parent element for one or more <execution> element.
- execution : configures the execution of a goal of the plugin.


In Maven, there are the build and the reporting plugins:

- Build plugins will be executed during the build and then, they should be configured in the <build/> element.
- Reporting plugins will be executed during the site generation and they should be configured in the <reporting/> element.


### position
Normally, the <build> block is placed after the project coordinates block and before the dependencies block.

### To show plugin details in configuration
Easiest way to know the available parameters for a goal is to run plugin help goal.
$ mvn compiler:help -Dgoal=compile -Ddetail
It will list the available parameters for compile goal of compiler.
But it will not show the default value of the parameters and to know the available parameters and also, the default value for each parameter, run help:describe goal of Maven Help plugin (maven-help-plugin).
$ mvn help:describe -Dplugin=compiler -Dmojo=compile -Ddetail
Note that maven-help-plugins uses -Dmojo for goal, instead of -Dgoal,


### Help Goal
Recent Maven plugins have generally an help goal to have in the command line the description of the plugin, with their parameters and types. For instance, to understand the javadoc goal, you need to call:
```bash
mvn javadoc:help -Ddetail -Dgoal=javadoc
```


#### Samples
- To include or exclude certain files or directories in the project Jar, configure Maven Jar Plugin with <include> and <exclude> parameters.
simple-app/pom.xml
  ...
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <configuration>
          <includes>
            <include>**/service/*</include>
          </includes>
        </configuration>
      </plugin>
    </plugins>
  </build>
  ...

- Maven Clean Plugin deletes the target directory by default and we may configure it to delete additional directories and files.
simple-app/pom.xml
  ...
  <build>
   <plugins>
    <plugin>
      <artifactId>maven-clean-plugin</artifactId>
      <configuration>
        <filesets>
        <filesets>
          <fileset>
            <directory>src/main/generated</directory>
            <followSymlinks>false</followSymlinks>
            <useDefaultExcludes>true</useDefaultExcludes>
            <includes>
              <include>*.java</include>
            </includes>
            <excludes>
              <exclude>Template*</exclude>
            </excludes>
          </fileset>
        </filesets>
      </configuration>
    </plugin>
   </plugins>
  </build>
  ...
The above configuration forces the clean plugin to delete *.java files in src/main/generated directory, but excludes the Template*.

### plugin execution
Firstly demonstrate plugin execution with an example. In Apache Ant, it’s quite easy to output echo any property or message during the build and many of us frequently use it to understand the build flow or to debug. But, Maven comes with no such feature, and only way to echo any message is to use Ant within Maven. The Apache AntRun Plugin provides the ability to run Ant tasks within Maven.
Let’s configure maven-antrun-plugin to output message to console.
simple-app/pom.xml
  ...
  <build>
    <plugins>
      <plugin>
        <artifactId>maven-antrun-plugin</artifactId>
        <executions>
          <execution>
            <goals>
              <goal>run</goal>
            </goals>
            <phase>compile</phase>
            <configuration>
              <tasks>
                <echo>Build Dir: ${project.build.directory}</echo>
              </tasks>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  ...
Build the project with  mvn package and it echoes the build directory name in compile phase.
 

#### sample
The clean and package arguments are build phases, while the dependency:copy-dependencies is a goal (of a plugin).

mvn clean dependency:copy-dependencies package
If this were to be executed, the clean phase will be executed first (meaning it will run all preceding phases of the clean lifecycle, plus the clean phase itself), and then the dependency:copy-dependencies goal, before finally executing the package phase (and all its preceding build phases of the default lifecycle).



#### concept
The element <executions>/<execution> allows you to configure the execution of a plugin goal. With it, you can accomplish the following things.
bind a plugin goal to a lifecycle phase.
configure plugin parameters of a specific goal.
configure plugin parameters of a specific goal such as compiler:compile, surefire:test etc., that are by default binds to a lifecycle phase


- 

## core concepts 
## phase
you may notice the second is simply a single word - package. Rather than a goal, this is a phase. A phase is a step in the build lifecycle, which is an ordered sequence of phases. When a phase is given, Maven will execute every phase in the sequence up to and including the one defined. For example, if we execute the compile phase, the phases that actually get executed are:

- validate
- generate-sources
- process-sources
- generate-resources
- process-resources
- compile

Maven Phases
Although hardly a comprehensive list, these are the most common default lifecycle phases executed.

- validate: validate the project is correct and all necessary information is available
- compile: compile the source code of the project
- test: test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed
- package: take the compiled code and package it in its distributable format, such as a JAR.
- integration-test: process and deploy the package if necessary into an environment where integration tests can be run
- verify: run any checks to verify the package is valid and meets quality criteria
- install: install the package into the local repository, for use as a dependency in other projects locally
- deploy: done in an integration or release environment, copies the final package to the remote repository for sharing with other developers and projects.
There are two other Maven lifecycles of note beyond the default list above. They are

- clean: cleans up artifacts created by prior builds
- site: generates site documentation for this project
Phases are actually mapped to underlying goals. The specific goals executed per phase is dependant upon the packaging type of the project. For example, package executes jar:jar if the project type is a JAR, and war:war if the project type is - you guessed it - a WAR.

An interesting thing to note is that phases and goals may be executed in sequence.

mvn clean dependency:copy-dependencies package
This command will clean the project, copy dependencies, and package the project (executing all phases up to package, of course).

--
Lifecycle and phases are just formal names for jobs and steps. Maven calls the jobs as lifecycles and tasks (or steps) as phases.
---
when a phase is invoked using mvn command, all preceding phases up to and including the invoked phase are executed sequentially. For example,
mvn compile – will run phases process-resources and then compile.
mvn test – will run phases process-resources, compile, process-test-resources, test-compile and finally test.
mvn install – will run phases process-resources, compile, process-test-resources, test-compile, test and finally install.
Like in clean lifecycle, in default lifecycle too, lifecycle phases by themselves don’t have capabilities to accomplish some task. For example, compile phase by itself can’t do anything but, it delegates compilation job to a plugin named maven-compiler-plugin.


### Some Phases Are Not Usually Called From the Command Line
The phases named with hyphenated-words (pre-*, post-*, or process-*) are not usually directly called from the command line. These phases sequence the build, producing intermediate results that are not useful outside the build. In the case of invoking integration-test, the environment may be left in a hanging state.



## Maven default repository

http://repo.maven.apache.org/maven2/

# Maven Settings
The settings element in the settings.xml file contains elements used to define values which configure Maven execution in various ways, like the pom.xml, but should not be bundled to any specific project, or distributed to an audience. These include values such as the local repository location, alternate remote repository servers, and authentication information.

There are two locations where a settings.xml file may live:

- The Maven install: ${maven.home}/conf/settings.xml
- A userâ€™s install: ${user.home}/.m2/settings.xml
The former settings.xml are also called `global settings`, the latter settings.xml are referred to as user settings. If both files exists, their contents gets merged, with the user-specific settings.xml being dominant.

Tip: If you need to create user-specific settings from scratch, itâ€™s easiest to copy the global settings from your Maven installation to your ${user.home}/.m2 directory. Mavenâ€™s default settings.xml is a template with comments and examples so you can quickly tweak it to match your needs.

## The contents of the settings.xml can be interpolated using the following expressions:

${user.home} and all other system properties (since Maven 3.0)
${env.HOME} etc. for environment variables
Note that properties defined in profiles within the settings.xml cannot be used for interpolation.


## All Maven pom.xml inherits from super POM

# Coc: Convention over Configuration
- Convention over configuration is a simple concept. Systems, libraries, and frameworks should assume reasonable defaults without requiring that unnecessary configuration systems should â€œjust work.â€ Popular frameworks such as Ruby on Rails and EJB3 have started to adhere to these principles in reaction to the configuration complexity of frameworks such as the initial Enterprise JavaBeans? (EJB) specifications. 
- â€œPopularized by the Ruby on Rails community, CoC emphasizes sensible defaults, thereby reducing the number of decisions to be made.â€
- â€œGradleâ€™s flexibility, like that of Ant, can be abused, which results in difficult and complex builds.
â€


# Notes
- This is the Project Object Model (POM), a declarative description of a project. 

## Coordinate
- â€œgroup, artifact, and version (**GAV**) coordinatesâ€
- Weâ€™ve highlighted the Maven coordinates for this project: **groupId, artifactId, version and packaging. These combined identifiers make up a projectâ€™s coordinates**.[3] Just as in any other coordinate system, a Maven coordinate is an address for a specific point in â€œspaceâ€: from general to specific. Maven pinpoints a project via its coordinates when one project relates to another, either as a dependency, a plugin, or a parent project reference. 
- Maven coordinates are often written using a colon as a delimiter in the following format: **groupId:artifactId:packaging:version**. 
- Projects undergoing active development can use a special identifier that marks **a version as a SNAPSHOT**.
- The packaging format of a project is also an important component in the Maven coordinates, but it isnâ€™t a part of a projectâ€™s unique identifiers. A projectâ€™s groupId:artifactId:version make that project unique; you canâ€™t have a project with the same three groupId, artifactId, and version identifiers.
- **packaging: The type of project, defaulting to jar**, describing the packaged output produced by a project. A project with packaging jar produces a JAR archive; a project with packaging war produces a web application.
- The core of Maven is pretty dumb; it doesnâ€™t know how to do much beyond parsing a few XML documents and keeping track of a lifecycle and a few plugins. Maven has been designed to delegate most responsibility to a set of Maven plugins that can affect the Maven lifecycle and offer access to goals. Most of the action in Maven happens in plugin goals that take care of things like compiling source, packaging bytecode, publishing sites, and any other task that needs to happen in a build. 
- You benefit from the fact that plugins are downloaded from a remote repository and maintained centrally. This is what is meant by universal reuse through Maven plugins.
- working developers are starting to realize that Maven not only simplifies the task of build management, it is helping to encourage a common interface between developers and software projects.

# Maven vs Ant
## The differences between Ant and Maven in this example are:
### Apache Ant 
Ant doesnâ€™t have formal conventions such as a common project directory structure; you have to tell Ant exactly where to find the source and where to put the output. Informal conventions have emerged over time, but they havenâ€™t been codified into the product.
- Ant is procedural; you have to tell Ant exactly what to do and when to do it. You have to tell it to compile, then copy, then compress.
- Ant doesnâ€™t have a lifecycle; you have to define goals and goal dependencies. You have to attach a sequence of tasks to each goal manually.
### Apache Maven
- Maven has conventions: in the example, it already knew where your source code was because you followed the convention. It put the bytecode in target/classes, and it produced a JAR file in target.
- Maven is declarative; all you had to do was create a pom.xml file and put your source in the default directory. Maven took care of the rest.
- Maven has a lifecycle, which you invoked when you executed mvn install. This command told Maven to execute a series of sequence steps until it reached the lifecycle. As a side effect of this journey through the lifecycle, Maven executed a number of default plugin goals that did things such as compile and create a JAR.
- A Maven plugin is a collection of one or more goals (see Figure 3-1). Examples of Maven plugins can be simple core plugins such as the Jar plugin that contains goals for creating JAR files, the Compiler plugin that contains goals for compiling source code and unit tests, or the Surefire plugin that contains goals for executing unit tests and generating reports.

#### Maven life cycle
- The second command we ran in the previous section was **mvn install**. This command didnâ€™t specify a plugin goal; instead, it specified a Maven lifecycle phase. A phase is a step in what Maven calls the â€œbuild lifecycle.â€ The build lifecycle is an ordered sequence of phases involved in building a project.
- **Plugin goals can be attached to a lifecycle phase**. As Maven moves through the phases in a lifecycle, it will execute the goals attached to each particular phase. Each phase may have zero or more goals bound to it. In the previous section, when you ran mvn install, you might have noticed that more than one goal was executed. Examine the output after running mvn install and take note of the various goals that are executed. 
- Maven steps through the phases preceding package in the Maven lifecycle; **executing a phase will first execute all proceeding phases in order**, ending with the phase specified on the command line. Each phase corresponds to zero or more goals, and since we havenâ€™t performed any plugin configuration or customization, this example binds a set of standard plugin goals to the default lifecycle. 

# Dependency
- â€œMaven provides **declarative dependency management**. With this approach, you declare your projectâ€™s dependencies in an external file called pom.xml. Maven will automatically download those dependencies and hand them over to your project for the purpose of building, testing, or packaging.â€
- â€œThe default remote repository with which Maven interacts is called Maven Central, and it is located at repo.maven.apache.org and uk.maven.org.â€
- â€œThe **internal repository manager** acts as a proxy to remote repositories. Because you have full control over the internal repository, you can regulate the types of artifacts allowed in your company. Additionally, you can also push your organizationâ€™s artifacts onto the server, thereby enabling collaboration. â€, such as Nexus

## Transitive Dependcies
- â€œA key benefit of Maven is that it automatically â€œdeals with transitive dependencies and includes them in your project.
- â€œMaven uses a technique known as **dependency mediation** to resolve version conflicts. Simply stated, dependency mediation allows Maven to pull the dependency that **is closest to the project in the tree**. In Figure 3-3, there are two versions of dependency B: 0.0.8 and 1.0.0. In this scenario, version 0.0.8 of dependency B is included in the project, because it is a direct dependency and closest to the tree. Now look at the three versions of dependency F: 0.1.3, 1.0.0, and 2.2.0. All three dependencies are at the same depth. In this scenario, Maven will use the **first-found dependency**, which would be 0.1.3, and not the latest 2.2.0 version.â€

## Dependency Scope
- â€œMaven uses the concept of scope, which allows you to specify **when and where** you need a particular dependency.â€ â€œMaven provides the following six scopes:â€
- â€œ**compile**: Dependencies with the compile scope are available in the class path in all phases on a project build, test, and run. This is the default scope.â€
- â€œ**provided**: Dependencies with the provided scope are available in the class path during the build and test phases. They donâ€™t get bundled within the generated artifact. Examples of dependencies that use this scope include Servlet api, JSP api, and so on.â€
- â€œ**runtime**: Dependencies with the runtime scope are not available in the class path during the build phase. Instead they get bundled in the generated artifact and are available during runtime.â€
- â€œ**test**: Dependencies with the test scope are available during the test phase. JUnit and TestNG are good examples of dependencies with the test scope.â€
- â€œ**system**: Dependencies with the system scope are similar to dependencies with the provided scope, except that these dependencies are not retrieved from the repository. Instead, a hard-coded path to the file system is specified from which the dependencies are used.â€
- â€œ**import**: The import scope is applicable for .pom file dependencies only. It allows you to include dependency management information from a remote .pom file. The import scope is available only in Maven 2.0.9 or later.â€


# Setup Proxy
- â€œMaven requires an Internet connection to download plug-ins and dependencies. Some companies employ HTTP proxies to restrict access to the Internet. In those scenarios, running Maven will result in Unable to download artifact errors. To address this, edit the settings.xml file and add the proxy information specific to your company.â€
```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <proxies>
  <proxy>
      <id>companyProxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>proxy.company.com</host>
      <port>8080</port>
      <username>proxyusername</username>
      <password>proxypassword</password>
      <nonProxyHosts />
    </proxy>
  </proxies>
 </settings>
```


# Multimodule projects
## Generate parent
```sh
mvn archetype:generate -DgroupId=com.apress.gswmbook -DartifactId=gswm-parent -Dversion=1.0.0-SNAPSHOT -DarchetypeGroupId=org.codehaus.mojo.archetypes -DarchetypeArtifactId=pom-root
```
Listing 6-5. Parent pom.xml File with Modules

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.apress.gswmbook</groupId>
  <artifactId>gswm-parent</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>pom</packaging>
  <name>gswm-parent</name>
  <modules>
    <module>gswm-web</module>
    <module>gswm-service</module>
    <module>gswm-repository</module>
  </modules>
</project>
```
## Generate web module
```sh
mvn archetype:generate -DgroupId=com.todzhang.mywebApp -DartifactId=main-web -Dversion=1.0.0-SNAPSHOT -Dpackage=war -DarchetypeArtifactId=maven-archetype-webapp
```
```xml
<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.apress.gswmbook</groupId>
    <artifactId>gswm-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
  </parent>
  <groupId>com.apress.gswmbook</groupId>
  <artifactId>gswm-web</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>war</packaging>
  <name>gswm-web Maven Webapp</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <finalName>gswm-web</finalName>
  </build>
</project>
```
## Generate a service jar module
```sh
mvn archetype:generate -DgroupId=com.todzhang.mywebApp -DartifactId=back-service -Dversion=1.0.0-SNAPSHOT -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```
- Notice that you didnâ€™t provide the package parameter, as the maven-archetype-quickstart produces a JAR project by default.
## To start the module
```sh
mvn packages
```
# Create archetype from project
```sh
mvn archetype:create-from-project
```
# Site life cycle
```sh
mvn site
```
- The site life cycle uses Mavenâ€™s site plug-in to generate the site for a single project. Once this command completes, a site folder gets created under the projectâ€™s target.
- Open the index.html file to browse the generated site. You will notice that Maven used the information provided in the pom.xml file to generate most of the documentation. It also automatically applied the default skin and generated the corresponding images and CSS files. 
- To regenerate site
```sh
mvn clean site
```

# JavaDoc
- Maven provides a Javadoc plug-in, which uses the Javadoc tool for generating Javadocs. Integrating the Javadoc plug-in simply involves declaring it in the reporting element of pom.xml file, as shown in Listing 7-4. Plug-ins declared in the pom reporting element are executed during site generation.
```xml
<project>
        <!â€”Content removed for brevity-->
 <reporting>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-javadoc-plugin</artifactId>
        <version>2.10.1</version>
      </plugin>
    </plugins>
  </reporting>
</project>
```
Then run **mvn clean site** to generate javadoc,  the apidocs folder created under gswm /target/site
# Unit test report
- Maven offers the Surefire plug-in that provides a uniform interface for running tests created by frameworks such as JUnit or TestNG. It also generates execution results in various formats such as XML and HTML. These published results enable developers to find and fix broken tests quickly.
The Surefire plug-in is configured in the same way as the Javadoc plug-in in the reporting section of the pom file. Listing 7-5 shows the Surefire plug-in configuration.
```xml
<project>
        <!â€”Content removed for brevity-->
  <reporting>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-report-plugin</artifactId>
        <version>2.17</version>
      </plugin>
    </plugins>
  </reporting>
</project>
```
-  you will see a Surefire Reports folder generated under gswm\target. It contains the test execution results in XML and TXT formats. The same information will be available in HTML format in the surefire-report.html file under site folder.

# Code coverate report
- Code coverage is a measurement of how much source code is being exercised by automated tests. Essentially, it provides an indication of the quality of your tests. Emma and Cobertura are two popular open source code coverage tools for Java.
In this section, you will use Cobertura for measuring this projectâ€™s code coverage. Configuring Cobertura is similar to other plug-ins, as shown in
```xml
<project>
        <!â€”Content removed for brevity-->
  <reporting>
    <plugins>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>cobertura-maven-plugin</artifactId>
        <version>2.6</version>
      </plugin>
   </plugins>
  </reporting>
</project>
```
# Find bug reports
- FindBugs is a tool for detecting defects in Java code. It uses static analysis to detect bug patterns, such as infinite recursive loops and null pointer dereferences. Listing 7-7 shows the FindBugs configuration.

```xml
<project>
        <!â€”Content removed for brevity-->
  <reporting>
    <plugins>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>findbugs-maven-plugin</artifactId>
        <version>3.0.0</version>
      </plugin>
   </plugins>
 </reporting>
</project>
```

# Integration with Nexus
-  Repository managers act as a proxy of public repositories, facilitate artifact sharing and team collaboration, ensure build stability, and enable the governance of artifacts used in the enterprise.
- Nexus is a popular open source repository manager from Sonatype. It is a web application that allows you to maintain internal repositories and access external repositories. It allows repositories to be grouped and accessed via a single URL. This enables the repository administrator to add and remove new repositories behind the scenes without requiring developers to change the configuration on their computers. Additionally, it provides hosting capabilities for sites generated using Maven site and artifact search capabilities.

## To enable nexus in Maven
- You will start by adding a distributionManagement element in the pom.xml file, as shown in Listing 8-1. This element is used to declare the location where the projectâ€™s artifacts will be when deployed. The repository element indicates the location where released artifacts will be deployed. Similarly, the snapshotRepository element identifies the location where the SNAPSHOT versions of the project will be stored.
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi=http://www.w3.org/2001/XMLSchema-instanceâ€ xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">  <modelVersion>4.0.0</modelVersion>

        <!-- Content removed for brevity -->
        <distributionManagement>
           <repository>
                <id>nexusReleases</id>
              <name>Releases</name>
        <url>http://localhost:8081/nexus/content/repositories/releases</url>
           </repository>
           <snapshotRepository>
              <id>nexusSnapshots</id>
              <name>Snapshots</name>
              <url>http://localhost:8081/nexus/content/repositories/snapshots</url>
           </snapshotRepository>
        </distributionManagement>
<!-- Content removed for brevity -->
</project>
```
- Out of the box, Nexus comes with **Releases and Snapshots repositories**. By default, SNAPSHOT artifacts will be stored in the Snapshots Repository, and release artifacts will be stored in the Releases repository.
Like most repository managers, deployment to Nexus is a protected operation. You provide the credentials needed to interact with Nexus in the **settings.xml** file.
- Listing 8-2. Settings.xml File with Server Information
- Listing 8-2 shows the settings.xml file with the server information. The Nexus deployment user with password deployment123 is provided out of the box. Notice that the IDs declared in the server tag â€” nexusReleases and nexusSnapshots must match the IDs of the repository and snapshotRepository declared in the pom.xml file. Replace the contents of the settings.xml file in the C:\Users\<<USER_NAME>>\.m2 folder with the code in Listing 8-2.
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
<servers>
   <server>
      <id>nexusReleases</id>
      <username>deployment</username>
      <password>deployment123</password>
   </server>
   <server>
      <id>nexusSnapshots</id>
      <username>deployment</username>
      <password>deployment123</password>
   </server>
</servers>
</settings>
```
- This concludes the configuration steps for interacting with Nexus. At the command line, run the command mvn deploy under the directory C:\apress\gswm-book\chapter8\gswm. Upon successful execution of the command, you will see the SNAPSHOT artifact under Nexus 


# POM

## Effective POM
At the start of every build, Maven internally merges project pomx.ml with Super POM and constructs a new POM which is known as Effective POM .

Earlier in Maven Lifecycle and Plugin Goals, we learned that Maven binds plugins goals to lifecycle phases. Actually, this magic happens in effective POM and it is highly instructive to go through the effective POM to know the what goes on under the hood.

### To dump effective POM
use Maven Help Plugin to dump the effective POM to a file for investigation.
```bash
$ mvn help:effective-pom -Doutput=target/effective-pom.xml
```

## Packaging
The first, and most common way, is to set the packaging for your project via the equally named POM element <packaging>. Some of the valid packaging values are jar, war, ear and pom. If no packaging value has been specified, it will default to jar.

Each packaging contains a list of goals to bind to a particular phase. For example, the jar packaging will bind the following goals to build phases of the default lifecycle.
## This is an almost standard set of bindings; however, some packagings handle them differently. For example, a project that is purely metadata (packaging value is pom) only binds goals to the install and deploy phases (for a complete list of goal-to-build-phase bindings of some of the packaging types, refer to the Lifecycle Reference).


# build tools options
Maven and Ant are just different approaches: imperative and declarative (see Imperative vs Declarative build systems)

Maven is better for managing dependencies (but Ant is ok with them too, if you use Ant+Ivy) and build artefacts. The main benefit from maven - its lifecycle. You can just add specific actions on correct phase, which seems pretty logical: just launch you integration tests on integration-test phase for example. Also, there are many existing plugins, which can could almost everything. Maven archetype is powerful feature, which allows you to quickly create project.

Ant is better for controlling of build process. Before your very first build you have to write you build.xml. If your build process is very specific, you have to create complicated scripts. For long-term projects support of ant-scripts could become really painful: scripts become too complicated, people, who has written them, could leave project, etc.

Both of them use xml, which could become too big in big long-term projects.

Anyway, you shoud read specific documentation (not hate-articles) on both. Also, there is ant-maven-plugin, which allow to launch ant-scripts with maven.

P.S. You can take a look on Gradle, which for me could provide more freedom than Maven, but is easier to use than Ant.

