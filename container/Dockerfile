FROM ubuntu

RUN apt-get update && apt-get -y install curl unzip jq && \
    curl -o packer.zip  https://releases.hashicorp.com/packer/1.0.0/packer_1.0.0_linux_amd64.zip && \
    unzip packer.zip

CMD ["/packer"]
