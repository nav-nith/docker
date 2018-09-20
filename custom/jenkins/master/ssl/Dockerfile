FROM jenkins/jenkins:latest

ENV DEBIAN_FRONTEND=noninteractive
ENV JENKINS_HOME /var/jenkins_home
ENV CERT_FOLDER "$JENKINS_HOME/.ssl"

ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000

USER root

# update
RUN apt-get update && apt-get upgrade -y

# SSL Setup
ENV JENKINS_OPTS --httpPort=-1 --httpsPort=443 --httpsCertificate="$CERT_FOLDER/*.pem" --httpsPrivateKey="$CERT_FOLDER/*.key"
EXPOSE 443

USER ${user}
