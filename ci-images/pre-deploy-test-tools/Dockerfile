FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive
WORKDIR /tools

# ---- base deps ----
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
      ca-certificates curl wget jq tar gzip xz-utils python3-pip coreutils gnupg && \
    pip install --no-cache-dir yamllint && \
    rm -rf /var/lib/apt/lists/*

# ---- versions as build args (pinned) ----
ARG KUBECTL_VERSION=v1.30.3
ARG KUSTOMIZE_VERSION=v5.4.3
ARG KUBECONFORM_VERSION=v0.6.7
ARG CONFTEST_VERSION=v0.57.0
ARG KUBE_SCORE_VERSION=v1.18.0
ARG YQ_VERSION=v4.44.2
ARG TRIVY_VERSION=v0.55.2

# helper for verified install
SHELL ["/bin/bash", "-euxo", "pipefail", "-c"]

# kubectl (official release + sha256)
RUN curl -sSLo /usr/local/bin/kubectl https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl && \
    curl -sSL https://dl.k8s.io/${KUBECTL_VERSION}/bin/linux/amd64/kubectl.sha256 -o /tmp/k.sha256 && \
    echo "$(cat /tmp/k.sha256)  /usr/local/bin/kubectl" | sha256sum -c - && \
    chmod +x /usr/local/bin/kubectl && rm -f /tmp/k.sha256

# kustomize
RUN curl -sSL -o /tmp/kustomize.tgz \
      https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2F${KUSTOMIZE_VERSION}/kustomize_${KUSTOMIZE_VERSION}_linux_amd64.tar.gz && \
    tar -xzf /tmp/kustomize.tgz -C /usr/local/bin && rm -f /tmp/kustomize.tgz

# kubeconform
RUN curl -sSL -o /tmp/kubeconform.tgz \
      https://github.com/yannh/kubeconform/releases/download/${KUBECONFORM_VERSION}/kubeconform-linux-amd64.tar.gz && \
    tar -xzf /tmp/kubeconform.tgz -C /usr/local/bin && rm -f /tmp/kubeconform.tgz

# conftest
RUN V="${CONFTEST_VERSION#v}"; \
    curl -sSL -o /tmp/conftest.tgz \
      https://github.com/open-policy-agent/conftest/releases/download/${CONFTEST_VERSION}/conftest_${V}_Linux_x86_64.tar.gz && \
    tar -xzf /tmp/conftest.tgz -C /usr/local/bin && rm -f /tmp/conftest.tgz && conftest --version

# kube-score
RUN curl -sSL -o /usr/local/bin/kube-score \
      https://github.com/zegl/kube-score/releases/download/${KUBE_SCORE_VERSION}/kube-score_ubuntu_amd64 && \
    chmod +x /usr/local/bin/kube-score

# yq
RUN curl -sSL -o /usr/local/bin/yq \
      https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64 && \
    chmod +x /usr/local/bin/yq

# Trivy (tarball + checksum)
RUN curl -sSL -o /tmp/trivy.tgz https://github.com/aquasecurity/trivy/releases/download/${TRIVY_VERSION}/trivy_${TRIVY_VERSION#v}_Linux-64bit.tar.gz && \
    curl -sSL -o /tmp/trivy.tgz.sha256sum https://github.com/aquasecurity/trivy/releases/download/${TRIVY_VERSION}/trivy_${TRIVY_VERSION#v}_Linux-64bit.tar.gz.sha256sum && \
    (cd /tmp && sha256sum -c trivy.tgz.sha256sum) && \
    tar -xzf /tmp/trivy.tgz -C /usr/local/bin trivy && \
    rm -f /tmp/trivy.tgz /tmp/trivy.tgz.sha256sum

# minimal oci labels
LABEL org.opencontainers.image.source="https://github.com/<org>/ci-cd-templates" \
      org.opencontainers.image.description="Pre-deploy test tools for K8s manifests" \
      org.opencontainers.image.licenses="MIT"

# non-root user
RUN useradd -m runner && mkdir -p /workspace && chown -R runner:runner /workspace
USER runner
WORKDIR /workspace

# version snapshot for logs
RUN echo '#!/bin/bash
set -e
echo "kubectl: $(kubectl version --client=true --output=yaml | grep gitVersion | awk '\''{print $2}'\'')"
echo "kustomize: $(kustomize version | tr -d \"{}\")."
echo "kubeconform: $(kubeconform -v)"
echo "conftest: $(conftest --version)"
echo "kube-score: $(kube-score version)"
echo "yq: $(yq --version)"
echo "trivy: $(trivy --version | head -n1)"
' > /usr/local/bin/toolbox-versions && chmod +x /usr/local/bin/toolbox-versions

CMD ["bash"]
