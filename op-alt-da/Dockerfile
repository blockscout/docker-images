ARG OP_ALT_DA_VERSION=0.16.0

FROM alpine:3.24

ARG OP_ALT_DA_VERSION

RUN apk add --no-cache ca-certificates wget curl && update-ca-certificates && \
    curl -fsSL https://github.com/celestiaorg/op-alt-da/releases/download/v${OP_ALT_DA_VERSION}/op-alt-da_${OP_ALT_DA_VERSION}_linux_amd64.apk -o op-alt-da.apk && \
    apk add --allow-untrusted op-alt-da.apk

EXPOSE 3100
ENTRYPOINT ["/usr/local/bin/da-server"]