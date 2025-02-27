# External Service

This policy allows services running inside the mesh to consume services that are not part of the mesh. The `ExternalService` resource allows you to declare specific external resources by name within the mesh, instead of implementing the default [passthrough mode](/docs/1.1.3/policies/mesh/#controlling-the-passthrough-mode). Passthrough mode allows access to any non-mesh host by specifying its domain name or IP address, without the ability to apply any traffic policies. The `ExternalService` resource enables the same observability, security, and traffic manipulation for external traffic as for services entirely inside the mesh

When you enable this policy, you should also [disable passthrough mode](mesh/#controlling-the-passthrough-mode) for the mesh and enable the [data plane proxy builtin DNS](../networking/dns/#data-plane-proxy-built-in-dns) name resolution.

## Usage

A simple HTTPS external service can be defined:

:::: tabs :options="{ useUrlFragment: false }"
::: tab "Kubernetes"
```yaml
apiVersion: kuma.io/v1alpha1
kind: ExternalService
mesh: default
metadata:
  namespace: default
  name: httpbin
spec:
  tags:
    kuma.io/service: httpbin
    kuma.io/protocol: http
  networking:
    address: httpbin.org:443
    tls: # optional
      enabled: true
      allowRenegotiation: false
      caCert: # one of inline, inlineString, secret
        inline: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t... # Base64 encoded cert
      clientCert: # one of inline, inlineString, secret
        secret: clientCert
      clientKey: # one of inline, inlineString, secret
        secret: clientKey
```

Then apply the configuration with `kubectl apply -f [..]`.

:::

::: tab "Universal"
```yaml
type: ExternalService
mesh: default
name: httpbin
tags:
  kuma.io/service: httpbin
  kuma.io/protocol: http
networking:
  address: httpbin.org:443
  tls:
    enabled: true
    allowRenegotiation: false
    caCert: # one of inline, inlineString, secret
      inline: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t... # Base64 encoded cert
    clientCert: # one of inline, inlineString, secret
      secret: clientCert
    clientKey: # one of inline, inlineString, secret
      secret: clientKey
```

Then apply the configuration with `kumactl apply -f [..]` or with the [HTTP API](/docs/1.2.0/documentation/http-api).

Universal mode is best combined with [transparent proxy](../networking/transparent-proxying/). For backward compatibility only, you can consume an external service from within the mesh by filling the proper `outbound` section of the relevant data plane resource:

```yaml
type: Dataplane
mesh: default
name: redis-dp
networking:
  address: 127.0.0.1
  inbound:
  - port: 9000
    tags:
      kuma.io/service: redis
  outbound:
  - port: 10000
    tags:
      kuma.io/service: httpbin
```

Then `httpbin.org` is accessible at `127.0.0.1:10000`.

:::
::::

### Accessing the External Service

Consuming the defined service from within the mesh for both Kubernetes and Universal deployments (assuming [transparent proxy](../networking/transparent-proxying/)) can be done:

* With the `.mesh` naming of the service `curl httpbin.mesh`. With this approach, specify port 80.
* With the real name and port, in this case `curl httpbin.org:443`. This approach works only with [the data plane proxy builtin DNS](../networking/dns/#data-plane-proxy-built-in-dns) name resolution.

Although the external service is HTTPS, it's consumed as plain HTTP. This is possible because when `networking.tls.enabled` is set to `true` then Envoy is responsible for originating and verifying TLS.

To consume the service using HTTPS, set the service protocol to `kuma.io/protocol: tcp` and `networking.tls.enabled=false`. This way application itself is responsible for originating and verifying TLS and Envoy is just passing the connection to a proper destination.

The first approach has an advantage that we can apply HTTP based policies, because Envoy is aware of HTTP protocol and can apply request modifications before the request is encrypted. Additionally, we can modify TLS certificates without restarting applications.

### Available policy fields

* `tags` the external service can include an arbitrary number of tags, where `kuma.io/service` is mandatory. The special `kuma.io/protocol` tag is also taken into account and supports the standard Kuma protocol values. It designates the specific protocol for the service.
* ` networking` describes the networking configuration of the external service
    * `address` is the address where the external service can be reached.
    * `tls` is the section to configure the TLS originator when consuming the external service
        * `enabled` turns on and off the TLS origination.
        * `allowRenegotiation` turns on and off TLS renegotiation. It's not recommended enabling this for [security reasons](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/transport_sockets/tls/v3/tls.proto).
          However, some servers require this setting to fetch client certificate after TLS handshake. TLS renegotiation is not available in TLS v1.3.
        * `caCert` the CA certificate for the external service TLS verification
        * `clientCert` the client certificate for mTLS
        * `clientKey` the client key for mTLS

As with other services, avoid duplicating service names under `kuma.io/service` with already existing ones. A good practice is to derive the tag value from the domain name or IP of the actual external service.
