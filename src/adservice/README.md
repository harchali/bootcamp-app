# Ad Service

The Ad service provides advertisement based on context keys. If no context keys are provided then it returns random ads.

## Building locally

The Ad service uses gradlew to compile/install/distribute. Gradle wrapper is already part of the source code. To build Ad Service, run:

```
./gradlew installDist
```
It will create executable script src/adservice/build/install/hipstershop/bin/AdService

### Upgrading gradle version
If you need to upgrade the version of gradle then run

```
./gradlew wrapper --gradle-version <new-version>
```

## Building for OPS

You need OpenJDK 8 to run this app.

Run the following gradle commands to install dependencies
```
./gradlew downloadRepos
./gradlew installDist
```

Then install the profiler
```
mkdir -p /opt/cprof && \
wget -q -O- https://storage.googleapis.com/cloud-profiler/java/latest/profiler_java_agent.tar.gz \
| tar xzv -C /opt/cprof && \
rm -rf profiler_java_agent.tar.gz
```

And finally, install the health probe
```
GRPC_HEALTH_PROBE_VERSION=v0.4.11 && \
wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
chmod +x /bin/grpc_health_probe
```

Default port is `9555` and the starting script is `build/install/hipstershop/bin/AdService`.