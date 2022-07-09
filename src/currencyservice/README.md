# currencyservice

This is a nodeJS app for currency management.

## Installing dependencies

Note: you will need python3, make and g++ to install the dependencies correctly.

```bash
# Execute this where package.json and package.lock files are available
npm install # For prod, add --only=production option
```

## Executing

Add the GRPC health probe
```
GRPC_HEALTH_PROBE_VERSION=v0.4.11 && \
wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
chmod +x /bin/grpc_health_probe
```

Execute the command `node server.js`
By default, it listens on port 7000