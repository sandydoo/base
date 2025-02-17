#--------------------------------------
# Image: base
#--------------------------------------
FROM ghcr.io/containerbase/ubuntu:20.04 AS base

RUN touch /.dummy

ARG APT_HTTP_PROXY
ARG CONTAINERBASE_CDN
ARG CONTAINERBASE_DEBUG
ARG CONTAINERBASE_LOG_LEVEL

ENV BASH_ENV=/usr/local/etc/env
SHELL ["/bin/bash" , "-c"]

ARG TARGETARCH

COPY dist/docker/ /
COPY dist/cli/containerbase-cli-${TARGETARCH} /usr/local/containerbase/bin/containerbase-cli

RUN install-containerbase

COPY --chown=1000:0 test/dotnet/test test
WORKDIR /test

#--------------------------------------
# build: dotnet 6.0 base image (LTS)
#--------------------------------------
FROM base AS build

RUN install-tool dotnet 6.0.416
RUN set -ex; dotnet --version | grep 6.0.

RUN ls -la $USER_HOME/.nuget/NuGet

USER 1000

RUN dotnet --info

SHELL [ "/bin/sh", "-c" ]
RUN dotnet --info

#--------------------------------------
# test: dotnet 6.0 (LTS)
#--------------------------------------
FROM build AS testa

RUN set -ex; \
    dotnet restore --use-lock-file

RUN set -ex;  \
    dotnet add package Newtonsoft.Json --version 12.0.3; \
    dotnet restore --force-evaluate


#--------------------------------------
# test: dotnet 8.0 (LTS)
#--------------------------------------
FROM base AS testb

# Do not change
RUN install-tool dotnet 6.0.416

# renovate: datasource=dotnet packageName=dotnet-sdk
RUN install-tool dotnet 8.0.303

# Test duplicate install
# renovate: datasource=dotnet packageName=dotnet-sdk
RUN install-tool dotnet 8.0.303
RUN set -ex; dotnet --version | grep 8.0.

USER 1000

RUN dotnet --info


RUN set -ex; \
    dotnet restore --use-lock-file

RUN set -ex;  \
    dotnet add package Newtonsoft.Json --version 12.0.3; \
    dotnet restore --force-evaluate; \
    dotnet build

#--------------------------------------
# test: dotnet 8.0 (non-root, LTS)
#--------------------------------------
FROM build AS testc

# only patch updates
# renovate: datasource=dotnet packageName=dotnet-sdk
RUN install-tool dotnet 8.0.303
RUN set -ex; dotnet --version | grep 8.0.

RUN set -ex; \
    dotnet restore --use-lock-file

RUN set -ex;  \
    dotnet add package Newtonsoft.Json --version 12.0.3; \
    dotnet restore --force-evaluate


#--------------------------------------
# test: dotnet latest (non-root)
#--------------------------------------
FROM build AS testd

# only patch updates
# renovate: datasource=dotnet packageName=dotnet-sdk
RUN install-tool dotnet 8.0.303
RUN set -ex; dotnet --version | grep 8.0.

RUN set -ex; \
    dotnet restore --use-lock-file

RUN set -ex;  \
    dotnet add package Newtonsoft.Json --version 12.0.3; \
    dotnet restore --force-evaluate

#--------------------------------------
# final
#--------------------------------------
FROM base

COPY --from=testa /.dummy /.dummy
COPY --from=testb /.dummy /.dummy
COPY --from=testc /.dummy /.dummy
# TODO: enable for net 9.0
# COPY --from=testd /.dummy /.dummy
