FROM python:3.13-slim
# Set by BuildKit per target platform; picks the matching grpc_health_probe binary
# (a hard-coded linux-amd64 one fails with "exec format error" on arm64).
ARG TARGETARCH=amd64

RUN apt-get update && apt-get install -y --no-install-recommends \
    wget && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

# Generate protobuf code from the proto file (copied into build context)
RUN python -m grpc_tools.protoc -Iproto --python_out=. --grpc_python_out=. proto/demo.proto

# Download grpc_health_probe
RUN GRPC_HEALTH_PROBE_VERSION=v0.4.61 && \
    wget -qO/app/grpc_health_probe \
    https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-${TARGETARCH} && \
    chmod +x /app/grpc_health_probe

# Runtime needs neither pip (its vendored urllib3/msgpack carry CVE-2026-97687/97689,
# CVE-2026-114613, GHSA-6v7p-g79w-8964) nor wget; remove both once the build is done.
RUN pip uninstall -y pip && \
    apt-get purge -y --auto-remove wget && \
    rm -rf /var/lib/apt/lists/*

USER 65532:65532

EXPOSE 5050

# Use opentelemetry-instrument for auto-instrumentation wrapping
ENTRYPOINT ["opentelemetry-instrument", "python", "checkout_service.py"]
