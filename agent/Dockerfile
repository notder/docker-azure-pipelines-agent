FROM ubuntu:18.04
# basics:
RUN apt-get update && \
    apt-get install -y software-properties-common apt-transport-https && \
    apt-get upgrade -y && \
    apt-add-repository ppa:git-core/ppa && \
    apt-get update && \
    apt-get install -y git iproute2 libuuid1 libicu60 wget curl zip unzip \
    vim  libunwind8 sudo iputils-ping gdebi-core python python-pip python3 python3-pip \
    build-essential libssl-dev
#users
RUN adduser --disabled-password --gecos '' agentuser && \
    adduser agentuser sudo && \
    echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
RUN mkdir /agent
WORKDIR /agent
# Microsoft keys:
RUN wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb && \
    dpkg -i packages-microsoft-prod.deb && \
    rm packages-microsoft-prod.deb
# netcore
RUN apt-get update && apt-get install -y dotnet-sdk-2.1
ENV PATH="${PATH}:/home/agentuser/.dotnet/tools"
# az cli:
RUN echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $(lsb_release -cs) main" | \
    tee /etc/apt/sources.list.d/azure-cli.list && \
    curl -L https://packages.microsoft.com/keys/microsoft.asc | apt-key add - && \
    apt-get update && \
    apt-get install -y azure-cli
# pwsh:
RUN apt-get install -y powershell
# nvm/node/yarn:
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash
RUN apt-get install -y nodejs
RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
RUN [ "/bin/bash", "-c", "source $HOME/.nvm/nvm.sh && nvm i 10" ]
RUN [ "/bin/bash", "-c", "source $HOME/.nvm/nvm.sh && nvm alias default 10" ]
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
    apt-get update && apt-get -y --no-install-recommends install yarn
# chrome:
RUN wget -q https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb && \
    gdebi --non-interactive google-chrome-stable_current_amd64.deb && \
    rm google-chrome-stable_current_amd64.deb
# kubectl:
RUN curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - && \
    echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | tee -a /etc/apt/sources.list.d/kubernetes.list && \
    apt-get update && \
    apt-get install -y kubectl
# helm:
RUN curl -sL https://raw.githubusercontent.com/helm/helm/master/scripts/get | bash
# terraform:
RUN wget -q https://releases.hashicorp.com/terraform/0.11.8/terraform_0.11.8_linux_amd64.zip && \
    unzip -d /usr/local/bin/ terraform_0.11.8_linux_amd64.zip && \
    rm terraform_0.11.8_linux_amd64.zip
# agent:
RUN curl -sOSL https://vstsagentpackage.azureedge.net/agent/2.141.0/vsts-agent-linux-x64-2.141.0.tar.gz && \
    tar xzf /agent/vsts-agent-linux-x64-2.141.0.tar.gz && \
    rm /agent/vsts-agent-linux-x64-2.141.0.tar.gz
COPY configureAgent.sh runAgent.sh configureAndRun.sh /agent/
RUN chmod +x configureAndRun.sh configureAgent.sh runAgent.sh && \
    chown agentuser:agentuser configureAgent.sh runAgent.sh configureAndRun.sh
RUN chown -R agentuser:agentuser /agent
USER agentuser
CMD [ "/agent/configureAndRun.sh" ]