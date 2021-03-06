FROM ruby:2.4-slim

ENV TERRAFORM_VERSION="0.10.7"
ENV DOCKER_VERSION 1.13.1

COPY ./tests/rspec/Gemfile* /tmp/app/

RUN apt-get update && \
    apt-get install --no-install-recommends -y -q \
    curl make unzip jq awscli wget xvfb xauth ssh openvpn build-essential && \
    apt-get clean

# Install gcloud
RUN echo "deb http://packages.cloud.google.com/apt cloud-sdk-jessie main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && \
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - && \
    apt-get update && \
    apt-get install --no-install-recommends -y -q \
    google-cloud-sdk

RUN cd /tmp/app && bundler install && rm -r /tmp/app

# Install kubectl
RUN curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && \
    chmod +x ./kubectl && \
    mv ./kubectl /bin/kubectl

# Install Terraform
RUN curl -L https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip | funzip > /usr/local/bin/terraform && chmod +x /usr/local/bin/terraform

# Install Chrome for installer gui tests
# Use Chrome beta because v60 or higher is needed for headless mode
RUN wget --quiet -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - && \
    sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list' && \
    apt-get update && \
    apt-get install --no-install-recommends -y -q \
    google-chrome-beta ca-certificates

# Install docker client to start k8s conformance test docker container
RUN curl https://get.docker.com/builds/Linux/x86_64/docker-${DOCKER_VERSION}.tgz | tar -xvz && \
    mv docker/docker /usr/local/bin/docker && \
    chmod +x /usr/local/bin/docker && \
    rm -r docker

# Add user jenkins (uid: 1000), this is needed among others for `ssh` to not
# complain about missing user.
RUN useradd -u 1000 -ms /bin/bash jenkins
