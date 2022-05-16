# POC - Preserve Source IP behind an OVH LB with Istio/Envoy

---

## Prerequisites

I'll use the IstioOperator to install Istio. So you'll need
the [istioctl](https://istio.io/latest/docs/ops/diagnostic-tools/istioctl/).

## Step-by-step installation

First install the operator in your cluster.

```shell
istioctl operator init
```

When this is done, you can apply a simple Istio configuration.

```shell
kubectl apply -f istio.yaml
```

> Please go read this file ! All the things that are important for this POC are preceded by a comment.

Then, we create two EnvoyFilters so that Envoy knows that we are using the proxy protocol.

```shell
kubectl apply -f envoy-filters.yaml
```

## Testing

We'll use the same testing
method [this guide](https://docs.ovh.com/gb/en/kubernetes/getting-source-ip-behind-loadbalancer/).

```shell
kubectl apply -f echo.yaml
```

Wait for the LoadBalancer service to give you an IP address.
Now you can test it using the LoadBalancer URL:

```shell
curl  xxx.xxx.xxx.xxx
```

And you should get the HTTP parameters of your request, including the right source IP in the x-forwarded-for header:
> N.B. : xx-xx-xx-xx is the LoadBalancer IP address. zz.zz.zz.zz should be your IP address.
```json
{
  "path": "/",
  "headers": {
    "host": "ip-xx-xx-xx-xx.sbg.lb.ovh.net",
    "x-forwarded-for": "zz.zz.zz.zz",
    "x-forwarded-proto": "http",
    "x-envoy-external-address": "zz.zz.zz.zz",
    "x-request-id": "1d8ba138-eea9-9d82-93f3-b012df84483e",
    "x-envoy-decorator-operation": "echo-service.echo.svc.cluster.local:80/*",
    "x-envoy-peer-metadata": "[REDACTED]",
    "x-envoy-peer-metadata-id": "[REDACTED]",
    "x-envoy-attempt-count": "1",
    "x-b3-traceid": "f5402345555336b06ffb989b95ee8648",
    "x-b3-spanid": "6ffb989b95ee8648",
    "x-b3-sampled": "1"
  },
  "method": "GET",
  "body": "",
  "fresh": false,
  "hostname": "ip-xx-xx-xx-xx.sbg.lb.ovh.net",
  "ip": "zz.zz.zz.zz",
  "ips": [
    "zz.zz.zz.zz"
  ],
  "protocol": "http",
  "query": {},
  "subdomains": [
    "lb",
    "sbg",
    "ip-xx-xx-xx-xx"
  ],
  "xhr": false,
  "os": {
    "hostname": "echo-deployment-b97d6c86f-cgcwj"
  },
  "connection": {}
}
```
