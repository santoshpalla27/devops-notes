# Maven Notes

```xml
<project>
  ...
  <packaging>jar</packaging>   <!-- or war -->
  ...
</project>
```

This decides to build a war or jar file

## Key Differences

| Feature | .jar | .war |
|---------|------|------|
| Target Application | Standalone Java app or library | Web application (Servlet-based) |
| Deployment | Anywhere with JVM | On web/app server |
| Executable | Yes (with Main-Class) | No (needs a servlet container) |
| Entry Point | main() method | web.xml or annotations like @WebServlet |
| Structure | Flat or library-structured | Follows WEB-INF web app layout |

```bash
yum install maven -y  # to install maven
```

Maven integration is the plugin to install maven

To install and maven java in windows

First install java and maven
Install java 8 and to install maven go to maven site and download apache-maven-bin.zip

Download the jdk and install and in c directory you will find java and for maven directly after extraction folder is available

Then go to the java path in c/program files/jdk and copy the path and go to environment variable in pc advanced and environment variables and in user var add new JAVA_HOME and paste the java bin path and in system variables edit path var paste java bin path same for maven

For maven change JAVA_HOME to M2_HOME

## Maven Lifecycle Phases

validate – validate the project is correct and all necessary information is available
compile – compile the source code of the project
test – test the compiled source code using a suitable unit testing framework. These tests should not require the code be package or deploy.
package – take the compiled code and package it in its distributable format, such as a JAR.
verify – run any checks on results of integration tests to ensure quality criteria are met
install – install the package into the local repository, for use as a dependency in other projects locally
deploy – done in the build environment, copies the final package to the remote repository for sharing with other developers and projects.

### Maven Lifecycle Phases

Maven has predefined phases in its build lifecycle:

| Phase | Description |
|-------|-------------|
| validate | Checks project structure |
| compile | Compiles the source code |
| test | Runs unit tests |
| package | Creates JAR/WAR file |
| verify | Runs integration tests |
| install | Installs artifact into local repository |
| deploy | Uploads artifact to a remote repository |

## Maven Build Lifecycle

Maven build lifecycle consists of different stages:

validate compile test package integration-test verify install deploy

```bash
mvn clean install  # to build mvn code clean removes previous builds
```

When we use install every step until install takes place

## Introduction to Maven

Maven is a build automation tool used primarily for Java projects.

It manages dependencies, build lifecycle, documentation, and deployment.

Uses POM (Project Object Model) file (pom.xml) for configuration.

Convention over configuration.

```bash
mvn clean          # Cleans the target directory
mvn compile        # Compiles source code
mvn package        # Creates a JAR/WAR file
mvn install        # Installs the package in the local repository
mvn deploy         # Deploys the package to a remote repository
mvn test           # Runs unit tests
mvn verify         # Runs integration tests and other checks
mvn site           # Generates project site documentation
```