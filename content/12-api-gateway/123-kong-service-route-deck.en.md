---
title : "Use decK to create Kong Objects"
weight : 123
---


Now, let's use **decK** to create Kong Objects. This time, you’ll create and expose a service to the [**HTTPbin**](http://httpbin.konghq.com) API. **HTTPbin** is an echo-type application that returns requests back to the requester as responses.


#### Ping Konnect with decK

Before start using decK, you should ping Konnect to check if the connecting is up. Note we assume you have the PAT environment variable set. Please, refer to the previous section to learn how to issue a PAT.

```
deck gateway ping --konnect-control-plane-name serverless-default --konnect-token $PAT
```


**Expected Output**
```
Successfully Konnected to the AcquaOrg organization!
```


#### Create a Kong Gateway Service and Kong Route

Create the following declaration first. Remarks:
* Note the ``host`` and ``port`` refers to the HTTPbin's endpoint ``http://httpbin.konghq.com:80``.
* The declaration tags the objects so you can managing them apart from other ones.

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
  host: httpbin.konghq.com
  port: 80
  routes:
  - name: httpbin-route
    paths:
    - /httpbin-route
EOF
```


#### Submit the declaration

Before submiting the declaration, run the following **deck** command to reset your Control Plane and delete all Kong Objects you might have created previously.

```
deck gateway reset --konnect-control-plane-name serverless-default --konnect-token $PAT -f
```

Now, you can use the following command to sync your Konnect Control Plane with the declaration.

```
deck gateway sync --konnect-token $PAT httpbin.yaml
```

**Expected Output**
```
creating service httpbin-service
creating route httpbin-route
Summary:
  Created: 2
  Updated: 0
  Deleted: 0
```

You should see your new service’s overview page.

![service1](/static/images/httpbin-service-route.png)

<!-- If you want to delete them run:

:::code{showCopyAction=true showLineNumbers=false language=shell}
deck gateway reset --konnect-control-plane-name kong-aws --konnect-token $PAT -f
::: -->


#### Consume the Route

We are to use the same ELB provisioned during the Data Plane deployment:

```
curl -v $DATA_PLANE_URL/httpbin-route/get
```

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
< x-kong-request-id: 738fe4313578a34415154c6833df4e40
< server: gunicorn/19.9.0
< date: Tue, 23 Sep 2025 12:10:29 GMT
< access-control-allow-origin: *
< access-control-allow-credentials: true
< x-kong-upstream-latency: 12
< x-kong-proxy-latency: 69
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
    "X-Kong-Request-Id": "738fe4313578a34415154c6833df4e40"
  }, 
  "origin": "186.204.54.49, 66.51.127.198, 172.16.12.194", 
  "url": "https://kong-cceb6a93c9usmc2hk.kongcloud.dev/get"
}
* Connection #0 to host kong-cceb6a93c9usmc2hk.kongcloud.dev left intact```


