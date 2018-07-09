FROM debian:jessie
LABEL maintainer="katia.talhi@alterway.fr"

WORKDIR /etc/ctmpl

RUN apt-get update && \
    apt-get install --no-install-recommends --no-install-suggests -y unzip && \
    rm -r /var/lib/apt/lists/*

ENV CONSUL_TEMPLATE_VERSION 0.19.4
ADD https://releases.hashicorp.com/consul-template/${CONSUL_TEMPLATE_VERSION}/consul-template_${CONSUL_TEMPLATE_VERSION}_linux_amd64.zip /tmp/consul-template.zip

RUN unzip /tmp/consul-template.zip -d /usr/bin && \
    chmod +x /usr/bin/consul-template && \
    rm /tmp/consul-template.zip

ENV DOCKER_VERSION 1.13.1
ADD https://get.docker.com/builds/Linux/x86_64/docker-${DOCKER_VERSION}.tgz docker.tgz
RUN tar --extract --file=docker.tgz docker/docker && \
    mv docker/docker /usr/bin/docker && \
    rm -Rf docker && \
    chmod +x /usr/bin/docker

COPY ctmpl /etc/ctmpl

ENTRYPOINT ["/usr/bin/consul-template"]
