FROM jenkins/jenkins:latest

ARG HOST_GID
ARG HOST_UID

ARG AGENT_NAME

# Install required plugins

RUN jenkins-plugin-cli --plugins ssh-slaves
RUN jenkins-plugin-cli --plugins blueocean


# Set the Jenkins user ID and group ID to match the host user
USER root
RUN apt-get update
RUN apt-get -y install sudo

RUN apt-get -y install iputils-ping

RUN apt-get update && apt-get install -y --no-install-recommends cron && rm -r /var/lib/apt/lists/*

RUN echo "jenkins:jenkins" | chpasswd
RUN echo "root:root" | chpasswd
RUN echo "jenkins ALL=(ALL) ALL" >> /etc/sudoers

RUN mkdir -p /var/lib/jenkins/.ssh

RUN chown -R jenkins:jenkins /var/lib/jenkins/.ssh

RUN chown -R jenkins:jenkins /var/jenkins_home
RUN chmod -R 777 /var/jenkins_home

USER jenkins
