FROM node:18.16.0 as builder

# Build frontend code which added upload/download button
WORKDIR /app
COPY html/package.json /app/
COPY html/yarn.lock /app/
RUN yarn config set network-timeout 100000
RUN yarn install
COPY html/ /app/
RUN yarn run build

# https://smartx1.slack.com/archives/C037V8QMKQA/p1727253372351649?thread_ts=1727244195.938559&cid=C037V8QMKQA
FROM docker.io/smartelf/helm-kubectl:1.25.16
SHELL [ "/bin/bash", "-c" ]

ARG TARGETARCH
ARG TTYD_VERSION=1.7.4
ARG KUBECOLOR_VERSION=0.4.0
ARG KUSTOMIZE_VERSION=5.5.0

RUN echo "https://mirrors.aliyun.com/alpine/v3.18/main/" >> /etc/apk/repositories \
    && apk update \
    && apk add wget gcc g++ make vim \
    && wget https://ohse.de/uwe/releases/lrzsz-0.12.20.tar.gz \
    && tar -xf lrzsz-0.12.20.tar.gz \
    && cd lrzsz-0.12.20 \
    && ./configure \
    && make \
    && make install \
    && apk add --no-cache ca-certificates \
    && ln -s /usr/local/bin/lrz	/usr/bin/rz \
    && ln -s /usr/local/bin/lsz	/usr/bin/sz \
    && apk add bash-completion \
    && source /usr/share/bash-completion/bash_completion \
    && echo 'source <(kubectl completion bash)' >>~/.bashrc \
    && echo 'alias k=kubectl' >>~/.bashrc \
    && echo 'alias kubectl=kubecolor' >> ~/.bashrc \
    && echo 'complete -o default -F __start_kubectl kubecolor' >> ~/.bashrc \
    && echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc \
    && source ~/.bashrc

RUN wget https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize/v${KUSTOMIZE_VERSION}/kustomize_v${KUSTOMIZE_VERSION}_linux_${TARGETARCH}.tar.gz \
    && tar -xvf kustomize_v${KUSTOMIZE_VERSION}_linux_${TARGETARCH}.tar.gz \
    && mv kustomize /usr/local/bin/kustomize \
    && chmod +x /usr/local/bin/kustomize \
    && kustomize version

RUN curl -LO https://github.com/kvaps/kubectl-node-shell/raw/master/kubectl-node_shell \
    && chmod +x ./kubectl-node_shell \
    && mv ./kubectl-node_shell /usr/local/bin/kubectl-node_shell

RUN wget https://github.com/kubecolor/kubecolor/releases/download/v${KUBECOLOR_VERSION}/kubecolor_${KUBECOLOR_VERSION}_linux_${TARGETARCH}.tar.gz \
    && tar xf kubecolor_${KUBECOLOR_VERSION}_linux_${TARGETARCH}.tar.gz -C /usr/local/bin/ kubecolor

RUN ttydArch="$(uname -m)" && echo "Building arch of $ttydArch.." \
    && curl -LO https://github.com/tsl0922/ttyd/releases/download/${TTYD_VERSION}/ttyd.${ttydArch} \
    && chmod +x ttyd.${ttydArch} \
    && mv ttyd.${ttydArch} /usr/local/bin/ttyd \
    && which ttyd

COPY --from=builder /app/dist/inline.html /usr/lib/ttyd/index.html
COPY ./docker/cloudshell/script/reset.sh  /usr/lib/ttyd/
COPY ./docker/cloudshell/script/startup.sh  /usr/lib/ttyd/

RUN mkdir /root/.kube \
    && chmod +x /usr/lib/ttyd/*.sh \
    && echo 'source /root/.env' >>~/.bashrc \
    && echo '' > .env

USER root
WORKDIR /root