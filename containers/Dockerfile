FROM alpine:edge AS git
ARG TAG
ENV TAG=${TAG:-main}
RUN apk update && apk add --no-cache \
	git
WORKDIR /tmp/
RUN git -c advice.detachedHead=false clone  --branch $TAG --depth=1 --recurse-submodules https://github.com/pfm-powerforme/vaultwarden-web.git vaultwarden-web
WORKDIR /tmp/vaultwarden-web/


FROM node:22-alpine AS builder
ARG TAG
ENV TAG=${TAG:-main}
ENV VAULT_FOLDER=bw_clients
RUN apk update && apk add --no-cache \
	gcc musl-dev g++ make py3-pip \
    git
WORKDIR /bw_web_builds
COPY --from=git /tmp/vaultwarden-web/ bw_clients/
WORKDIR /bw_web_builds/bw_clients/
RUN export VAULT_VERSION=$(git rev-parse HEAD)
RUN npm ci
WORKDIR /bw_web_builds/bw_clients/apps/web
RUN npm run dist:oss:selfhost
RUN printf '{"version":"%s"}'  $(echo $TAG | grep -Eo '[^\/v]*$') > build/vw-version.json


FROM scratch AS runtime
COPY --from=builder /bw_web_builds/bw_clients/apps/web/build/ /web-vault
CMD [""]