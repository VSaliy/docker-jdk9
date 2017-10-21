# Build a Docker Image with JDK 9

Table of Contents

[Create a Docker Image using JDK 9](https://github.com/VSaliy/docker-jdk9/tree/dev#create-a-docker-image-using-jdk-9)

[Create a Docker Image using JDK 9 and Alpine Linux](https://github.com/VSaliy/docker-jdk9/tree/dev#create-a-docker-image-using-jdk-9-and-alpine-linux)

[Create a Docker Image using JDK 9 and a Java application](https://github.com/VSaliy/docker-jdk9/tree/dev#create-a-docker-image-using-jdk-9-and-a-java-application)

[Reduce the size of a Docker Image using JDK 9 and a Java application](https://github.com/VSaliy/docker-jdk9/tree/dev#reduce-the-size-of-a-docker-image-using-jdk-9-and-a-java-application)

### Create a Docker Image using JDK 9

Create a new directory 'docker-jdk9'

```
git init
git remote add origin https://github.com/VSaliy/docker-jdk9.git
git checkout -b dev
```
Create a new text file [jdk-9-debian-slim.Dockerfile](jdk-9-debian-slim.Dockerfile)

```
# A JDK 9 with Debian slim
FROM debian:stable-slim
# Download from http://jdk.java.net/9/
# ADD http://download.java.net/java/GA/jdk9/9/binaries/openjdk-9_linux-x64_bin.tar.gz /opt
ADD openjdk-9_linux-x64_bin.tar.gz /opt
# Set up env variables
ENV JAVA_HOME=/opt/jdk-9
ENV PATH=$PATH:$JAVA_HOME/bin
CMD ["jshell", "-J-XX:+UnlockExperimentalVMOptions", \
               "-J-XX:+UseCGroupMemoryLimitForHeap", \
               "-R-XX:+UnlockExperimentalVMOptions", \
               "-R-XX:+UseCGroupMemoryLimitForHeap"]
```

```
git add .
git commit -m "start commit"
git push --set-upstream origin dev
```

`curl -LO http://download.java.net/java/GA/jdk9/9/binaries/openjdk-9_linux-x64_bin.tar.gz`

`docker image build -t jdk-9-debian-slim -f jdk-9-debian-slim.Dockerfile .`

`docker image ls`

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jdk-9-debian-slim   latest              a23d7cc00b2f        50 seconds ago      400MB
debian              stable-slim         d30525fb4ed2        10 days ago         55.3MB
```

`docker container run -m=200M -it --rm jdk-9-debian-slim`

`jshell>`

```
jshell> Runtime.getRuntime().maxMemory() / (1 << 20)
$1 ==> 100
```
### Create a Docker Image using JDK 9 and Alpine Linux

Create a new text file [jdk-9-alpine.Dockerfile](jdk-9-alpine.Dockerfile)

```
# A JDK 9 with Alpine Linux
FROM alpine:3.6
# Add the musl-based JDK 9 distribution
RUN mkdir /opt
# Download from http://jdk.java.net/9/
# ADD http://download.java.net/java/jdk9-alpine/archive/181/binaries/jdk-9-ea+181_linux-x64-musl_bin.tar.gz
ADD jdk-9-ea+181_linux-x64-musl_bin.tar.gz /opt
# Set up env variables
ENV JAVA_HOME=/opt/jdk-9
ENV PATH=$PATH:$JAVA_HOME/bin
CMD ["jshell", "-J-XX:+UnlockExperimentalVMOptions", \
               "-J-XX:+UseCGroupMemoryLimitForHeap", \
               "-R-XX:+UnlockExperimentalVMOptions", \
               "-R-XX:+UseCGroupMemoryLimitForHeap"]
```

```
git add .
git commit -m "JDK 9 and Alpine Linux"
git push --set-upstream origin dev
```

`curl -LO http://download.java.net/java/jdk9-alpine/archive/181/binaries/jdk-9-ea+181_linux-x64-musl_bin.tar.gz`

`docker image build -t jdk-9-alpine -f jdk-9-alpine.Dockerfile .`

`docker image ls`

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jdk-9-alpine        latest              6adf085ab630        4 seconds ago       356MB
jdk-9-debian-slim   latest              f68e97e4f78f        15 minutes ago      400MB
debian              stable-slim         d30525fb4ed2        10 days ago         55.3MB
alpine              3.6                 76da55c8019d        5 weeks ago         3.97MB
```
### Create a Docker Image using JDK 9 and a Java application

`git submodule add https://github.com/VSaliy/helloworld-java-9.git`

`cd helloworld-java-9`

```
docker container run --volume //d/dev/src/IdeaProjects/docker-jdk9/helloworld-java-9:/helloworld-java-9 --workdir /helloworld-java-9 -it --rm openjdk:9-jdk-slim ./mvnw package
```
```
./mvnw package
```

[helloworld-jdk-9.Dockerfile](https://github.com/VSaliy/helloworld-java-9/blob/master/helloworld-jdk-9.Dockerfile)

```
# Hello world application with JDK 9 and Debian slim
FROM jdk-9-debian-slim
COPY target/helloworld-1.0-SNAPSHOT.jar /opt/helloworld/helloworld-1.0-SNAPSHOT.jar
# Set up env variables
CMD java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap \
  -cp /opt/helloworld/helloworld-1.0-SNAPSHOT.jar org.examples.java.App
```

`docker image build -t helloworld-jdk-9 -f helloworld-jdk-9.Dockerfile .`

Run the jdeps tool to see what modules the application depends on:

`docker container run -it --rm helloworld-jdk-9 jdeps --list-deps /opt/helloworld/helloworld-1.0-SNAPSHOT.jar`

### Reduce the size of a Docker Image using JDK 9 and a Java application

```
docker run --rm --volume //d/dev/src/IdeaProjects/docker-jdk9/helloworld-java-9:/out jdk-9-debian-slim jlink --module-path /opt/jdk-9/jmods --verbose --add-modules java.base --compress 2 --no-header-files --output /out/target/openjdk-9-base_linux-x64
```

[helloworld-jdk-9-base.Dockerfile](https://github.com/VSaliy/helloworld-java-9/blob/master/helloworld-jdk-9-base.Dockerfile)

`docker image build -t helloworld-jdk-9-base -f helloworld-jdk-9-base.Dockerfile .`
