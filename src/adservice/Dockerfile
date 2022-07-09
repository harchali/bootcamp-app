FROM openjdk:8-slim as builder

WORKDIR /app

COPY gradle gradle
COPY gradlew .
COPY build.gradle .
RUN ./gradlew downloadRepos

COPY . .
RUN ./gradlew installDist

FROM openjdk:8-jre-slim

EXPOSE 9555/tcp

RUN apt-get -y update && apt-get install -y wget && rm -rf /var/lib/apt/lists/*
RUN mkdir -p /opt/cprof && \
    wget -q -O- https://storage.googleapis.com/cloud-profiler/java/latest/profiler_java_agent.tar.gz \
    | tar xzv -C /opt/cprof && \
    rm -rf profiler_java_agent.tar.gz

RUN GRPC_HEALTH_PROBE_VERSION=v0.4.11 && \
    wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe

WORKDIR /app
COPY --from=builder /app .

ENTRYPOINT ["/app/build/install/hipstershop/bin/AdService"]
