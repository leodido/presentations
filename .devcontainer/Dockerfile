FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

# git-lfs is required because slide PDFs are tracked with Git LFS
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends git-lfs \
    && apt-get clean && rm -rf /var/lib/apt/lists/*
