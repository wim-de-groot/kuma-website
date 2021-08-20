# Configure proxy policies

## Structure of a policy

Dataplane proxy policies all follow the same basic structure:

```yaml
sources:
- match:
    kuma.io/service: ... # unique name OR '*'
    ... # (optionally) other tags

destinations:
- match:
    kuma.io/service: ... # unique name OR '*'
    ... # (optionally) other tags

conf:
  ... # policy-specific configuration
```

* `sources` - list of selectors that specify the dataplane objects where network traffic originates
* `destinations` - list of selectors that specify the dataplane object the source traffic is sent to 
* `conf` - configuration to apply to network traffic between `sources` and `destinations`

Kuma assumes that every dataplane object represents a service, even if it's a cron job that doesn't normally handle incoming traffic. This means the `service` tag is required for `sources` and `destinations`. Wildcard values are supported.

All policies support arbitrary tags in `sources` selectors, there are tag limitations for `destinations` selectors. For example, policies that are applied on the client side of a connection between two dataplane objects do not support arbitrary tags in the `destinations` selector. Only the `service` tag is supported in this case. Such policies include `TrafficRoute`, `TrafficLog`, and `HealthCheck`.

For example, this policy applies to all network traffic between all dataplane objects:

```yaml
sources:
- match:
    kuma.io/service: '*'

destinations:
- match:
    kuma.io/service: '*'

conf:
  ...
```

This policy applies only to network traffic between dataplane objects for the specified services:

```yaml
sources:
- match:
    kuma.io/service: web

destinations:
- match:
    kuma.io/service: backend

conf:
  ...
```

You can also provide additional tags to further limit policy scope:

```yaml
sources:
- match:
    kuma.io/service: web
    cloud:   aws
    region:  us

destinations:
- match:
    kuma.io/service: backend
    version: v2      # not all policies support arbitrary tags in `destinations` selectors

conf:
  ...
```

