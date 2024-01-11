# Dockerize your Scala application

## Create a hello world project
 - sbt new scala/hello-world.g8

##  Run the hello world project
 - sbt run

## Package the application
### The JavaAppPackaging archetype from sbt-native-packager provides a default application structure and executable ### scripts to launch your application.
### Add the sbt-native-packager to your plugins.sbt (Create a new one under project folder)
 - addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "1.7.1")

### Enable the JavaAppPackaging plugin in your build.sbt with
 - enablePlugins(JavaAppPackaging)

### This can be done using the following command.
 - sbt stage

### The result is available under target/universal/stage
 - tree ./target/universal/stage/

### Test the generated script
 - bash ./target/universal/stage/bin/hello-world

### The usage below show some useful option already available into generated script, like :
 - verbose and debug options
 - a multi main class support
 - selection of java version to use
 - JVM options

### bash target/universal/stage/bin/hello-world -h
 
## Generate a docker image for the application
### The Docker Plugin from sbt-native-packager implement the following features
 - generate a Dockerfile based on JavaAppPackaging archetype stage.
 - sbt integration to build the docker image

### Enable the Docker Plugin in your build.sbt with
 - enablePlugins(DockerPlugin)

### Run this command to generate a Dockerfile
 - sbt docker:stage

### To build the docker image run the following command
 - sbt docker:publishLocal

### The result is a docker image named with the same name as the project and with the same version as the project.
 - docker image ls

### Launch this command to test
 - docker run --rm -ti hello-world:1.0

## Optimise the docker image
### The generated docker image have 525MB, let try to optimise this size. The docker history give as the size of each layer. Here we use grep to get only layer that the size is above or equals 1MB. The third column of this output show the Dockerfile command that generate this layer. If we check the Dockerfile we can see that only the first command COPY --chown=demiourgos728 is present. The other command come from the openjdk:8 base image
- docker history hello-world:1.0 | grep MB

### In fact openjdk:8 base image have 510MB as size
- docker image ls | grep openjdk | grep "8 "

### Below a list of some openjdk image and we can see that adoptopenjdk jre version the smallest one
- docker image ls | grep openjdk | grep "jdk" | sort -h -k7

### The Docker Plugin provide an option to override the base image.
 - Add this line in your build.sbt
 - dockerBaseImage := "adoptopenjdk:11.0.7_10-jre-hotspot"
 - and run again
 - sbt docker:publishLocal

### Verify that the base image has changed into Dockerfile
 - more target/docker/stage/Dockerfile | grep FROM

### The new generated docker image is 54% smaller
 - docker image ls

### Check that application still working
 - docker run --rm -ti hello-world:1.0

### By default, sbt Native Packager will create a daemon user named demiourgos728. To change name of the user, add this line in your build.sbt
 - daemonUser in Docker := "daemon"

# Copyright - https://enjoytechnology.netlify.app/2020/05/11/dockerize-scala-application/#optimise-the-docker-image
