# checkoutservice

For building the app, we uses the `Dockferfile` at the root of this repository, producing a `checkoutservice` binary.

### To execute the app, on a linux os

Install the GRPC probes in your env
```
GRPC_HEALTH_PROBE_VERSION=v0.4.11 && \
wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
chmod +x /bin/grpc_health_probe
```

Then execute `./checkoutservice`, listening by default on port 5050