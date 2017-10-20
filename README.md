# Build a Docker Image with JDK 9

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
