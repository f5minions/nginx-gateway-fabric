# syntax=docker/dockerfile:1.6
# this is here so we can grab the latest version of kind and have dependabot keep it up to date
FROM kindest/node:v1.29.1

FROM golang:1.21

WORKDIR /go/src/github.com/nginxinc/nginx-gateway-fabric/conformance/tests/

COPY --link go.mod /go/src/github.com/nginxinc/nginx-gateway-fabric/
COPY --link go.sum /go/src/github.com/nginxinc/nginx-gateway-fabric/
RUN go mod download

COPY --link conformance/tests /go/src/github.com/nginxinc/nginx-gateway-fabric/conformance/tests/
