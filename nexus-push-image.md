How to push docker image to Nexus repo:

1. Create a Dockerfile:

FROM openjdk:17-jdk-buster

RUN apt-get update && \
    apt-get install -y telnet curl dnsutils iputils-ping && \
    rm -rf /var/lib/apt/lists/*

CMD ["bash"]

2. Create an image:
# docker build -t nexus-java17 .
3. Test the image:
# docker run -it --rm nexus-java17
4. Create a tag for the image:
# docker tag ae377e21e345 nexus.digirella.local:8090/java17:1.1 (here ae377e21e345 is the image id)
5. Add the repo URL to insecure connections of docker (we must add a hostname to /etc/hosts file):
# vi /etc/docker/daemon.json
{
"insecure-registries": ["nexus.digirella.local:8090"]
}
6. Login to the repo:
# docker login nexus.digirella.local:8090
7. Push the image to repo:
# docker push nexus.digirella.local:8090/java17:1.1
