---
title : "Response Transformer"
weight : 153
---

The [Response Transformer](https://docs.konghq.com/hub/kong-inc/response-transformer/) plugin modifies the upstream response (e.g. response from the server) before returning it to the client.

In this section, you will configure the Response Transformer plugin on the Kong Route. Specifically, you will configure Kong Konnect to add a new header "demo: injected-by-kong" before responding to the client.


#### Create the Response Transformer Plugin

Take the plugins declaration and enable the **Response Transformer** plugin to the Route.

```
cat > response-transformer.yaml << 'EOF'
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
  - name: response-transformer-route
    paths:
    - /response-transformer-route
    plugins:
    - name: response-transformer
      instance_name: response-transformer1
      config:
        add:
          headers:
          - demo:injected-by-kong
EOF
```


Submit the declaration
```
deck gateway sync --konnect-token $PAT response-transformer.yaml
```


### Verify
Test to make sure Kong transforms the request to the echo server and httpbin server. 

```
curl --head $DATA_PLANE_URL/response-transformer-route/get
```

```
HTTP/2 200 
content-type: application/json
content-length: 526
x-kong-request-id: 29f8371c1b1cc446119b7f5df69a6128
server: gunicorn/19.9.0
date: Tue, 23 Sep 2025 12:40:54 GMT
access-control-allow-origin: *
access-control-allow-credentials: true
demo: injected-by-kong
x-kong-upstream-latency: 12
x-kong-proxy-latency: 1
via: 1.1 kong/3.11.0.0-enterprise-edition
```


**Expected Results** Notice that ``demo: injected-by-kong`` is injected in the header.


#### Cleanup

Reset the Control Plane to ensure that the plugins do not interfere with any other modules in the workshop for demo purposes and each workshop module code continues to function independently.

```
deck gateway reset --konnect-control-plane-name serverless-default --konnect-token $PAT -f
```

In real world scenario, you can enable as many plugins as you like depending on your use cases.

Kong-gratulations! have now reached the end of this module by configuring the Kong Route to include ``demo: injected-by-kong`` before responding to the client. You can now click **Next** to proceed with the next module.