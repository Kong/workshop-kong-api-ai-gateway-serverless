---
title : "API Key Authentication"
weight : 151
---

To get started with API Authentication, let's implement a basic Key Authentication mechanism. API Keys are one of the foundamental security mechanisms provided by Konnect. In order to consume an API, the consumer should inject a previously created API Key in the header of the request. The API consumption is allowed if the Gateway recognizes the API Key. Consumers add their API key either in a query string parameter, a header, or a request body to authenticate their requests and consume the application.

A Kong Consumer represents a consumer (user or application) of a Service. A Kong Consumer is tightly coupled to an Authentication mechanism the Kong Gateway provides.

![kong_consumer](/static/images/kong_consumer.png)

Please, check the [**Key-Auth** plugin](https://docs.konghq.com/hub/kong-inc/key-auth/) plugin and [**Kong Consumer**](https://docs.konghq.com/gateway/latest/key-concepts/consumers/) documentation pages to learn more about them.



### Enable the Key Authentication Plugin on the Kong Route

```
cat > key-auth.yaml << 'EOF'
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
    - /key-auth-route
  plugins:
  - name: key-auth
    instance_name: key-auth1
EOF
```

```
deck gateway sync --konnect-token $PAT key-auth.yaml
```



#### Consume the Route

Now, if you try the Route, you'll get a specific **401** error code meaning that, since you don't have any API Key injected in your request, you are not allowd to consume it.

```
curl -i $DATA_PLANE_URL/key-auth-route/get
```

```
HTTP/2 401 
date: Tue, 23 Sep 2025 12:28:41 GMT
content-type: application/json; charset=utf-8
x-kong-request-id: 5082dc799dcc913c3e27581f84eba120
www-authenticate: Key
content-length: 96
x-kong-response-latency: 1
server: kong/3.11.0.0-enterprise-edition

{
  "message":"No API key found in request",
  "request_id":"5082dc799dcc913c3e27581f84eba120"
}
```


#### Create a Kong Consumer

In order to consume the Route we need to create a Kong Consumer. Here's its declaration:

```
cat > key-auth.yaml << 'EOF'
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
    - /key-auth-route
  plugins:
  - name: key-auth
    instance_name: key-auth1
consumers:
- keyauth_credentials:
  - key: "123456"
  username: consumer1
EOF
```


Submit the declaration
```
deck gateway sync --konnect-token $PAT key-auth.yaml
```



#### Consume the Route with the API Key

Now, you need to inject the Key you've just created, as a header, in your requests. Using HTTPie, you can do it easily like this:

```
curl --head $DATA_PLANE_URL/key-auth-route/get -H 'apikey:123456'
```

```
HTTP/2 200 
content-type: application/json
content-length: 702
x-kong-request-id: b54df81da8dde905312cd55d5600f638
server: gunicorn/19.9.0
date: Tue, 23 Sep 2025 12:29:57 GMT
access-control-allow-origin: *
access-control-allow-credentials: true
x-kong-upstream-latency: 10
x-kong-proxy-latency: 2
via: 1.1 kong/3.11.0.0-enterprise-edition
```

Of course, if you inject a wrong key, you get a specific error like this:
```
% curl --head $DATA_PLANE_URL/key-auth-route/get -H 'apikey:12'
HTTP/2 401 
date: Tue, 23 Sep 2025 12:30:23 GMT
content-type: application/json; charset=utf-8
x-kong-request-id: 31c73846e3a250366a42d805f0282b4b
www-authenticate: Key
content-length: 81
x-kong-response-latency: 1
server: kong/3.11.0.0-enterprise-edition
```


**NOTE**

* The header has to have the API Key name, which is, in our case, ``apikey``. That was the default name provided by Konnect when you enabled the **Key Authentication** on the Kong Route. You can change the plugin configuration, if you will.


### Kong Consumer Policies

With the API Key policy in place, we can control the incoming requests. However, the policies implemented by the other plugins are the same regardless the consumer.

It's important then to be able to define specific policies for each one of these consumers. For example, it would be great to define Rate Limiting policies for different consumers like this:

* consumer1:
    * apikey = 123456
    * rate limiting policy = 5 rpm
* consumer2:
    * apikey = 987654
    * rate limiting policy = 8 rpm

Doing that, the Data Plane is capable to not just protect the Route but to identify the consumer based on the key injected to enforce specific policies to the consumer.

For this section we're implementing a Rate Limiting policy. Keep in mind that a Consumer might have other plugins also enabled such as [Request Transformer](https://docs.konghq.com/hub/kong-inc/request-transformer/), [TCP Log](https://docs.konghq.com/hub/kong-inc/tcp-log/), etc.


#### New Consumer

Create the second ``consumer2``, just like you did with the first one, with the ``987654`` key.

```
cat > key-auth.yaml << 'EOF'
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
    - /key-auth-route
  plugins:
  - name: key-auth
    instance_name: key-auth1
consumers:
- keyauth_credentials:
  - key: "123456"
  username: consumer1
- keyauth_credentials:
  - key: "987654"
  username: consumer2
EOF
```

Submit the declaration
```
deck gateway sync --konnect-token $PAT key-auth.yaml
```



If you will, you can inject both keys to your requests.

```
curl --head $DATA_PLANE_URL/key-auth-route/get -H 'apikey:123456'
```

or

```
curl --head $DATA_PLANE_URL/key-auth-route/get -H 'apikey:987654'
```


#### Consumers' Policy

Now let's enhance the plugins declaration enabling the Rate Limiting plugin to each one of our consumers.

```
cat > key-auth.yaml << 'EOF'
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
    - /key-auth-route
  plugins:
  - name: key-auth
    instance_name: key-auth1
consumers:
- keyauth_credentials:
  - key: "123456"
  username: consumer1
  plugins:
  - name: rate-limiting
    instance_name: rate-limiting1
    config:
      minute: 5
- keyauth_credentials:
  - key: "987654"
  username: consumer2
  plugins:
  - name: rate-limiting
    instance_name: rate-limiting2
    config:
      minute: 8
EOF
```


Submit the declaration
```
deck gateway sync --konnect-token $PAT key-auth.yaml
```



#### Consumer the Route using different API Keys.

First of all let's consume the Route with the Consumer1's API Key:

```
curl --head $DATA_PLANE_URL/key-auth-route/get -H 'apikey:123456'
```

**Expected Output**

```
HTTP/2 200 
content-type: application/json
content-length: 702
x-kong-request-id: 863f8f3fc16dd32bdf8f045465857c7e
ratelimit-limit: 5
ratelimit-remaining: 4
x-ratelimit-limit-minute: 5
x-ratelimit-remaining-minute: 4
ratelimit-reset: 35
server: gunicorn/19.9.0
date: Tue, 23 Sep 2025 12:33:25 GMT
access-control-allow-origin: *
access-control-allow-credentials: true
x-kong-upstream-latency: 8
x-kong-proxy-latency: 2
via: 1.1 kong/3.11.0.0-enterprise-edition
```

Now, let's consume it with the Consumer2's API Key. As you can see the Data Plane is processing the Rate Limiting processes independently.

```
curl --head $DATA_PLANE_URL/key-auth-route/get -H 'apikey:987654'
```

**Expected Output**

```
HTTP/2 200 
content-type: application/json
content-length: 702
x-kong-request-id: 3c99310c94ed3157f1cdb4604e0a8c4e
ratelimit-limit: 8
ratelimit-remaining: 7
x-ratelimit-limit-minute: 8
x-ratelimit-remaining-minute: 7
ratelimit-reset: 9
server: gunicorn/19.9.0
date: Tue, 23 Sep 2025 12:33:51 GMT
access-control-allow-origin: *
access-control-allow-credentials: true
x-kong-upstream-latency: 9
x-kong-proxy-latency: 1
via: 1.1 kong/3.11.0.0-enterprise-edition
```

If we keep sending requests using the first API Key, eventually, as expected, we'll get an error code:


```
curl --head $DATA_PLANE_URL/key-auth-route/get -H 'apikey:123456'
```


**Expected Output**

```
HTTP/2 429 
date: Tue, 23 Sep 2025 12:34:21 GMT
content-type: application/json; charset=utf-8
x-kong-request-id: 3e890328c589ee56b57a0171c4c89267
retry-after: 39
ratelimit-limit: 5
ratelimit-remaining: 0
x-ratelimit-limit-minute: 5
x-ratelimit-remaining-minute: 0
ratelimit-reset: 39
content-length: 92
x-kong-response-latency: 0
server: kong/3.11.0.0-enterprise-edition
```

However, the second API Key is still allowed to consume the Kong Route:

```
curl --head $DATA_PLANE_URL/key-auth-route/get -H 'apikey:987654'
```

**Expected Output**

```
HTTP/2 200 
content-type: application/json
content-length: 702
x-kong-request-id: 86f7c102950a06452a4e0f66c290f30e
ratelimit-limit: 8
ratelimit-remaining: 7
x-ratelimit-limit-minute: 8
x-ratelimit-remaining-minute: 7
ratelimit-reset: 26
server: gunicorn/19.9.0
date: Tue, 23 Sep 2025 12:34:34 GMT
access-control-allow-origin: *
access-control-allow-credentials: true
x-kong-upstream-latency: 8
x-kong-proxy-latency: 1
via: 1.1 kong/3.11.0.0-enterprise-edition
```

Kong-gratulations! have now reached the end of this module by authenticating the API requests with a key and associating different consumers with policy plans. You can now click **Next** to proceed with the next module.

### Optional Reading

Applying Kong Plugins on Services, Routes or Globally helps us to implement an extensive list of policies in the API Gateway layer. However, so far, we are not controlling who is sending the requests to the Data Plane. That is, anyone who has the Runtime Instance ELB address is capable to send requests to it and consumer the Services.

API Gateway Authentication is an important way to control the data that is allowed to be transmitted using your APIs. Basically, it checks that a particular consumer has permission to access the API, using a predefined set of credentials.

Kong Gateway has a library of plugins that provide simple ways to implement the best known and most widely used methods of API gateway authentication. Here are some of the commonly used ones:

* Basic Authentication
* Key Authentication
* OAuth 2.0 Authentication
* LDAP Authentication
* OpenID Connect

Kong Plugin Hub provides documentation about all [Authentication](https://docs.konghq.com/hub/#authentication) based plugins. Refer to the following link to read more about [API Gateway Authentication](https://konghq.com/learning-center/api-gateway/api-gateway-authentication)
