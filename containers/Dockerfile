FROM docker.io/library/alpine:edge AS git
ARG TAG
ENV TAG=${TAG:-main}
RUN apk update && apk add --no-cache \
	git
WORKDIR /
RUN git -c advice.detachedHead=false clone --branch $TAG --depth=1 --recurse-submodules https://github.com/pfm-powerforme/vaultwarden-frontend.git source
WORKDIR /source/


FROM docker.io/library/node:lts-alpine AS builder
ARG TAG
ENV TAG=${TAG:-main}
ENV VAULT_FOLDER=bw_clients
RUN apk update && apk add --no-cache \
	gcc musl-dev g++ make py3-pip \
    git
WORKDIR /
COPY --from=git /source/ build/
WORKDIR /build/
RUN export VAULT_VERSION=$(git rev-parse HEAD)
RUN npm ci
WORKDIR /build/apps/web
RUN npm run dist:oss:selfhost
RUN printf '{"version":"%s"}'  $(echo $TAG | grep -Eo '[^\/v]*$') > build/vw-version.json


FROM scratch AS runtime
COPY --from=builder /build/apps/web/build/ /frontend/
CMD [""]