# Pinned so a rebuild cannot silently change the Alpine release.
ARG BASE_IMAGE=alpine:3.24
# Go 1.27.3 fixes CVE-2026-111943 in crypto/x509. go.mod needs 1.25.3 or later.
FROM golang:1.27.3-alpine AS builder
ARG VERSION=latest

COPY  . /go/src/github.com/zalando/postgres-operator
WORKDIR /go/src/github.com/zalando/postgres-operator

RUN GO111MODULE=on go mod vendor \
    && CGO_ENABLED=0 go build -o build/postgres-operator -v -ldflags "-X=main.version=${VERSION}" cmd/main.go

FROM ${BASE_IMAGE}
LABEL maintainer="Team ACID @ Zalando <team-acid@zalando.de>"
LABEL org.opencontainers.image.source="https://github.com/zalando/postgres-operator"

# We need root certificates to deal with teams api over https
RUN apk -U upgrade --no-cache \
    && apk add --no-cache curl ca-certificates

# CVE-2026-111497: fail the build unless the base carries the fixed brotli-libs
RUN apk version -t "$(apk list -I brotli-libs | cut -d' ' -f1 | sed 's/^brotli-libs-//')" 1.2.1-r0 | grep -q '[=>]'

COPY --from=builder /go/src/github.com/zalando/postgres-operator/build/* /

RUN addgroup -g 1000 pgo
RUN adduser -D -u 1000 -G pgo -g 'Postgres Operator' pgo

USER 1000:1000

ENTRYPOINT ["/postgres-operator"]
