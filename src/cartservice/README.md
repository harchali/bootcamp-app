# cartservice

Handle the cart.

## Building for OPS

You need Dotnet 6 to run this app. SDK for building, and only deps for running.

Run the following dotnet commands to install dependencies and build
```
dotnet restore cartservice.csproj -r linux-musl-x64
dotnet publish cartservice.csproj -p:PublishSingleFile=true -r linux-musl-x64 --self-contained true -p:PublishTrimmed=True -p:TrimMode=Link -c release -o /cartservice --no-restore
```
This will generate a single, self-contained binary named `cartservice`.


In the running env, install the health probe
```
GRPC_HEALTH_PROBE_VERSION=v0.4.11
wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64
chmod +x /bin/grpc_health_probe
```

Env vars
```
# Choose the port it needs to listen to, default is 7070
ENV ASPNETCORE_URLS http://*:<PORT>
ENV DOTNET_EnableDiagnostics=0
```

To start the app, run `./cartservice` binary.