# Timeout

This policy enables Kuma to set timeouts on the outbound connections depending on the protocol.

## Usage

Specify the proxy to configure with the `sources` selector, and the outbound connections from the proxy with the `destinations` selector.

The policy lets you configure timeouts for `HTTP`, `GRPC`, and `TCP` protocols.

## Example

:::: tabs :options="{ useUrlFragment: false }"
::: tab "Kubernetes"
```yaml
apiVersion: kuma.io/v1alpha1
kind: Timeout
mesh: default
metadata:
  name: timeouts-backend
spec:
  sources:
    - match:
        kuma.io/service: '*'
  destinations:
    - match:
        kuma.io/service: 'backend'
  conf:
    # connectTimeout defines time to establish connection, 'connect_timeout' on Cluster, default 10s
    connectTimeout: 10s
    tcp:
      # 'idle_timeout' on TCPProxy, disabled by default
      idleTimeout: 1h
    http:
      # 'timeout' on Route, disabled by default
      requestTimeout: 5s
      # 'idle_timeout' on Cluster, disabled by default
      idleTimeout: 1h
    grpc:
      # 'stream_idle_timeout' on HttpConnectionManager, disabled by default
      streamIdleTimeout: 5m
      # 'max_stream_duration' on Cluster, disabled by default
      maxStreamDuration: 30m
```
We will apply the configuration with `kubectl apply -f [..]`.
:::

::: tab "Universal"
```yaml
type: Timeout
mesh: default
name: timeouts-backend
sources:
  - match:
      kuma.io/service: '*'
destinations:
  - match:
      kuma.io/service: 'backend'
conf:
  # connectTimeout defines time to establish connection, 'connect_timeout' on Cluster, default 10s
  connectTimeout: 10s
  tcp:
    # 'idle_timeout' on TCPProxy, disabled by default
    idleTimeout: 1h
  http:
    # 'timeout' on Route, disabled by default
    requestTimeout: 5s
    # 'idle_timeout' on Cluster, disabled by default
    idleTimeout: 1h
  grpc:
    # 'stream_idle_timeout' on HttpConnectionManager, disabled by default
    streamIdleTimeout: 5m
    # 'max_stream_duration' on Cluster, disabled by default
    maxStreamDuration: 30m
```
We will apply the configuration with `kumactl apply -f [..]` or via the [HTTP API](/docs/1.1.2/documentation/http-api).
:::
