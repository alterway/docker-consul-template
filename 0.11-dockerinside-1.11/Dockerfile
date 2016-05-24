FROM debian:jessie
MAINTAINER Ophélie Mauger <ophelie.mauger@alterway.fr>

ENV DOCKER_VERSION 1.11.1
RUN apt-get update && \
    apt-get install -y unzip wget && \
    rm -r /var/lib/apt/lists/* && \
    wget -O docker.tgz https://get.docker.com/builds/Linux/x86_64/docker-${DOCKER_VERSION}.tgz && \
    tar --extract --file=docker.tgz docker/docker && \
    mv docker/docker /usr/bin/docker && \
    rm -Rf docker && \
    chmod +x /usr/bin/docker

ENV CONSUL_TEMPLATE_VERSION 0.11.1
ADD https://releases.hashicorp.com/consul-template/${CONSUL_TEMPLATE_VERSION}/consul-template_${CONSUL_TEMPLATE_VERSION}_linux_amd64.zip /tmp/consul-template.zip

RUN unzip /tmp/consul-template.zip -d /usr/bin && \
    chmod +x /usr/bin/consul-template && \
    rm /tmp/consul-template.zip

COPY ctmpl /etc/ctmpl
WORKDIR /etc/ctmpl

ENTRYPOINT ["/usr/bin/consul-template"]
