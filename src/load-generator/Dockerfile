FROM --platform=$BUILDPLATFORM golang:1.25-alpine AS builder
ARG TARGETPLATFORM
ARG TARGETOS
ARG TARGETARCH
WORKDIR /build
RUN apk --no-cache add git
# Pin xk6 to last Go-1.25-compatible release; xk6 >=v1.4.13 requires Go 1.26 (ISI-6772 CI fix / C1 flag)
RUN CGO_ENABLED=0 go install go.k6.io/xk6/cmd/xk6@v1.4.12
RUN CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} xk6 build \
    --output /k6 \
    --with github.com/henrikrexed/henrikrexed-xk6-output-opentelemetry@latest \
    --with github.com/Dynatrace/xk6-output-dynatrace@latest

FROM alpine:3.21
RUN apk --no-cache add ca-certificates && \
    adduser -D -u 12345 k6

COPY --from=builder /k6 /usr/bin/k6
COPY test.js /scripts/test.js
COPY run.sh /scripts/run.sh

USER root
RUN chmod +x /scripts/run.sh
USER k6

ENV FRONTEND_URL=http://frontend:8080
ENV K6_VUS=5
ENV K6_DURATION=87600h
ENV K6_OUTPUT_MODE=opentelemetry
ENV K6_OTEL_GRPC_EXPORTER_INSECURE=true
ENV K6_OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
ENV OTEL_SERVICE_NAME=load-generator

ENTRYPOINT ["/scripts/run.sh"]
