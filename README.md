# TcpTunnel

TcpTunnel is a program implemented in C# (.NET 7.0) that allows to tunnel TCP connections through a server (gateway) to a remote machine,
for example to access services that are running behind a firewall or NAT.

A working configuration consists of three nodes:
- **Gateway**: Runs on a (server) machine that is accessible for both proxy endpoints (e.g. on a public server).
  It listens for incoming TCP connections from the proxy endpoints (proxy-client and proxy-server) and forwards
  data from one proxy endpoint to the corresponding partner proxy endpoint.
- **Proxy-Server**: Connects to the **Gateway** and listens for incoming TCP connections on previously configured
  ports. When a connection arrives, it forwards it to the **Gateway**, which in turn forwards the connection to
  the **Proxy-Client**.
- **Proxy-Client**: Connects to the **Gateway** and waits for forwarded connections received through the
  **Gateway** from the **Proxy-Server**. When receiving such a forwarded connection, it opens a TCP connection
  to the specified endpoint and forwards the data to it.

For example, imagine you have some TCP service (like a VNC server) running on a machine within a LAN that
has internet access (maybe only through NAT so it's not possible to use port forwarding or a VPN), and you
want to securely connect to this service from a machine on another network.
Additionally, you have a server (e.g. VPS) with a publicly domain and you have a SSL/TLS certificate for it.

In this case, you can use the TcpTunnel with the following configuration:
- Run the **Gateway** on the VPS server, configure it to listen at a specific TCP port using SSL/TLS (SSL/TLS
  for the Gateway is currently only supported on **Windows**), and to allow a session with an ID and password.
- Run the **Proxy-Client** on the machine that has access to the TCP service (VNC server), configuring it to
  connect to the host and port of the Gateway.
- Run the **Proxy-Server** on your machine where you want to connect to the TCP service (VNC server), configuring
  it to connect to the host and port of the Gateway, and to listen on a specific TCP port (like 5920) that
  should get forwarded to the Proxy-Client to a specific host and TCP port (like localhost:5900).

The following image illustrates this approach:
![](tcptunnel-illustration.png)

## Configuration
The TcpTunnel is configured via an XML file using the name `settings.xml`. When building the application,
sample settings will get copied to the output directory which you can use as a template.

## Features:
- Uses async I/O for high scalability.
- Supports SSL/TLS (currently on Windows only) and password authentication for the gateway connections.
- Multiplexes multiple (tunneled) TCP connections over a single one.
- Uses a window for flow control for tunneled connections.
- Can be installed as service on Windows.

## Building:
- Install the [.NET 7.0 SDK](https://dotnet.microsoft.com/download) or higher.
- On Windows, you can use one of the `PUBLISH-xyz.cmd` files to publish the app, either as self-contained app
  (with native AOT compilation), or as framework-dependent app (so it needs the .NET Runtime to be installed).
- Otherwise, you can publish for the current platform with the following command (as framework-dependent app): 
  ```
  dotnet publish "TcpTunnel/TcpTunnel.csproj" -f net7.0 -c Release -p:PublishSingleFile=true --no-self-contained
  ```

## Development TODOs:
- Support SSL/TLS certificates from a certificate file (.pfx), so that it can be used with the gateway under Linux.
- Use a different password storage mechanism so that they don't have to be specified in cleartext in the
  XML settings file.
- Add more documentation.
- Support installing/running as service on Linux e.g. via `systemd`.