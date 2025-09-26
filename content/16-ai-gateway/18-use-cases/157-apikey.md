---
title : "Key Auth"
weight : 157
---

In this section, you will configure the **Key-Auth** plugin on the Kong Route to protect Amazon Bedrock.


#### Add Kong Key Authentication plugin and Kong Consumer

Add a KongPlugin resource for authentication, specifically the **Key-Auth** plugin. Note that, besides describing the plugin configuration, the declaration also creates a **Kong Consumer**, named ``user1``, with an API Key (``123456``) as its credential.

```
cat > ai-key-auth.yaml << 'EOF'
_format_version: "3.0"
_konnect:
  control_plane_name: serverless-default
_info:
  select_tags:
  - llm
services:
- name: service1
  host: localhost
  port: 32000
  routes:
  - name: openai-route
    paths:
    - /openai-route
    plugins:
    - name: ai-proxy
      instance_name: ai-proxy-openai
      config:
        route_type: llm/v1/chat
        auth:
          header_name: Authorization
          header_value: Bearer ${{ env "DECK_OPENAI_API_KEY" }}
        model:
          provider: openai
          name: gpt-4.1
          options:
            temperature: 1.0
    - name: key-auth
      instance_name: key-auth1
      enabled: true
consumers:
- keyauth_credentials:
  - key: "123456"
  username: user1
EOF
```


Apply the declaration with decK:
```
deck gateway reset --konnect-control-plane-name serverless-default --konnect-token $PAT -f
deck gateway sync --konnect-token $PAT ai-key-auth.yaml
```



#### Verify authentication is required
New requests now require authentication

```
curl -i -X POST \
  --url $DATA_PLANE_URL/openai-route \
  --header 'Content-Type: application/json' \
   --data '{
   "messages": [
     {
       "role": "user",
       "content": "Who is Jimi Hendrix?"
     }
   ]
  }'
```

* Expect response

The response is a ``HTTP/1.1 401 Unauthorized``, meaning the Kong Gateway Service requires authentication.

```
HTTP/2 401 
date: Wed, 24 Sep 2025 20:14:07 GMT
content-type: application/json; charset=utf-8
x-kong-request-id: 95b133433cd56510d863734de2c1a16f
www-authenticate: Key
content-length: 96
x-kong-response-latency: 1
server: kong/3.11.0.0-enterprise-edition

{
  "message":"No API key found in request",
  "request_id":"95b133433cd56510d863734de2c1a16f"
}
```

#### Send another request with an API key

Use the apikey to pass authentication to access the services.

```
curl -i -X POST \
  --url $DATA_PLANE_URL/openai-route \
  --header 'Content-Type: application/json' \
  --header 'apikey: 123456' \
  --data '{
   "messages": [
     {
       "role": "user",
       "content": "Who is Jimi Hendrix?"
     }
   ]
  }'
```

The request should now respond with a  **HTTP/1.1 200 OK**.

When submitting requests, the API Key name is defined, by default, ``apikey``. You can change the plugin configuration, if you will.



