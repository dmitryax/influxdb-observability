#syntax=docker/dockerfile:1.2
FROM golang:1.17-alpine AS builder

COPY . /go/src/jaeger-query-influxdb
WORKDIR /go/src/jaeger-query-influxdb

RUN \
  --mount=type=cache,id=jaeger-influxdb-gocache,sharing=locked,target=/root/.cache/go-build \
  --mount=type=cache,id=jaeger-influxdb-gomodcache,sharing=locked,target=/go/pkg/mod \
  du -cshx /root/.cache/go-build /go/pkg/mod && \
  go install ./cmd/jaeger-query-influxdb && \
  du -cshx /root/.cache/go-build /go/pkg/mod

FROM alpine:3.14

COPY --from=iox:local /usr/bin/influxdb_iox /usr/bin/influxdb_iox
COPY --from=jaegertracing/jaeger-query:1.28 /go/bin/query-linux /usr/bin/jaeger-query
COPY --from=builder /go/bin/jaeger-query-influxdb /usr/bin/jaeger-query-influxdb

ENV SPAN_STORAGE_TYPE grpc-plugin
ENV GRPC_STORAGE_PLUGIN_BINARY /usr/bin/jaeger-query-influxdb

RUN apk add --no-cache runit
RUN mkdir -p /etc/runit && touch /etc/runit/stopit && chmod 0 /etc/runit/stopit

#ENTRYPOINT /usr/bin/jaeger-query
#CMD runit-init
