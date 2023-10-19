FROM jenkins/ssh-agent:latest-jdk11 AS base

#Install Dotnet SDK
RUN apt-get update && apt-get upgrade -y
RUN apt-get install sudo && apt-get install wget


RUN wget -q https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb

RUN dpkg -i packages-microsoft-prod.deb
RUN apt-get update && apt-get install -y apt-transport-https
RUN apt-get update
RUN apt-get install -y dotnet-sdk-7.0

RUN chmod -R 777 /etc/environment && chmod -R 777 /etc/ssh

USER jenkins