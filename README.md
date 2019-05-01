# m2package

> This project is intended to be used to pull down COTs dependencies (via Maven) for internal repositories used in closed-development projects.

# Quick Start

> Note that Gradle will remove `$HOME/.m2` so that you have a "fresh" local repository.

1. Ensure that you have Maven in your `PATH`

2. Execute: `./gradlew` (Linux) or `gradlew.bat` (Windows)

3. Navigate to `m2package` folder and modify the `pom.xml` with the dependencies you need

4. Execute: `./gradlew build` to generate `build/m2package-<epoch>.zip`

5. The zip file contains the POM file as well as all the dependencies that was downloaded in your local `.m2` folder.

# Vagrant Environment

A `Vagrantfile` is provided so that you can have a virtualized environment with Gradle and Maven installed. It also comes with a local instance of [Apache Archive](https://archiva.apache.org). The Vagrant environment locks Maven to use the local Apache Archiva which also proxies to Maven Central for artifacts - building will not only download to `$HOME/.m2` but also populates Archiva internal repository.

1. Navigate to `/vagrant` to access this project

2. Execute: `vagrant up`

3. Execute: `vagrant ssh`

4. Follow Quick Start

5. Archiva is installed in `/opt/archiva` and started by default - installation instructions can be found [here](https://archiva.apache.org/docs/2.2.3/quick-start.html)

6. Archiva is running in Vagrant on port 8080 but you can reach it on your host machine at `http://localhost:8000`.

# POM Configurations

## Minimum

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>sample</groupId>
  <artifactId>m2package</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>m2package</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

## Vagrant Apache Archiva

```
<repositories>
  <repository>
    <id>archiva</id>
    <url>https://localhost:8080/repository/internal</url>
  </repository>
</repositories>

<dependencies>
  <dependency>
    <!-- Dependency from the archiva repository -->
    <exclusions>
      <exclusion>  <!-- declare the exclusion here -->
        <groupId></groupId>
        <artifactId></artifactId>
      </exclusion>
    </exclusions>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <version>2.0.9.RELEASE</version>
    <scope>compile</scope>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.0.9.RELEASE</version>
    <scope>compile</scope>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
    <version>2.0.9.RELEASE</version>
    <scope>compile</scope>
  </dependency>
  ....
</dependencies>
```

# Archiva Configurations and Gradle

1. Archiva is configured with a Remote Repository to be Maven Central. Within the Vagrant environment, Maven is locked down to only talk to Archiva; therefore, whenever it reaches Archiva for artifacts, Archiva will reach out to Maven Central for the libraries and update its internal repository.

2. A Gradle build can use the same Archiva instance for dependencies.

3. Apache Archiva is preconfigured with two managed repositories: cots (proxy to Maven Central) and plugins (proxy to Gradle plugins).

4. A Gradle `buildscript` tag can be pointed to the Archiva instance for plugins (i.e. Spring Boot Gradle Plugin).

   - Create a Remote Repository in Archiva pointing to `https://plugins.gradle.org/m2`

   - Create a Proxy Connector for a Managed Repository (i.e. plugins or internal) pointing to the Gradle Plugin Remote Repository.

   - Assure that the `Repository Observer - <repo>` (i.e. Repository Observer - plugins) role is assigned to the guest user

   - For example, in the Gradle build script, add the follow to `build.gradle`:

   ```
   buildscript {
     repositories {
       maven {
         url "http://localhost:8080/repository/plugins"
       }
     }
     dependencies {
       classpath 'com.moowork.gradle:gradle-node-plugin:1.2.0'
       classpath "org.springframework.boot:spring-boot-gradle-plugin:2.1.4.RELEASE"
     }
   }

   repositories {
     maven { url "http://localhost:8080/repository/cots" }
   }

   dependencies {
     compile group: 'org.springframework.boot', name: 'spring-boot-starter', version: '2.1.4.RELEASE'
   }
   ```

# Resources

- https://archiva.apache.org/docs/1.4-M4/userguide/using-repository.html

- Commands

   `mvn --offline <target>`

   `mvn dependency:go-offline` - Goal that resolves all project dependencies, including plugins and reports and their dependencies [reference](https://maven.apache.org/plugins/maven-dependency-plugin/go-offline-mojo.html).
