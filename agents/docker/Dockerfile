FROM jenkins/ssh-agent:latest-jdk11 AS base

ARG HOST_GID
ARG HOST_UID

#Install Docker CLI
RUN apt-get update && apt-get install -y curl wget gpg apt-transport-https ca-certificates gnupg-agent software-properties-common
RUN install -m 0755 -d /etc/apt/keyrings

RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
RUN chmod a+r /etc/apt/keyrings/docker.gpg

RUN add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable edge test"

RUN echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu xenial stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null

RUN apt-get update
RUN apt-get -y install sudo

RUN apt-get -y install docker-ce docker-ce-cli containerd.io

RUN usermod -u $HOST_UID jenkins
RUN echo "jenkins:jenkins" | chpasswd
RUN echo "root:root" | chpasswd
RUN echo "jenkins ALL=(ALL) ALL" >> /etc/sudoers

RUN groupmod -g $HOST_GID docker || echo 'Group already exists'
RUN usermod -aG docker jenkins

RUN chmod -R 777 /etc/environment && chmod -R 777 /etc/ssh

USER jenkins