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

# test openshift compatibility 1000<>1001
COPY --chown=1001:0 test/erlang/test test
WORKDIR /test

#--------------------------------------
# build
#--------------------------------------
FROM base AS build

RUN prepare-tool erlang

#--------------------------------------
# test: erlang (root)
#--------------------------------------
FROM base AS testa

# https://hexdocs.pm/elixir/1.15.0/compatibility-and-deprecations.html#compatibility-between-elixir-and-erlang-otp
# renovate: datasource=github-releases packageName=containerbase/erlang-prebuild versioning=docker
RUN install-tool erlang 27.0.0.0

# renovate: datasource=docker versioning=docker
RUN install-tool elixir 1.17.2


USER 1001

RUN set -ex; \
    cd a; \
    mix deps.update --all;


#--------------------------------------
# test: erlang (v24,user,openshift)
#--------------------------------------
FROM build AS testb

USER 1001

# https://hexdocs.pm/elixir/1.16.0/compatibility-and-deprecations.html#compatibility-between-elixir-and-erlang-otp
RUN install-tool erlang 24.3.4.17

RUN install-tool elixir 1.16.3


RUN set -ex; \
    cd a; \
    mix deps.update --all;


#--------------------------------------
# test: erlang (v22,user,openshift)
#--------------------------------------
FROM build AS testc

USER 1001

# https://hexdocs.pm/elixir/1.13.4/compatibility-and-deprecations.html#compatibility-between-elixir-and-erlang-otp
RUN install-tool erlang 22.3.4.27

RUN install-tool elixir 1.13.4

RUN set -ex; \
    cd a; \
    mix deps.update --all;

#--------------------------------------
# final
#--------------------------------------
FROM base

COPY --from=testa /.dummy /.dummy
COPY --from=testb /.dummy /.dummy
COPY --from=testc /.dummy /.dummy
