[//]: # (Implicit Links Within Project)

[1]: https://docs.gitlab.com/ee/ci/docker/using_kaniko.html   "GitLab Kaniko Docs"
[2]: https://dev.to/ipo/using-kaniko-to-build-and-publish-container-image-with-github-action-on-github-self-hosted-runners-d5m   "Rollson Article"
[3]: https://github.com/GoogleContainerTools/kaniko   "Google Container Tools"
[4]: https://madhuakula.com/kubernetes-goat/docs/scenarios/scenario-2/docker-in-docker-exploitation-in-kubernetes-containers/   "KubeGoat"
[5]: https://owasp.org/www-project-kubernetes-top-ten/   "OWASP Kubernetes Top 10"

# Kaniko Builds on GitHub Actions

This directory contains a sample pipeline to push docker images to GitHub Container Registry, using Kaniko, a container tool by Google.

## What's the Problem?

[Kubernetes OWASP Top 10][5] states that DIND is 'K01' - The top cause of Kubernetes exploitation. In this example repository, we want to build a docker image inside of another docker image. This is called running 'Docker in Docker' or 'DIND'. A docker container *cannot* run it's own Docker Daemon, instead, it must hook into the parent Daemon via a UNIX socket.

![A diagram demonstrating a malicious Supply Chain Attack on a Docker Host.](.README_IMAGES/dind_example.png)

Building an image using the Docker Daemon requires privileged access to it - You may have come across this before in another guise when running Docker on Linux, requiring the addition of a Docker User to the SUDOERS file.

We often build docker containers from other public images, or build docker containers with software packages taken from external sources. If a malicious attacker manages to infiltrate the supply chain of our own software's dependencies, there is a strong chance that they will target the privileged `/var/run/docker.sock` socket - It's privileged, and can do a lot of damage, from stealing intellectual property, to exploit propegation, even escaping the host system.

Madhu Akula has written an amazing series of articles explaining Kubernetes issues in detail, and I would recommend taking the time to [follow this tutorial through][4], just to see how *scary* and *straightforward* it is for a rogue package to exploit a UNIX socket misconfiguration and escape the host system to view other containers.

## What is Kaniko?

[Kaniko][3] is a tool written by Google (though not *officially* supported commercially), which utilises user space 