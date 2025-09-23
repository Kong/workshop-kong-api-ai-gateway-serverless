---
title : "Import an OpenAPI specification"
weight : 122
---


For our first Kong Service, we are going to import an OpenAPI spec (OAS) to our Control Plane. The Control Plane will convert the spec into Kong Service and Kong Routes.

* Download the [bankong.yaml](/code/bankong.yaml) spec.
* In your Control Plane, click on **Import via OAS spec**.
* Choose the ``bankong.yaml`` spec and click **Continue**.
* Review the Import Summary and click Import
* Notice the Services and Routes that will be imported
* Notice declarative representation of this import as well (more on this later)


### Test the API Deployment

```
curl $DATA_PLANE_URL/transactions
```


You should get a response from the API
Notice the x-kong-* headers like request id, proxy latency, upstream latency
Run the request again, what do you notice about the proxy latency now?
 



#### Consume the Route

We are to use the same ELB provisioned during the Data Plane deployment:

{{<highlight>}}
curl -v $DATA_PLANE_URL/httpbin-route/get
{{</highlight>}}

If successful, you should see the **httpbin** output:

```
* Host kong-cceb6a93c9usmc2hk.kongcloud.dev:443 was resolved.
* IPv6: (none)
* IPv4: 66.51.127.198
*   Trying 66.51.127.198:443...
* Connected to kong-cceb6a93c9usmc2hk.kongcloud.dev (66.51.127.198) port 443
* ALPN: curl offers h2,http/1.1
* (304) (OUT), TLS handshake, Client hello (1):
*  CAfile: /etc/ssl/cert.pem
*  CApath: none
* (304) (IN), TLS handshake, Server hello (2):
* (304) (IN), TLS handshake, Unknown (8):
* (304) (IN), TLS handshake, Certificate (11):
* (304) (IN), TLS handshake, CERT verify (15):
* (304) (IN), TLS handshake, Finished (20):
* (304) (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / AEAD-CHACHA20-POLY1305-SHA256 / [blank] / UNDEF
* ALPN: server accepted h2
* Server certificate:
*  subject: CN=*.kongcloud.dev
*  start date: Aug 15 23:28:00 2025 GMT
*  expire date: Nov 13 23:27:59 2025 GMT
*  subjectAltName: host "kong-cceb6a93c9usmc2hk.kongcloud.dev" matched cert's "*.kongcloud.dev"
*  issuer: C=US; O=Let's Encrypt; CN=E6
*  SSL certificate verify ok.
* using HTTP/2
* [HTTP/2] [1] OPENED stream for https://kong-cceb6a93c9usmc2hk.kongcloud.dev/httpbin-route/get
* [HTTP/2] [1] [:method: GET]
* [HTTP/2] [1] [:scheme: https]
* [HTTP/2] [1] [:authority: kong-cceb6a93c9usmc2hk.kongcloud.dev]
* [HTTP/2] [1] [:path: /httpbin-route/get]
* [HTTP/2] [1] [user-agent: curl/8.7.1]
* [HTTP/2] [1] [accept: */*]
> GET /httpbin-route/get HTTP/2
> Host: kong-cceb6a93c9usmc2hk.kongcloud.dev
> User-Agent: curl/8.7.1
> Accept: */*
> 
* Request completely sent off
< HTTP/2 200 
< content-type: application/json
< content-length: 500
< x-kong-request-id: b3517f65a230d3091265e724d7d3ba14
< server: gunicorn/19.9.0
< date: Mon, 22 Sep 2025 19:59:58 GMT
< access-control-allow-origin: *
< access-control-allow-credentials: true
< x-kong-upstream-latency: 10
< x-kong-proxy-latency: 131
< via: 1.1 kong/3.11.0.0-enterprise-edition
< 
{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Connection": "keep-alive", 
    "Host": "httpbin.konghq.com", 
    "User-Agent": "curl/8.7.1", 
    "X-Forwarded-Host": "kong-cceb6a93c9usmc2hk.kongcloud.dev", 
    "X-Forwarded-Path": "/httpbin-route/get", 
    "X-Forwarded-Prefix": "/httpbin-route", 
    "X-Kong-Request-Id": "b3517f65a230d3091265e724d7d3ba14"
  }, 
  "origin": "186.204.54.49, 66.51.127.198, 172.16.12.194", 
  "url": "https://kong-cceb6a93c9usmc2hk.kongcloud.dev/get"
}
* Connection #0 to host kong-cceb6a93c9usmc2hk.kongcloud.dev left intact
```


Kong-gratulations! have now reached the end of this module by having your first service set up, running, and routing traffic proxied through a Kong data plane. You can now click **Next** to proceed with the next module.

