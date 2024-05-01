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
# docker tag ae377e21e345 nexus-docker.eagro.az/java17:1.1 (here ae377e21e345 is the image id)
5. Add the repo URL to insecure connections of docker (we must add a hostname to /etc/hosts file):
# vi /etc/docker/daemon.json
{
"insecure-registries": ["nexus-repo.minagro.local:8090"]
}
6. Login to the repo:
# docker login nexus-docker.eagro.az
7. Push the image to repo:
# docker push nexus-docker.eagro.az/java17:1.1

############################################################
1. If we want to pull our image to K8S cluster, first we must to create secret of type 'kubernetes.io/dockerconfigjson' with the following specifiction:
# kubectl create secret docker-registry private-reg-cred --docker-server=nexus-repo.minagro.local:8090 --docker-username=admin --docker-password=<nexus_admin_password> --docker-email=dock_user@myprivateregistry.com
2. Then we have to add to '/etc/containerd/config.toml' the following section:

[plugins."io.containerd.grpc.v1.cri".registry.mirrors."nexus-repo.minagro.local:8090"]
          endpoint = ["http://nexus-repo.minagro.local:8090"]
        [plugins."io.containerd.grpc.v1.cri".registry.configs]
          [plugins."io.containerd.grpc.v1.cri".registry.configs."nexus-repo.minagro.local:8090".tls]
            insecure_skip_verify = true
            
3. After adding these lines, restart containerd service:
# systemctl restart containerd

4. Then, in the definition file, we must specify the following field under spec section:
imagePullSecrets:
         - name: private-reg-cred
