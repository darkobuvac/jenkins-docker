FROM jenkins/ssh-agent:latest-jdk11 AS base

ARG HOST_GID
ARG HOST_UID

RUN apt-get update && apt-get install -y curl wget gpg apt-transport-https ca-certificates gnupg-agent software-properties-common
RUN install -m 0755 -d /etc/apt/keyrings

RUN apt-get update
RUN apt-get -y install sudo

#Intall node js

RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

ENV NODE_MAJOR=20

RUN echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list

RUN apt-get update && sudo apt-get install nodejs -y && sudo apt-get install nano

WORKDIR /etc/newman

COPY ./src .

RUN curl -o- "https://dl-cli.pstmn.io/install/linux64.sh" | sh

RUN npm install -g newman newman-reporter-html newman-reporter-junitfull newman-reporter-htmlextra newman-reporter-slackmsg ajv

WORKDIR /

RUN touch /home/jenkins/.ssh/authorized_keys

RUN chown -R jenkins:jenkins /home/jenkins && chmod 750 /home/jenkins && chmod 700 /home/jenkins/.ssh && chmod 600 /home/jenkins/.ssh/authorized_keys

RUN chmod -R 777 /etc/environment && chmod -R 777 /etc/ssh

RUN npm -v

USER jenkins

WORKDIR /home/jenkins