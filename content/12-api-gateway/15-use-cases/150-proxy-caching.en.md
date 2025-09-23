---
title : "Proxy Caching"
weight : 150
---

[Proxy Caching](https://docs.konghq.com/hub/kong-inc/proxy-cache/) provides a reverse proxy cache implementation for Kong. It caches response entities based on configurable response code and content type, as well as request method. It can cache per-Consumer or per-API. Cache entities are stored for a configurable period of time, after which subsequent requests to the same resource will re-fetch and re-store the resource. Cache entities can also be forcefully purged via the Admin API prior to their expiration time.

### Kong Gateway Plugin list

Before enabling the **Proxy Caching**, let's check the list of plugins Konnect provides. Inside the ``serverless-default`` Control Plane, click on **Plugins** menu option and **+ New plugin**. You should the following page with all plugins available:

![proxy_cache](/static/images/plugins.png)

### Enabling a Kong Plugin on a Kong Service
Create another declaration with ``plugins`` option. With this option you can enable and configure the plugin on your Kong Service.

```
cat > httpbin.yaml << 'EOF'
_format_version: "3.0"
_konnect:
  control_plane_name: serverless-default
_info:
  select_tags:
  - httpbin-service-route
services:
- name: httpbin-service
  tags:
  - httpbin-service-route
  host: httpbin.konghq.com
  port: 80
  plugins:
  - name: proxy-cache
    instance_name: proxy-cache1
    config:
      strategy: memory
      cache_ttl: 30
  routes:
  - name: httpbin-route
    tags:
    - httpbin-service-route
    paths:
    - /httpbin-route
EOF
```


For the plugin configuration we used the following settings:
* **strategy** with ``memory``. The plugin will use the Runtime Instance's memory to implement to cache.
* **cache_ttl** with ``30``, which means the plugin will clear all data that reached this time limit.

All plugin configuration paramenters are described inside **[Kong Plugin Hub](https://docs.konghq.com/hub/)** portal, in its specific [documentation page](https://docs.konghq.com/hub/kong-inc/proxy-cache/).

#### Submit the new declaration
```
deck gateway sync --konnect-token $PAT httpbin.yaml
```

**Expected Output**
```
creating plugin proxy-cache for service httpbin-service
Summary:
  Created: 1
  Updated: 0
  Deleted: 0
```


#### Consume the Service

If you consume the service again, you'll see some new headers describing the caching status:

```
curl -i $DATA_PLANE_URL/httpbin-route/get
```

```
HTTP/2 200 
content-type: application/json
content-length: 500
x-kong-request-id: abd3f90c6ecbbb0a0939fb2edab2b40d
x-cache-key: f44e43eff1a09eeb35e0436a117f95f9363267d79089b8bdd950f85ca6247e97
x-cache-status: Miss
server: gunicorn/19.9.0
date: Tue, 23 Sep 2025 12:16:15 GMT
access-control-allow-origin: *
access-control-allow-credentials: true
x-kong-upstream-latency: 13
x-kong-proxy-latency: 1
via: 1.1 kong/3.11.0.0-enterprise-edition

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
    "X-Kong-Request-Id": "abd3f90c6ecbbb0a0939fb2edab2b40d"
  }, 
  "origin": "186.204.54.49, 66.51.127.198, 172.16.12.194", 
  "url": "https://kong-cceb6a93c9usmc2hk.kongcloud.dev/get"
}
```

Notice that, for the first request we get **Miss** for the **X-Cache-Status** header, meaning that the Runtime Instance didn't have any data avaialble in the cache and had to connect to the Upstream Service, ``httpbin.org``.

If we send a new request, the Runtime Instance has all it needs to satify the request, therefore the status is **Hit**. Note that the latency time has dropped considerably.

```
% curl -i $DATA_PLANE_URL/httpbin-route/get
HTTP/2 200 
content-type: application/json
x-kong-request-id: 4ad5c907f84c167c3eb3f716200ae17c
x-cache-key: f44e43eff1a09eeb35e0436a117f95f9363267d79089b8bdd950f85ca6247e97
date: Tue, 23 Sep 2025 12:16:46 GMT
server: gunicorn/19.9.0
age: 3
x-cache-status: Hit
access-control-allow-origin: *
access-control-allow-credentials: true
content-length: 500
x-kong-upstream-latency: 0
x-kong-proxy-latency: 1
via: 1.1 kong/3.11.0.0-enterprise-edition

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
    "X-Kong-Request-Id": "70184e54f9235642a310362396089529"
  }, 
  "origin": "186.204.54.49, 66.51.127.198, 172.16.12.194", 
  "url": "https://kong-cceb6a93c9usmc2hk.kongcloud.dev/get"
}
```

### Enabling a Kong Plugin on a Kong Route

Now, we are going to define a Rate Limiting policy for our Service. This time, you are going to enable the **Rate Limiting** plugin to the Kong Route, not to the Kong Gateway Service. In this sense, new Routes defined for the Service will not have the Rate Limiting plugin enabled, only the Proxy Caching.

```
cat > httpbin.yaml << 'EOF'
_format_version: "3.0"
_konnect:
  control_plane_name: serverless-default
_info:
  select_tags:
  - httpbin-service-route
services:
- name: httpbin-service
  tags:
  - httpbin-service-route
  host: httpbin.konghq.com
  port: 80
  plugins:
  - name: proxy-cache
    config:
      strategy: memory
      cache_ttl: 30
  routes:
  - name: httpbin-route
    tags:
    - httpbin-service-route
    paths:
    - /httpbin-route
    plugins:
    - name: rate-limiting
      instance_name: rate-limiting1
      config:
        minute: 3
EOF
```

The configuration includes:
* **minute** as ``3``, which means the Route can be consumed only 3 times a given minute.



#### Submit the declaration
```
deck gateway sync --konnect-token $PAT httpbin.yaml
```


#### Consume the Service

If you consume the service again, you'll see, besides the caching related headers, new ones describing the status of current rate limiting policy:

```
curl -i $DATA_PLANE_URL/httpbin-route/get
```

```
HTTP/2 200 
content-type: application/json
content-length: 500
x-kong-request-id: dc73a70617cde444eada947550425656
ratelimit-limit: 3
ratelimit-remaining: 2
x-ratelimit-limit-minute: 3
x-ratelimit-remaining-minute: 2
ratelimit-reset: 29
x-cache-key: f44e43eff1a09eeb35e0436a117f95f9363267d79089b8bdd950f85ca6247e97
x-cache-status: Miss
server: gunicorn/19.9.0
date: Tue, 23 Sep 2025 12:19:31 GMT
access-control-allow-origin: *
access-control-allow-credentials: true
x-kong-upstream-latency: 8
x-kong-proxy-latency: 1
via: 1.1 kong/3.11.0.0-enterprise-edition

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
    "X-Kong-Request-Id": "dc73a70617cde444eada947550425656"
  }, 
  "origin": "186.204.54.49, 66.51.127.198, 172.16.12.194", 
  "url": "https://kong-cceb6a93c9usmc2hk.kongcloud.dev/get"
}
```


If you keep sending new requests to the Runtime Instance, eventually, you'll get a **429** error code, meaning you have reached the consumption rate limiting policy for this Route.

```
% curl -i $DATA_PLANE_URL/httpbin-route/get
HTTP/2 429 
date: Tue, 23 Sep 2025 12:19:37 GMT
content-type: application/json; charset=utf-8
x-kong-request-id: bf61f204c78bc7404721310d6a35ec11
retry-after: 23
ratelimit-limit: 3
ratelimit-remaining: 0
x-ratelimit-limit-minute: 3
x-ratelimit-remaining-minute: 0
ratelimit-reset: 23
content-length: 92
x-kong-response-latency: 0
server: kong/3.11.0.0-enterprise-edition

{
  "message":"API rate limit exceeded",
  "request_id":"bf61f204c78bc7404721310d6a35ec11"
}
```

### Enabling a Kong Plugin globally

Besides scoping a plugin to a Kong Service or Route, we can apply it globally also. When we do it so, all Services ans Routes will enforce the police described by the plugin.

For example, let's apply the Proxy Caching plugin globally.

```
cat > httpbin.yaml << 'EOF'
_format_version: "3.0"
_konnect:
  control_plane_name: serverless-default
_info:
  select_tags:
  - httpbin-service-route
plugins:
- name: proxy-cache
  config:
    strategy: memory
    cache_ttl: 30
services:
- name: httpbin-service
  tags:
  - httpbin-service-route
  host: httpbin.konghq.com
  port: 80
  routes:
  - name: httpbin-route
    tags:
    - httpbin-service-route
    paths:
    - /httpbin-route
    plugins:
    - name: rate-limiting
      instance_name: rate-limiting1
      config:
        minute: 3
EOF
```


#### Submit the declaration
```
deck gateway sync --konnect-token $PAT httpbin.yaml
```

* Expected output

Note the first Proxy Cache instance is deleted to get the Control Plane state synced with the declaratio:

```
creating plugin proxy-cache (global)
deleting plugin proxy-cache for service httpbin-service
Summary:
  Created: 1
  Updated: 0
  Deleted: 1
```

After testing the configuration, reset the Control Plane:

```
deck gateway reset --konnect-control-plane-name serverless-default --konnect-token $PAT -f
```


