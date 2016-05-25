FROM debian:jessie
MAINTAINER Nicolas Berthe <nicolas.berthe@alterway.fr>

RUN apt-get update && \
    apt-get install --no-install-recommends --no-install-suggests -y unzip && \
    rm -r /var/lib/apt/lists/*

ENV CONSUL_TEMPLATE_VERSION 0.12.1
ADD https://releases.hashicorp.com/consul-template/${CONSUL_TEMPLATE_VERSION}/consul-template_${CONSUL_TEMPLATE_VERSION}_linux_amd64.zip /tmp/consul-template.zip

RUN unzip /tmp/consul-template.zip -d /usr/bin && \
    chmod +x /usr/bin/consul-template && \
    rm /tmp/consul-template.zip

COPY ctmpl /etc/ctmpl
WORKDIR /etc/ctmpl

ENTRYPOINT ["/usr/bin/consul-template"]
